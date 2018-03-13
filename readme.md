# promise学习

## promise的释义

## 状态


## promise的设计
  ```javascript
  new Promise(
    /* 执行器 */
    function(resolve, reject) {
      // 一段时间的异步
      resolve(); // 数据处理完成
      reject(); // 数据处理出错
    }
  )
    .then(function(res){
      // 成功
    },function(err){
      // 失败
    })
  ```

## promise例子
```javascript
// 简单的调用方法
console.log('run');
new Promise( resolve => {
  setTimeout( ()=>{
    resolve('hello');
  }, 1000);
}).then( value => {
  console.log(value + ' world');
})

// then值的传递
new Promise( resolve => {
  setTimeout( ()=>{
    resolve('hello');
  }, 1000);
}).then( value =>{
  console.log(value);
  return new Promise( resolve => {
    setTimeout( ()=>{
      resolve('world'); // 这里并没有进行值传递
    }, 1000);
  })
}).then( value => {
  console.log(value + ' world');
})

// promise是可以引用的,无论在哪里只要promise变量有then都会推送到队列
let promise = new Promise( resolve => {
  setTimeout(()=>{
      resolve('next1');
  }, 1000);
});

setTimeout(()=>{
  promise.then( value => {
    console.log(value);
  })
}, 3000);

```

## promise的小问题
+ 假设所有的doSth和doSthElse函数返回的都是promise实例
+ 第一个then的值会作为第二个then的调用对象
+ 可能是undefined，也可能是你第一次调用的promise对象，也可能是你在then里新构建的promise


+ 问题一：
```
doSth().then(function(){
  return doSthElse();
}).then(finalHandler);
执行流程：
  doSth函数
  |-----|
        doSthElse(undefined)
        |-----|
              finalHandler(resultOfDoSthElse)
              |-----|
```

+ 问题二：
```
doSth().then(function(){
  doSthElse(); // 没有return相当于返回空
}).then(finalHandler);
执行流程：
  doSth函数
  |-----|
        doSthElse(undefined)
        |-----|
        finalHandler(undefined)
        |-----|
```

+ 问题三：
```
doSth().then(doSthElse()).then(finalHandler); // then里的值如果不是一个函数哪呢就会忽略这个then,所以是同步执行的

执行流程：
  doSth函数
  |-----|
  doSthElse(undefined)
  |-----|
        finalHandler(resultOfDoSth)
        |-----|
```

+ 问题四：
```
doSth().then(doSthElse).then(finalHandler); // 如果then里面的值如果是函数且返回了新的promise对象，那么后续的then就会用新的promise对象
执行
doSth函数
|-----|
      doSthElse(undefined)
      |-----|
            finalHandler(resultOfDoSthElse)
            |-----|
```

## promise.all方法与map函数
``` javascript
// 所有的示例都要完成
var arr = [1,2,3,4,5,6];
Promise.all( arr.map( item => { // 将数组循环，并返回一个数组
  return new Promise( resolve => { // 返回promise对象
    setTimeout( () => {
      console.log(item);
      resolve(item);
    }, 1000);
  });
}));
```

## promise.race方法

``` javascript
// 只要有一个完成就完成
let p1 = new Promise( resolve => {
  setTimeout( () => {
    resolve('p1');
  }, 2000);
});
let p2 = new Promise( resolve => {
  setTimeout( () => {
    resolve('p2');
  }, 1000);
});
Promise.race([p1, p2]).then( value => {
  console.log(value);
});
```

##promise实现队列

方法一:each方式
 
```javascript
function queue(things) {
  let promise = Promise.resolve(); // 生成一个promise对象（http://t.cn/R4tBSME）
  things.each(thing => { // 循环需要操作的对象
    promise = promise.then( () => {
      return new Promise( resolve => { // 返回新的promise对象
        yourFunction(thing, () => { // 这里是你要执行的异步函数
          resolve(); // 这里改变异步的状态
        });
      });
    });
  });
  return promise; // 只保留要执行的上一个promise的状态
}

queue(['lots', 'of', 'things', ......]);
```

方法二：readuce方法
 
```javascript
function queue(things) {
  return things.reduce( (promise, thing) => {
    return promise.then( ()=>{
      return new Promise( resolve => {
        // yourFunction(thing, () => { // 这里是你要执行的异步函数
        //   resolve(); // 这里改变异步的状态
        // });
        setTimeout(() => { // 这里是你要执行的异步函数
          console.log(thing);
          resolve(thing); // 这里改变异步的状态
        }, 1000);
      });
    });
  }, Promise.resolve()); // 会返回一个promise的引用
}

queue(['lots', 'of', 'things']).then(function(res){ // 执行最后一次promise的结果
  console.log(res);
});
```

## 把回调函数包装成promise

```javascript
function getList() {
  return new Promise( resolve => {
    $.ajax(...
    success: (data) => {
      resolve(data);
    })
  });
};
```

## promise的兼容性

IE下支持不是很好，其他浏览器大都支持了。

在IE下是不能使用Promise对象的，如何破？
> 1. JQuery的defered对象       
> 2. Bluebird的Promise ployfill类库        


## Fetch API 是XMLHttpRequest的现代化替代方案。

> 更强大也更友好。
> 直接返回一个Promise实例

样例代码：
```javascript
fetch('xxx.json').then( res =>{
  return res.json();
}).then( json => {
  ...
}).catch( err => {
  console.log(err);
})
```