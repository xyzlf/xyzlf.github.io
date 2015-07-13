---
layout: post
title: Java中隐藏的this变量和局部变量可能引发的内存泄露问题
date: 2015-07-13 12:54:48
categories: [Java, Android]
tags: [JVM, MemoryLeak]
---
###背景
众所周知，在Java中，成员方法内可以使用this来引用当前对象，使用起来特别方便。但是在JVM中方法是在方法区中，所有的类的对象都共用了一个方法区，那么JVM是怎么知道this是指向哪个对象的呢？  
其实为了实现这一功能，Java的处理方式很简单，在编译时，为每个成员方法都默默的添加了一个参数this，当调用这个方法时，把当前对象以参数的形式传进去即可。
<!--more-->
本文重点不在this是怎么来的，因此简单验证下：

一个简单的java类：
{% highlight java %}
import java.util.Arrays;

public class Test {

    public Test() {
    }

    public void hi() {
    }

    public static void staticHi() {
    }

}
{% endhighlight java %}
编译后，使用javap来查看class文件
{% highlight java %}
~ javap -verbose Test.class
Classfile /Users/Johnny/Downloads/Test.class
  Last modified 2015-7-13; size 280 bytes
  MD5 checksum 5f8b96a983d277edb1974e472cf0b62a
  Compiled from "Test.java"
public class Test
  SourceFile: "Test.java"
  minor version: 0
  major version: 51
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #3.#12         //  java/lang/Object."<init>":()V
   #2 = Class              #13            //  Test
   #3 = Class              #14            //  java/lang/Object
   #4 = Utf8               <init>
   #5 = Utf8               ()V
   #6 = Utf8               Code
   #7 = Utf8               LineNumberTable
   #8 = Utf8               hi
   #9 = Utf8               staticHi
  #10 = Utf8               SourceFile
  #11 = Utf8               Test.java
  #12 = NameAndType        #4:#5          //  "<init>":()V
  #13 = Utf8               Test
  #14 = Utf8               java/lang/Object
{
  public Test();
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 5: 0
        line 7: 4

  public void hi();
    flags: ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 11: 0

  public static void staticHi();
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=0, locals=0, args_size=0
         0: return
      LineNumberTable:
        line 15: 0
}
{% endhighlight java %}
看Test方法和hi方法的args_size=1，源码中是0参数的，因此可以验证确实是偷偷的添加了一个参数，当然这里肯定是this参数了。  

###Memory Leak
回归正题，上面验证了成员方法中会默默的添加一个this参数，而参数其实是放在方法局部变量表中的。因此如果参数没有被明确赋值为null的话，那么这个参数就一直是以强引用的形态指向了该对象。在一般情况下，方法和对象的生命周期是一样的，也就是对象不存在了，方法也不会被调用，不会存在泄露。但是有些情况下，比如一些异步的操作，导致对象本来应该被回收掉时，方法还在被调用，因此这期间就会存在短暂的泄露，当然如果是耗时的方法，泄露会表现的比较明显。
下面是在有异步任务的情况下的泄露测试。

{% highlight java %}
import java.lang.ref.WeakReference;

/**
 * 测试隐藏的this引用带来的Memory Leak
 */
public class Test {
    /**
     * 主要用来标记test方法是否被调用，避免Test实例提前被gc
     */
    public static boolean sHasRun = false;

    private static class MyThread extends Thread {
        protected WeakReference<Test> mTestRef;
        protected boolean mTestLeak;

        public MyThread(Test t, boolean testLeak) {
            mTestRef = new WeakReference<Test>(t);
            mTestLeak = testLeak;
        }

        @Override
        public void run() {
            if (mTestLeak) {
                // 泄露点1：局部变量，如果执行的方法是耗时的，并且在同一个线程执行的话，那么在这个方法执行完成前，局部变量是不会被回收的。
                Test t = mTestRef.get();
                if (t != null) {
                    // 泄露点2：成员方法，因为在编译时，默认会给每个成员方法加上this参数，this指向了当前类的实例。
                    // 方法参数本身就是一个局部变量，因此类此泄露点1，必须等该方法执行完成后，这个局部变量的强引用才会清除。
                    t.testLeak();
                }
            } else {
                // 解决泄露1：不保存局部变量
                if (mTestRef.get() != null) {
                    // 解决泄露2：调用静态方法而不是成员方法
                    // 当然这里可能会发生NPE，因为可能在判空后test调用前Test实例被gc了，因此try catch下
                    try {
                        mTestRef.get().test();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

    /**
     * 模拟耗时方法
     */
    private static void test() {
        sHasRun = true;
        System.out.println("time-consuming method start");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("time-consuming method end");
        sHasRun = false;
    }

    void testLeak() {
        test();
    }

    public void start(boolean testLeak) {
        new MyThread(this, testLeak).start();
    }

    public static void main(String[] args) {
        WeakReference<Test> testRef;
        boolean testLeak = true;
        {
            Test t = new Test();
            testRef = new WeakReference<Test>(t);
            t.start(testLeak);
            t = null;
        }
        while (testRef.get() != null) {
            if (sHasRun) {
                System.gc();
            } else {
                System.out.println("thread not run.");
            }
        }
        System.out.println("Test instance has cleaned.");
    }
}
{% endhighlight java %}
当testLeak为true时，打印结果为
{% highlight java %}
thread not run.
time-consuming method start
time-consuming method end
Test instance has cleaned.
{% endhighlight java %}
当testLeak为false时，打印结果为
{% highlight java %}
thread not run.
time-consuming method start
Test instance has cleaned.
time-consuming method end
{% endhighlight java %}
从打印结果可以看出，在测试泄露时，必须等到耗时方法执行结束后，该对象才能被gc，改进后的测试，在耗时方法未执行结束前，对象就已经可以被gc了。

### 解决方案
1. 为了解决隐藏的this参数这个问题，可以从根源上尽可能去除方法中this局部变量的存在，比如将方法改成static的。
2. 对于局部变量，比如弱引用的调用时，不要用局部变量保存从弱引用中get出来的值等。
