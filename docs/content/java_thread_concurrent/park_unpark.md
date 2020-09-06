---
title: "park unpark"
layout: page
date: 2019-03-09 00:00
---

[TOC]

# park & unpark

## 简单使用

```java
public class Main {

    static void parkUnParkTest() throws Exception{
        Thread boyThread = new Thread(() -> {
            System.out.println("boy: 我要吃鸡");
            System.out.println("boy: park1");
            LockSupport.park();
            System.out.println("boy: park2");
            LockSupport.park();
            System.out.println("boy: 开始吃鸡了");
        });

        Thread girlThread = new Thread(() -> {
            System.out.println("girl play");
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("girl: 允许");
            // unpark两次，但是permit不会叠加
            LockSupport.unpark(boyThread);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("girl: 允许2");
            LockSupport.unpark(boyThread);
        });
        girlThread.start();
        TimeUnit.SECONDS.sleep(2);
        boyThread.start();
    }

    static void parkUnParkTest2() throws Exception{
        Thread boyThread = new Thread(() -> {
            System.out.println("boy: 我要吃鸡");
            System.out.println("boy: park1");
            LockSupport.park();
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("boy: park2");
            LockSupport.park();
            System.out.println("boy: 开始吃鸡了");
        });

        Thread girlThread = new Thread(() -> {
            System.out.println("girl play");
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("girl: 允许");
            LockSupport.unpark(boyThread);
            System.out.println("girl: 允许2");
            LockSupport.unpark(boyThread);
        });
        girlThread.start();
        TimeUnit.SECONDS.sleep(2);
        boyThread.start();
    }

    static void parkUnParkTest3() throws Exception{
        Thread boyThread = new Thread(() -> {
            System.out.println("boy: 我要吃鸡");
            System.out.println("boy: park1");
            LockSupport.park();
            System.out.println("-------boy: over");
        });

        Thread sonThread = new Thread(() -> {
            System.out.println("son: 我要drink");
            System.out.println("son: park1");
            LockSupport.park();
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("-------son: over");
        });

        Thread girlThread = new Thread(() -> {
            System.out.println("girl play");
            try {
                TimeUnit.SECONDS.sleep(4);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("girl: 允许");
            LockSupport.unpark(boyThread);
            System.out.println("girl: 允许2");
            LockSupport.unpark(sonThread);
        });
        girlThread.start();
        TimeUnit.SECONDS.sleep(2);
        boyThread.start();
        sonThread.start();
    }

    public static void main(String[] args) throws Exception {
        parkUnParkTest3();
    }
}
```

## UNSAFE.park(false, 0L); (native void park(boolean var1, long var2);)

### HotSpot park.hpp Parker对象

```java
class Parker : public os::PlatformParker {
private:
  // 表示许可
  volatile int _counter ;
  Parker * FreeNext ;
  JavaThread * AssociatedWith ; // Current association

public:
  Parker() : PlatformParker() {
    _counter       = 0 ;
    FreeNext       = NULL ;
    AssociatedWith = NULL ;
  }
protected:
  ~Parker() { ShouldNotReachHere(); }
public:
  // For simplicity of interface with Java, all forms of park (indefinite,
  // relative, and absolute) are multiplexed into one call.
  void park(bool isAbsolute, jlong time);
  void unpark();

  // Lifecycle operators
  static Parker * Allocate (JavaThread * t) ;
  static void Release (Parker * e) ;
private:
  static Parker * volatile FreeList ;
  static volatile int ListLock ;

};
```

### os::PlatformParker（os_linux.hpp）

```java
class PlatformParker : public CHeapObj<mtInternal> {
  protected:
    enum {
        REL_INDEX = 0,
        ABS_INDEX = 1
    };
    int _cur_index;  // which cond is in use: -1, 0, 1
    // 互斥变量类型
    pthread_mutex_t _mutex [1] ;
    // 条件变量类型
    pthread_cond_t  _cond  [2] ; // one for relative times and one for abs.

  public:       // TODO-FIXME: make dtor private
    ~PlatformParker() { guarantee (0, "invariant") ; }

  public:
    PlatformParker() {
      int status;
      status = pthread_cond_init (&_cond[REL_INDEX], os::Linux::condAttr());
      assert_status(status == 0, status, "cond_init rel");
      status = pthread_cond_init (&_cond[ABS_INDEX], NULL);
      assert_status(status == 0, status, "cond_init abs");
      status = pthread_mutex_init (_mutex, NULL);
      assert_status(status == 0, status, "mutex_init");
      _cur_index = -1; // mark as unused
    }
};
```

### park方法 c++ 源码

* 首先尝试`Atomic::xchg`获取permit,如果能够获取到，则设置permit为0，然后park方法直接返回（Atomic::xchg操作是线程安全的操作，控制单个cpu直接操作祝内存）
* 线程中断，park超时等判断处理
* 构造ThreadBlockInVM，进入`safepoint region`
* 检查`_counter`是不是>0，如果是，则把_counter设置为0，unlock mutex并返回
* 否则，再判断等待的时间，然后再调用`pthread_cond_wait`函数等待，如果等待返回，则把_counter设置为0，unlock mutex并返回

```java
void Parker::park(bool isAbsolute, jlong time) {
  // Ideally we'd do something useful while spinning, such
  // as calling unpackTime().

  // Optional fast-path check:
  // Return immediately if a permit is available.
  // We depend on Atomic::xchg() having full barrier semantics
  // since we are doing a lock-free update to _counter.
  if (Atomic::xchg(0, &_counter) > 0) return;

  Thread* thread = Thread::current();
  assert(thread->is_Java_thread(), "Must be JavaThread");
  JavaThread *jt = (JavaThread *)thread;

  // Optional optimization -- avoid state transitions if there's an interrupt pending.
  // Check interrupt before trying to wait
  if (Thread::is_interrupted(thread, false)) {
    return;
  }

  // Next, demultiplex/decode time arguments
  timespec absTime;
  if (time < 0 || (isAbsolute && time == 0) ) { // don't wait at all
    return;
  }
  if (time > 0) {
    unpackTime(&absTime, isAbsolute, time);
  }


  // Enter safepoint region
  // Beware of deadlocks such as 6317397.
  // The per-thread Parker:: mutex is a classic leaf-lock.
  // In particular a thread must never block on the Threads_lock while
  // holding the Parker:: mutex.  If safepoints are pending both the
  // the ThreadBlockInVM() CTOR and DTOR may grab Threads_lock.
  ThreadBlockInVM tbivm(jt);

  // Don't wait if cannot get lock since interference arises from
  // unblocking.  Also. check interrupt before trying wait
  if (Thread::is_interrupted(thread, false) || pthread_mutex_trylock(_mutex) != 0) {
    return;
  }

  int status ;
  if (_counter > 0)  { // no wait needed
    _counter = 0;
    status = pthread_mutex_unlock(_mutex);
    assert (status == 0, "invariant") ;
    // Paranoia to ensure our locked and lock-free paths interact
    // correctly with each other and Java-level accesses.
    OrderAccess::fence();
    return;
  }

#ifdef ASSERT
  // Don't catch signals while blocked; let the running threads have the signals.
  // (This allows a debugger to break into the running thread.)
  sigset_t oldsigs;
  sigset_t* allowdebug_blocked = os::Linux::allowdebug_blocked_signals();
  pthread_sigmask(SIG_BLOCK, allowdebug_blocked, &oldsigs);
#endif

  OSThreadWaitState osts(thread->osthread(), false /* not Object.wait() */);
  jt->set_suspend_equivalent();
  // cleared by handle_special_suspend_equivalent_condition() or java_suspend_self()

  assert(_cur_index == -1, "invariant");
  if (time == 0) {
    _cur_index = REL_INDEX; // arbitrary choice when not timed
    status = pthread_cond_wait (&_cond[_cur_index], _mutex) ;
  } else {
    _cur_index = isAbsolute ? ABS_INDEX : REL_INDEX;
    status = os::Linux::safe_cond_timedwait (&_cond[_cur_index], _mutex, &absTime) ;
    if (status != 0 && WorkAroundNPTLTimedWaitHang) {
      pthread_cond_destroy (&_cond[_cur_index]) ;
      pthread_cond_init    (&_cond[_cur_index], isAbsolute ? NULL : os::Linux::condAttr());
    }
  }
  _cur_index = -1;
  assert_status(status == 0 || status == EINTR ||
                status == ETIME || status == ETIMEDOUT,
                status, "cond_timedwait");

#ifdef ASSERT
  pthread_sigmask(SIG_SETMASK, &oldsigs, NULL);
#endif

  _counter = 0 ;
  status = pthread_mutex_unlock(_mutex) ;
  assert_status(status == 0, status, "invariant") ;
  // Paranoia to ensure our locked and lock-free paths interact
  // correctly with each other and Java-level accesses.
  OrderAccess::fence();

  // If externally suspended while waiting, re-suspend
  if (jt->handle_special_suspend_equivalent_condition()) {
    jt->java_suspend_self();
  }
}
```

### unpark方法源码

* 使用`_mutex`互斥锁操作设置 _counter = 1，即提供一个许可；
* 如果_counter之前的值是0，则还要调用`pthread_cond_signal`唤醒在park中等待的线程：

```java
void Parker::unpark() {
  int s, status ;
  status = pthread_mutex_lock(_mutex);
  assert (status == 0, "invariant") ;
  s = _counter;
  _counter = 1;
  if (s < 1) {
    // thread might be parked
    if (_cur_index != -1) {
      // thread is definitely parked
      if (WorkAroundNPTLTimedWaitHang) {
        status = pthread_cond_signal (&_cond[_cur_index]);
        assert (status == 0, "invariant");
        status = pthread_mutex_unlock(_mutex);
        assert (status == 0, "invariant");
      } else {
        status = pthread_mutex_unlock(_mutex);
        assert (status == 0, "invariant");
        status = pthread_cond_signal (&_cond[_cur_index]);
        assert (status == 0, "invariant");
      }
    } else {
      pthread_mutex_unlock(_mutex);
      assert (status == 0, "invariant") ;
    }
  } else {
    pthread_mutex_unlock(_mutex);
    assert (status == 0, "invariant") ;
  }
}
```

### park unpark总结

可以简单理解为：用mutex和condition保护了一个_counter（许可）的变量，park操作将这个变量置为了0；unpark操作将这个变量置为1
