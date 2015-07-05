---
layout: post
title: 内部类对宿主类private成员的访问 
date: 2015-07-05 19:25:05
categories: [android, java]
tags: [class]
---
摘自http://developer.android.com/intl/zh-cn/training/articles/perf-tips.html#PackageInner
<!--more-->
Consider the following class definition:

{% highlight java %}
public class Foo {
    private class Inner {
        void stuff() {
            Foo.this.doStuff(Foo.this.mValue);
        }
    }

    private int mValue;

    public void run() {
        Inner in = new Inner();
        mValue = 27;
        in.stuff();
    }

    private void doStuff(int value) {
        System.out.println("Value is " + value);
    }
}
{% endhighlight java %}

What's important here is that we define a private inner class (Foo$Inner) that directly accesses a private method and a private instance field in the outer class. This is legal, and the code prints "Value is 27" as expected.

The problem is that the VM considers direct access to Foo's private members from Foo$Inner to be illegal because Foo and Foo$Inner are different classes, even though the Java language allows an inner class to access an outer class' private members. To bridge the gap, the compiler generates a couple of synthetic methods:

{% highlight java %}
/*package*/ static int Foo.access$100(Foo foo) {
    return foo.mValue;
}
/*package*/ static void Foo.access$200(Foo foo, int value) {
    foo.doStuff(value);
}
{% endhighlight java %}

The inner class code calls these static methods whenever it needs to access the mValue field or invoke the doStuff() method in the outer class. What this means is that the code above really boils down to a case where you're accessing member fields through accessor methods. Earlier we talked about how accessors are slower than direct field accesses, so this is an example of a certain language idiom resulting in an "invisible" performance hit.

If you're using code like this in a performance hotspot, you can avoid the overhead by declaring fields and methods accessed by inner classes to have package access, rather than private access. Unfortunately this means the fields can be accessed directly by other classes in the same package, so you shouldn't use this in public API.

也就是说在内部类中对宿主类的私有成员的访问时，会生成 static 的 access$xxx 方法来对宿主类的私有成员的访问做桥接。从而会增加类的方法数以及调用的开销。