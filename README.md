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
var a = () => { console.log(this) }
var b = function () {
  console.log(this)
}
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
```
使用传统的为this命名的方法绑定this
```
"use strict";

var _this2 = void 0;

var a = function a() {
  console.log(_this2);
};

var b = function b() {
  console.log(this);
};

var obj = {
  prop: 1,
  func: function func() {
    var _this3 = this;

    var _this = this;

    var innerFunc = function innerFunc() {
      var _this_in = _this3;
      _this3.prop = 1;
    };

    var innerFunc1 = function innerFunc1() {
      this.prop = 1;
    };
  }
};

```
### 3. 对象中括号属性
```
const prop2 = "PROP2";
var obj = {
    ['prop']: 1,
    ['func']: function() {
        console.log('func');
    },
    [prop2]: 3
};
```
新增了一个_defineProperty函数,使用了Object.defineProperty方法
```
"use strict";

var _obj;

function _defineProperty(obj, key, value) {
    if (key in obj) {
        Object.defineProperty(obj, key, {
            value: value,
            enumerable: true,
            configurable: true,
            writable: true
        });
    } else {
        obj[key] = value;
    }
    return obj;
}

var prop2 = "PROP2";
var obj = (_obj = {}, _defineProperty(_obj, 'prop', 1), _defineProperty(_obj, 'func', function func() {
    console.log('func');
}), _defineProperty(_obj, prop2, 3), _obj);
```
### 4. Object.assign
```
const obj = {a: 'a'};
const obj_new = Object.assign({}, obj, {b: 'b'});
```
在只使用基础包的情况下，对Object.assign并不进行转译
```
"use strict";

var obj = {
  a: 'a'
};
var obj_new = Object.assign({}, obj, {
  b: 'b'
});
```
### 5. Map, Set
```
const map = new Map([['a', 1], ['b', 2]]);
const set = new Set([1, 2, 3, 1]);
```
同样不转译
```
"use strict";

var map = new Map([['a', 1], ['b', 2]]);
var set = new Set([1, 2, 3, 1]);

```
### 6. 数组遍历方法
```
const arr = [1, 2, 3];
arr.forEach(function(a) {
  console.log(a);
})
const arr_new = arr.map(function(a) {
 return a - 1
});
```
同样不转译
```
"use strict";

var arr = [1, 2, 3];
arr.forEach(function (a) {
  console.log(a);
});
var arr_new = arr.map(function (a) {
  return a - 1;
});
```
### 7. 类
```
class Animal {
  constructor(name, type) {
    this.name = name;
    this.type = type;
  }

  walk() {
    console.log('walk');
  }

  run() {
    console.log('run')
  }

  static getType() {
    return this.type;
  }

  get getName() {
    return this.name;
  }

  set setName(name) {
    this.name = name;
  }
}

class Dog extends Animal {
  constructor(name, type) {
    super(name, type);
  }

  get getName() {
    return super.getName();
  }
}
```
定义了大量的辅助方法来实现类。
转换后（由于代码太长，先省略辅助的方法）：

```
/**
......一堆辅助方法，后文详述
**/
var Animal = (function () {
    function Animal(name, type) {
                // 此处是constructor的实现，用_classCallCheck来判定constructor正确与否
        _classCallCheck(this, Animal);

        this.name = name;
        this.type = type;
    }
        // _creatClass用于创建类及其对应的方法
    _createClass(Animal, [{
        key: 'walk',
        value: function walk() {
            console.log('walk');
        }
    }, {
        key: 'run',
        value: function run() {
            console.log('run');
        }
    }, {
        key: 'getName',
        get: function get() {
            return this.name;
        }
    }, {
        key: 'setName',
        set: function set(name) {
            this.name = name;
        }
    }], [{
        key: 'getType',
        value: function getType() {
            return this.type;
        }
    }]);

    return Animal;
})();

var Dog = (function (_Animal) {
        // 子类继承父类
    _inherits(Dog, _Animal);

    function Dog(name, type) {
        _classCallCheck(this, Dog);
                // 子类实现constructor
                // babel会强制子类在constructor中使用super，否则编译会报错
        return _possibleConstructorReturn(this, Object.getPrototypeOf(Dog).call(this, name, type));
    }

    _createClass(Dog, [{
        key: 'getName',
        get: function get() {
                       // 跟上文使用super调用原型链的super编译解析的方法一致，
                       // 也是自己写了一个回溯prototype原型链
            return _get(Object.getPrototypeOf(Dog.prototype), 'getName', this).call(this);
        }
    }]);

    return Dog;
})(Animal);
```

```
// 检测constructor正确与否
function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}
```

```
// 创建类
var _createClass = (function() {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            // es6规范要求类方法为non-enumerable
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            // 对于setter和getter方法，writable为false
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }
    return function(Constructor, protoProps, staticProps) {
        // 非静态方法定义在原型链上
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        // 静态方法直接定义在constructor函数上
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
})();
```

```
// 继承类
function _inherits(subClass, superClass) {
   // 父类一定要是function类型
    if (typeof superClass !== "function" && superClass !== null) {
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
    }
    // 使原型链subClass.prototype.__proto__指向父类superClass，同时保证constructor是subClass自己
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    // 保证subClass.__proto__指向父类superClass
    if (superClass) 
        Object.setPrototypeOf ? 
        Object.setPrototypeOf(subClass, superClass) :    subClass.__proto__ = superClass;
}
```

```
// 子类实现constructor
function _possibleConstructorReturn(self, call) {
    if (!self) {
        throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
    }
    // 若call是函数/对象则返回
    return call && (typeof call === "object" || typeof call === "function") ? call : self;
}
```
### 8.解构
```
const obj = {a: 'a', b: 'b', c: 'c'};
const arr = [1, 2, 3];
const { a, b, c } = obj;
const [ d, e, f ] = arr;
```
没有使用特别方法，逐步分解
```
"use strict";

var obj = {
  a: 'a',
  b: 'b',
  c: 'c'
};
var arr = [1, 2, 3];
var a = obj.a,
    b = obj.b,
    c = obj.c;
var d = arr[0],
    e = arr[1],
    f = arr[2];
```
### 9.函数默认值
```
var a = function ({a='a', b}) {
  console.log(a, b);
}
a({});
```
```
"use strict";

var a = function a(_ref) {
  var _ref$a = _ref.a,
      a = _ref$a === void 0 ? 'a' : _ref$a,
      b = _ref.b;
  console.log(a, b);
};

a({});
```
### 总结
babel只是转译新标准引入的语法，比如ES6的箭头函数转译成ES5的函数；而新标准引入的新的原生对象，部分原生对象新增的原型方法，新增的API等（如Proxy、Set等），这些babel是不会转译的。需要用户自行引入polyfill来解决
 

## babel兼容
ES6特性|兼容性
---|:--
箭头函数|支持
类的声明和继承|部分支持，IE8不支持
增强的对象字面量|支持
字符串模板|支持
解构|支持，但注意使用方式
参数默认值，不定参数，拓展参数|支持
let与const|支持
for of|IE不支持
iterator, generator|不支持
模块 module、Proxies、Symbol|不支持
Map，Set 和 WeakMap，WeakSet|不支持
Promises、Math，Number，String，Object 的新API|不支持
export & import|支持
数组拷贝|支持
### 1. 常用方法的兼容性
- Object.assign

