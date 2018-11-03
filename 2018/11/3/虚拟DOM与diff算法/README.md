# 虚拟DOM与diff算法

### 1.为什么需要虚拟DOM？

DOM是很慢的，其元素非常庞大，页面的性能问题大部分都是由JS操作DOM引起的。如果对前端工作进行抽象的话，主要就是维护状态和更新视图；而更新视图和维护状态都需要DOM操作。其实近年来，前端的框架主要发展方向就是解放DOM操作的复杂性。 

***更新DOM是非常昂贵的操作***

### 2.什么是虚拟DOM？

> 通俗易懂的来说就是用一个简单的对象去代替复杂的DOM对象。虚拟DOM因为是纯粹的JS对象，所以操作它会很高效

### 3.diff算法

虚拟DOM（即`vdom`）是树状结构，其节点为`vnode`,`vnode`与DOM中的`Node`一一对应。对`vnode`的操作最终会转换成对DOM中`Node`的操作。所以为了实现高效的DOM操作，一套高效的虚拟DOM diff算法显得很有必要。

![React’s diff algorithm ](https://ss.csdn.net/p?http://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ6g2LN1MRr5fftnrxTkMUvHckvhwHDkYgutqibCW6ZvVZfIhlsjIP9YbQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

这是一张很经典的图，出自《React’s diff algorithm》，Vue的diff算法也同样，即仅在同级的vnode间做diff，递归地进行同级vnode的diff，最终实现整个DOM树的更新。下面介绍同级vnode diff的细节。

#### 3.1举例

<img src="https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ6k7SoPnKvw5hxxs7dfiaFvBO2R43iaJQEQ8G26PiaBNfRNn20xeaFcUjAg/0?wx_fmt=png" width="100%">



> 如上图的例子，更新前是1到10排列的Node列表，更新后是乱序排列的Node列表。罗列一下图中有以下几种类型的节点变化情况：
>
> （1）、头部相同、尾部相同的节点：如1、10
>
> （2）、头尾相同的节点：如2、9（处理完头部相同、尾部相同节点之后）
>
> （3）、新增的节点：11
>
> （4）、删除的节点：8
>
> （5）、其他节点：3、4、5、6、7

#### 3.2 简单diff算法

简单的diff算法可以这样设计：

逐个遍历`newVdom`的节点，找到它在`oldVdom`中的位置，如果找到了就移动对应的DOM元素，如果没找到说明是新增节点，则新建一个节点插入。遍历完成之后如果`oldVdom`中还有没处理过的节点，则说明这些节点在`newVdom`中被删除了，删除它们即可。

仔细思考一下，几乎每一步都要做移动DOM的操作，这在DOM整体结构变化不大时的开销是很大的，实际上DOM变化不大的情况现实中经常发生，很多时候我们只需要变更某个节点的文本而已。

接下来我们看一下Vue的diff实现。

####3.3Vue中的diff实现

上图例子中我画上了`oldStart`+`oldEnd`，`newStart`+`newEnd`这样2对指针，分别对应`oldVdom`和`newVdom`的起点和终点。起止点之前的节点是待处理的节点，Vue不断对`vnode`进行处理同时移动指针直到其中任意一对起点和终点相遇。处理过的节点Vue会在`oldVdom`和`newVdom`中同时将它标记为已处理（标记方法后文中有介绍）。Vue通过以下措施来提升diff的性能。

* **优先处理特殊场景**

  * 头部的同类型节点、尾部的同类型节点 

    这类节点更新前后位置没有发生变化，所以不用移动它们对应的DOM 

  * 头尾/尾头的同类型节点 

    这类节点位置很明确，不需要再花心思查找，直接移动DOM就好 

    **处理了这些场景之后，一方面一些不需要做移动的DOM得到快速处理，另一方面待处理节点变少，缩小了后续操作的处理范围，性能也得到提升 **

* **原地复用**

  “原地复用”是指Vue会尽可能复用DOM，尽可能不发生DOM的移动。**Vue在判断更新前后指针是否指向同一个节点，其实不要求它们真实引用同一个DOM节点，实际上它仅判断指向的是否是同类节点（比如2个不同的div，在DOM上它们是不一样的，但是它们属于同类节点），如果是同类节点，那么Vue会直接复用DOM，这样的好处是不需要移动DOM。**再看上面的实例，假如10个节点都是div，那么整个diff过程中就没有移动DOM的操作了。 

#### 3.4解析实例

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ6RN3YKLjXa2MMPiaL0QCRBdmwK4HRjE5vo0MZly2dSZ0AZ3F5pEWiaiaXg/0?wx_fmt=png)

先看一张整体视图，整个diff分两部分： 

（1）第一部分是一个循环，循环内部是一个分支逻辑，每次循环只会进入其中的一个分支，**每次循环会处理一个节点，处理之后将节点标记为已处理（`oldVdom`和`newVdom`都要进行标记，如果节点只出现在其中某一个`vdom`中，则另一个`vdom`中不需要进行标记），标记的方法有2种，当节点正好在`vdom`的指针处，移动指针将它排除到未处理列表之外即可，否则就要采用其他方法，Vue的做法是将节点设置为`undefined`。 **

（2）循环结束之后，可能`newVdom`或者`oldVdom`中还有未处理的节点，如果是`newVdom`中有未处理节点，则这些节点是新增节点，做新增处理。如果是`oldVdom`中有这类节点，则这些是需要删除的节点，相应在DOM树中删除之 。

**整个过程是逐步找到更新前后`vdom`的差异，然后将差异反应到DOM树上（也就是`patch`），特别要提一下Vue的`patch`是即时的，并不是打包所有修改最后一起操作DOM（React则是将更新放入队列后集中处理），朋友们会问这样做性能很差吧？实际上现代浏览器对这样的DOM操作做了优化，并无差别。** 

##### 3.4.1处理头部的同类型节点 

处理头部的同类型节点，即`oldStart`和`newStart`指向同类节点的情况，如下图中的节点1 

这种情况下，将节点1的变更更新到DOM，然后对其进行标记，标记方法是`oldStart`和`newStart`后移1位即可，过程中不需要移动DOM（**更新DOM或许是要的，比如属性变更了，文本内容变更了等等**） 

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ6Au4ibQUGVibXAuaTJh9RLnjbvdQFOyutdafzWrMT3iayBqIcwSWzLu3Pw/0?wx_fmt=png)

##### 3.4.2处理尾部的同类型节点 

处理尾部的同类型节点，即`oldEnd`和`newEnd`指向同类节点的情况，如下图中的节点10 

与情况（1）类似，这种情况下，将节点10的变更更新到DOM，然后`oldEnd`和`newEnd`前移1位进行标记，同样也不需要移动DOM 。

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ69RiaqpE0Q10XL92L8oNdIV2Ps73KW3Dz08lFkR08piak633VtrX9qaDg/0?wx_fmt=png)

##### 3.4.3处理头尾/尾头的同类型节点 

处理头尾/尾头的同类型节点，即`oldStart`和`newEnd`，以及`oldEnd`和`newStart`指向同类节点的情况，如下图中的节点2和节点9 。

先看节点2，其实是往后移了，移到哪里？移到`oldEnd`指向的节点（即节点9）后面，移动之后标记该节点，将`oldStart`后移1位，`newEnd`前移一位 。

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ6P17n9O3K22cvofhkuHo8OS4btwgrz9m00XA1v73J37pXVnHMq4673A/0?wx_fmt=png)

操作结束之后情况如下图 

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ67bC4nG3hBOMbUaJreLXAV9icKuodMUQ5IIzkDCn4dtrkFBwswECB2ibw/0?wx_fmt=png)

同样地，节点9也是类似的处理，处理完之后成了下面这样 

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ69usIT7f8Py6eaJqOdx9cuo3VoxofO3kj4P8H56BkjbKAJhEfbHaa0w/0?wx_fmt=png)

##### 3.4.4处理新增的节点 

`newStart`来到了节点11的位置，在`oldVdom`中找不到节点11，说明它是新增的 。

那么就创建一个新的节点，插入DOM树，插到什么位置？插到`oldStart`指向的节点（即节点3）前面，然后将`newStart`后移1位标记为已处理（注意`oldVdom`中没有节点11，所以标记过程中它的指针不需要移动），处理之后如下图 ：

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ69Fsa6J3STJJkpAj7lzyF0JMcjWyTzsjKA2sSh0uFiatMVqyLgXFpJmw/0?wx_fmt=png)

##### 3.4.5处理更新的节点 

经过第（4）步之后，`newStart`来到了节点7的位置，在`oldVdom`中能找到它而且不在指针位置（**查找`oldVdom`中`oldStart`到`oldEnd`区间内的节点**），说明它的位置移动了 。

那么需要在DOM树中移动它，移到哪里？移到`oldStart`指向的节点（即节点3）前面，**与此同时将节点标记为已处理，跟前面几种情况有点不同，`newVdom`中该节点在指针下，可以移动`newStart`进行标记，而在`oldVdom`中该节点不在指针处，所以采用设置为`undefined`的方式来标记（一定要标记吗？后面会提到）** 。

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ6aW3c1w7GBxhia5MRcLMrxJoEJWVWlVx1kNlsas8Tjmhf0Fs6daHoLgw/0?wx_fmt=png)

处理之后就成了下面这样 

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ6ozBAzSYMpcu3BoAm2ibzMHcHDY5o2Vzryc5wcrC52B5wtBTqiaiccSl8g/0?wx_fmt=png)

##### 3.4.6处理3、4、5、6节点 

经过第（5）步处理之后，我们看到了令人欣慰的一幕，`newStart`和`oldStart`又指向了同一个节点（即都指向节点3），很简单，按照（1）中的做法只需移动指针即可，非常高效，3、4、5、6都如此处理，处理完之后如下图 :

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ6BYl9pVMmnWevic3dibPzsZOTDlR9YVn4KdXHc80aWzFTiaygLEmtlTkHw/0?wx_fmt=png)

##### 3.4.7处理需删除的节点 

经过前6步处理之后（实际上前6步是循环进行的），朋友们看`newStart`跨过了`newEnd`，它们相遇啦！而这个时候，`oldStart`和`oldEnd`还没有相遇，说明这2个指针之间的节点（包括它们指向的节点，即上图中的节点7、节点8）是此次更新中被删掉的节点。 

OK，那我们在DOM树中将它们删除，**再回到前面我们对节点7做了标记，为什么标记是必需的？标记的目的是告诉Vue它已经处理过了，是需要出现在新DOM中的节点，不要删除它，所以在这里只需删除节点8。 **

**在应用中也可能会遇到`oldVdom`的起止点相遇了，但是`newVdom`的起止点没有相遇的情况，这个时候需要对`newVdom`中的未处理节点进行处理，这类节点属于更新中被加入的节点，需要将他们插入到DOM树中。 **

![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/70ke266reibiadFhB9jmmAklSLp2opYkZ6TA2Tb6C4fjUIIwHm8hActqApZ7susibu6aQQdSe0NHpbJHH3bOTHo6A/0?wx_fmt=png)



至此，整个diff过程结束了

Vue的diff算法与动态规划算法中的经典案例“计算a到b的最小编辑距离”看上去有些相似，实际完全不同，Vue的diff相对来说轻量很多，感兴趣的朋友可以查阅相关资料进行了解。



贴心附上[原文链接](https://blog.csdn.net/m6i37jk/article/details/78140159)









