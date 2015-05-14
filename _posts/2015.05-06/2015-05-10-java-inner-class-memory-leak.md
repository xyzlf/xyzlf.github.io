---
layout: post
title: Java中内部类的内存泄露问题
date: 2015-05-10 14:23:00
category: Java
tags: MemoryLeak Java Android
---
**&nbsp;&nbsp;&nbsp;&nbsp;Java中内部类使用方便，但是在使用过程中，也会存在比较隐蔽的风险，以下介绍下Java内部类中可能存在的内存泄露问题。**    
<!--more-->
{% highlight java %}
package com.example.temptemp;

import android.app.Activity;
import android.os.Bundle;
import android.util.Log;
import android.widget.TextView;

public class SecondActivity extends Activity {
    byte[] bigData = new byte[10 * 1024 * 1024];
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ((TextView) findViewById(R.id.txt)).setText("SecondActivity");
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // test();
        test2();
    }
    private void test() {
        new Thread() {
            @Override
            public void run() {
                super.run();
                Log.e("SS", "start");
                try {
                    Thread.sleep(15000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Log.e("SS", "end");
            }
        }.start();
    }


    private static void test2() {
        new Thread() {

            @Override
            public void run() {
                super.run();
                Log.e("SS", "start");
                try {
                    Thread.sleep(15000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Log.e("SS", "end");
            }
        }.start();
    }
}
{% endhighlight java %}
&nbsp;&nbsp;&nbsp;&nbsp;这是一个简单的Activity，里面有两个简单的方法，test()和test2()，test()和test2()的唯一区别就是一个是静态的，一个非静态的。
同时这个类有个成员变量bigData，主要是分配10M的内存，便于查看内存的变化。
测试方法：
只调用test方法：当Activity销毁后，发现只有等线程结束后，该Activity才能被回收；
只调用test2方法：当Activity销毁后，发现该Activity能立即被回收。
原因就是Java中内部类（匿名内部类也一样）会有宿主类的强引用。也就是this变量的来源。是编译默认行为。用javap查看class文件可以看出。
{% highlight java %}
Compiled from "SecondActivity.java"
class com.example.temptemp.SecondActivity$1 extends java.lang.Thread {
  final com.example.temptemp.SecondActivity this$0;

  com.example.temptemp.SecondActivity$1(com.example.temptemp.SecondActivity);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #10                 // Field this$0:Lcom/example/temptemp/SecondActivity;
       5: aload_0
       6: invokespecial #12                 // Method java/lang/Thread."<init>":()V
       9: return

  public void run();
    Code:
       0: aload_0
       1: invokespecial #20                 // Method java/lang/Thread.run:()V
       4: ldc           #22                 // String SS
       6: ldc           #24                 // String start
       8: invokestatic  #26                 // Method android/util/Log.e:(Ljava/lang/String;Ljava/lang/String;)I
      11: pop
      12: ldc2_w        #32                 // long 15000l
      15: invokestatic  #34                 // Method java/lang/Thread.sleep:(J)V
      18: goto          26
      21: astore_1
      22: aload_1
      23: invokevirtual #38                 // Method java/lang/InterruptedException.printStackTrace:()V
      26: ldc           #22                 // String SS
      28: ldc           #43                 // String end
      30: invokestatic  #26                 // Method android/util/Log.e:(Ljava/lang/String;Ljava/lang/String;)I
      33: pop
      34: return
    Exception table:
       from    to  target type
          12    18    21   Class java/lang/InterruptedException
}
{% endhighlight java %}  

注意看**this$0**，就是对宿主类的引用。

结论：在内部类中这种内存泄露一般很容易被忽略，比如经常会在Activity onDestroy中进行一些回收，或者同步工作，这时也应当避免做耗时的操作，就算耗时操作用匿名内部类的Thread来做，也同样可能造成泄露。