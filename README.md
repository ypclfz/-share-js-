# -share-js-
- https://imweb.io/topic/561f9352883ae3ed25e400f5
- https://github.com/lcxfs1991/blog/issues/9
## 使用配置
```
"devDependencies": {
    "@babel/cli": "^7.6.3",
    "@babel/core": "^7.6.3",
    "@babel/preset-env": "^7.6.3"
  },
  "babel": {
    "presets": [
      ["@babel/preset-env", {
        "targets": "> 0.25%, not dead"
      }]
    ]
  }
  ```
## babel编译结果解析
### 1. var,const,let
```
const a = 'aaa';
let b = 'bbb';
var c = 'cccc';
{
  const a = 'a';
  let b = 'b';
  var c = 'c';
}
```
通过改变变量名来实现const let 的局部作用域, const 不可变更通过编译检测实现
```
"use strict";

var a = 'aaa';
var b = 'bbb';
var c = 'cccc';
{
  var _a = 'a';
  var _b = 'b';
  var c = 'c';
}
```
### 2. 箭头函数
```
var obj = {
  prop: 1,
  func: function() {
    var _this = this;

    var innerFunc = () => {
      var _this_in = this;
      this.prop = 1;
    };

    var innerFunc1 = function() {
      this.prop = 1;
    };
  },
};
var a = () => { this.c = 'ccc' }
```
使用传统的为this命名的方法绑定this
```
"use strict";

var _this3 = void 0;

var obj = {
  prop: 1,
  func: function func() {
    var _this2 = this;

    var _this = this;

    var innerFunc = function innerFunc() {
      var _this_in = _this2;
      _this2.prop = 1;
    };

    var innerFunc1 = function innerFunc1() {
      this.prop = 1;
    };
  }
};

var a = function a() {
  _this3.c = 'ccc';
};
```
