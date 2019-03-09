---
title: "对象的wait和notify方法"
layout: page
date: 2019-03-09 00:00
---

[TOC]

# wait & nofiy

都是基类`Object`中的方法

* `object's monitor`
* `wait set`
* synchronized

## wait 源码说明1

```java
 /**
     * Causes the current thread to wait until either another thread invokes the
     * {@link java.lang.Object#notify()} method or the
     * {@link java.lang.Object#notifyAll()} method for this object, or a
     * specified amount of time has elapsed.
     * <p>
     * The current thread must own this object's monitor.
     * <p>
     * This method causes the current thread (call it <var>T</var>) to
     * place itself in the wait set for this object and then to relinquish
     * any and all synchronization claims on this object. Thread <var>T</var>
     * becomes disabled for thread scheduling purposes and lies dormant
     * until one of four things happens:
     * <ul>
     * <li>Some other thread invokes the {@code notify} method for this
     * object and thread <var>T</var> happens to be arbitrarily chosen as
     * the thread to be awakened.
     * <li>Some other thread invokes the {@code notifyAll} method for this
     * object.
     * <li>Some other thread {@linkplain Thread#interrupt() interrupts}
     * thread <var>T</var>.
     * <li>The specified amount of real time has elapsed, more or less.  If
     * {@code timeout} is zero, however, then real time is not taken into
     * consideration and the thread simply waits until notified.
     * </ul>
     * The thread <var>T</var> is then removed from the wait set for this
     * object and re-enabled for thread scheduling. It then competes in the
     * usual manner with other threads for the right to synchronize on the
     * object; once it has gained control of the object, all its
     * synchronization claims on the object are restored to the status quo
     * ante - that is, to the situation as of the time that the {@code wait}
     * method was invoked. Thread <var>T</var> then returns from the
     * invocation of the {@code wait} method. Thus, on return from the
     * {@code wait} method, the synchronization state of the object and of
     * thread {@code T} is exactly as it was when the {@code wait} method
     * was invoked.
     * <p>
     * A thread can also wake up without being notified, interrupted, or
     * timing out, a so-called <i>spurious wakeup</i>.  While this will rarely
     * occur in practice, applications must guard against it by testing for
     * the condition that should have caused the thread to be awakened, and
     * continuing to wait if the condition is not satisfied.  In other words,
     * waits should always occur in loops, like this one:
     * <pre>
     *     synchronized (obj) {
     *         while (&lt;condition does not hold&gt;)
     *             obj.wait(timeout);
     *         ... // Perform action appropriate to condition
     *     }
     * </pre>
     * (For more information on this topic, see Section 3.2.3 in Doug Lea's
     * "Concurrent Programming in Java (Second Edition)" (Addison-Wesley,
     * 2000), or Item 50 in Joshua Bloch's "Effective Java Programming
     * Language Guide" (Addison-Wesley, 2001).
     *
     * <p>If the current thread is {@linkplain java.lang.Thread#interrupt()
     * interrupted} by any thread before or while it is waiting, then an
     * {@code InterruptedException} is thrown.  This exception is not
     * thrown until the lock status of this object has been restored as
     * described above.
     *
     * <p>
     * Note that the {@code wait} method, as it places the current thread
     * into the wait set for this object, unlocks only this object; any
     * other objects on which the current thread may be synchronized remain
     * locked while the thread waits.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @param      timeout   the maximum time to wait in milliseconds.
     * @throws  IllegalArgumentException      if the value of timeout is
     *               negative.
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of the object's monitor.
     * @throws  InterruptedException if any thread interrupted the
     *             current thread before or while the current thread
     *             was waiting for a notification.  The <i>interrupted
     *             status</i> of the current thread is cleared when
     *             this exception is thrown.
     * @see        java.lang.Object#notify()
     * @see        java.lang.Object#notifyAll()
     */
public final native void wait(long timeout) throws InterruptedException;
```

## 例子1

```java
public class Main {

    public static void main(String[] args) throws Exception{
        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (this) {
                    System.out.println("A start");
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    System.out.println("A notify");
                    // 唤醒对应的线程
                    this.notify();
                }
            }
        });

        Thread threadB = new Thread(()-> {
            System.out.println("B start");
            synchronized(threadA) {
                try {
                    // 使得当前执行的线程B等待
                    threadA.wait(3000);
                    System.out.println("B running");
                } catch (Exception e) {
                    System.out.println("B Exception");
                    e.printStackTrace();
                }
                System.out.println("B end");
            }
        });

        threadA.start();
        threadB.start();

        threadA.join();
        threadB.join();
        System.out.println("main end");
    }
}
/* ouput
A start
B start
A notify
B running
B end
main end
*/
```

`synchronized(threadA)`锁定threadA（获得threadA的monitor）

`wait`操作会释放对象的monitor， 如果原先没有锁定，则会抛出`java.lang.IllegalMonitorStateException`

```java
Thread threadB = new Thread(()-> {
            System.out.println("B start");
            try {
                // 使得当前执行的线程B等待
                threadA.wait(3000);
                System.out.println("B running");
            } catch (Exception e) {
                System.out.println("B Exception");
                e.printStackTrace();
            }
            System.out.println("B end");
        });
```

* output

```java
A start
B start
B Exception
B end
java.lang.IllegalMonitorStateException
	at java.lang.Object.wait(Native Method)
	at com.parallel.Main.lambda$main$0(Main.java:31)
	at java.lang.Thread.run(Thread.java:748)
A notify
main end
```
