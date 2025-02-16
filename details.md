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
# 4-滤镜：刷新后再次删除时，匹配不上，会多一些type，opaque类似的键
这段代码实现了一个 Fabric.js 的图片工具插件，提供了多种滤镜效果。代码中注释提到了一个已知问题，我将详细分析这个问题，并指出代码中可能存在的其他潜在问题：

**1. `removeSpecificFilters` 方法中的已知问题 (注释中提到的)**

   - **问题描述:**  "目前还存在些问题，如刷新后再次删除时，匹配不上，会多一些 type，opaque 类似的键，需要后续排查"
   - **原因分析:**
      - **`filtersMap` 的数据来源不稳定:**  `removeSpecificFilters` 方法中的 `filtersMap` 数据来源有两个：`activeObject.filtersMap` 和 `filtersMapData`。当 `activeObject.filtersMap` 为空时，会使用 `filtersMapData`。这导致 `filtersMap` 的数据可能不一致，尤其是在刷新或重新加载图片后。
        *   `activeObject.filtersMap`： 这是存储在 Fabric.js 对象上的自定义属性，用于记录已应用的滤镜。
        *   `filtersMapData`： 这个参数的来源不明确，可能是外部传入的，也可能是组件内部状态。
      - **`isEqualFilterIgnoringType` 的比较不完全可靠:**  这个函数通过将滤镜对象转换为 JSON 字符串并忽略 `type` 字段来比较滤镜是否相同。这种方法存在以下问题：
         *   **属性顺序:**  JSON 字符串化时，对象的属性顺序可能会影响结果。如果两个滤镜对象的属性顺序不同，即使它们逻辑上相同，`JSON.stringify` 也会认为它们不同。
         *    **其他属性差异:** 除了 `type`，滤镜对象可能还包含其他属性（如注释中提到的 `opaque`），这些属性的差异会导致 `isEqualFilterIgnoringType` 返回 `false`，即使滤镜在视觉上是相同的。
         * **浮点数精度:** 如果滤镜参数包含浮点数，`JSON.stringify` 可能会因为精度问题导致比较失败。
      - **刷新/重新加载导致的状态不一致:**  当图片刷新或重新加载后，`activeObject.filtersMap` 可能会被重置，而 `filtersMapData` 可能仍然保留旧的数据。这导致 `removeSpecificFilters` 无法正确匹配和移除滤镜。

   - **改进建议:**
      1.  **统一 `filtersMap` 数据源:**  应该始终使用 `activeObject.filtersMap` 作为滤镜映射数据的来源。避免使用外部的 `filtersMapData`，除非有非常明确的理由。
      2.  **更可靠的滤镜比较:**  不要依赖 `JSON.stringify` 来比较滤镜对象。可以创建一个更健壮的比较函数，逐个比较滤镜对象的关键属性（如 `brightness`、`contrast`、`matrix` 等）。对于浮点数，可以使用 `Math.abs(a - b) < epsilon` 的方式进行比较，其中 `epsilon` 是一个很小的容差值。
      3.  **考虑使用 `filter.type`:** 如果每种滤镜的 `type` 属性是唯一的，可以直接使用 `filter.type` 来识别和移除滤镜，而不需要复杂的比较逻辑。例如：

          ```javascript
          activeObject.filters = activeObject.filters.filter(
            (filter) => !filterIdentifiers.includes(filter.type)
          );
          ```
      4. **在对象创建的时候就添加上`filtersMap`**

**2. 潜在问题：`filtersMap` 的滥用**

   - **问题描述:**  代码中大量使用了 `filtersMap` 来自定义属性来跟踪滤镜，这可能会导致代码难以维护和理解。
   - **原因分析:**
      -  **非标准属性:**  `filtersMap` 不是 Fabric.js 的标准属性，过度依赖自定义属性会降低代码的可读性和可移植性。
      - **与 Fabric.js 内部状态的潜在冲突:**  如果 Fabric.js 未来版本使用了同名的属性，可能会导致冲突。
      - **代码冗余:**  在多个方法中（`setConvolute`、`setOldPhoto`、`setHighlight`、`removeSpecificFilters`）都重复了对 `filtersMap` 的操作，增加了代码冗余。

   - **改进建议:**
      - **尽量使用 Fabric.js 的标准 API:**  Fabric.js 提供了 `filters` 数组来管理滤镜，应该尽量使用这个数组。
      - **封装滤镜操作:**  可以将滤镜的添加、移除、查找等操作封装成更高级别的函数，避免直接操作 `filtersMap`。
      - **考虑使用 `filter.type` 或其他标准属性:**  如果 `filter.type` 不足以唯一标识滤镜，可以考虑使用其他标准属性（如 `filter.id`，如果 Fabric.js 提供了的话）。

**3. 潜在问题：`setOldPhoto`和`setHighlight`的滤镜添加方式**

   - **问题描述:** `setOldPhoto`和`setHighlight`方法直接向 `activeObject.filters` 数组添加了多个滤镜，没有移除可能存在的旧滤镜（除了sepias）

    - **原因分析**
        - **重复添加**：如果用户多次调用`setOldPhoto`，则会不断地向`filters`数组添加滤镜，造成滤镜效果叠加，而不是替换。虽然使用了`filtersMap`来标记滤镜已添加，但这仅仅阻止了重复进入`setOldPhoto`的逻辑，并没有阻止滤镜的重复添加。
    - **改进建议**
          像其他的设置滤镜的方法一样，先移除，再添加

          ```typescript
          // 移除旧的滤镜，如果有的话
          activeObject.filters = activeObject?.filters?.filter(
            (f) =>
              !(
                f instanceof fabric.Image.filters.Noise ||
                f instanceof fabric.Image.filters.Brightness ||
                f instanceof fabric.Image.filters.Contrast ||
                f instanceof fabric.Image.filters.Sepia
              )
          );
            // 添加新的滤镜并更新filtersMap
            // @ts-ignore
            activeObject.filters.push(filter1, filter2, filter3, filter4);
          ```
**4. 潜在问题：部分get方法直接返回了0**
    例如`getShadows`

**总结:**

这段代码的主要问题是 `removeSpecificFilters` 方法中滤镜比较逻辑的不可靠性，以及过度使用自定义属性 `filtersMap` 导致的潜在问题。通过改进滤镜比较方法、统一 `filtersMap` 数据源、封装滤镜操作，并尽量使用 Fabric.js 的标准 API，可以提高代码的健壮性、可读性和可维护性。

