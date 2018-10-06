## MDN上关于border-radius的介绍
* one,two,three,or four `<length>` or `<percentage>` values. This is used to set a single radius for the coners.
* followed optionally by "/" and one, two, three, or four `<length>` or `<percentage>` values. This is used to set an additional radius, so you can have elliptical corners.

贴心附上链接，请[点击这里](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius)
## 深度挖掘border-radius
今天我们只关注上面的第二个点，也就是border-radius的八个值

首先来看一下效果（多图预警）：

![效果一](./img/img01.png)

![效果二](https://user-gold-cdn.xitu.io/2018/10/6/1664764571960ee8?w=153&h=80&f=png&s=1244)

![效果三](https://user-gold-cdn.xitu.io/2018/10/6/16647646679b1de3?w=163&h=81&f=png&s=1971)

![效果四](https://user-gold-cdn.xitu.io/2018/10/6/1664764e0242d164?w=184&h=180&f=png&s=3645)

![效果五](https://user-gold-cdn.xitu.io/2018/10/6/1664764f7ba156a8?w=153&h=148&f=png&s=2751)

来几个酷炫的按钮（大图预警）：

![按钮一](https://user-gold-cdn.xitu.io/2018/10/6/166476647285452f?w=428&h=184&f=png&s=13126)

![按钮二](https://user-gold-cdn.xitu.io/2018/10/6/166476676ca514ff?w=431&h=229&f=png&s=9711)

![按钮三](https://user-gold-cdn.xitu.io/2018/10/6/1664766917c41f07?w=240&h=236&f=png&s=8573)

![按钮四](https://user-gold-cdn.xitu.io/2018/10/6/16647669eb2c056d?w=228&h=226&f=png&s=13758)

这四个按钮不仅用了border-radius还用了box-shadow和linear-gradient

#### 简单回顾一下border-radius的八个值

```
/* 除了长度单位还可以用%表示 */
border-radius: 10px/20px;
border-radius: 10px 20px/20px 10px;
border-radius: 10px 20px 30px/30px 20px 10px;
border-radius: 10px 20px 30px 40px/40px 30px 20px 10px;
```

#### 实现的原理

以下图为例


![例图](https://user-gold-cdn.xitu.io/2018/10/6/1664774bc393002e?w=295&h=146&f=png&s=3003)

代码如下
```
border-radius: 100px/50px;
```
大家应该都清楚，上面的代码也是简写形式，完整形式应该如下：
```
border-radius: 100px 100px 100px 100px/50px 50px 50px 50px;
```
我们简单分析一下上面的代码。“/”的左边表示的其实是左上，右上，右下，左下四个角水平方向的半径，而“/”右边表示的左上，右上，右下，左下四个角垂直方向的半径。

由于四个角是类似的，我们以左上角为例，作用原理图如下（图有点丑，没办法，审美有问题，怎么绘制怎么改颜色都不能变好看......）：

![原理图](https://user-gold-cdn.xitu.io/2018/10/6/16648429701941ce?w=447&h=214&f=png&s=15811)

**我们用一个水平半径为100px（左上角椭圆中的黄线），垂直半径为50px（左上角椭圆中的红线）的椭圆紧贴左上角（椭圆的上部紧贴矩形的上部，椭圆的右部紧贴矩形的右部）。左上角矩形多余的区域就会被裁剪掉。这是裁剪一个角，如果裁剪四个角就会变成上上图的样子。**

#### 简单练习

* 这是啥图形之我也不知道叫啥
![练习题一图](https://user-gold-cdn.xitu.io/2018/10/6/166484d86a614cf7?w=153&h=80&f=png&s=1244)

代码如下：
```
border-radius: 100% 50%/0 100%;
```
* 一片叶子

![练习题二图](https://user-gold-cdn.xitu.io/2018/10/6/166484fc322a989f?w=163&h=81&f=png&s=1971)
代码如下：
```
border-radius: 0 100%/0 100%;
```
* 倾斜的椭圆

![练习题三图](https://user-gold-cdn.xitu.io/2018/10/6/1664851b5aec3fe3?w=153&h=148&f=png&s=2751)

```
border-radius: 100% 50%/100% 50%;
```
* 扭曲的相框

![练习题四图](https://user-gold-cdn.xitu.io/2018/10/6/1664852bcc2fede9?w=184&h=180&f=png&s=3645)

```
border-radius: 150px 400px 150px 400px/400px 150px 400px 150px;
```

#### 酷炫按钮练习

* 
![酷炫按钮练习题一图](https://user-gold-cdn.xitu.io/2018/10/6/16648569a2ec1bdd?w=428&h=184&f=png&s=13126)
代码如下（包含渐变和阴影）：
```
background: linear-gradient(to bottom,#FBBCD0,#FBAAC3);
box-shadow: 0 10px 10px #B9174C;
border-radius: 150px 14px/150px 14px;
```
*
![酷炫按钮练习题二图](https://user-gold-cdn.xitu.io/2018/10/6/16648591f35046e9?w=431&h=229&f=png&s=9711)
代码如下（包含渐变和阴影）：
```
background: linear-gradient(to bottom,#E8F6D9,#BEE595);
box-shadow: 0 10px 10px #4F821D;
border-radius: 50px/100px;
```
*

![酷炫按钮练习题三图](https://user-gold-cdn.xitu.io/2018/10/6/166485a1406b22a0?w=240&h=236&f=png&s=8573)
代码如下（包含渐变和阴影）：
```
background: linear-gradient(to bottom,#D7EAFD,#A3C7E8);
box-shadow: 0 10px 10px #4179AB;
border-radius: 50px 50px 20px 20px/200px 200px 20px 20px;
```

*

![酷炫按钮练习题四图](https://user-gold-cdn.xitu.io/2018/10/6/166485b0653935da?w=228&h=226&f=png&s=13758)
代码如下（包含渐变和阴影）：
```
background: linear-gradient(to bottom,#FDE0AF,#E4B260);
border-radius: 15px 15px 50% 50%/15px 15px 100% 100%;
box-shadow: 0 10px 10px #986206;
```

## 总结

其实你只要看懂了上面的原理图，关于border-radius就可以算是理解了。剩下的就是发挥你的创造力去制作出不同效果的图形，可以结合渐变和阴影制作出很漂亮的图形。

#### 额外收获

在研究border-radius的时候竟然收获到了如何用 DIV 来模拟梯形，学习处处有惊喜！


![梯形](https://user-gold-cdn.xitu.io/2018/10/6/166486d080d13f7a?w=213&h=78&f=png&s=936)

这里只附上代码，对DIV模拟梯形感兴趣并且不清楚怎么实现的朋友可以参考一下，若您都会，请跳过
```
height: 0;
width: 400px;
border-bottom: 200px solid lightgreen;
border-left: 100px solid transparent;
border-right: 100px solid transparent;
/* 
    实现梯形实际是相当于在实现三角形的基础上(width=0)更改width。
    这里实现的是等腰梯形。若想实现非等腰梯形，可以让左右边框的宽度不相等。
*/
```

#### 题外话

这是我在掘金上的第一篇博客。可能有很多不当之处，欢迎大家批评指教。我昨晚在研究
border-radius 和写这篇博客时参考了部分其他人的博客，但是他们的博客都不是原文出处，应该是转发他人的，而且还没有附上原文出处链接。所以我在这里就不附上这些链接了。

虽然参考了他人的文章，但这篇博文的确是我一字一字手打的，没有任何粘贴复制。而且上面的原理图之类的都是我自己重新绘制的，保证原图出处在这。练习中酷酷的按钮都是我在网络上找的按钮图然后自己用圆角、阴影、渐变一点一点制作出来的。

欢迎转发。码字不易，转发时请自觉附上原文链接，谢谢合作。
