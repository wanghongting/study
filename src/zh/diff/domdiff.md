### dom diff算法与虚拟dom的故事
1.【虚拟DOM】：render 虚拟DOM + diff算法+更新必要的DOM元素

![diff运行图](/study/diff.webp)
优点：最终表现在dom上的修改只是部分的变更，可以保证高效的渲染，提高网页性能
缺点：首次渲染大量dom时，由于多了一层Dom的计算，会比innerHtml慢一点
虚拟DOM的理解误区
对虚拟DOM的理解往往停留在：通过JavaScript对象模拟原生DOM，加上DOM Diff 极大提升了DOM操作的性能。然而，虚拟DOM最大的意义不在于性能的提升（JavaScript对象比DOM对象性能高），而在于抽象了DOM的具体实现（对DOM进行了一层抽象），这在浏览器中使用 React时不是特别明显，因为写的DOM标签跟原生的没有区别，并且最终都被渲染成了DOM，在React Native中将会有很好的说明。

详情可参考以下链接：
<a href='http://www.php.cn/js-tutorial-411868.html'>http://www.php.cn/js-tutorial-411868.html</a>