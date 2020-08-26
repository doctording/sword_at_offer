---
title: "Java IO面试题"
layout: page
date: 2019-03-18 00:00
---

[TOC]

# FileInputStream 在使用完以后，不关闭流，想二次使用可以怎么操作

```java
/**
    * Creates a <code>FileInputStream</code> by
    * opening a connection to an actual file,
    * the file named by the <code>File</code>
    * object <code>file</code> in the file system.
    * A new <code>FileDescriptor</code> object
    * is created to represent this file connection.
    * <p>
    * First, if there is a security manager,
    * its <code>checkRead</code> method  is called
    * with the path represented by the <code>file</code>
    * argument as its argument.
    * <p>
    * If the named file does not exist, is a directory rather than a regular
    * file, or for some other reason cannot be opened for reading then a
    * <code>FileNotFoundException</code> is thrown.
    *
    * @param      file   the file to be opened for reading.
    * @exception  FileNotFoundException  if the file does not exist,
    *                   is a directory rather than a regular file,
    *                   or for some other reason cannot be opened for
    *                   reading.
    * @exception  SecurityException      if a security manager exists and its
    *               <code>checkRead</code> method denies read access to the file.
    * @see        java.io.File#getPath()
    * @see        java.lang.SecurityManager#checkRead(java.lang.String)
    */
public FileInputStream(File file) throws FileNotFoundException {
    String name = (file != null ? file.getPath() : null);
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkRead(name);
    }
    if (name == null) {
        throw new NullPointerException();
    }
    if (file.isInvalid()) {
        throw new FileNotFoundException("Invalid file path");
    }
    fd = new FileDescriptor();
    fd.attach(this);
    path = name;
    open(name);
}
```

```java
/**
    * Opens the specified file for reading.
    * @param name the name of the file
    */
private native void open0(String name) throws FileNotFoundException;

// wrap native call to allow instrumentation
/**
    * Opens the specified file for reading.
    * @param name the name of the file
    */
private void open(String name) throws FileNotFoundException {
    open0(name);
}
```

fileinputStream 就是通过open方法（内部私有方法，底层是native系统方法）来打开文件的，所以如果要重新读取这个文件，不重新创建对象，那么只要调用这个方法就可以了；（由于是私有方法，所以可以利用反射来处理）

```java
public class Main {

    public static void testFileinp() throws Exception {
        String path = "/Users/mubi/git_workspace/java8/test.txt";
        String path2 = "/Users/mubi/git_workspace/java8/test2.txt";
        String path3 = "/Users/mubi/git_workspace/java8/test3.txt";
        FileInputStream inputStream = new FileInputStream(path);
        FileOutputStream outputStream = new FileOutputStream(path2);
        FileOutputStream outputStream2 = new FileOutputStream(path3);

        int len;
        byte[] by = new byte[8192];
        while ((len = inputStream.read(by)) != -1) {
            outputStream.write(by, 0, len);
        }
        // 利用反射再次打开文件
        if (inputStream.read() == -1) {
            Class in = FileInputStream.class;
            Method openo = in.getDeclaredMethod("open0", String.class);
            openo.setAccessible(true);
            openo.invoke(inputStream, path);
        }
        while ((len = inputStream.read(by)) != -1) {
            outputStream2.write(by, 0, len);
        }
        // 最后关闭流
        outputStream.close();
    }

    public static void main(String[] args) throws Exception{
        testFileinp();
    }

}
```
