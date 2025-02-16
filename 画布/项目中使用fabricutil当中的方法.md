好的，我将详细介绍你提到的这些 Fabric.js 工具函数和方法的用法，并提供示例代码：

**1. `fabric.util.transformPoint(point, transform, invert)`**

   - **用途:** 对一个点 (point) 应用一个 2x3 的变换矩阵 (transform)。
   - **参数:**
      - `point`: 一个包含 `x` 和 `y` 属性的对象，表示要变换的点。例如：`{ x: 10, y: 20 }`
      - `transform`: 一个包含 6 个元素的数组，表示 2x3 的变换矩阵。 矩阵形式如下：
         ```
         [ a, b, c, d, e, f ]
         ```
         这对应于以下变换：
         ```
         x' = ax + cy + e
         y' = bx + dy + f
         ```
         其中 `(x, y)` 是原始坐标，`(x', y')` 是变换后的坐标。
         *   `a`, `d`: 缩放 (scale)
         *   `b`, `c`: 倾斜 (skew)
         *   `e`, `f`: 平移 (translate)
      - `invert` (可选): 一个布尔值，如果为 `true`，则应用变换矩阵的逆变换。

   - **返回值:** 一个新的点对象，包含变换后的 `x` 和 `y` 坐标。

   - **示例:**

     ```javascript
     // 假设有一个点
     const point = { x: 10, y: 20 };

     // 定义一个变换矩阵：放大 2 倍，并向右平移 5 个单位，向下平移 10 个单位
     const transform = [2, 0, 0, 2, 5, 10];

     // 应用变换
     const transformedPoint = fabric.util.transformPoint(point, transform);
     console.log(transformedPoint); // 输出: { x: 25, y: 50 }

     // 应用逆变换
     const originalPoint = fabric.util.transformPoint(transformedPoint, transform, true);
     console.log(originalPoint); // 输出: { x: 10, y: 20 } (回到原始点)
      // 使用invertTransform
     const invertTransform_ = fabric.util.invertTransform(transform);
     const originalPoint_ = fabric.util.transformPoint(transformedPoint, invertTransform_, false);

     ```
    **`fabric.util.invertTransform(transform)`**:计算转换矩阵的逆矩阵

**2. `fabric.util.groupSVGElements(elements, options, path)`**

   - **用途:** 将一组 SVG 元素 (通常是解析 SVG 字符串得到的) 组合成一个 Fabric.js 的 `fabric.Group` 对象。
   - **参数:**
      - `elements`: 一个数组，包含 Fabric.js 对象（通常是 `fabric.Path`、`fabric.Rect`、`fabric.Circle` 等）。这些对象通常是通过 `fabric.loadSVGFromString` 或 `fabric.loadSVGFromURL` 解析 SVG 得到的。
      - `options`: 一个对象，包含用于创建 `fabric.Group` 的选项。这些选项与 `fabric.Group` 构造函数的选项相同。
      - `path` (可选): SVG 文件的路径（如果 SVG 是从文件加载的）。

   - **返回值:** 一个 `fabric.Group` 对象，包含所有提供的 SVG 元素。

   - **示例:**

     ```javascript
     // 假设有一个 SVG 字符串
     const svgString = `
       <svg width="200" height="100">
         <rect x="10" y="10" width="50" height="30" fill="red" />
         <circle cx="80" cy="50" r="20" fill="blue" />
       </svg>
     `;

     // 解析 SVG 字符串
     fabric.loadSVGFromString(svgString, (objects, options) => {
       // 将解析得到的 SVG 元素组合成一个 Group
       const group = fabric.util.groupSVGElements(objects, options);

       // 将 Group 添加到画布
       canvas.add(group);
       canvas.renderAll();
     });
     ```

**3. `fabric.util.object.clone(object, deep)`**

   - **用途:** 克隆一个对象。
   - **参数:**
      - `object`: 要克隆的对象。
      - `deep` (可选): 一个布尔值。如果为 `true`，则执行深拷贝（递归克隆所有嵌套的对象和数组）；如果为 `false` 或未提供，则执行浅拷贝（只克隆对象本身，嵌套的对象和数组仍然是引用）。

   - **返回值:** 克隆后的对象。

   - **示例:**
     ```javascript
     // 浅拷贝
     const obj1 = { a: 1, b: { c: 2 } };
     const obj2 = fabric.util.object.clone(obj1);
     obj2.b.c = 3;
     console.log(obj1.b.c); // 输出: 3 (obj1.b 和 obj2.b 指向同一个对象)

     // 深拷贝
     const obj3 = { a: 1, b: { c: 2 } };
     const obj4 = fabric.util.object.clone(obj3, true);
     obj4.b.c = 3;
     console.log(obj3.b.c); // 输出: 2 (obj3.b 和 obj4.b 是不同的对象)
     ```
**4. `fabric.util.degreesToRadians(degrees)`**

   - **用途:** 将角度值转换为弧度值。
   - **参数:**
      - `degrees`: 要转换的角度值（以度为单位）。

   - **返回值:** 对应的弧度值。

   - **示例:**

     ```javascript
     const degrees = 90;
     const radians = fabric.util.degreesToRadians(degrees);
     console.log(radians); // 输出: 1.5707963267948966 (π/2)
     ```
**`fabric.util.radiansToDegrees(radians)`**:弧度转角度

**5. `fabric.util.sizeAfterTransform(width, height, options)`**
计算经过形变之后的尺寸.
  - **参数**
     -`width`: 原始宽度
     -`height`:原始高度
      -options: 一个包含形变信息的对象, 通常包括`scaleX`,`scaleY`,`skewX`,`skewY`等.
  -**返回值**
  一个包含`x`,`y`的对象,表示形变后的尺寸.

**6. `fabric.util.loadImage(url, callback, context, options)`**

   - **用途:** 加载一张图片。
   - **参数:**
      - `url`: 要加载的图片的 URL。
      - `callback`: 加载完成后的回调函数。该函数接收两个参数：
         - `image`: 一个 `HTMLImageElement` 对象，表示加载的图片。如果加载失败，则为 `null`。
         - `error`: 一个错误对象，如果加载过程中发生错误。
      - `context` (可选): 回调函数的执行上下文 (即 `this` 值)。
      - `options` (可选): 一个对象，可以包含以下属性：
         - `crossOrigin`:  图片的 `crossOrigin` 属性值。用于处理跨域图片加载。常见的值有 `'anonymous'` 和 `'use-credentials'`。

   - **返回值:** 无。

   - **示例:**

     ```javascript
     const imageUrl = 'path/to/your/image.png';

     fabric.util.loadImage(
       imageUrl,
       (image, error) => {
         if (image) {
           // 创建一个 Fabric.js 的 Image 对象
           const fabricImage = new fabric.Image(image);

           // 将 Image 对象添加到画布
           canvas.add(fabricImage);
           canvas.renderAll();
         } else {
           console.error('图片加载失败:', error);
         }
       },
       null, // context
       { crossOrigin: 'anonymous' } // options
     );
     ```

**7. `fabric.util.multiplyTransformMatrices(a, b, is2x2)`**

   - **用途:**  计算两个变换矩阵的乘积。
   - **参数:**
      -   `a`:  第一个变换矩阵（一个包含 6 个元素的数组）。
      -   `b`:  第二个变换矩阵（一个包含 6 个元素的数组）。
      - `is2x2` (可选). 如果为true，则矩阵被视为2x2矩阵

   - **返回值:**  一个新的数组，表示两个矩阵相乘的结果。

   - **示例:**

     ```javascript
     // 定义两个变换矩阵
     const matrixA = [2, 0, 0, 2, 10, 20]; // 缩放 2 倍，平移 (10, 20)
     const matrixB = [1, 0.5, 0, 1, 5, 0]; // 水平倾斜 0.5，平移 (5, 0)

     // 计算两个矩阵的乘积
     const resultMatrix = fabric.util.multiplyTransformMatrices(matrixA, matrixB);

     console.log(resultMatrix); // 输出: [2, 1, 0, 2, 20, 20]
     ```

**8. `fabric.util.createClass(parent, properties)`**
   - 用途
      创建一个类，可以指定父类和类的属性和方法
   - 参数
      -  `parent`:可选，要继承的父类
      -  `properties`: 包含类的属性和方法的对象
   - 返回值
       一个新的类
```javascript
var Animal = fabric.util.createClass({
  initialize: function(name) {
    this.name = name;
  },
  say: function(message) {
    console.log(this.name + ' says: ' + message);
  }
});

// 创建 Animal 的实例
var dog = new Animal('Dog');
dog.say('Woof!'); // 输出: Dog says: Woof!

// 创建一个继承自 Animal 的子类
var Cat = fabric.util.createClass(Animal, {
  initialize: function(name, color) {
    this.callSuper('initialize', name); // 调用父类的构造函数
    this.color = color;
  },
  meow: function() {
    console.log(this.name + ' meows. Color: ' + this.color);
  }
});
var cat = new Cat('Whiskers', 'Gray');
cat.say('Meow!');   // 输出: Whiskers says: Meow!
cat.meow(); // 输出: Whiskers meows. Color: Gray

```
**9. `fabric.util.object.extend(destination, source, deep)`**

- 用途
   将source的属性复制到destination
- 参数
  - `destination`: 目标对象，将会被修改
  -  `source`： 来源，它的属性会被复制到目标对象
  - `deep`：可选，是否深度拷贝，默认为`false`
```javascript
  var obj1 = { a: 1, b: { c: 2 } };
var obj2 = { b: { d: 4 }, e: 5 };

// 浅拷贝
fabric.util.object.extend(obj1, obj2);
console.log(obj1); // 输出: { a: 1, b: { d: 4 }, e: 5 }
console.log(obj1.b === obj2.b); // 输出: true,  b是同一个对象

var obj3 = { a: 1, b: { c: 2 } };
var obj4 = { b: { d: 4 }, e: 5 };
//深拷贝
fabric.util.object.extend(obj3,obj4,true);
console.log(obj3) // 输出: { a: 1, b: { c: 2, d: 4 }, e: 5 }
console.log(obj3.b === obj4.b); // 输出: false  b是不同的对象

```

这些就是你提到的 Fabric.js 工具函数和方法的详细用法。希望这些解释和示例能帮助你更好地理解和使用它们！
