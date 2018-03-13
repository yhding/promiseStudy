# promise学习

## promise的释义

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
  