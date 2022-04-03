#### 2022/3/31

1. ##### `.gitignore`更新生效

     ```bash
    git rm -r --cached .
    ```

1. ##### 浅克隆与深克隆

2. `浅克隆`：只进行一级克隆，基本类型为值传递，对象类型引用传递，和原来的对象公用相同的堆地址

   方法有：`展开运算符...`，`slice`，`Object.assign(target, source)`

   函数克隆直接通过浅克隆就能实现，因为函数克隆的时候会在内存中单独开辟一片空间，互不影响（这里不太明白）

   `深克隆`：对象所有的属性和值都会一层一层完全复制，新对象的修改不会影响原来的对象，如果克隆对象的某个值还是对象的话

* 方法一：`js.parse(js.stringify())`
  
    原理：先把原对象转为字符串，然后再parse成一个全新的对象，就实现了内容的深度克隆
  
    问题：可以处理`Number`，`String`，`Array`这种能够被 `js`正确表示的数据结构，但是函数处理后会变为`null`，正则对象处理后会变为空对象`{}`
- 方法二：自己写

  > - 递归调用函数
  > - 函数开头判断类型
  > - 对于数组或者对象则通过`for in`写法递归复制所有的键值

  例子
  
  ```js
  let obj1 = [
    10,
    20,
    { name: "小宝贝", parents: ["大宝贝1", "大宝贝2"] },
    /^\d+$/,
    function () {},
  ];
  ```
  
  
  
  ```js
  function deepClone(obj) {
    if (!obj || obj instanceof Function)
      return obj; /* 如果为falsy或者为函数，不需要克隆 */
  
    if (typeof obj != "object") return obj; /* 如果不是对象，不需要克隆 */
  
    /* 如果为正则，新建正则对象 */
    if (obj.constructor === RegExp) return new RegExp(obj);
    /* 日期同理 */
    if (obj.constructor === Date) return new Date(obj);
  
    /* 否则就是对象或者数组 */
    const clone = obj.constructor();
    for (const key in obj) {
      /* 遍历键，保证是obj自己的 */
      if (obj.hasOwnProperty(key)) {
        clone[key] = deepClone(obj[key]);
      }
    }
  
    return clone;
  }
  
  let clone = deepClone(obj1);
  clone[3].parents = ["大宝贝0", "大宝贝1"];
  
  console.log(`原对象：${clone}`);
  console.log(`克隆对象：${obj1}`);
  ```

![image-20220331162324004](README_imgs/image-20220331162324004.png)

- 另一个对象的例子：

  ```js
  let obj2 = {
  			name: '小明',
  			ke: ['语文', '数学', '英语'],
  			color: {
  				n: 'red',
  				m: 'blue'
  			}
  		};
  
  let clone = deepClone(obj2);
  clone.color.n = "lightblue";
  console.log(clone.color.n === obj2.color.n) // false
  
  ```

- 同样的可以改变一下顺序

  > 1. 先`for in`循环键
  > 2. 然后再对这个值的各种情况进行处理
  >    - 基本类型或者Function，直接复制
  >    - RegEx或者Date类型，构造函数复制一下
  >    - 数组或者对象，递归调用函数

  ```js
  function deepClone(obj) {
    const clone = new obj.constructor(); /* 新建对象，Array或者Object */
    for (const key in obj) {
      /* 循环键 */
      if (obj.hasOwnProperty(key)) {
        if (obj[key] == null || typeof obj[key] != "object") {
          clone[key] = obj[key]; /* 排除null和其他基本类型，以及function */
        } else if (/RegEx|Date/.test(_type(obj[key]))) {
          /* 构造函数创建一个新的正则或者日期对象 */
          clone[key] = new obj[key].constructor(obj[key]);
        } else {
          /* 循环调用了 */
          clone[key] = deepClone(obj[key]);
        }
      }
    }
    return clone;
  }
  
  /* 打印类型，如正则对象输出‘Object RegEx’ */
  function _type(obj) {
    return Object.prototype.toString.call(obj);
  }
  ```


1. ##### JavaScript数据类型

   > 各种数据类型主要的特点，各自的区别以及判断的方法

* 首先JavaScript是一种弱类型/动态语言，不用提前声明变量的类型，一个变量可以保存不同类型的数据

  ```js
  let foo = 42; // 现在是Number
  foo = "I'm a String"; // 现在是String
  foo = false; // 现在是Boolean
  ```

* 基本数据类型有七种，`Boolean`, `Null`, `Undefined`, `Number`, `String`, `Bigint`, `Symbol`;

  基本数据类型处在语言底层，值不可变，比如说JavaScript的String是不可变的；所以也称为原始值，Primitive Values

* 对象类型，Object，是一组属性的集合

* `Boolean`布尔类型：表示一个逻辑实体，两种取值，true或者false；判断语句的条件本质上会转换成布尔值；PS：布尔对象和布尔值是不同的

* `Null`类型：只有一种取值：`null`；`null`表示缺少，指代变量未指向任何对象；

  PS：`null`会相等（`==`）于undefined，但不会强等于（`===`）；因为前者会执行类型转换

* `Undefined`类型：一个没有赋值的变量就会默认为`undefined`；它是全局对象的一个属性，即全局作用域的一个变量，其初始值就是基本类型`undefined`；这个属性是不能配置或重写的；

* `Number`数字类型：基于IEEE754标准的双精度64位二进制浮点数，能表示`(2^53-1) ~ (2^53-1)`的数字，除了表示浮点数，还有三个带符号的值，`+Infinity`、`-Infinity`和`NaN`（非数值，Not-a-Number），判断数值是否在JS表达的范围之内，可以使用`Number.MAX_VALUE`和`Number.Min_VALUE`；

* `Bigint`：可以表示任意精度的整数，使用ta甚至可以超过Number类型的安全整数范围。通过在整数的末尾附加字母`n`或者调用构造函数来创建的；通过引用Bigint，可以在数字递增时，操作超过`Number.MAX_SAGE_INTEGER`也能返回预期的结果；不能与Number相互运算，否则，将抛出`TypeError`

* `String`类型：用于表示文本数据。字符串一旦创建，不可更改；但是，可以通过对源字符串的操作来创建新的字符串。例如

  * 获取字串可通过直接按照索引选择某些字母，或者使用<u>`substr()`</u>;
  * 字符串拼接使用连接运算符`+`或者<u>`concat`</u>；

* `Symbol`类型，它是一种唯一且不可修改的基本数据类型；

  * 通过调用`Symbol`函数创建数据实例；
  * 可以用作对象的键，用于创建匿名的对象属性；它唯一的作用就是创建私用成员；
  * 但是这个键是不能通过`for in `来打印的，也不会出现在`Object.getOwnPropertyNames()`的返回数组里；
  * 要访问这个属性只能通过创建时的原始symbol值来访问，或者遍历`Object.getOwnPropertySymbols()`返回的数组来访问；
  * 访问全局的symbol注册表的方法，只能通过反射方法，`Symbol.for(tokenString)`返回一个symbol值，反之，`Symbol.keyFor(symbolValue)`从注册表返回一个token字符串

4. ##### 基本数据类型和对象数据类型的区别

- 存储方式的区别
  - 基本数据类型变量的存储只涉及到栈区，在内存栈区存着变量标识符和对应的值，也就是基本数据类型
  - 而对象数据类型变量的存储涉及到内存栈区和的堆区（这里堆区是指内存里里的堆内存）；栈内存里保存变量标识符和存储在堆内存里的对象的地址，所以本质上所有对象类型变量存储的都是指针，一个对象类型变量赋值给另一个变量，就是把指针赋值了一份过去，指向的还是同一个对象；而为什么对象类型要放在堆内存中，一方面是对象的大小并不固定，可以改变；另一方面是引用变量操作查找更加方便；
- 两个变量之间的比较
  - 基本数据类型变量比较的时候比较的是值，`==`在比较之前会进行数据类型转换，比如`Boolean`类型和`Number`类型比较的时候会先把`true`变成`1`，或者`false`变成`0`；
  - 而对象类型的变量比较的时候比的是存在栈内存里的指针，所有哪怕两个对象的内容完全一样，这两个对象的地址不同，比较引用变量的时候也是不同的（我好啰嗦）
- 能否增删改
  - 基本数据类型处在语言底层，值不可变，不能添加属性；以`String`类型为例
  - 对象类型的值可以改变同时能添加属性

5. ##### 对象类型

* `对象`：在计算机科学中，对象是指内存中可以被`标识符`引用的的一块区域；

* `标识符`：代码中用来标识变量、函数或者属性的字符序列；

* `键/属性`：在JavaScript中，对象可以看作一组属性的集合；对象的键可以是`字符串`或者`Symbol`类型；每个属性(`property`)都有对应的`attributes`（翻译成特性）来控制；

  对象的属性包括两种`数据属性`和`访问器属性`

* `数据属性`以键值对的形式存在，而其描述符称为数据描述符，描述符是对象，包括一下属性
  * `value`：这个属性的数据值，可以是任何Javascript类
  * `writable`：`Boolean`值，决定属性值（`vlaue`）能不能修改
  * `Enumerable`：`Boolean`值，决定这个属性能不能被`for...in`循环来枚举
  * `Configurable`：`Boolean`值，决定除了`vlaue`和`Writable`以外的特性能不能修改
* `访问器属性`：有一个或者两个访问器函数（`get`和`set`来存取数值）
  * `Get`：为函数对象或者`undefined`
  * `Set`：同上
  * `Enumberable`：同上
  * `Configurable`：`Boolean`值，如果是`false`，表示属性不能删除，且不能转换成数据属性
* "标准的"对象和函数：
  - 一个JavaScript对象就是键和值之间的映射，键和值的要求见上，对象非常符合`哈希表`的要求
  - 函数则是一个附带了可被调用功能的常规对象，一样有键值
* `日期对象`（`Date`）：显示日期，使用JavaScript内置的`Date`对象
* 有序集，包括数组`Array`和类数组`Typed Arrays`
  * `Array`：是一种整数作为键，并且键与长度`length`相关的对象
  * 数组对象继承了`Array.prototype`的一些操作数组的方法，如`indexOf`或`push`，是列表或集合的最优选择
  * 类数组（`Typed Arrays`）是ES6新定义的JavaScript对象

* 带键的集合：`Map`，`Set`，`WeakMap`，`WeakSet`

  这些数据结构将对对象的引用，即引用对象的标识符作为键；`Map`和`WeakMap`将值和对象关联起来，`Set`和`WeakSet`表示一组对象。`WeakMaps`和`WeakSet`的键是不可枚举的，对象是弱引用的；且对于`WeakMap`来说键只能是对象，对于WeakSet来说值也只能是对象，而不能是原始类型；弱引用的特点使得他们有更好的垃圾回收机制优化。

  `与对象的对比`

  查询时间上：其实在ES5也能实现Maps和Sets，但对象不能进行比较，所以查询时线性的时线性的，而ES6的原生实现（包括WeakMap）的查询时间是相对恒定的，对数增长

  数据绑定方面：通常，原生对象上也能设置属性或者使用`data-*`属性，这样也能将数据绑定到DOM元素上，然后有缺陷，这样在任何脚本内，数据都运行在同样的上下文。而相较之下，`Map`和`WeakMap`能够方便的将数据*`私密`*地绑定到一个对象上

* 结构化数据：`js`（`JavaScript Object Notion`）

  是一种轻量级地数据交换格式，来源于JavaScript，同时也被多种语言所使用，其被用来构建通用地数据结构

* `typeof`操作符用于判断对象类型，具体的示例和注意事项参：[`MDN typeof`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof)


#### 2022/04/01

1. ##### 二叉树的层序遍历

> 输入为二叉树的根节点`root`，返回节点值的层序遍历数组，逐层从左到右访问所有节点。

* 遍历每一层，首先结果集假如当前层的的节点值数组，同时得到非空的下一层节点数组

  ```js
  /**
   * @param {TreeNode} root
   * @return {number[][]}
   */
  var levelOrder = function (root) {
      if (!root) return [];/* 保证root非空 */
      let NodesByLevel = [root];/* 存储每一层的节点，保证非空 */
      let temp = [];
      const resArr = [];
  
      while (NodesByLevel.length) {
          resArr[resArr.length] = [];/* 新加一层 */
          for (const node of NodesByLevel) {
              resArr[resArr.length - 1].push(node.val);/* push value */
              if (node.left) temp.push(node.left);/* 下一层节点 */
              if (node.right) temp.push(node.right);
          }
          /* 更新节点列表 */
          NodesByLevel = temp;
          temp = [];
      }
      return resArr;
  };
  ```

* 优化后，可以只要一个二维数组，原地更新就行

  ```js
  /**
   * @param {TreeNode} root
   * @return {number[][]}
   */
  var levelOrder = function (root) {
      if (!root) return [];/* 保证root非空 */
      let resArr = [[root]];/* 初始化二维数组 */
      let level = 0;/* 当前层 */
      let cur;/* 当前节点 */
  
      /* 遍历最后一层 */
      while (resArr[level].length) {
          /* 加一层 */
          resArr[level + 1] = [];
          for (const index in resArr[level]) {
              cur = resArr[level][index];
              /* 先加入下一层 */
              if (cur.left) resArr[level + 1].push(cur.left);
              if (cur.right) resArr[level + 1].push(cur.right);
              resArr[level][index] = cur.val;/* 更新为节点值 */
          }
          level++;/* 更新当前层 */
      }
  
      return resArr.slice(0, resArr.length - 1);/* 去掉最后一层空的 */
  };
  ```

* 也可以采用队列先进先出的特性进行层序遍历，就是JavaScript的数组shitft操作可能比较耗时间，但也挺快的，是时间复杂度O(n)的写法

  ```js
  /**
   * @param {TreeNode} root
   * @return {number[][]}
   */
  var levelOrder = function (root) {
      if (!root) return [];/* 保证root非空 */
      const nodesQueue = [root];/* 记录当前层和下一层节点，作为队列使用 */
      const resArr = [];
      let cur;/* 当前节点 */
      let size;
  
      /* 直到队列空了 */
      while (nodesQueue.length) {
          /* 得到当前层的节点数 */
          size = nodesQueue.length;
          /* 结果加一层 */
          resArr[resArr.length] = [];
          /* 层序遍历，先进先出 */
          while (size--) {
              cur = nodesQueue.shift();/* 拿出第一个 */
              /* 下一层入队列 */
              if (cur.left) nodesQueue.push(cur.left);
              if (cur.right) nodesQueue.push(cur.right);
              /* 记录节点值 */
              resArr[resArr.length - 1].push(cur.val);
          }
      }
  
      return resArr;
  };
  ```

* 递归法

  同样的，层序遍历也能递归，不过需要传入结果数组指针，以及当前结点所在层级

  ```js
  /**
   * @param {TreeNode} root
   * @return {number[][]}
   */
  var levelOrder = function (root) {
      if (!root) return [];
      const resArr = [[]];/* 结果数组 */
      depthFirst(resArr, root, 0);
      return resArr;
  
      /* 递归方法，传入结点以及所在层级，从0开始 */
      function depthFirst(arr, node, level) {
          /* 如果arr层数不够，加一层 */
          if (arr.length - 1 < level) {
              arr[level] = [];
          }
  
          /* 加入当前值 */
          arr[level].push(node.val);
  
          /* 递归调用，从左到右 */
          if (node.left) depthFirst(arr, node.left, level + 1);
          if (node.right) depthFirst(arr, node.right, level + 1);
      }
  };
  ```

2. ##### 二叉树的层序遍历Ⅱ

   > 输入同样是根节点root，返回的是自底向上的层序遍历。逐层从左向右遍历

* 仍然是原地遍历得到结果数组，然后将数组翻转一下得到结果

  ```js
  /**
   * @param {TreeNode} root
   * @return {number[][]}
   */
  var levelOrderBottom = function (root) {
      /* 首先进行从上至下的层序遍历 */
      if (!root) return [];
      /* 初始化二维数组 */
      const resArr = [[root]];
      let level = 0;/* 当前层 */
  
      /* 遍历当前层 */
      while (true) {
          let nextLevel = [];/* 下一层结点 */
  
          for (const index in resArr[level]) {
              let cur = resArr[level][index];/* 当前结点 */
              /* 先加入下一层，然后更新这一层为值 */
              if (cur.left) nextLevel.push(cur.left);
              if (cur.right) nextLevel.push(cur.right);
              resArr[level][index] = cur.val;
          }
  
          if (!nextLevel.length) break;
          resArr.push(nextLevel);
          level++;
      }
  
      /* 第二步，将数组逆转 */
      for (let left = 0, right = resArr.length - 1; left < right; left++, right--)
          [resArr[left], resArr[right]] = [resArr[right], resArr[left]]
  
      return resArr;
  };
  ```


3. ##### 二叉树的右视图

   输入是根节点root，返回从右侧看从上到下每一层的最后一个节点；

   `注意`：由于用于不知道下一层的最后一个节点是这一层那个节点的孩子，故而需要从左到右得到所有的孩子

* 方法一，原地递归

  ```js
  /**
   * @param {TreeNode} root
   * @return {number[]}
   */
  var rightSideView = function (root) {
      if (!root) return [];
      /* 层序遍历，只保留每一层的最后一个 */
      const resArr = [[root]];
      let level = 0;
      /* 循环，直到下一层为空 */
      while (true) {
          let nextLevel = [];/* 初始化下一层 */
          /* 遍历当前层 */
          for (let index = 0; index < resArr[level].length; index++) {
              let cur = resArr[level][index];
              /* 从左至右得到下一层所有非空节点 */
              if(cur.left) nextLevel.push(cur.left);
              if(cur.right) nextLevel.push(cur.right);
          }
          /* 得到当前层的最后一个的值 */
          resArr[level] = resArr[level].pop().val;
  
          /* 判断下一层还有没有 */
          if (!nextLevel.length) return resArr;
          /* 更新level，遍历下一层 */
          resArr[level + 1] = nextLevel;
          level++;
      }
  };
  ```

* 使用队列，先进先出层序遍历

  ```js
  /**
   * @param {TreeNode} root
   * @return {number[]}
   */
  var rightSideView = function (root) {
      if (!root) return [];
      /* 使用队列进行层序遍历 */
      const nodesQueue = [root];
      const resArr = [];
      let cur;
  
      /* 遍历每一层 */
      while (nodesQueue.length) {
          /* 当前层节点数 */
          let size = nodesQueue.length;
          while (size--) {
              cur = nodesQueue.shift();
  
              /* 下一层入队 */
              if (cur.left) nodesQueue.push(cur.left);
              if (cur.right) nodesQueue.push(cur.right);
          }
          resArr.push(cur.val);/* 最右侧的值进结果 */
      }
  
      return resArr;
  };
  ```

* 递归法，必然深度优先，从右往左，传一个深度level和数组，节点进去，只有当当前层的最右还没有确定的时候才会push数值进结果数组

  ```js
  /**
   * @param {TreeNode} root
   * @return {number[]}
   */
  var rightSideView = function (root) {
      if (!root) return [];
      const resArr = [];
      depthFirst(resArr, root, 0);
      return resArr;
  
      /* 递归方法，从右往左调用递归方法 */
      function depthFirst(arr, node, level) {
          /* 如果当前层的最右节点还没确定 */
          if (arr.length - 1 < level)
              arr[level] = node.val;
  
          /* 从右往左递归 */
          if (node.right) depthFirst(arr, node.right, level + 1);
          if (node.left) depthFirst(arr, node.left, level + 1);
      }
  };
  ```

##### 4. 判断一个对象是数组

* 使用`instanceof`进行判断

  `instanceof`可以用于判断指定构造函数的`prototype`属性是否出现在指定实例对象的原型链的任何位置，返回一个`Boolean`值

  ```js
  a = [];
  console.log(a instanceof Array); //true
  ```

  然而也有例外，例如在多个全局环境（`multi-globals`）的情况下，为了保证不出现混乱，不同的全局环境（如`iframe`）中的数组原型就不是一样了，如下: `iframe`中的`Array`构造函数和全局`window`的的`Array`构造函数原型就不会全等

  ```js
  const iframe1 = document.createElement('iframe'); // 创建一个iframe对象
  document.body.append(iframe1);
  var xArray = windows.frames[0].Array; // 得到iframe1环境中的数组构造函数
  
  let arr = new xArray();
  /* 因为iframe会产生新的全局环境 */
  console.log(arr instanceof Array); //false
  ```

* 使用`constructor`进行判断

  `constructor`会指向对象的构造函数，同样的一个对象的构造函数也是可以手动更改的：

  ```js
  let arr = [];
  arr.constructor = Object
  ```

  而且同样会出现多个全局变量的情况

* 使用`Object.prototype.toString.call`进行判断

  `Object`原型中默认的`toString`方法会输出“`object`”+具体类型；数组之类的重写了这个方法，故而需要使用原型方法来`call`，它不仅能判断`Data`，`Array`之类的，还能判断`Number`，`String`这种原始值类型

  ```js
  console.log(Object.prototype.toString.call(/my-type/g); 
  // [object RegEx]
  let arr = [];
  console.log(Object.prototype.toString.call(arr);
  // [object Array]
  console.log(Object.prototype.toString.call(1);
  // [object Number]
  ```

  同样在多全局环境中也能输出理想结果

- 使用`Array.isArray`

  `Array.isArray`是ES5提出来的，简单好用，同样能解决多环境问题

  ```
  const arr = [];
  console.log(Array.isArray(arr)); // true
  ```

  而且如果ES5之前不支持的话，可以自己封装一下

  ```js
  if(!Array.isArray){
  	Array.isArray = function(arg){
          return Object.prototype.toString.call(arg) === "[object Array]";
      }
  }
  ```


#### 2022-04-22

1. ##### 二叉树的层平均值

   给定一个**`非空`**二叉树的根节点root，以数组的形式返回每一层节点的平均值。保证精度在10^-5以内

   仍旧采用队列先进先出，注意节点出去是shift

   ```js
   /**
    * @param {TreeNode} root
    * @return {number[]}
    */
   var averageOfLevels = function (root) {
       const nodeQueue = [root];/* 队列辅助层序遍历 */
       const resArr = [];
       while (nodeQueue.length) {
           /* 当前层的节点数 */
           let size = nodeQueue.length;
           let sum = 0;/* 初始化每一层的和 */
           for (let i = 0; i < size; i++) {
               let cur = nodeQueue.shift();
               sum += cur.val;
               if (cur.left) nodeQueue.push(cur.left);
               if (cur.right) nodeQueue.push(cur.right);
           }
           resArr.push(sum / size);/* 当前层节点的平均值 */
       }
   
       return resArr;
   };
   ```


2. ##### N叉树的层序遍历

   给定一个N叉树的`root`节点，返回层序遍历，其构造函数如下：

   ```js
   function Node(val,children) {
       this.val = val;
       this.children = children;
    };
   ```

   采用递归，在递归函数中去循环递归调用所有子节点

   ```js
   /**
    * @param {Node|null} root
    * @return {number[][]}
    */
   var levelOrder = function (root) {
       /* 采用递归的写法 */
       if (!root) return [];
       const resArr = [];
       levelDown(resArr, root, 0);
       return resArr;
   
       /* 递归，传入数组，当前节点，当前层级（从0开始） */
       function levelDown(arr, node, level) {
           /* 如果结果中还没有这一层，加一层 */
           if (arr.length - 1 < level) {
               arr[level] = [];
           }
           arr[level].push(node.val);
   
           /* 从左往右递归children，传入level+1 */
           for (const child of node.children) {
               if (child) levelDown(arr, child, level + 1);
           }
       }
   };
   ```

3. ##### 在每个树行中找最大值

   > 给定root，层序遍历，找到每一层的最大值

   采用队列方法如下：

   ```js
   /**
    * @param {TreeNode} root
    * @return {number[]}
    */
   var largestValues = function (root) {
       if (!root) return [];
       /* 使用队列，先进先出 */
       const resArr = [];
       const nodesQueue = [root];
   
       while (nodesQueue.length) {
           let size = nodesQueue.length;/* 得到当前层节点数 */
           let max = nodesQueue[0].val;/* 初始化最大值 */
   
           while (size--) {
               let cur = nodesQueue.shift();
               /* 得到更大的 */
               max = Math.max(max, cur.val);
               if (cur.left) nodesQueue.push(cur.left);
               if (cur.right) nodesQueue.push(cur.right);
           }
           resArr.push(max);
       }
   
       return resArr;
   };
   ```


4. ##### 反转二叉树

   > 输入二叉树的根节点，然后左右翻转这颗二叉树，然后返回根节点

* 递归，前序遍历，将每个节点的左右孩子都交换一下，不能中序遍历，因为这样原左边节点会翻转两次，原右边节点翻不到

  ```js
  /**
   * @param {TreeNode} root
   * @return {TreeNode}
   */
  var invertTree = function (root) {
      if (root) invertNode(root);
      return root;
  
      function invertNode(node) {
          /* 当前左右子节点交换一下 */
          [node.left, node.right] = [node.right, node.left];
  
          /* 递归调用 */
          if (node.left) invertNode(node.left);
          if (node.right) invertNode(node.right);
      }
  };
  ```

* 直接用一个堆栈来存还没有翻转的节点

  ```js
  /**
   * @param {TreeNode} root
   * @return {TreeNode}
   */
  var invertTree = function (root) {
      if (!root) return root;
      unInvertedNodes = [root];/* 一个堆栈存储还没翻转的节点 */
      let len = 1;
      while (len--) {
          /* 得到当前要翻的节点 */
          let cur = unInvertedNodes.pop();
  
          /* 翻转 */
          [cur.left, cur.right] = [cur.right, cur.left];
  
          /* 存入 */
          if (cur.left) unInvertedNodes[len++] = cur.left;
          if (cur.right) unInvertedNodes[len++] = cur.right;
      }
  
      return root;
  };
  ```

* 也可以层序遍历，用一个队列来翻转

  ```js
  /**
   * @param {TreeNode} root
   * @return {TreeNode}
   */
  var invertTree = function (root) {
      if (!root) return root;
      let unInvertedNodes = [root];/* 队列，层序遍历 */
  
      while (unInvertedNodes.length) {
          let cur = unInvertedNodes.shift();
  
          /* 翻转 */
          [cur.left, cur.right] = [cur.right, cur.left];
  
          if (cur.left) unInvertedNodes.push(cur.left);
          if (cur.right) unInvertedNodes.push(cur.right);
      }
  
      return root;
  };
  ```


5. ##### HTTP缓存机制

   > 借鉴资源：
   >
   > - `MDN`： [`HTTP缓存`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching#varying%20responses) 		[`Cache-Control`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)
   > - 掘金：[`轻松理解HTTP缓存策略-蒋鹏飞`](https://segmentfault.com/a/1190000038562294)

* 缓存是一种保存资源副本并且再下次请求时直接使用该副本的技术

* HTTP缓存减少了等待时间和网络流量，进而减少了显示资源表示形式所需的时间。这使得网站更加具有响应性

* 缓存种类很多，大致分为两类：`私有缓存`和`共享缓存`。共享缓存存储的响应能被多个用户使用，私有缓存则只针对特定用户。以下的HTTP缓存策略主要针对浏览器或者代理缓存；部署在服务器上的缓存还包括：`网管缓存`、`CDN`、`反向代理缓存`、`和负载均衡器`等。为站点和web应用提供更好的稳定性、性能和扩展性

* `HTTP缓存策略`的存在是为了解决客户端和服务端的信息不对称问题。

* `浏览器缓存`/`私有缓存`：针对单独用户，浏览器缓存拥有用户通过HTTP下载的所有文档；这些缓存一来为浏览过的网页提供前后向导航、保存网页、查看源码等功能。同样能够在某种程度上保证离线浏览。

* `代理缓存`/共享缓存：热门资源通过架设一个web代理来作为本地网络基础的一部分提供给用户。减少网络拥堵和延迟。

* 常见的需要使用到缓存的案例（一般是针对`GET`请求的响应）：

  * 检索成功的响应，`GET`请求返回`200`，表示`ok`，包含html文档，图片或者文档的响应
  * 永久重定向：`301`
  * 错误响应（客户端方）：`404`页面
  * 不完全响应：`206`，只返回部分信息
  * 除了`GET`请求外，如果在响应中匹配到了被定义的cache键名（疑惑）
  * 也可以通过一些关键字来有区分地存储和使用不同响应来组成缓存，如`Vary`键则决定了不同设备对应不同地缓存

* `缓存控制技术`：

* 主要是HTTP/1.1定义的`Cache-Control`头来决定应该采用哪种缓存策略，请求头和响应头都会带有这个属性，通过它提供的不同的值来决定采用哪种缓存策略。

  常用的头如下：

  | 属性              | 应用                                                                                                                              |
  | ----------------- | :-------------------------------------------------------------------------------------------------------------------------------- |
  | `no-store`        | 强制不缓存，不存储任何关于客户端请求和服务器响应的内容，每次都得重新请求内容                                                      |
  | `no-cache`        | 可以缓存，但每次发出的请求都会发到服务器重新验证缓存是否可用                                                                      |
  | `private`         | 响应头可带，表示只能被单个用户缓存，不能作为共享缓存                                                                              |
  | `public`          | 响应头可带，表示可以被任何中间人缓存，如代理服务器，`CDN`等，通常是不能被中间人缓存的页面，如带验证信息的页面或者请求方法为`POST` |
  | `max-age`         | 表示缓存的过期时间，单位为秒；与`Expires`相反，它是相对于请求的时间，`Expires`是一个固定的时间                                    |
  | `s-maxage`        | 覆盖`max-age`或者`Expires`头，仅适用于共享缓存                                                                                    |
  | `must-revalidate` | 表示资源过期后，必须在成功向服务器验证之后才能使用缓存资源，即`协商缓存`                                                          |
  
  `举例说明：`
  
  禁止缓存可以用以下的头：
  
  ```js
  Cache-Control: no-strore
  ```
  
  缓存静态资源时可以让其同时存在公共缓存中
  
  ```js
  Cache-Control:public, max-age=31536000
  ```
  
  若要设置成客户端可以缓存，但必须强制每次验证缓存
  
  ```js
  Cache-Control: no-cache
  ```
  
  ```js
  Cache-Control: max-age=0, must-revalidate
  ```

* `Pragma`头

  `Pragma`是`HTTP/1.0`标准中定义的一个`header`属性，其作用跟`Cache-Control：no-cache`相同，可以用于向后兼容`HTTP/1.0`的客户端

* `Expires`头

  包含过期的日期/时间，优先级低于`Cache-Control`响应头设置的`max-age`或者`s-maxage`

* 故而在客户端发出请求是检索到有缓存时，会首先计算其新鲜度，如果有`Cache-control`的`max-age`或者`s-maxage`，则以它作为参考，否则就去看`Expires`头是否存在

* 倘若已经缓存已经过期了，或者缓存的响应头中设置了使用缓存之前必须先进行缓存验证，就会在每次需要重新发出请求使用缓存时进行缓存验证。而缓存验证时往往会带上之前服务器发过来的响应头中的校验器。校验器其实就是一种同步的事件标志。分为两种，强校验器和弱校验器。

  * `Etag`是一种缓存的强校验器，优先级更高。`Etag`其实就是从资源本身算出来的一个`hash`值或者版本号，客户端请求验证的时候会在请求头带上`Etag`的值，然后对应的键是`If-None-Match`。意思就是问服务器，你那边的Etag和我这个是不是不`Match`，不`Match`就把新的资源传过来
  * `Last-Modified`响应头则是一种若校验器，表示每一次资源修改的时间，说它弱是因为精度只到一秒，而`Etag`的时间颗粒度更高，而且有的时候文件更新的而内容并不会边，这个时候`Etag`也不会更新，更加的准确，而对于有`Last-Modified`的响应头信息的缓存，验证的时候会带上`If-Modified-Since`，也很直观，意思就是问服务器自从上次修改时间是不是又改了

* 当服务器处理缓存验证的请求的时候，会返回`200`ok，表示会返回更新之后的正常的结果。或者`304` `Not Modified`，而不返回内容，表示客户端可以使用缓存。`304`和`200`的响应头都可以更新缓存文件的过期时间，如果设置了`max-age`之类的响应头的话。

* 总而言之，如果设置了过期时间相关的头的话，就会先计算缓存还新不新鲜，如果过期了就在进行缓存验证，如果又缓存验证相关的头的话就把这些头带上。当然，服务器可以进行的设置还是很多的。

* `Vary`响应头

  `Vary`响应头决定了对于后续的请求，是否要请求一个新的资源还是使用缓存的内容。比如`Vary`的键设置成了

  `Content-Encoding`这回将缓存了的资源的`encoding`和请求的`encoding`进行比较看有没有，使用`Vary`头明显增强了内容服务的动态多样性。

  再比如说，如果需要区分移动端和客户端的展示内容，避免再不同的终端展示错误的布局和内容呢等，就可以使用如下响应头：

  ```js
  Vary：User-Agent
  ```

  同时这可以帮助`Google`或者其他的搜索引擎更好地发现页面的移动版本。



#### 2022-04-03

1. ##### 填充每个节点的下一个右侧节点指针

   给了一个完美二叉树，填充所有的节点的next指针为下一个右侧节点，没有就默认是null；输入根节点，输出根节点。二叉树的定义如下：

   ```js
   function Node(val,left,right,next){
       this.val = val === undefined ? 0 : val;
       this.left = left === undefined ? null : left;
       this.right = right === undefined ? null : right;
       this.next = next === undefeined ? null : next;
   }
   ```

* 法一、采用队列进行层序遍历

  ```js
  /**
   * @param {Node} root
   * @return {Node}
   */
  var connect = function (root) {
      /* 队列，先进先出进行层序遍历 */
      if (!root) return root;
      const nodesQueue = [root];
  
      while (nodesQueue.length) {
          let size = nodesQueue.length;/* 当前层的节点数 */
          while (size--) {
              let cur = nodesQueue.shift();/* 获得队首 */
              /* 填充下一个，判断是否当前层最右节点 */
              if (size != 0) cur.next = nodesQueue[0];
              /* 左右子节点入队 */
              if (cur.left) nodesQueue.push(cur.left);
              if (cur.right) nodesQueue.push(cur.right);
          }
      }
  
      return root;
  };
  ```

* 法二，两个数组，一个保存当前层，临时数组保存下一层

  ```js
  /**
   * @param {Node} root
   * @return {Node}
   */
  var connect = function (root) {
      if (!root) return root;
      let curLevel = [root];/* 当前层 */
  
      /* 遍历当前层 */
      while (true) {
          let nextLevel = [];/* 下一层 */
          let size = curLevel.length;
          for (let i = 0; i < size; i++) {
              /* 填充下一个 */
              if (i < size - 1) curLevel[i].next = curLevel[i + 1];
              if (curLevel[i].left) nextLevel.push(curLevel[i].left);
              if (curLevel[i].right) nextLevel.push(curLevel[i].right);
          }
  
          /* 还有没有下一层 */
          if (!nextLevel.length) return root;
          curLevel = nextLevel;/* 更新当前层 */
      }
  };
  ```

* 递归，深度优先，不过会带一个数组和当前节点所在深度

  ```js
  /**
   * @param {Node} root
   * @return {Node}
   */
  var connect = function (root) {
      if (!root) return root;
      const arr = [];/* 数组，记录每一层的一个节点，从左往右更新 */
      depthFirst(arr, root, 0);/* 调用递归 */
      return root;
  
      /* 接受一个数组，节点以及其所在深度 */
      function depthFirst(arr, node, level) {
          /* 判断是不是还没有遍历过这一层 */
          if (arr.length - 1 < level) {
              arr[level] = node;
          } else {
              /* 填充next指针并且更新数组 */
              arr[level].next = node;
              arr[level] = node;
          }
  
          /* 递归调用，从左往右填 */
          if (node.left) depthFirst(arr, node.left, level + 1);
          if (node.right) depthFirst(arr, node.right, level + 1);
      }
  };
  ```

2. 对称二叉树

​	输入一个根节点root，判断它的值是否轴对称

* 迭代法，层序遍历，一层一层地查

  ```js
  /**
   * @param {TreeNode} root
   * @return {boolean}
   */
  var isSymmetric = function (root) {
      if (!root) return true;
      /* 层序遍历 */
      let curLevel = [root];
  
      while (curLevel.length) {
          let nextLevel = [];
          /* 首先遍历，将当前层转为值，得到下一层 */
          for (const index in curLevel) {
              let cur = curLevel[index];
              if (!cur) continue;/* 保证cur不是null */
              nextLevel.push(cur.left);
              nextLevel.push(cur.right);
              curLevel[index] = cur.val;
          }
  
          /* 判断当前层是否对称 */
          for (let left = 0, right = curLevel.length - 1; left < right; left++, right--) {
              if (curLevel[left] != curLevel[right]) return false;
          }
  
          curLevel = nextLevel;/* 更新 */
      }
  
      return true;
  };
  ```

* 递归，先得到层序遍历的二维数组，然后判断是否对称

  ```js
  /**
   * @param {TreeNode} root
   * @return {boolean}
   */
  var isSymmetric = function (root) {
      /* 递归层序遍历，得到二维数组 */
      const resArr = [];
      depthFirst(resArr, root, 0);
  
      /* 遍历二维数组的每一层 */
      for (let index = 0; index < resArr.length - 1; index++) {
          let curLevel = resArr[index];
          for (let left = 0, right = curLevel.length - 1; left < right; left++, right--) {
              if (curLevel[left] != curLevel[right]) return false;
          }
      }
      return true;
  
      function depthFirst(arr, node, level) {
          if (resArr.length - 1 < level) {
              /* 如果还没有遍历到这一层，则在结果加一层 */
              resArr[level] = [];
          }
          if (!node) {
              /* 如果为null */
              resArr[level].push(null);
              return;
          }
          resArr[level].push(node.val);
  
          /* 递归调用 */
          depthFirst(arr, node.left, level + 1);
          depthFirst(arr, node.right, level + 1);
      }
  };
  ```

* 递归法，前序遍历，左树以中左右的顺序，右树以中右左的顺序，得到两个数组，然后再比较是不是一样的

  ```js
  /**
   * @param {TreeNode} root
   * @return {boolean}
   */
  /* 递归左树 */
  function leftTreeTraversal(arr, node) {
      if (!node) {
          arr.push(null);
          return;
      }
  
      arr.push(node.val);
  
      /* 不遍历叶子节点的孩子节点 */
      if (!node.left && !node.right) {
          return;
      }
  
      /* 中左右的顺序遍历左树 */
      leftTreeTraversal(arr, node.left);
      leftTreeTraversal(arr, node.right);
  }
  
  /* 递归右树 */
  function rightTreeTraversal(arr, node) {
      if (!node) {
          arr.push(null);
          return;
      }
  
      arr.push(node.val);
  
      /* 不遍历叶子节点的孩子节点 */
      if (!node.left && !node.right) {
          return;
      }
      /* 中右左的顺序遍历右数 */
      rightTreeTraversal(arr, node.right);
      rightTreeTraversal(arr, node.left);
  }
  var isSymmetric = function (root) {
      // 前序遍历对比两个子树，递归方法
      if (!root) return true;
      const leftTreeArray = [];
      const rightTreeArray = [];
  
      leftTreeTraversal(leftTreeArray, root.left);
      rightTreeTraversal(rightTreeArray, root.right);
  
      if (leftTreeArray.length != rightTreeArray.length) return false;
  
      /* 遍历两个数组 */
      for (const index in leftTreeArray) {
          if (leftTreeArray[index] != rightTreeArray[index]) return false;
      }
      return true;
  };
  ```

* 递归方法，递归分别比较两边的内测和外侧

  ```js
  /**
   * @param {TreeNode} root
   * @return {boolean}
   */
  var isSymmetric = function (root) {
      if (!root) return true;
      /* 递归比较左树和右树 */
      return compareTrees(root.left, root.right);
  };
  
  /* 递归方法，返回Boolean值 */
  function compareTrees(left, right) {
      if (!left && right) return false;   /* 如果有一方是空节点 */
      else if (left && !right) return false;
      else if (!left && !right) return true;  /* 如果都为空则是相同 */
      else if (left.val != right.val) return false;/* 接下来才能比较当前节点的值 */
  
      /* 分别比较下一层的外侧的里侧 */
      return compareTrees(left.left, right.right) && compareTrees(left.right, right.left);
  }
  ```

* 使用堆栈来模拟递归，不过堆栈里的元素是要比对的二元数组

  ```js
  /**
   * @param {TreeNode} root
   * @return {boolean}
   */
  var isSymmetric = function (root) {
      if (!root) return true;
      const nodesStack = [[root.left, root.right]];
      /* 使用栈来存储未递归的部分，先外侧再内侧 */
      while (nodesStack.length) {
          /* 出栈 */
          let len = nodesStack.length;
          let left = nodesStack[len - 1][0];
          let right = nodesStack[len - 1][1];
          nodesStack.pop();
  
          /* 先比较值 */
          if (!right && left) return false;
          else if (right && !left) return false;
          else if (!right && !left) continue;
          else if (right.val != left.val) return false;
  
          /* 然后分别将内侧和外侧放入堆栈 */
          nodesStack.push([left.right, right.left]);
          nodesStack.push([left.left, right.right]);
      }
      return true;
  };
  ```


3. ##### 前端页面性能优化

   > 借鉴：
   >
   > - `MDN`: [`Populating the pages How browsers work`](https://developer.mozilla.org/zh-CN/docs/Web/Performance/How_browsers_work)
   > - `掘金`: [`聊一聊前端性能优化--俊劫`](https://juejin.cn/post/6911472693405548557)
   > - `SegmentFault：`：[`前端性能优化 24 条建议---谭光志`](https://segmentfault.com/a/1190000022205291)

* 首先要善用工具，浏览器提供的各种调试工具

  * `NetWork`面板：

    可以看到所有资源的加载情况，评估影响页面性能的因素，面板底部还有各种信息：包括`requests`的数量，`DOMContentLoaded`即`DOM`渲染完成的时间，`Load`即当前页面所有资源加载完成的时间

    同时可以判断那些资源对当前页面加载无用做出相应优化

  * 瀑布流`Waterfall`

    点进每个资源可以看到每个HTTP请求过程的每个步骤的具体用时，包括:

    * `Queueing`：资源入队时间
    * `Stalled`： 再队列中停止时间
    * `DNS lookup`：DNS解析时间
    * `Initial connection`：建立HTTP连接时间
    * `SSL`：建立安全性连接的时间
    * `TTFB`：等待服务器返回数据的时间
    * `Content Download`：资源下载时间

    很显然，每次`HTTP`请求的耗时都不仅仅是资源下载的时间，故而建议将多个小文件合并为一个大文件来减少`HTTP`请求次数，进而减少过多的时间耗损

  * Lighthouse

    根据chrome的一些策略自动对网站做一个质量评估，并且会给出一些优化的建议；

    `Frist Contentful Paint` 首屏渲染时间，1s内为绿色

    `Speed Index` 速度指数，4s内为绿色

    `Time to Interactive` 到页面可交互的时间

    `...`

  - Performance

    录制网页，给出相当多的数据以及分析

* 减少cookie的传输

  cookie的传输会造成带宽浪费，

  - 故而可以一定程度上减少cookie中存储的东西

  - 同时静态资源就不使用cookie了，使用其他域名时就不会自动带上cookie

* 减少重排重绘

  首先要了解浏览器渲染一个页面的整个过程

  对于一个用户而言，良好的冲浪体验包括两个部分：`页面内容的快速加载`和`流畅的交互`；而要实验流畅的交互，开发者要确保网站从流畅的网页坤东到点击响应的交互体验。而要实现这个目标渲染时间是重中之重。

  首先浏览器加载并且渲染一个页面包括以下步骤。

  * 首先是`导航`，用户再地址栏输入目标`url`提交表单等

  * 然后是`DNS查找`，要访问一个网站，首先要得到其服务器的IP地址，如果以前没有访问过，就需要进行`DNS`查找。再第一次初始化请求后，这个`IP地址`会被缓存一段时间，当然可能加载一个页面会需要多次`DNS查找`，因为可能`fonts`，`images`，`scripts`，`ads`都有着不同的主机名

  * `TCP Handshake`，然后浏览器会通过`TCP“三次握手”`来与服务器建立连接，三次握手技术经常称为“`SYN`-`SYN`-`ACK`”，其实更准确的说是`SYN`，`SYN-ACK`，`ACK`；因为客户端和服务器需要发送三个消息才能协商完毕。而此时真正的请求还没有发出

  * `TLS`协商，要再`HTTPS`上建立安全的连接，必须要进行令一番三次握手。更准确地说是TLS协商。这个过程验证了服务器，同时决定了用什么密钥来进行加密通信。保证再进行真实的数据传输之前建立安全的连接。

    ![The DNS lookup, the TCP handshake, and 5 steps of the TLS handshake including clienthello, serverhello and certificate, clientkey and finished for both server and client.](README_imgs/ssl.jpg)

    如上图所示，浏览器在发送真正的请求之前，一共和服务器进行了八次往返。如`MDN`上说，以下整个过程才能称之为`导航`：

    `The DNS lookup, the TCP handshake, and 5 steps of the TLS handshake including clienthello, serverhello and certificate, clientkey and finished for both server and client.`

  - 下一步就是响应了

    建立了`web`服务器的连接后，浏览器会代表用户发出一个初始的`HTTP GET`请求，得到一个网页。

  - `TCP慢开始/14kb规则`

    为了避免拥塞，TCP会初始`14Kb`的响应然后逐步增加发送数据的数量，直到达到网络的最大带宽。这是为了逐渐建立一个适合网络能力的传输速度。

  * `拥塞控制`

    服务器通过`TCP`包发送数据时，客户端会返回`ACK`确认帧，而如果发的太快了，可能会出现丢包现象，然后就不会有确认帧。拥塞控制算法正是通过这个过程来决定合适的发送速率。



