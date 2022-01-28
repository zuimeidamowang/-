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

## Rem不是银弹

rem是弹性布局的一种实现方式，弹性布局可以算作响应式布局的一种，但响应式布局不是弹性布局，弹性布局强调等比缩放，100%还原；响应式布局强调不同屏幕要有不同的显示，比如媒体查询

> 用户选择大屏幕有两个几个出发点，有些人想要更大的字体，更大的图片，比如老花眼的我；有些人想要更多的内容，并不想要更大的图标；有些人想要个镜子。。。——颜海镜

我认为一般内容型的网站，都不太适合使用rem，因为大屏用户可以自己选择是要更大字体，还是要更多内容，一旦使用了rem，就剥夺了用户的自由，比如百度知道，百度经验都没有使用rem布局；一些偏向app类的，图标类的，图片类的，比如淘宝，活动页面，比较适合使用rem，因为调大字体时并不能调大图标的大小

rem可以做到100%的还原度，但同事rem的制作成本也更大，同时使用rem还有一些问题，下面我们一一列举下：

首先是字体的问题，字体大小并不能使用rem，字体的大小和字体宽度，并不成线性关系，这个函数我还没算出来，所以字体大小不能使用rem；由于设置了根元素字体的大小，会影响所有没有设置字体大小的元素，因为字体大小是会继承的，难道要每个元素都显示设置字体大小？？？

我们可以在body上做字体修正，比如把body字体大小设置为16px，但如果用户自己设置了更大的字体，此时用户的设置将失效，比如合理的方式是，将其设置为用户的默认字体大小

```
html {fons-size: width / 100}
body {font-size: 16px}
```

那字体的大小如何实现响应式呢？可以通过修改body字体的大小来实现，同时所有设置字体大小的地方都是用em单位，对就是em，因为只有em才能实现，同步变化，我早就说过em就是为字体而生的

当然不同屏幕字体大小相同也是非常合理和不错的效果，需要你自己做决策

```
@media screen and (min-width: 320px) {
    body {font-size: 16px}
}
@media screen and (min-width: 481px) and (max-width:640px) {
    body {font-size: 18px}
}
@media screen and (min-width: 641px) {
    body {font-size: 20px}
}

p {font-size: 1.2em}
p a {font-size: 1.2em}
```

第二，如果用户在PC端浏览，页面过宽怎么办？一般我们都会设置一个最大宽度，大于这个宽度的话页面居中，两边留白

```
var clientWidth = document.documentElement.clientWidth;
clientWidth = clientWidth < 780 ? clientWidth : 780;
document.documentElement.style.fontSize = clientWidth / 100 + 'px';
```

设置body的宽度为100rem，并水平居中

```
body {
    margin: auto;
    width: 100rem
}
```

第三，如果用户禁用了js怎么破？其实这种用户真不多了，要不放弃吧。。。

首先可以添加noscript标签提示用户

```
<noscript>开启JavaScript，获得更好的体验</noscript>
```

给html添加一个320时的默认字体大小，保证页面可以显示

```
html {fons-size: 3.2px}
```

如果你想要更好的体验，不如添加媒体查询吧

```
@media screen and (min-width: 320px) {
    html {font-size: 3.2px}
}
@media screen and (min-width: 481px) and (max-width:640px) {
    html {font-size: 4.8px}
}
@media screen and (min-width: 641px) {
    html {font-size: 6.4px}
}
```

rem不是银弹，这个世上也没有银弹，每个方案都有其优点，也有其缺点，学会做出选择和妥协

rem仅能做到内容的缩放，但是对于非矢量资源，比如图片放大时的失真，并无法解决，这个以后有缘再讨论。

## Rem布局方案

通过上面可以得出实现缩放布局，共有四种方案，下面做一个对比

| 缩放布局        | 用户体验 | 兼容性       | 依赖js | 超大屏幕 | 修正字体 |
| :-------------- | :------- | :----------- | :----- | :------- | :------- |
| rem+media-query | 可       | IOS4.1 AN2.1 | √      | √        | ×        |
| rem+js          | 良       | IOS4.1 AN2.1 | ×      | √        | ×        |
| rem+vw          | 优       | IOS6.1 AN4.4 | √      | √        | ×        |
| vw              | 优       | IOS6.1 AN4.4 | √      | ×        | √        |

如果要求兼容性，建议rem+js方案，需要解决的问题如下：

- 修正body字体大小
- 浏览器禁用js（可选）
- 宽度限制，超大屏幕居中（可选）
- 字体缩放（可选）

如果兼容性满足，建议使用rem+vw方案，需要解决的问题如下：

- 修正body字体大小
- 宽度限制，超大屏幕居中（可选）
- 字体缩放（可选）

但是上面的方案还有个问题，就是分成100份的话，假设屏幕宽度320，此时html大小是3.2px，但浏览器支持最小字体大小是12px，怎么办？那就分成10份呗，只要把上面的100都换成10就好了

下面给一个rem+js方案的完整例子，css的计算没有使用预处理器，这个很简单

html代码如下

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">

    <title>rem布局——rem+js</title>
</head>
<body>
    <noscript>开启JavaScript，获得更好的体验</noscript>

    <div class="p1">
        宽度为屏幕宽度的50%，字体大小1.2em
        <div class="s1">
            字体大小1.2.em
        </div>
    </div>

    <div class="p2">
        宽度为屏幕宽度的40%，字体大小默认
        <div class="s2">
            字体大小1.2em
        </div>
    </div>
</body>
</html>
```

css代码如下

```
html {
    font-size: 32px; /* 320/10 */
}
body {
    font-size: 16px; /* 修正字体大小 */
    /* 防止页面过宽 */
    margin: auto;
    padding: 0;
    width: 10rem;
    /* 防止页面过宽 */
    outline: 1px dashed green;
}

/* js被禁止的回退方案 */
@media screen and (min-width: 320px) {
    html {font-size: 32px}
    body {font-size: 16px;}
}
@media screen and (min-width: 481px) and (max-width:640px) {
    html {font-size: 48px}
    body {font-size: 18px;}
}
@media screen and (min-width: 641px) {
    html {font-size: 64px}
    body {font-size: 20px;}
}

noscript {
    display: block;
    border: 1px solid #d6e9c6;
    padding: 3px 5px;
    background: #dff0d8;
    color: #3c763d;
}
/* js被禁止的回退方案 */

.p1, .p2 {
    border: 1px solid red;
    margin: 10px 0;
}

.p1 {
    width: 5rem;
    height: 5rem;

    font-size: 1.2em; /* 字体使用em */
}

.s1 {
    font-size: 1.2em; /* 字体使用em */
}

.p2 {
    width: 4rem;
    height: 4rem;
}
.s2 {
    font-size: 1.2em /* 字体使用em */
}
```

js代码如下

```
var documentElement = document.documentElement;

function callback() {
    var clientWidth = documentElement.clientWidth;
    // 屏幕宽度大于780，不在放大
    clientWidth = clientWidth < 780 ? clientWidth : 780;
    documentElement.style.fontSize = clientWidth / 10 + 'px';
}

document.addEventListener('DOMContentLoaded', callback);
window.addEventListener('orientationchange' in window ? 'orientationchange' : 'resize', callback);
```

完整的例子如下

- [rem+js的例子](http://yanhaijing.com/rem/rem-and-js.html)
- [rem+vw的例子](http://yanhaijing.com/rem/rem-and-vw.html)
- [vw的例子](http://yanhaijing.com/rem/vw.html)

页面效果如下

![img](https://yanhaijing.com/blog/520.png)


如果对本文有什么疑问，欢迎留言讨论；如果觉得本文对你有帮助，那就赶紧赞赏吧，^_^

最后填一下开头埋的雷吧，demo在[这里](http://yanhaijing.com/rem/demo.html)，代码如下

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

先来看第一组的答案

```
p1：font-size: 16px; line-height: 32px
s1：font-size: 32px; line-height: 32px
s2：font-size: 32px; line-height: 64px
```

和你的答案一样吗？下面来解释下

- p1 无需解释
- s1 em作为字体单位，相对于父元素字体大小；line-height继承父元素计算值
- s2 em作为行高单位时，相对于自身字体大小

再来看看第二组的答案

```
p2：font-size: 16px; line-height: 32px
s5：font-size: 32px; line-height: 64px
s6：font-size: 32px; line-height: 64px
```

意不意外？惊不惊喜？下面来解释下

- p2 `line-height: 2`自身字体大小的两倍
- s5 数字无单位行高，继承原始值，s5的line-height继承的2，自身字体大小的两倍
- s6 无需解释