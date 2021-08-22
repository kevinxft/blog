---
title: 撸一个Promise
date: 2021-08-22 08:50:39
tags: javascript
---

## 手撸一个 Promise

- https://mp.weixin.qq.com/s/-2vIKWRPzdXCZlPupw4XcA

- 一个**Promise**包括私有状态，内部方法，静态方法等。

### 私有属性(外部无法访问)

- 状态: **pending** 初始化状态,**fulfilled** 兑现(完成),**rejected**拒绝
- 值 **PromiseState** **PromiseResult**
- 值由 **resolve**或**reject**决定

### 实例方法

- then
- catch
- finally

### 静态方法

- Promise.reject
- Promise.resolve
- Promise.race
- Promise.all
- Promise.allSettled
- Promise.any

## 实现

- 基础类

```ts
enum PROMISE_STATES ｛
  PENDING = 'pending',
  FULFILLED = 'fulfilled',
  REJECTED = 'rejected'
｝

type PromiseStates = PROMISE_STATES.PENDING | PROMISE_STATES.FULFILLED | PROMISE.REJECTED

export const isFunction = (fn: any): boolean => typeof fn === 'function';
export const isObject = (obj: any): boolean => typeof obj === 'object';

export interface ICallbackFn {
  (value?: any): any
}

type CallbackParams = ICallbackFn | null

export interface IExecutorFn {
  (resolve: ICallbackFn, reject: ICallbackFn): any
}


class PromiseLike {
  protected PromiseState: PromiseStates;
  protected PromiseResult: any;

  resolveCallbackQueues: Array<ICallbackFn>
  rejectCallbackQueues: Array<ICallbackFn>


  construtor(executor) {
    if (!isFunction(executor)) {
      throw new Error('Promise resolver undefined is not a function')
    }
    this.PromiseState = PROMISE_STATES.PENDING
    this.PromiseResult = undefined

    // 保存两个事件的数组
    this.resolveCallbackQueues = []
    this.rejectCallbackQueues = []

    executor(this._resolve, this._reject)
  }

  static all(promises: Array<ICallbackFn>) {
    return new PromiseLike((resolve, reject) => {
      const len = promises.length
      let resolvedPromisesCount = 0
      let resolvedPromisesResult = <any>[]
      for (let i = 0; i < len; i++) {
        const currentPromise = promises[i]
        PromiseLike.resolve(currentPromise)
        .then((res: any) => {
          resolvedPromisesCount++
          resolvedPromisesResult[i] = res
          if (resolvedPromisesCount ===  len) {
            resolve(resolvedPromisesResult)
          }
        })
        .catch((err: any) => {
          reject(err)
        })
      }
    })
  }

  static race(promises: Array<ICallbackFn>) {
    return new PromiseLike((resolve, reject) => {
      for (leet i = 0; i < promises.length; i++) {
        const currentPromise = promises[i]
        PromiseLik.resolve(currentPromise)
          .then((res: any) => {
            resolve(res)
          })
          .catch((err: any) => {
            reject(err)
          })
      }
    })
  }

  static is(promise: PromiseType) {
    return promise instanceof PromiseLike
  }

  static resolve(value?: any) {
    // 如果已经是promise，就不需要再封装了
    if (PromiseLike.is(value)) {
      return value
    }
    return new PromiseLike(resolve => resolve(value))
  }

  static reject(value?: any) {
    return new PromiseLike((resolve, reject) => reject(value))
  }

  _resolve = (value?: any) => {
    const resolveCb = () => {
      if (this.PromiseState !== PROMISE_STATES.PENDING) {
        return
      }
      while (this.resolveCallbackQueues.length > 0) {
        const fn = this.resolveCallbackQueues.shift()
        fn && fn(value)
      }
      this.PromiseState = PROMISE_STATES.FULFILLED
      this.PromiseResult = value
    }
    // 让任务变成异步
    setTimeout(resolveCb,0)
  }

  _reject = (value?: any) => {
    const rejectCb = () => {
      if (this.PromiseState !== PROMISE_STATES.PENDING) {
        return
      }
      while(this.rejectCallbackQueues.length > 0) {
        const fn = this.rejectCallbackQueues.shift()
        fn && fn(value)
      }
      this.PromiseState = PROMISE_STATES.REJECTED
      this.PromiseResult = value
    }
    // 让任务变成异步
    setTimeout(resolveCb, 0)
  }

  then = (onFulfilled: CallbackParams, onRejected: CallbackParams) => {
    // 默认处理
    onFulfilled = isFunction(onFulfilled) ? onFulfilled : value => value
    onRejected = isFunction(onRejected) ? onRejected : err => { throw err; }

    return new PromiseLike((resolve, reject) => {

      const handleFulfilled = (val: any) => {
        try {
          const res = onFulfilled(val)
          resolve(res)
        } catch (error) {
          reject(error)
        }
      }

      const handleRejected = (val: any) => {
        try {
          const res = onRejected(val)
          reject(res)
        } catch(error) {
          reject(error)
        }
      }

      switch (this.PromiseState) {
        case PENDING:
          isFunction(onFulfilled) && this.resolveCallbackQueues.push(onFulfilled)
          isFunction(onRejected) && this.rejectCallbackQueues.push(onRejected)
        case FULFILLED:
          handleFulfilled(this.PromiseResult)
          break
        case REJECTED:
          handleRejected(this.PromiseResult)
          break
      }
      return this
    }
  }

  catch = (rejectedCb: CallbackParams) => {
    return this.then(null, rejectedCb)
  }

  finally = (finallyCb: CallbackParams) => {
    return this.then(
      val => PromiseLike.resolve(finallyCb && finallyCb()).then(() => val),
      err => PromiseLike.reject(finallyCb && finallyCb()).then(() => { throw err })
    )
  }

}

```

### 要点

- 有自己的私有状态，也有改变私有状态的方法
- 接收一个执行函数作为参数，执行函数内部是预定义好的私有方法
- 私有状态一旦改变(兑现或拒绝)后不可逆
