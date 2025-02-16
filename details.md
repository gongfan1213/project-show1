# 1
- canvas只能压缩image/jpeg 或 image/webp格式，所以输出图片格式统一为image/jpeg
# 2
这段代码中的注释提到了一个与 Fabric.js 相关的 bug，具体是：

**问题：** 在编组（group）的情况下，通过 `setSrc` 方法修改图片的 `src` 值（这里是用 `base64Original` 替换原来的缩略图），会导致图片在缩放时变得模糊。

**原因推测：** 注释中说是 "Fabric 的一个未知 bug"，这意味着具体原因不清楚，可能是 Fabric.js 内部在处理编组内图片 `src` 更新和缩放时存在的逻辑问题。  以下是可能的原因，更偏向底层的推测：

1.  **缓存机制问题：** Fabric.js 为了优化性能，会对图像进行缓存。当 `setSrc` 改变了图像的源，但缩放操作可能仍然使用了旧的缓存图像（缩略图），导致显示模糊。 尤其是在编组内，缓存的更新可能没有正确地同步。

2.  **渲染机制问题：**  Fabric.js 的渲染管线可能在编组内更新图像 `src` 后，没有正确地处理图像的重新采样或抗锯齿，导致在缩放时出现模糊。

3.  **ClipPath 相关问题:** 代码中提到了 `removeClipPathAndCache()` 和 `restoreClipPath()`，这表明图像可能应用了裁剪路径。在编组内，`setSrc` 与裁剪路径的交互可能存在问题，导致渲染不正确。

4. **图像对象状态：** 改变图片对象的scale可能会与setSrc产生冲突。

**代码中的临时解决方案：**

注释中提到的临时解决方案是使用克隆的方式来解决：

*   `canvasEditor?.cloneTextureGroup(object, base64Original)`：  克隆整个包含原始图像的新编组。这意味着不是直接修改现有图像对象的 `src`，而是创建一个全新的、包含正确图像数据的编组。
*   `canvasEditor?.canvas.add(group)`: 添加克隆的编组到画布。
*  `canvasEditor?.canvas.remove(object)`：移除原来的编组。

**为什么克隆能解决问题（推测）:**

*   **全新的对象：** 克隆创建了一个全新的 Fabric.js 对象，绕过了可能存在问题的 `setSrc` 更新逻辑。新的对象从一开始就拥有正确的图像数据，避免了缓存或渲染机制的错误。
*   **状态重置：** 克隆过程可能重置了与图像相关的内部状态，确保所有属性（包括缩放、裁剪路径等）都基于新的图像数据正确设置。

**其他值得注意的点：**
* `stopSaveHistory` 和 `startSaveHistory`: 停止和开启记录历史,说明作者可能之前在这个问题上尝试了很多方法,通过控制历史记录的方式来试图绕过bug.

**总结:**

这段代码中的 bug 是 Fabric.js 内部在特定条件下（编组内、使用 `setSrc` 更新图像、缩放）的一个已知问题（但具体原因未知）。代码作者采用了克隆整个编组的临时方法来规避这个问题，以确保图像在替换为高分辨率版本后，缩放时保持清晰。  如果需要彻底解决，可能需要深入 Fabric.js 源码进行调试，或者向 Fabric.js 社区报告该 bug。
# 3
这段 React 代码实现了一个文件上传组件，其中包含了一些针对浏览器兼容性的处理。我将逐一分析这些兼容性处理：

**1. `setTimeout` 替代 `requestIdleCallback` (主要兼容性)**

   - **代码:**
     ```javascript
     setTimeout(() => {
       doUpload({ ..._f, status: 'uploading' });
     }, 10)
     ```

   - **原意图 (被注释掉的代码):**
     ```javascript
     // if ('requestIdleCallback' in window) {
     //   requestIdleCallback((deadline) => {
     //     if (deadline.timeRemaining() > 0) {
     //       doUpload({ ..._f, status: 'uploading' });
     //     }
     //   })
     // } else {
     //   // ... setTimeout ...
     // }
     ```

   - **兼容性问题:**  `requestIdleCallback` 是一个较新的 API，用于在浏览器空闲时执行任务，以避免阻塞主线程。然而，它的兼容性存在以下问题：
      - **浏览器支持:**  较旧的浏览器（如 IE）不支持 `requestIdleCallback`。
      - **低帧率问题 (关键):**  在电脑帧率较低的情况下，`deadline.timeRemaining()` 可能一直为 0，导致回调函数永远不会执行，上传请求无法发出。

   - **`setTimeout` 的兼容性:**  `setTimeout` 是一个非常古老的 API，几乎所有浏览器都支持。即使设置一个很小的延迟（如 10 毫秒），也能确保代码在主线程的下一个事件循环中执行，避免了 `requestIdleCallback` 在低帧率下的问题。

   - **为什么 `setTimeout` 更好:**  虽然 `requestIdleCallback` 在理想情况下可以更好地利用浏览器空闲时间，但为了兼容性和稳定性，`setTimeout` 是一个更可靠的选择，尤其是在需要处理大量文件上传的情况下。

**2. `[].slice.call(files)`**

   - **代码:**
     ```javascript
     onChange={(e: React.ChangeEvent<HTMLInputElement>) => {
       const files = e.target.files;
       if (files) {
         handleFilesChange([].slice.call(files));
         // ...
       }
     }}
     ```

   - **兼容性问题:**  `e.target.files` 返回的是一个 `FileList` 对象，它是一个类数组对象 (array-like object)，而不是真正的数组。在一些较旧的浏览器中，直接对 `FileList` 使用数组方法（如 `forEach`、`map`）可能会出现问题。
     FileList对象没有提供forEach的方法.

   - **解决方法:**  `[].slice.call(files)`  是一种将类数组对象转换为真正数组的常用技巧。它利用了 `slice` 方法的特性：当 `slice` 方法不带参数调用时，会返回数组的一个浅拷贝。通过 `call` 方法将 `slice` 的 `this` 指向 `FileList` 对象，就可以将其转换为一个真正的数组。

   - **更现代的方法 (但此代码未使用):**  `Array.from(files)`  是 ES6 引入的更简洁的方法，可以将类数组对象或可迭代对象转换为数组。如果不需要兼容非常旧的浏览器，`Array.from` 是更好的选择。  代码中`useImperativeHandle`中, 就使用了`Array.from()`方法.

**3.  `inputRef?.current?.click()` (可选链)**

- 代码
```javascript
const handleBrowseFile = () => {
    inputRef?.current?.click()
}
```
- 兼容性问题：`?.`是可选链操作符，`?.`之前的对象如果不存在，则表达式返回`undefined`，如果存在则继续计算。如果浏览器不支持可选链，会报语法错误。
- 兼容方法： 可以用`&&`来代替
```javascript
const handleBrowseFile = () => {
  inputRef && inputRef.current && inputRef.current.click()
}
```

**4. `inputRef?.current && (inputRef.current.value = '')` (可选链和短路)**

   - **代码:**
     ```javascript
     inputRef.current && (inputRef.current.value = '');
     ```

   - **兼容性:**  这行代码本身没有直接的兼容性问题，它利用了 JavaScript 的短路逻辑。
     *   `inputRef.current`: 这里检查了`inputRef.current`是否存在。
     *   `&&`: 如果 `inputRef.current` 为 `null` 或 `undefined`，整个表达式会短路，后面的赋值操作不会执行，避免了错误。

**总结:**

这段代码中最重要的兼容性处理是使用 `setTimeout` 替代 `requestIdleCallback`，以确保在各种浏览器和帧率条件下都能可靠地发起文件上传请求。 `[].slice.call(files)` 也是一个重要的兼容性处理，但可以用更现代的 `Array.from(files)` 代替（如果不需要支持非常旧的浏览器）。可选链的处理也是为了代码的健壮性。
