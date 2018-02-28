title: Redux更新数据之组件复用
date: 2018-02-11 18:41:43
tags: 
- react 
- redux
---


这是一篇关于我常用的关于如何进行组件复用的方法总结。
先来张图：
![image](http://7xnpna.com1.z0.glb.clouddn.com/redux%E6%B5%81%E7%A8%8B.png)

简单来说，redux更新数据的步骤如上面的流程(好吧，我只画了一部分)

先来了解这种组件复用的更新的流程：
<!--more-->
1.  `buildConstant` 方法，这是用来构造不同 `actionType` 的 ，`action` 和 `reducer`里都用到这个方法，他们通过传入一致的 `PREFIX`, `CONST` 返回一致的 `actionType`

![image](http://7xnpna.com1.z0.glb.clouddn.com/buildconstant.jpg)

2.  在 **action** 里,我们传入参数 **PREFIX** , 结合方法 **buildConstant** ,生成 **actionType** 返回整个 **Action**

![image](http://7xnpna.com1.z0.glb.clouddn.com/action.png)


3.  在 **reducer** 里，同样传入参数 **PREFIX**，保持和在 **action** 里定义的一样，生成 **actionType**, 返回整个 **Reducer**


![image](http://7xnpna.com1.z0.glb.clouddn.com/reducer.png)

4.  好了，上面方法写好了，当我们需要触发 **action**,更新数据时，只需要传入一个  **PREFIX** ，从而实例一个 **action** ,注意这个 **action** 是封装过的，就是第2步里返回的整个 **action(userAction)**，同样 **Reducer** 的使用也一样，需要传入和 **action** 的 一致的 **PREFIX**

![image](http://7xnpna.com1.z0.glb.clouddn.com/action%E8%B0%83%E7%94%A8.png)


Reucer的调用

![image](http://7xnpna.com1.z0.glb.clouddn.com/reducer%E8%B0%83%E7%94%A8.png)

现在可以看看最开始的 **redux** 更新的图，发现确实可以更新数据，那为什么要多此一举传入 **PREFIX** 更新数据，为什么不和平常一样好好写 **actionType** 呢，直接调用 **action**，为什么还要封装多一层呢？

**我们先来分析多传入一个 PREFIX 有什么特点呢？**

```
我们传入不同的PREFIX，会生成不同的 actionType.
dispatch 不同的 actionType， reducer 会根据不同的 actionType 更新对应的 state。
```

你可能在想不就是要个不一样的 actionType吗？直接自己定义不一样不就可以了吗？比如下面这样子的

哈哈，聪明的你想的没错，正常情况，我们可以直接定义一个 **actionType** 就可以了，完全不要传入 **PREFIX**，然后弄得这么复杂。。。。

可是，如果你写的组件是需要复用的，不仅仅是只是UI层面上的复用，还包括组件的 **action**, **state** 也要复用，那么这个方法就起作用了。这个时候你只需要传入一个 PREFIX，你就可以实例一个新的组件了，
这个组件有自己的 **action** 和 **state**,你可以直接调用该组件的**action**, 获取该组件的**state**, 不需要额外写。

简单来说，上面的这种写法主要适用于组件复用的场景，该组件的特点是实例化之后有自己的 **state**, **action**，如果你希望组件可以复用，并且希望触发他们的 **action**，更新不同的 **state** 你就可以使用这种方法啦。

其实之前也尝试用过这种方法封装过一个分页器，想了解更多可以参考我的Github。。[戳我戳我 react-redux-pagination](https://github.com/SelinaYu/react-redux-pagination)