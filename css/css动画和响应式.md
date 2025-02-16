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
