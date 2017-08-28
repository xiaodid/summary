# TIPS

1. 在componentWillMount中调用setState，不会触发re-render。但componentDidMount会。
2. 在遍历数组生成组件时，要给一个key，而且这个key不能是数组的index。一定要是一个唯一的值。
3. shouldComponentUpdate方法，是React性能提升的精髓。应尽量多的使用该方法。
4. setState是异步操作，合成新的state要在下一次update的时候才会执行，所以，在setState后马上读取state里的新内容是错误的。如果要在state被set后触发什么操作，可以给setState传一个callback。
5.

# REFERNCES

1. http://reactkungfu.com/2015/12/dive-into-react-codebase-transactions/
1. https://facebook.github.io/react/
1. http://blog.csdn.net/u013510838/article/details/55669742
1. https://bogdan-lyashenko.github.io/Under-the-hood-ReactJS/
