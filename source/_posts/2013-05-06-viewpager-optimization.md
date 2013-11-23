---
title: "关于ViewPager性能的讨论"
date: 2013-05-06 10:05:11 +0800
categories:
- android
comments: true

---

> [@王中晟edison][] 
> 发现一个小trick...`ViewPager`的`setOffScreeLimit`, 如果小的话页面内使用`requestLayout`的`Animation`就会很快，如果大的话页面之间的`Animation`就会快，所以如果我们根据动画的需要动态改变这个数字的话就会最大程度的优化整个App的动画！ [@KorukH][] [@developerWorks][]
>> KorukH：这个方法应该是设置系统为你缓存页面（`page`）的，缓存的页面不会重新`recreate`，所以速度会更快，在重写`ViewPager`的时候用的比较多。 <http://t.cn/zTYYJSP>
> > >王中晟edison：是的，但是大家没有想到的是反过来使用这个属性来优化页面里面的动画。因为页面同时在缓存得越多，`requestLayout()`就会慢。
>>>>KorukH: 这个倒不知道，有没有文章介绍？缓存会影响`requestLayout`… 官方文档好像没提到这方面
>>>>>王中晟edison：Romain Guy在这里有说过。因为所有Fragment的Views都会在`ViewPager`里面，所以如果你缓存了以后`requestLayout`会在所有的页面上都call.你页面里面的`requestLayout`就会出现。 
___  
我是那天在把一个`TabActivity`转换成`FragmentActivity`时，发现里面的一个`Animation`突然变得很慢了发现的，如果只在有`requestLayout`()的`Animation`的那个页面将缓存变低（`onPageScrollState`是Idle时），而在其他页面将缓存变高，就可以在大部分地方都Smooth了。  
___ 
Romain Guy's StackOverflow Answer: <http://t.cn/zTYT6pr>
___

>>>>>> KorukH：回复@王中晟edison:我不敢说听明白了… “只在有requestLayout()的Animation的那个页面将缓存变低”这句什么意思？idle状态的时候ViewPager就停止滚动了吧，还是说有动画的那个页面不可见时将缓存调高，可见时调低？

>>>>>>> 王中晟edison：回复@KorukH: 假设这么个情形：你有4个Tabs, Tab#4有用requestLayout的动画。如果你把setOffScreenPageLimit(4)的话，4个Tabs之间的移动会非常圆滑。但是Tab #4上面的动画会很慢。如果你把setOffScreenPageLimit(1)得话，4个页面之间的移动会很卡，但是Tab#4上面的就很顺畅了。
___
>>>>>>> 王中晟edison：回复@KorukH: 所以如果你在你的onPageChangedListener上通过知道用户在哪个界面来改变的话，就最好了。也就是说，用户到Tab #4的时候我们setOffScreenPageLimit(1)，但是其他任何页面上都setOffScreenPageLimit(4),这样页面1->2->3->4之间的滑动还是顺畅的，只有3<-4之间的会卡一点，#4上面的页会顺畅
___
>>>>>>> 王中晟edison：哦，当然，最好是在onPageScrolledChanged Idle得时候，不然可能会在滑动时有意想不到的NPE.
___
>>>>>>>> KorukH：回复@王中晟edison:是不是有点太hack的感觉了。不能像Romain Guy说的用其他方法？另外idle的时候界面上显示的就是滑动结束只有一个page吧，不是说用户正在滑动中？
>>>>>>>>> 王中晟edison(纽约工程师):回复@KorukH: 话说Romain Guy..我的搭档去年在Google I/O见过他，说他很臭屁和牛逼，所以我们想问核心团队问题现在一般都是通过Android Developer Advocate的那帮人，他们很好心，有什么难解的问题都可以问他们，他们解决问题的效率有时候要比在stackoverflow上高，特别是device特别的问题，他们都遇过


> [崔海斌DX][]
应该说的是`setOffscreenPageLimit`吧，此语句主要是用来设置当前页面左右的页面限制的，超出此范围的`Fragment`会被杀掉，因此，当此限制增加的时候，`Fragment`会被更多创建出来并且保持不被杀掉，因此当滑动时因为已经创建了，动画就更加流畅。但当有大量复杂页面存在时，会耗费大量资源，因此使用要小心


	2013年5月6日 下午1:20



[@王中晟edison]:http://weibo.com/wzsddtc "@王中晟edison"
[@KorukH]:http://weibo.com/n/KorukH "@KorukH"
[@developerWorks]:http://weibo.com/n/developerWorks "@developerWorks"
[崔海斌DX]:http://weibo.com/billytsuiƒƒ