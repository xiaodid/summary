# TRANSACTION

`Transaction`是React中大量使用的一种操作模式，在mount component，update component中都有使用。

这里的`Transaction`，和数据库操作的transaction（事务）不太一样，React的`Transaction`没有commit和rollback的操作。

## 解决的问题
* 原子操作？，一系列的操作做为一个整体，这个整体完成以后才能进行下一个操作，两个操作不能有交叉
* pattern of pre-condition -- operation -- post-condition。试想，现在要写一个计算复杂销售报表的程序，但如果计算出来的数据中，利润是负数的话，就恢复上一次的报表。
``` javascript
var salesReport = {
  // ...
  totalDollarsEarned: 782
};

function calculateStateAfterRiskyInvestments(salesReport) {
  var savedOldReport = JSON.load(JSON.stringify(salesReport)); /* Cloning! (`pre-condition) */

  /* Complicated manipulation of many fields of the sales report... */

  /* Postcondition */
  if(salesReport.totalDollarsEarned < 0) {
    Object.assign(salesReport, savedOldReport); /* Restoration of fields (post-condition). */
  }
}
```

另外一个例子是发送消息

```
single message
open connection  -->  msg            --> close connection

multiple messages
open connection  --> msg1/msg2/msg3  --> close connection
```

## 原子操作
在Transaction中，定义了isInTransaction

``` javascript
isInTransaction: function(): boolean {
  return !!this._isInTransaction;
}
```

在`Transaction`开始的时候，会检查是否`isInTransaction`为真，如果是，则抛出异常
``` javascript
perform (
  method: T, scope: any,
  a: A, b: B, c: C, d: D, e: E, f: F,
) {
  // 判断是否已经在transaction中，如果是，则抛出异常
  invariant(
    !this.isInTransaction(),
    'Transaction.perform(...): Cannot initialize a transaction when there ' +
    'is already an outstanding transaction.'
  );

  // other codes
}
```

## initialize-perform-close
`Transaction`提供了一套初始化－执行－收尾的操作模式，并保证被该执行过程在执行过程中只有一个在执行。

先看Transaction的注释，src/renderers/shared/utils/Transaction.js
```
 * `Transaction` creates a black box that is able to wrap any method such that
 * certain invariants are maintained before and after the method is invoked
 * (Even if an exception is thrown while invoking the wrapped method). Whoever
 * instantiates a transaction can provide enforcers of the invariants at
 * creation time. The `Transaction` class itself will supply one additional
 * automatic invariant for you - the invariant that any transaction instance
 * should not be run while it is already being run. You would typically create a
 * single instance of a `Transaction` for reuse multiple times, that potentially
 * is used to wrap several different methods. Wrappers are extremely simple -
 * they only require implementing two methods.
 *
 *                       wrappers (injected at creation time)
 *                                      +        +
 *                                      |        |
 *                    +-----------------|--------|--------------+
 *                    |                 v        |              |
 *                    |      +---------------+   |              |
 *                    |   +--|    wrapper1   |---|----+         |
 *                    |   |  +---------------+   v    |         |
 *                    |   |          +-------------+  |         |
 *                    |   |     +----|   wrapper2  |--------+   |
 *                    |   |     |    +-------------+  |     |   |
 *                    |   |     |                     |     |   |
 *                    |   v     v                     v     v   | wrapper
 *                    | +---+ +---+   +---------+   +---+ +---+ | invariants
 * perform(anyMethod) | |   | |   |   |         |   |   | |   | | maintained
 * +----------------->|-|---|-|---|-->|anyMethod|---|---|-|---|-|-------->
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | +---+ +---+   +---------+   +---+ +---+ |
 *                    |  initialize                    close    |
 *                    +-----------------------------------------+
```

`Transaction`由initialize，perform，close三部分组成。
* 如果某一个initialize抛出异常，接下来的initialize会被继续执行。但perform不会被执行。没有出错的initialize对应的close会被执行
* 如果perform抛出异常，则所有close都会被执行。
* 如果某一个close抛出异常，剩下的close会继续执行

创建一个Transaction
``` javascript
// wrapper 1
var RESET_BATCHED_UPDATES = {
  initialize: emptyFunction,
  close: function() {
    ReactDefaultBatchingStrategy.isBatchingUpdates = false;
  },
};

// wrapper 2
var FLUSH_BATCHED_UPDATES = {
  initialize: emptyFunction,
  close: ReactUpdates.flushBatchedUpdates.bind(ReactUpdates),
};

var TRANSACTION_WRAPPERS = [FLUSH_BATCHED_UPDATES, RESET_BATCHED_UPDATES];

function ReactDefaultBatchingStrategyTransaction() {
  this.reinitializeTransaction();
}

Object.assign(
  ReactDefaultBatchingStrategyTransaction.prototype,
  Transaction,
  {
    // 实现抽象方法
    getTransactionWrappers: function() {
      return TRANSACTION_WRAPPERS;
    },
  }
);

var transaction = new ReactDefaultBatchingStrategyTransaction();
```


perform:
``` javascript
/**
 * Executes the function within a safety window. Use this for the top level
 * methods that result in large amounts of computation/mutations that would
 * need to be safety checked. The optional arguments helps prevent the need
 * to bind in many cases.
 *
 * @param {function} method Member of scope to call.
 * @param {Object} scope Scope to invoke from.
 * @param {Object?=} a Argument to pass to the method.
 * @param {Object?=} b Argument to pass to the method.
 * @param {Object?=} c Argument to pass to the method.
 * @param {Object?=} d Argument to pass to the method.
 * @param {Object?=} e Argument to pass to the method.
 * @param {Object?=} f Argument to pass to the method.
 *
 * @return {*} Return value from `method`.
 */
perform (
  method: T, scope: any,
  a: A, b: B, c: C, d: D, e: E, f: F,
) {
  // 判断是否已经在transaction中，如果是，则抛出异常
  invariant(
    !this.isInTransaction(),
    'Transaction.perform(...): Cannot initialize a transaction when there ' +
    'is already an outstanding transaction.'
  );
  var errorThrown;
  var ret;
  try {
    this._isInTransaction = true;
    // Catching errors makes debugging more difficult, so we start with
    // errorThrown set to true before setting it to false after calling
    // close -- if it's still set to true in the finally block, it means
    // one of these calls threw.
    errorThrown = true;
    // 初始化
    this.initializeAll(0);

    // 执行wrapped method
    ret = method.call(scope, a, b, c, d, e, f);
    errorThrown = false;
  } finally {
    try {
      if (errorThrown) {
        // If `method` throws, prefer to show that stack trace over any thrown
        // by invoking `closeAll`.
        try {
          this.closeAll(0);
        } catch (err) {
        }
      } else {
        // Since `method` didn't throw, we don't want to silence the exception
        // here.
        this.closeAll(0);
      }
    } finally {
      this._isInTransaction = false;
    }
  }
  return ret;
}
```
