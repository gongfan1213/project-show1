<img width="1020" alt="image" src="https://github.com/user-attachments/assets/d55bfc9d-1091-4ff4-864d-bb3a16163d0d" />
<img width="993" alt="image" src="https://github.com/user-attachments/assets/083bba31-33fc-44cc-b6d8-be73dba47513" />
<img width="954" alt="image" src="https://github.com/user-attachments/assets/bdab78bc-1f6c-4574-8cc3-c37ae193ef83" />
<img width="1028" alt="image" src="https://github.com/user-attachments/assets/d3be2817-af91-46ad-a2d8-48dc322b6aee" />
<img width="999" alt="image" src="https://github.com/user-attachments/assets/63a38def-bf61-49fd-8136-089c413d6ac1" />
<img width="957" alt="image" src="https://github.com/user-attachments/assets/31b798e2-a03d-4d9b-bcf7-41781c8596fb" />
<img width="991" alt="image" src="https://github.com/user-attachments/assets/6cfa9f28-fab5-47e3-ae58-30f9ad69c295" />
<img width="1001" alt="image" src="https://github.com/user-attachments/assets/dc3a2b08-d2f7-4850-a683-4a9287ec42b8" />
<img width="998" alt="image" src="https://github.com/user-attachments/assets/10845a07-6c4f-404b-b5d9-7ab7e5ed41cc" />
<img width="1065" alt="image" src="https://github.com/user-attachments/assets/9cdaba6c-d566-45c4-b54d-356817bcec48" />

好的，下面是对上述内容中性能优化方法的整理，包括问题诊断、处理方案和相关工具：

**性能优化总结**

**1. Lighthouse 简介与使用**

*   **简介：** Lighthouse 是一款开源自动化工具，用于评估网页质量，包括性能、无障碍功能、最佳做法和 SEO 等方面。它会给出评分、具体问题和改进建议。
*   **使用：** 在 Chrome 开发者工具中打开 Lighthouse 选项卡，点击“分析网页加载情况”按钮，即可生成报告。

**2. Lighthouse 性能评分指标**

Lighthouse 性能评分主要基于以下五个指标：

*   **首次内容绘制（FCP）：** 页面开始显示内容的速度。
*   **最大内容绘制（LCP）：** 页面主要内容的加载速度。
*   **总阻塞时间（TBT）：** 页面在加载过程中响应用户输入的能力。
*   **速度指数（SI）：** 页面内容在视口内可见的速度。
*   **累积布局偏移（CLS）：** 页面的视觉稳定性。

优化目标是：

*   FCP、LCP、TBT、SI 越小越好。
*   CLS 越接近 0 越好。

**3. 性能问题诊断与优化方案**

| 问题                               | 诊断方法                                                                                             | 处理方案                                                                                                                                                                                                                                                                                                              | 相关工具/插件                                                                                                                                                                                                 |
| :--------------------------------- | :--------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 总阻塞时间（TBT）过大             | Lighthouse 报告                                                                                       | 使用 `react-lazy-load-image-component` 插件懒加载图片，减少主线程任务耗时。                                                                                                                                                                                                                                      | `react-lazy-load-image-component`                                                                                                                                                                      |
| 未使用的 CSS 和 JS                 | 1.  开发者工具 → “运行命令” → 输入 `coverage` → “显示覆盖范围”  2.  点击“重新运行”，查看 CSS/JS 使用情况（红色区域为未使用） | 1.  移除未使用的 CSS 和 JS 代码。  2.  使用 `gatsby-plugin-purgecss` 插件（Gatsby 项目）自动移除未使用的 CSS。                                                                                                                                                                                                    | `gatsby-plugin-purgecss`                                                                                                                                                                              |
| 图片格式未优化                     | Lighthouse 报告                                                                                       | 1.  使用 WebP 格式图片。  2.  使用 `react-lazy-load-image-component` 插件懒加载图片。  3.  使用 `gatsby-plugin-image` 组件（Gatsby 项目）自动优化图片格式。  4.  对于 PNG 图片，使用 `tinypng` 等工具压缩。                                                                                              | `react-lazy-load-image-component`, `gatsby-plugin-image`, `tinypng`                                                                                                                               |
| 启用文本压缩                       | Lighthouse 报告                                                                                       | 接口请求的响应头添加 `content-encoding: gzip` 字段。                                                                                                                                                                                                                                                             |                                                                                                                                                                                                            |
| JavaScript 执行时间过长           | Lighthouse 报告，开发者工具“性能”面板                                                                       | 1.  增强服务器性能（例如，部署新的测试环境）。  2.  缩减代码量，拆分代码，移除未使用的代码。                                                                                                                                                                                                                             |                                                                                                                                                                                                            |
| 主线程工作时间过长                 | Lighthouse 报告，开发者工具“性能”面板                                                                       | 1.  优化代码，减少不必要的计算和操作。  2.  使用 Web Workers 将部分任务移出主线程。                                                                                                                                                                                                                                 | Web Workers                                                                                                                                                                                                 |
| 浏览器渲染程序进程主线程负载过高 | 开发者工具“性能”面板，查看网络记录中耗时较长的点，点击查看下方具体情况                                                   | 减少 JavaScript 代码量（代码拆分、懒加载）； 优化样式和布局（减少复杂 CSS 规则和布局计算）； 优化渲染（减少重绘和重排）； 减少解析和编译时间（优化 JS 代码）；优化垃圾回收（优化内存管理策略）                                                                                                                                    |   Chrome DevTools                                                                                                                                                                                     |
**4.优化图片加载的具体实现**
* **加载本地静态图片**
    *   **`StaticImage` (Gatsby):**
    ```jsx
     import { StaticImage } from 'gatsby-plugin-image';

     <StaticImage src="../images/test.png" alt="test" />
     ```
    *注意：* `StaticImage` 仅支持相对路径，不支持通过 props 传递路径。

    *   **`GatsbyImage` (Gatsby):**
    ```jsx
    import { GatsbyImage, getImage } from 'gatsby-plugin-image';
    import { graphql, useStaticQuery } from 'gatsby';

    export default function MyGatsbyImage(props) {
    const { imageName, width, height, alt = '', style } = props;
    const data = useStaticQuery(graphql`
        query {
            allFile(filter: { extension: { regex: "/(jpg|jpeg|png|gif)/" } }) {
            nodes {
                relativePath
                childImageSharp {
                gatsbyImageData(layout: CONSTRAINED)
            }
        }
      }
      }
      `);
      const imageNode = data.allFile.nodes.find(node => node.relativePath === imageName);
      const image = getImage(imageNode.childImageSharp.gatsbyImageData);
     return (
        <GatsbyImage
          image={image}
          alt={alt}
          style={{ borderRadius: '8px', width, height, ...style }}
          />
      );
     }
     ```
*   **加载接口动态渲染图片:**
    ```jsx
    import { LazyLoadImage } from 'react-lazy-load-image-component';
    import 'react-lazy-load-image-component/src/effects/blur.css';
    <LazyLoadImage
      src={src} // 图片地址
      alt={alt} // 描述图片的内容
      crossOrigin="anonymous" // 设置跨域属性
      effect="blur" // 加载效果
      style={{ objectFit: 'contain', width: '100%', height: '100%', ...style }}
      onClick={handleClick} // 点击图片
      onError={(e) => { e.currentTarget.src = img_error }} // 图片加载失败后的展示图
      placeholderSrc={placeholderImage} // 图片加载完成之前显示的占位符
      beforeLoad={handleBeforeLoad} // 图片开始加载之前调用
      afterLoad={handleAfterLoad} // 图片加载完成之后调用
    {...reset}
     />
    ```
**5. 总结**

通过以上方法，可以全面诊断和优化网页性能，提升用户体验。主要优化手段包括：

*   **代码优化：** 减少代码量、移除未使用的代码、优化代码逻辑。
*   **资源优化：** 图片格式优化、图片懒加载、文本压缩。
*   **渲染优化：** 减少主线程工作、优化样式和布局计算、使用 Web Workers。
*   **构建优化：** 使用相关插件（如 `gatsby-plugin-purgecss`、`gatsby-plugin-image`）优化项目工程。

通过这些综合措施，可以有效提升网页的加载速度、响应速度和视觉稳定性，从而提供更好的用户体验。
