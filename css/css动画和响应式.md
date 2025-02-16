这段 SCSS 代码中定义了几种动画效果，主要用于加载状态。我将逐个分析这些动画：

**1. `loading` (通用旋转动画)**

   - **定义:**

     ```scss
     @keyframes loading {
       to {
         transform: rotate(360deg);
       }
     }
     ```

   - **应用:**

     ```scss
     .Steps1_loading {
       width: RelativeWidth(20);
       height: RelativeWidth(20);
       animation: loading 2s linear infinite;
     }
     // 以及其他几个地方的 .Print_loading, .Generate_btn_loading 等
     ```

   - **效果:**  这是一个标准的旋转动画。`@keyframes loading` 定义了一个名为 `loading` 的关键帧动画。
      - `to { transform: rotate(360deg); }`:  表示动画的最终状态是将元素旋转 360 度（即完整的一圈）。
      - `animation: loading 2s linear infinite;`:  将动画应用于元素。
         - `loading`:  使用名为 `loading` 的关键帧动画。
         - `2s`:  动画持续时间为 2 秒。
         - `linear`:  动画以恒定速度进行（线性过渡）。
         - `infinite`:  动画无限循环播放。

   - **总结:** 这个动画会让元素（通常是一个加载图标）持续地、平滑地旋转，表示正在加载或处理中。

**2. `loading1` (另一个旋转动画，仅用于 `changeStyle_box` 内的 `.Subtract_image`)**

   - **定义:**

     ```scss
     @keyframes loading1 {
       to {
         transform: rotate(360deg);
       }
     }
     ```

   - **应用:**

     ```scss
     .Subtract_image {
       width: RelativeWidth(48);
       height: RelativeWidth(48);
       animation: loading1 2s linear infinite;
     }
     ```

   - **效果:**  与 `loading` 动画完全相同，只是名称不同，并且只应用在`.changeStyle_box .blur_overlay .Subtract_image`这个元素上。可能是为了区分不同的加载场景，或者未来可能对这个动画进行修改。

**3. `RemoveBgState_loading` (应用于滤镜和风格项目的加载)**
    - 定义
    ```scss
      @keyframes loading {
          to {
            transform: rotate(360deg);
          }
        }
    ```
    - 应用
    ```scss
        .RemoveBgState_loading {
          position: absolute;
          top: RelativeWidth(20);
          left: RelativeWidth(20);
          right: RelativeWidth(20);
          bottom: RelativeWidth(20);
          width: RelativeWidth(60);
          height: RelativeWidth(60);
          animation: loading 2s linear infinite;
          cursor: not-allowed;
        }
    ```
    - 效果： 和`loading`是一样的, 都是旋转. 但是这里添加了`cursor: not-allowed;`,表示鼠标放上去是禁止点击的图标.

**总结:**

这段 SCSS 代码中主要使用了旋转动画来表示加载状态。`@keyframes` 定义了动画的关键帧，`animation` 属性将动画应用于元素。通过设置 `animation` 属性的不同值（持续时间、过渡类型、循环次数），可以控制动画的具体表现。 这种旋转动画是加载指示器的常见做法，能够直观地告诉用户系统正在处理中。

**关于 `RelativeWidth` 和 `RelativeHeight`**

这两个是 SCSS 函数，用于实现响应式布局：

- `@function RelativeWidth($target)`:  将基于 1920px 设计稿的宽度值 `$target` 转换为相对于视口宽度 (viewport width) 的百分比值 (`vw`)。
-   `@function RelativeHeight($target)`: 将基于1080px的设计稿的高度值转换为相对于视口高度的百分比
- **计算公式:**
  - `RelativeWidth`:  `(目标宽度 / 1920) * 100vw`
  - `RelativeHeight`: `(目标高度 / 1080) * 100vh`

- **好处:** 这种方法可以使页面元素在不同屏幕尺寸下保持相对比例，从而实现响应式布局。
```js
// 计算相对宽度的函数
@function RelativeWidth($target) {
  @return calc(#{$target} / 1920 * 100vw);
}

@function RelativeHeight($target) {
  @return calc(#{$target} / 1080 * 100vh);
}
```
# sass创建响应式布局
这两个是用 Sass（SCSS 语法）编写的函数，用于创建响应式布局。让我详细解释一下它们的作用、工作原理以及为什么这样写：

**1. 作用：**

- `RelativeWidth($target)`:  将一个以像素 (px) 为单位的宽度值（`$target`，通常是设计稿中的宽度）转换为相对于视口宽度（viewport width, `vw`）的百分比值。
- `RelativeHeight($target)`:  将一个以像素 (px) 为单位的高度值（`$target`，通常是设计稿中的高度）转换为相对于视口高度（viewport height, `vh`）的百分比值。

**2. 工作原理：**

- **`@function`:**  这是 Sass 中定义函数的关键字。
- **`RelativeWidth($target)` / `RelativeHeight($target)`:**  函数名和参数。`$target` 是一个占位符，表示传入的像素值。
- **`@return`:**  指定函数的返回值。
- **`calc(...)`:**  这是一个 CSS 函数，用于执行计算。在 Sass 中，`calc()` 函数通常用于混合不同单位的计算（例如像素和百分比）。
- **`#{$target}`:**  这是 Sass 中的插值语法。它将 Sass 变量 (`$target`) 的值插入到 CSS 属性值中。  如果不使用插值，`calc()` 函数会把 `$target` 当作一个普通的字符串，而不是一个数值。
- **`1920` / `1080`:**  这些是设计稿的基准宽度和高度（以像素为单位）。  通常，UI 设计师会基于一个固定的屏幕尺寸（例如 1920x1080）来设计界面。
- **`100vw`:**  `vw` 是视口宽度单位。`1vw` 等于视口宽度的 1%。`100vw` 表示视口宽度的 100%，即整个视口宽度。
- **`100vh`:**  `vh` 是视口高度单位。`1vh` 等于视口高度的 1%。`100vh` 表示视口高度的 100%，即整个视口高度。

**计算公式:**

- `RelativeWidth`:  `(目标宽度 / 1920) * 100vw`  =  `(目标宽度 / 设计稿宽度) * 视口宽度`
- `RelativeHeight`: `(目标高度 / 1080) * 100vh` = `(目标高度 / 设计稿高度) * 视口高度`

**3. 为什么这样写？**

这种写法是为了实现**响应式布局**，使网页元素在不同屏幕尺寸下能够自动缩放，保持与设计稿相似的比例。

- **基于设计稿的固定尺寸:**  UI 设计师通常会提供一个固定尺寸的设计稿（例如 1920x1080）。
- **转换为相对单位:**  前端开发者需要将设计稿中的固定像素值转换为相对单位（如 `vw`、`vh`、`%`），以便在不同屏幕上自适应。
- **`RelativeWidth` 和 `RelativeHeight` 函数:**  这两个函数提供了一种方便的方式来执行这种转换。  你只需要传入设计稿中的像素值，函数会自动计算出相应的 `vw` 或 `vh` 值。
- **使用 `calc()`:**  `calc()` 函数是必要的，因为它允许你在 CSS 中进行不同单位的计算。  如果没有 `calc()`，你就不能直接将像素值与 `vw` 或 `vh` 相乘。
- **Sass 的优势:** Sass（或 SCSS）提供了函数、变量、插值等功能，使得编写响应式样式更加方便和可维护。

**示例：**

假设设计稿中有一个按钮，宽度为 192px，高度为 54px。使用这两个函数，你可以这样写：

```scss
.my-button {
  width: RelativeWidth(192);  // 转换为 (192 / 1920) * 100vw = 10vw
  height: RelativeHeight(54); // 转换为 (54 / 1080) * 100vh = 5vh
}
```

这样，无论屏幕宽度和高度是多少，按钮的宽度将始终是视口宽度的 10%，高度将始终是视口高度的 5%。

**总结:**

`RelativeWidth` 和 `RelativeHeight` 函数是一种常见的、基于设计稿尺寸创建响应式布局的技术。它们利用 Sass 的功能和 `calc()` 函数，将固定像素值转换为相对视口的单位，从而使网页元素在不同屏幕上自适应。
