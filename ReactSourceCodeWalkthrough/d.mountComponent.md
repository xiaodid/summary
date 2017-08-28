# mountComponent


这里分析ReactCompositeComponent的mountComponent方法。ReactDomComponent的mountComponent，大家可以自己分析。

ReactCompositeComponent.mountComponent
``` javascript
mountComponent: function(
    transaction,
    hostParent,
    hostContainerInfo,
    context
  ) {
    this._context = context;
    this._mountOrder = nextMountID++;
    this._hostParent = hostParent;
    this._hostContainerInfo = hostContainerInfo;

    var publicProps = this._currentElement.props;
    var publicContext = this._processContext(context);

    var Component = this._currentElement.type;

    var updateQueue = transaction.getUpdateQueue();

    // Initialize the public class
    var doConstruct = shouldConstruct(Component);
    var inst = this._constructComponent(
      doConstruct,
      publicProps,
      publicContext,
      updateQueue
    );
    var renderedElement;

    // Support functional components
    if (!doConstruct && (inst == null || inst.render == null)) {
      renderedElement = inst;
      warnIfInvalidElement(Component, renderedElement);
      inst = new StatelessComponent(Component);
      this._compositeType = CompositeTypes.StatelessFunctional;
    } else {
      if (isPureComponent(Component)) {
        this._compositeType = CompositeTypes.PureClass;
      } else {
        this._compositeType = CompositeTypes.ImpureClass;
      }
    }

    // These should be set up in the constructor, but as a convenience for
    // simpler class abstractions, we set them up after the fact.
    inst.props = publicProps;
    inst.context = publicContext;
    inst.refs = emptyObject;
    inst.updater = updateQueue;

    this._instance = inst;

    // Store a reference from the instance back to the internal representation
    ReactInstanceMap.set(inst, this);

    var initialState = inst.state;
    if (initialState === undefined) {
      inst.state = initialState = null;
    }

    this._pendingStateQueue = null;
    this._pendingReplaceState = false;
    this._pendingForceUpdate = false;

    var markup;
    if (inst.unstable_handleError) {
      markup = this.performInitialMountWithErrorHandling(
        renderedElement,
        hostParent,
        hostContainerInfo,
        transaction,
        context
      );
    } else {
      markup = this.performInitialMount(renderedElement, hostParent, hostContainerInfo, transaction, context);
    }

    if (inst.componentDidMount) {
      transaction.getReactMountReady().enqueue(inst.componentDidMount, inst);
    }

    return markup;
  }
```

ReactCompositeComponent.mountComponent主要完成以下工作：
* 该component的type是一个function，通过new component.prototype.type()来创建一个type的实例，并构建component的props和state
* 调用```this.performInitialMount```，这个方法会调用组件的生命周期函数：```componentWillMount```。
* 在```this.performInitialMount```中，会递归处理子组件，调用子组件的```mountComponent```，直到整个组件树被构件完成。并返回该组件的DOM Element。
* 在挂载组件的DOM Element后，会调用另一个生命周期方法：```componentDidMount```。但componentDidMount方法不是立即被调用，而是放到```updateQueue```中，在mount过程结束后，```transaction```会在close中触发noticeAll事件。这个会触发```componentDidMount```调用。

``` javascript
performInitialMount: function(renderedElement, hostParent, hostContainerInfo, transaction, context) {
  var inst = this._instance;

  var debugID = 0;

  if (inst.componentWillMount) {
    inst.componentWillMount();
    // When mounting, calls to `setState` by `componentWillMount` will set
    // `this._pendingStateQueue` without triggering a re-render.
    // 在componentWillMount中调用setState，不会触发重新渲染，但如果是异步操作还是会触发重新渲染，比如调接口
    // 在ReactUpdates.batchedUpdates中，通过ReactDefaultBatchingStrategyTransaction来启动挂载。在close的时候，会调用ReactUpdates.flushBatchedUpdates。这里会把所有dirty的Component再update一次。
    // 在componentWillMount中调用setState，会生成一个dirtyComponent，在batchedUpdate的transaction结束的时候，会调用ReactUpdates.flushBatchedUpdates。这里面会对每一个dirtyComponent再重新render。但这里会判断_pendingStateQueue是不是为空，如果不为空，则不update，因为已经update过了。
    if (this._pendingStateQueue) {
      inst.state = this._processPendingState(inst.props, inst.context);
    }
  }

  // If not a stateless component, we now render
  if (renderedElement === undefined) {
    // 调用当前Component的render方法，返回当前component对应ReactElement
    renderedElement = this._renderValidatedComponent();
  }

  var nodeType = ReactNodeTypes.getType(renderedElement);
  this._renderedNodeType = nodeType;
  // 创建子组件对应的ReactComponent
  var child = this._instantiateReactComponent(
    renderedElement,
    nodeType !== ReactNodeTypes.EMPTY /* shouldHaveDebugID */
  );
  this._renderedComponent = child;

  // 挂载子组件。子组件又挂载孙组件，一直递归下去
  // 先创建对应的ReactComponent，在调用对应component的mountComponent方法，触发组件的生命周期方法
  var markup = ReactReconciler.mountComponent(
    child,
    transaction,
    hostParent,
    hostContainerInfo,
    this._processChildContext(context),
    debugID
  );

  return markup;
},
```
