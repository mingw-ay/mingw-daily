#### 2022/3/31

1. ##### `.gitignore`更新生效

  ```bash
   git rm -r --cached .
  ```

2. ##### 浅克隆与深克隆

   `浅克隆`：只进行一级克隆，基本类型为值传递，对象类型引用传递，和原来的对象公用相同的堆地址

   方法有：`展开运算符...`，`slice`，`Object.assign(target, source)`

   函数克隆直接通过浅克隆就能实现，因为函数克隆的时候会在内存中单独开辟一片空间，互不影响（这里不太明白）

   `深克隆`：对象所有的属性和值都会一层一层完全复制，新对象的修改不会影响原来的对象，如果克隆对象的某个值还是对象的话

* 方法一：`JSON.parse(JSON.stringify())`

​		原理：先把原对象转为字符串，然后再parse成一个全新的对象，就实现了内容的深度克隆

​		问题：可以处理`Number`，`String`，`Array`这种能够被 `json`正确表示的数据结构，但是函数处理后会变为`null`，正则对象处理后会变为空对象`{}`

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


3. ##### JavaScript数据类型

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
* `键/属性`：在JavaScript中，对象可以看作一组属性的集合；对象的键可以是`字符串`或者`Symbol`类型；每个属性(`property`)都有对应的`attributes`来控制；


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


