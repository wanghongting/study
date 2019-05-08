### dom diff算法与虚拟dom的故事

#### 【虚拟DOM】：render 虚拟DOM + diff算法+更新必要的DOM元素


优点：最终表现在dom上的修改只是部分的变更，可以保证高效的渲染，提高网页性能

缺点：首次渲染大量dom时，由于多了一层Dom的计算，会比innerHtml慢一点

#### 虚拟DOM的理解误区

对虚拟DOM的理解往往停留在：通过JavaScript对象模拟原生DOM，加上DOM Diff 极大提升了DOM操作的

性能。然而，虚拟DOM最大的意义不在于性能的提升（JavaScript对象比DOM对象性能高），而在于抽象了

DOM的具体实现（对DOM进行了一层抽象），这在浏览器中使用 React时不是特别明显，因为写的DOM标签跟

原生的没有区别，并且最终都被渲染成了DOM，在React Native中将会有很好的说明。

详情可参考以下链接：

<a href='http://www.php.cn/js-tutorial-411868.html'>http://www.php.cn/js-tutorial-411868.html</a>

### 下面是具体diff 算法

###react diff

##### 与传统树的diff的区别

计算一棵树形结构转换成另一棵树形结构的最少操作，是一个复杂且值得研究的问题。传统 diff 算法通过循

环递归对节点进行依次对比，效率低下，算法复杂度达到 O(n^3)

### react diff策略

Web UI 中 DOM 节点跨层级的移动操作特别少，可以忽略不计。

拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构。

对于同一层级的一组子节点，它们可以通过唯一 id 进行区分。

### 1.tree diff

基于策略一，对树进行分层比较，两棵树只会对同一层次的节点进行比较。

React 通过 updateDepth 对 Virtual DOM 树进行层级控制，同一个父节点下的所有子节点。

![treediff](/study/treediff.png)

##### 什么是 DOM 节点跨层级的移动操作？

A 节点（包括其子节点）整个被移动到 D 节点下

![tree2](/study/tree2.png)

如果出现了 DOM 节点跨层级的移动操作，React diff 会有怎样的表现呢？

React 只会简单的考虑同层级节点的位置变换，而对于不同层级的节点，只有创建和删除操作。

当根节点发现子节点中 A 消失了，就会直接销毁 A；当 D 发现多了一个子节点 A，则会创建新的 A（包括子

节点）作为其子节点。此时，React diff 的执行情况：create A -> create B -> create C -> delete A。

## 注意：

在开发组件时，保持稳定的 DOM 结构会有助于性能的提升。例如，可以通过 CSS 隐藏或显示节点，而不是真的

移除或添加 DOM 节点。

### 2.component diff

依据策略二

如果是同一类型的组件，按照原策略继续比较 virtual DOM tree。

如果不是，则将该组件判断为 dirty component，从而替换整个组件下的所有子节点。

对于同一类型的组件，有可能其 Virtual DOM 没有任何变化，如果能够确切的知道这点那可以节省大量的 diff 

运算时间，因此 React 允许用户通过 shouldComponentUpdate() 来判断该组件是否需要进行 diff。

React 判断 D 和 G 是不同类型的组件，就不会比较二者的结构，而是直接删除 component D，重新创建 component

 G 以及其子节点，即使D 和 G的结构很相似

![componentdiff](/study/componentdiff.png)

### 3.element diff

当节点处于同一层级时，React diff 提供了三种节点操作，分别为：INSERT_MARKUP（插入）、MOVE_EXISTING（移动）

和 REMOVE_NODE（删除）。INSERT_MARKUP，新的 component 类型不在老集合里， 即是全新的节点，需要对新节点执行

插入操作。

MOVE_EXISTING，在老集合有新 component 类型，且 element 是可更新的类型，generateComponentChildren 已调用 

receiveComponent，这种情况下 prevChild=nextChild，就需要做移动操作，可以复用以前的 DOM 节点。

REMOVE_NODE，老 component 类型，在新集合里也有，但对应的 element 不同则不能直接复用和更新，需要执行删除操作，

或者老component 不在新集合里的，也需要执行删除操作。

eg: 新老集合进行 diff 差异化对比，发现 B != A，则创建并插入 B 至新集合，删除老集合 A；以此类推，创建并插入 A、

D 和 C，删除B、C 和 D。

![elementold](/study/elementold.png)

带来的问题：都是相同的节点，但由于位置发生变化，导致需要进行繁杂低效的删除、创建操作，其实只要对这些节点进行位置

移动即可

react优化策略：允许开发者对同一层级的同组子节点，添加唯一 key 进行区分

![elementdiff](/study/elementdiff.png)

##### 优化后diff实现：

对新集合的节点进行循环遍历，通过唯一 key 可以判断新老集合中是否存在相同的节点

如果存在相同节点，则进行移动操作，但在移动前需要将当前节点在老集合中的位置child._mountIndex与lastIndex(访问过的

节点在老集合中最右的位置即最大的位置)进行比较，if (child._mountIndex < lastIndex)，则进行节点移动操作从新集合中

取得 B，判断老集合中存在相同节点 B

B 在老集合中的位置 B._mountIndex = 1

初始 lastIndex = 0

不满足 child._mountIndex < lastIndex 的条件，因此不对 B 进行移动操作

更新 lastIndex = Math.max(prevChild._mountIndex, lastIndex) lastIndex更新为1

将 B 的位置更新为新集合中的位置prevChild._mountIndex = nextIndex，此时新集合中 B._mountIndex = 0，nextIndex++

以上主要分析新老集合中存在相同节点但位置不同时，对节点进行位置移动的情况，如果新集合中有新加入的节点且老集合存在需要删

除的节点，

那么 React diff 又是如何对比运作的呢？

![step](/study/step.png)


新建：从新集合中取得 E，判断老集合中不存在相同节点 E，则创建新节点 E

lastIndex不做处理

E 的位置更新为新集合中的位置，nextIndex++

删除：当完成新集合中所有节点 diff 时，最后还需要对老集合进行循环遍历，判断是否存在新集合中没有但老集合中仍存在的节点，

发现存在这样的节点 D，因此删除节点 D

### 以上三个操作是diff算法的核心 如下图：

![answer](/study/answer.png)

### react diff的问题

理论上 diff 应该只需对 D 执行移动操作，然而由于 D 在老集合的位置是最大的，导致其他节点的 _mountIndex < lastIndex，

造成 D 没有执行移动操作，而是 A、B、C 全部移动到 D 节点后面的现象

![problem](/study/problem.png)

总结：

React 通过制定大胆的 diff 策略，将 O(n3) 复杂度的问题转换成 O(n) 复杂度的问题；

React 通过分层求异的策略，对 tree diff 进行算法优化；

React 通过相同类生成相似树形结构，不同类生成不同树形结构的策略，对 component diff 进行算法优化；

React 通过设置唯一 key的策略，对 element diff 进行算法优化；

建议，在开发组件时，保持稳定的 DOM 结构会有助于性能的提升；

建议，在开发过程中，尽量减少类似将最后一个节点移动到列表首部的操作，当节点数量过大或更新操作过于频繁时，在一定程度上

会影响React 的渲染性能。

以上就是React中虚拟dom与diff算法的讲解（附代码）的详细内容。

# 侵权必删！