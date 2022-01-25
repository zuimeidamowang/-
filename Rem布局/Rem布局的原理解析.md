# Rem布局的原理解析

## 什么是Rem

rem和em很容易混淆，其实两个都是css的单位，并且也都是相对单位，现有的em，css3才引入的rem，在介绍rem之前，我们先来了解下em

> em作为font-size的单位时，其代表父元素的字体大小，em作为其他属性单位时，代表自身字体大小——MDN

问s1、s2、s5、s6的`font-size`和`line-height`分别是多少px，先来想一想，结尾处有答案和解释

```
<div class="p1">
    <div class="s1">1</div>
    <div class="s2">1</div>
</div>
<div class="p2">
    <div class="s5">1</div>
    <div class="s6">1</div>
</div>
.p1 {font-size: 16px; line-height: 32px;}
.s1 {font-size: 2em;}
.s2 {font-size: 2em; line-height: 2em;}

.p2 {font-size: 16px; line-height: 2;}
.s5 {font-size: 2em;}
.s6 {font-size: 2em; line-height: 2em;}
```

em可以让我们的页面更灵活，更健壮，比起到处写死的px值，em似乎更有张力，改动父元素的字体大小，子元素会等比例变化，这一变化似乎预示了无限可能

有些人提出用em来做弹性布局页面，但其复杂的计算让人诟病，甚至有人专门做了个px和em的[计算器](https://vasilis.nl/nerd/code/emcalc/)，不同节点像素值对应的em值，o(╯□╰)o

![img](https://yanhaijing.com/blog/519.png)

em做弹性布局的缺点还在于牵一发而动全身，一旦某个节点的字体大小发生变化，那么其后代元素都得重新计算，X﹏X

> rem作用于非根元素时，相对于根元素字体大小；rem作用于根元素字体大小时，相对于其出初始字体大小——MDN

rem取值分为两种情况，设置在根元素时和非根元素时，举个例子

```
/* 作用于根元素，相对于原始大小（16px），所以html的font-size为32px*/
html {font-size: 2rem}

/* 作用于非根元素，相对于根元素字体大小，所以为64px */
p {font-size: 2rem}
```

rem有rem的优点，em有em的优点，尺有所短，寸有所长，我一直不觉得技术没有什么对错，只有适合不适合，有对错的是使用技术的人，杰出与优秀的区别就在于能否选择合适的技术，并让其发挥优势

我一直觉得em就是为字体和行高而生的，有些时候子元素字体就应该相对于父元素，元素行高就应该相对于字体大小；而rem的有点在于统一的参考系

## Rem布局原理

rem布局的本质是什么？这是我问过很多人的一个问题，但得到的回答都差强人意。

其实rem布局的本质是等比缩放，一般是基于宽度，试想一下如果UE图能够等比缩放，那该多么美好啊

假设我们将屏幕宽度平均分成100份，每一份的宽度用x表示，`x = 屏幕宽度 / 100`，如果将x作为单位，x前面的数值就代表屏幕宽度的百分比

```
p {width: 50x} /* 屏幕宽度的50% */
```

如果想要页面元素随着屏幕宽度等比变化，我们需要上面的x单位，不幸的是css中并没有这样的单位，幸运的是在css中有rem，通过rem这个桥梁，可以实现神奇的x

通过上面对rem的介绍，可以发现，如果子元素设置rem单位的属性，通过更改html元素的字体大小，就可以让子元素实际大小发生变化

```Css
html {font-size: 16px}
p {width: 2rem} /* 32px*/

html {font-size: 32px}
p {width: 2rem} /*64px*/
```

如果让html元素字体的大小，恒等于屏幕宽度的1/100，那1rem和1x就等价了

```
html {fons-size: width / 100}
p {width: 50rem} /* 50rem = 50x = 屏幕宽度的50% */
```

如何让html字体大小一直等于屏幕宽度的百分之一呢？ 可以通过js来设置，一般需要在页面dom ready、resize和屏幕旋转中设置

```
document.documentElement.style.fontSize = document.documentElement.clientWidth / 100 + 'px';
```

那么如何把UE图中的获取的像素单位的值，转换为已rem为单位的值呢？公式是`元素宽度 / UE图宽度 * 100`，让我们举个例子，假设UE图尺寸是640px，UE图中的一个元素宽度是100px，根据公式`100/640*100 = 15.625`

```
p {width: 15.625rem}
```

下面来验证下上面的计算是否正确，下面的表格是UE图等比缩放下，元素的宽度

| UE图宽度 | UE图中元素宽度 |
| :------- | :------------- |
| 640px    | 100px          |
| 480px    | 75px           |
| 320px    | 50px           |

下面的表格是通过我们的元素在不同屏幕宽度下的计算值

| 页面宽度 | html字体大小    | p元素宽度        |
| :------- | :-------------- | :--------------- |
| 640px    | 640/100 = 6.4px | 15.625*6.4=100px |
| 480px    | 480/100=4.8px   | 15.625*4.8=75px  |
| 320px    | 320/100=3.2px   | 15.625*3.2=50px  |

可以发现，UE图宽度和屏幕宽度相同时，两边得出的元素宽度是一致的

上面的计算过程有些繁琐，可以通过预处理的function来简化过程，下面是sass的例子，less类似

```
$ue-width: 640; /* ue图的宽度 */

@function px2rem($px) {
  @return #{$px/$ue-width*100}rem;
}

p {
  width: px2rem(100);
}
```

上面的代码编译完的结果如下

```
p {width: 15.625rem}
```

其实有了postcss后，这个过程应该放到postcss中，源代码如下

```
p {width: 100px2rem}
```

postcss会对px2rem这个单位进行处理，处理后的结果如下，感兴趣的话来写一个px2rem的postcss插件吧

```
p {width: 15.625rem}
```

## 比Rem更好的方案

上面提到想让页面元素随着页面宽度变化，需要一个新的单位x，x等于屏幕宽度的百分之一，css3带来了rem的同时，也带来了vw和vh

> vw —— 视口宽度的 1/100；vh —— 视口高度的 1/100 —— MDN

聪明的你也许一经发现，这不就是单位x吗，没错根据定义可以发现1vw=1x，有了vw我们完全可以绕过rem这个中介了，下面两种方案是等价的，可以看到vw比rem更简单，毕竟rem是为了实现vw么

```
/* rem方案 */
html {fons-size: width / 100}
p {width: 15.625rem}

/* vw方案 */
p {width: 15.625vw}
```

vw还可以和rem方案结合，这样计算html字体大小就不需要用js了

```
html {fons-size: 1vw} /* 1vw = width / 100 */
p {width: 15.625rem}
```

虽然vw各种优点，但是vw也有缺点，首先vw的兼容性不如rem好，使用之前要看下

| 兼容性 | Ios  | 安卓 |
| :----- | :--- | :--- |
| rem    | 4.1+ | 2.1+ |
| vw     | 6.1+ | 4.4+ |

另外，在使用弹性布局时，一般会限制最大宽度，比如在pc端查看我们的页面，此时vw就无法力不从心了，因为除了width有max-width，其他单位都没有，而rem可以通过控制html根元素的font-size最大值，而轻松解决这个问题