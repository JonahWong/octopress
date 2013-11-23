---
title: "Android新增的导航模式-Navigation Drawer"
date: 2013-05-18 12:00:11 +0800
categories:
- android
comments: true

---

## Navigation Drawer  - 第一部分   

这篇文章是我在Google I/O 2013大会期间写的。会上`Google`发布了很多新的东西，其中之一就是第13次修订的`v4 support library`（v4表示最低支持`Api Level` 4，同理v13表示最低支持`Api Level `13）。这个版本的v4包对`Navigation Drawer`界面设计模式提供了官方支持。这种这种设计模式在过去的时间里迅速发展，被许多应用采纳。但在此之前都没有官方支持以及相应的规范。但此刻这种情况不存在了，官方推出了[官方设计指南](http://developer.android.com/design/patterns/navigation-drawer.html)。  

在最近的关于Android的[`Adapter`](http://blog.stylingandroid.com/archives/1679)`系列文章中，我们用Spinner导航的方式创建了一个应用，今天这篇文章中，我们会将那个Spinner导航来用`Navigation Drawer`来实现。  

首先我们需要做的是获取最新的v4兼容包（13次修订版），本文中的项目是`maven`项目，所以如果你不打算用`maven`的话，你需要删除已经通过`maven`方式导入的包，然后重新导入。   

然后我们就可以修改之前的布局文件，大致就是将其包含在`DrawerLayout`（v4包新增的布局）中，接着添加一个`ListView`用来显示导航列表。     

```
 <android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >
 
    <FrameLayout
        android:id="@+id/main"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >
    </FrameLayout>
 
    <ListView
        android:id="@+id/drawer"
        android:layout_width="240dp"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:background="#FFF"
        android:choiceMode="singleChoice" />
 
</android.support.v4.widget.DrawerLayout>      
```        
接下来我们需要改变`MainActivity`中的代码来把`Adapter`给到`ListView`，之前我们是把`Adapter`给到`Spinner`的。代码如下：

```   
@Override
protected void onCreate(Bundle savedInstanceState)
{
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    final String[] names = 
        getResources().getStringArray(R.array.nav_names);
    final String[] classes = 
        getResources().getStringArray(R.array.nav_classes);
    ArrayAdapter<String> adapter = 
        new ArrayAdapter<String>(
            getActionBar().getThemedContext(), 
            android.R.layout.simple_list_item_1, names);
 
    final DrawerLayout drawer = 
        (DrawerLayout)findViewById(R.id.drawer_layout);
    final ListView navList = 
        (ListView) findViewById(R.id.drawer);
    navList.setAdapter(adapter);
    navList.setOnItemClickListener(new OnItemClickListener()
    {
 
        @Override
        public void onItemClick(AdapterView<?> parent, 
                View view, final int pos, long id)
        {
            drawer.setDrawerListener( 
                new DrawerLayout.SimpleDrawerListener()
            {
                @Override
                public void onDrawerClosed(View drawerView)
                {
                    super.onDrawerClosed(drawerView);
                    FragmentTransaction tx = 
                        getSupportFragmentManager()
                            .beginTransaction();
                    tx.replace(R.id.main, 
                        Fragment.instantiate(
                            MainActivity.this, 
                            classes[pos]));
                    tx.commit();
                }
            });
            drawer.closeDrawer(navList);
        }
    });
 
    FragmentTransaction tx = 
        getSupportFragmentManager().beginTransaction();
    tx.replace(R.id.main,
        Fragment.instantiate(MainActivity.this, classes[0]));
    tx.commit();
    }   

```
下面我们来分析一下上面的代码：   
首先，我们需要一个不同的布局文件，因为`ListView`中的布局和之前`Spinner`中的不尽相同。接下来通过`findViewById`方法分别找到`DrawerLayout`和`ListView`，然后为`ListView`设置适配器。
___
然后就需要处理一下`ListView`的点击事件——当`ListView`中某项被点击的时候，我们先关闭`drawer`，关闭完成后，我们再将`Fragment`呈现出来。     
___
我们之所以先把`drawer`关闭然后再加载`Fragment`是因为如果关闭`drawer`和加载`Fragment`同时进行的话，可能会有卡顿现象出现。当然，你也可以选择先加载`Fragment`然后再关闭`drawer`，但在本例中为了简单起见，我们选择先关闭`drawer`。  
___   
最后，我们还需要设置默认的`Fragment`。之前我们用`Spinner`的时候，`Spinner`的默认选项使得我们可以通过`OnNavigationListener`来创建相应的`Fragment`。同样的，当我们用今天这个新的导航方式的时候，我们也必须手动设置一个默认的`Fragment`显示。    
___
完成上面这些操作，就可以得到一个简单的`Navigation Drawer`了。当我们从屏幕左侧边缘滑动的时候，侧面导航栏就会出现。   

![Navigation Drawer](http://blog.stylingandroid.com/wp-content/uploads/2013/05/simple.png)

下一篇文章中，我们会讲一下`Navigation Drawer`如何与`ActionBar`整合起来，通过`ActionBar`来控制`Navigation Drawer`的显示和隐藏之间的切换。  
本文中用到源码可以到[这里](https://bitbucket.org/StylingAndroid/adapters/src/03263bae4437e5e26c5d433bbfacb3ffd429a1cd/?at=NavigationDrawerPart1)下载。   
___
[原文链接](http://blog.stylingandroid.com/archives/1793)