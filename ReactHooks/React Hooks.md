# `React Hooks`

## 1 为什么要有`React Hooks`

介绍`Hooks`之前，首先要给大家说一下`React`的组件创建方式，一种是**类组件**，一种是**纯函数组件**，并且`React`团队希望，组件不要变成复杂的容器，最好只是数据流的管道。开发者根据需要，组合管道即可。也就是说**组件的最佳写法应该是函数，而不是类。**
 但是我们知道，在以往开发中*类组件*和*纯函数组件*的区别是很大的，纯函数组件有着类组件不具备的多种特点，简单列举几条

- 纯函数组件**没有状态**
- 纯函数组件**没有生命周期**
- 纯函数组件没有`this`
- 只能是纯函数

这就注定，我们所推崇的函数组件，只能做UI展示的功能，涉及到状态的管理与切换，我们不得不用类组件或者redux，但我们知道类组件的也是有缺点的，比如，遇到简单的页面，你的代码会显得很重，并且每创建一个类组件，都要去继承一个`React`实例，至于`Redux`,更不用多说，很久之前`Redux`的作者就说过，“能用`React`解决的问题就不用`Redux`”,等等一系列的话。关于`React`类组件`redux`的作者又有话说

> - 大型组件很难拆分和重构，也很难测试。
> - 业务逻辑分散在组件的各个方法之中，导致重复逻辑或关联逻辑。
> - 组件类引入了复杂的编程模式，比如 render props 和高阶组件。
