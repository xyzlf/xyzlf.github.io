---
layout: post
title: Fragment实例化，Fragment生命周期源码分析
date: 2015-05-19 22:21:34
category: Android
tags: Android Fragment UI
---
&nbsp;&nbsp;&nbsp;&nbsp;以下从FragmentActivity.onCreate开始分析Fragment的实例化，以及生命周期。<!--more-->  
1. android.support.v4.app.FragmentActivity#onCreate  
{% highlight java %}
@Override
protected void onCreate(Bundle savedInstanceState) {
    mFragments.attachActivity(this, mContainer, null);
    // Old versions of the platform didn't do this!
    if (getLayoutInflater().getFactory() == null) {
        getLayoutInflater().setFactory(this);
    }
    super.onCreate(savedInstanceState);
   // ....
}
{% endhighlight java %}
&nbsp;&nbsp;&nbsp;&nbsp;这里主要是对LayoutInflater设置了Factory（注意：在同一个Activity中使用getBaseContext和使用Activity.this通过LayoutInflater.from获得的LayoutInflater是同一个，但是不同的Activity获得的不是同一个），这个Factory的作用主要是用来作为hook，便于FragmentActivity通过“fragment”Tag来创建Fragment实例。
&nbsp;&nbsp;&nbsp;&nbsp;然后设置android.app.Activity#setContentView(int) 会触发android.view.LayoutInflater#inflate(int, android.view.ViewGroup)->android.view.LayoutInflater#createViewFromTag。  
2. android.view.LayoutInflater#createViewFromTag  
{% highlight java %}
View view;
   if (mFactory2 != null) {
       view = mFactory2.onCreateView(parent, name, viewContext, attrs);
   } else if (mFactory != null) {
       view = mFactory.onCreateView(name, viewContext, attrs);
   } else {
       view = null;
   }
{% endhighlight java %}
&nbsp;&nbsp;&nbsp;&nbsp;这里会调用mFactory的onCreateView方法，这里的mFactory也就是FragmentActivity。  
3. android.support.v4.app.FragmentActivity#onCreateView  
{% highlight java %}
@Override
public View onCreateView(String name, @NonNull Context context, @NonNull AttributeSet attrs) {
    if (!"fragment".equals(name)) {
        return super.onCreateView(name, context, attrs);
    }

    final View v = mFragments.onCreateView(name, context, attrs);
    if (v == null) {
        return super.onCreateView(name, context, attrs);
    }
    return v;
}
{% endhighlight java %}
&nbsp;&nbsp;&nbsp;&nbsp;先判断是否是“fragment”Tag，如果不是，返回super.onCreateView，父类是直接返回null，这样会丢给LayoutInflater中自己处理。如果是“fragment”Tag，那么会调用FragmentManagerImpl的onCreateView去处理。  
4. android.support.v4.app.FragmentManagerImpl#onCreateView
{% highlight java %}
@Override
public View onCreateView(String name, Context context, AttributeSet attrs) {
    /// ...
    if (fragment == null) {
        fragment = Fragment.instantiate(context, fname);
        fragment.mFromLayout = true;
        fragment.mFragmentId = id != 0 ? id : containerId;
        fragment.mContainerId = containerId;
        fragment.mTag = tag;
        fragment.mInLayout = true;
        fragment.mFragmentManager = this;
        fragment.onInflate(mActivity, attrs, fragment.mSavedFragmentState);
        addFragment(fragment, true);

    } 
    // ...
    return fragment.mView;
}
{% endhighlight java %}
&nbsp;&nbsp;&nbsp;&nbsp;这里会直接调用Fragment.instantiate方法去构造Fragment实例。  
5. android.support.v4.app.Fragment#instantiate(android.content.Context, java.lang.String, android.os.Bundle)  
{% highlight java %}
public static Fragment instantiate(Context context, String fname, Bundle args) {
        Class<?> clazz = sClassMap.get(fname);
        if (clazz == null) {
            // Class not found in the cache, see if it's real, and try to add it
            clazz = context.getClassLoader().loadClass(fname);
            sClassMap.put(fname, clazz);
        }
        Fragment f = (Fragment)clazz.newInstance();
        if (args != null) {
            args.setClassLoader(f.getClass().getClassLoader());
            f.mArguments = args;
        }
        return f;
    
}
{% endhighlight java %}
&nbsp;&nbsp;&nbsp;&nbsp;直接通过class.newInstance反射调用了Fragment的无参构造来获得新的Fragment实例。同时在第4步最后会调用addFragment来把新的Fragment添加到FragmentManagerImpl中。  
6. android.support.v4.app.FragmentManagerImpl#addFragment
{% highlight java %}
if (moveToStateNow) {
    moveToState(fragment);
}
{% endhighlight java %}
&nbsp;&nbsp;&nbsp;&nbsp;这个方法核心点会去进行Fragment的状态变换。  
7. android.support.v4.app.FragmentManagerImpl#moveToState(android.support.v4.app.Fragment, int, int, int, boolean)
{% highlight java %}
if (DEBUG) Log.v(TAG, "moveto CREATED: " + f);
if (f.mSavedFragmentState != null) {
    f.mSavedFragmentState.setClassLoader(mActivity.getClassLoader());
    f.mSavedViewState = f.mSavedFragmentState.getSparseParcelableArray(
            FragmentManagerImpl.VIEW_STATE_TAG);
    f.mTarget = getFragment(f.mSavedFragmentState,
            FragmentManagerImpl.TARGET_STATE_TAG);
    if (f.mTarget != null) {
        f.mTargetRequestCode = f.mSavedFragmentState.getInt(
                FragmentManagerImpl.TARGET_REQUEST_CODE_STATE_TAG, 0);
    }
    f.mUserVisibleHint = f.mSavedFragmentState.getBoolean(
            FragmentManagerImpl.USER_VISIBLE_HINT_TAG, true);
    if (!f.mUserVisibleHint) {
        f.mDeferStart = true;
        if (newState > Fragment.STOPPED) {
            newState = Fragment.STOPPED;
        }
    }
}
f.mActivity = mActivity;
f.mParentFragment = mParent;
f.mFragmentManager = mParent != null
        ? mParent.mChildFragmentManager : mActivity.mFragments;
f.mCalled = false;
f.onAttach(mActivity);
if (!f.mCalled) {
    throw new SuperNotCalledException("Fragment " + f
            + " did not call through to super.onAttach()");
}
if (f.mParentFragment == null) {
    mActivity.onAttachFragment(f);
}

if (!f.mRetaining) {
    f.performCreate(f.mSavedFragmentState);
}
f.mRetaining = false;
if (f.mFromLayout) {
    // For fragments that are part of the content view
    // layout, we need to instantiate the view immediately
    // and the inflater will take care of adding it.
    f.mView = f.performCreateView(f.getLayoutInflater(
            f.mSavedFragmentState), null, f.mSavedFragmentState);
    if (f.mView != null) {
        f.mInnerView = f.mView;
        if (Build.VERSION.SDK_INT >= 11) {
            ViewCompat.setSaveFromParentEnabled(f.mView, false);
        } else {
            f.mView = NoSaveStateFrameLayout.wrap(f.mView);
        }
        if (f.mHidden) f.mView.setVisibility(View.GONE);
        f.onViewCreated(f.mView, f.mSavedFragmentState);
    } else {
        f.mInnerView = null;
    }
}
{% endhighlight java %}
&nbsp;&nbsp;&nbsp;&nbsp;这个方法依次调用了onAttach，onCreate，onCreateView，onViewCreated方法，这也就是Fragment的生命周期中比较重要的几个方法，同时，因为这些方法都是在Activity.onCreate中进行的，所以这些方法都会在onActivityCreated之前调用。  
8. android.support.v4.app.FragmentActivity#onStart
{% highlight java %}
@Override
protected void onStart() {
    super.onStart();

    mStopped = false;
    mReallyStopped = false;
    mHandler.removeMessages(MSG_REALLY_STOPPED);

    if (!mCreated) {
        mCreated = true;
        mFragments.dispatchActivityCreated();
    }
    // ...
    mFragments.dispatchStart();
    // ...
}
{% endhighlight java %}
&nbsp;&nbsp;&nbsp;&nbsp;这里会调用onActivityCreated和onStart方法。  

#### 以上就是Fragment创建的过程和一些生命周期的方法的调用顺序。  
结论：  
1. Fragment必须有无参构造，否则无法被实例化。  
2. Fragment生命周期：Activity.onCreate->Fragment.onAttach->Fragment.onCreate->Fragment.onCreateView->Fragment.onViewCreated->Activity.onStart->Fragment.onActivityCreated->Fragment.onStart
