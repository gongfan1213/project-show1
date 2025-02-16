# requestIdllecallback使用
## taskqueue
`taskQueue` 在 `HistoryPlugin` 中的 `historySave` 方法中使用。具体来说，它的作用和性能优化体现在以下几个方面：

1.  **防止频繁的、立即的历史记录保存操作阻塞主线程：**

    *   当用户在 Fabric.js 画布上进行操作（添加、修改、删除对象）时，会触发 `object:added`、`object:modified`、`object:removed` 等事件。
    *   这些事件都会触发 `historySave` 方法。如果每次触发都立即执行历史记录保存（包括 `getJSON`、深拷贝、`history.push`、事件触发等），可能会导致短时间内多次执行耗时操作，阻塞主线程。
    *   `taskQueue` 将这些历史保存操作放入一个队列中，**异步地、按顺序地**执行，而不是立即执行。

2.  **异步执行，避免阻塞：**

    *   `historySave` 方法将历史保存操作包装成一个 `Promise`，并将这个 `Promise` 对应的任务（`task`）放入 `taskQueue`。
    *   `processQueue` 方法负责从队列中取出任务并执行。由于是异步执行，即使历史保存操作比较耗时，也不会阻塞主线程。
     * 虽然这里没有用 `requestIdleCallback`, 但 `processQueue` 会依次执行队列中的任务，这已经实现了异步和避免阻塞的效果。它相当于一个简单的、没有利用浏览器空闲时间的任务调度器。

3.  **合并短时间内的多次操作：**

    *   如果用户在短时间内进行了多次操作（例如，连续移动一个对象多次），`taskQueue` 会将这些操作对应的历史保存任务合并。
    *   `processQueue` 会依次处理这些任务，但由于是异步的，它们不会同时执行，而是按顺序执行，避免了不必要的重复保存。虽然它不会像防抖/节流那样减少保存次数，但至少保证了按顺序、异步地执行，避免了阻塞。

4. **避免重复的历史记录**
    *  在添加到队列前, 会先比较最后一次历史记录和当前的json, 如果相同则不加入队列

**性能优化总结：**

`taskQueue` 在 `HistoryPlugin` 中主要起到了以下性能优化作用：

*   **将同步操作变为异步操作：** 将 potentially 耗时的历史记录保存操作变为异步，避免阻塞主线程。
*   **按顺序执行：** 确保历史记录按照用户操作的顺序保存。
*   **合并短时间内的多次操作（有限的）：** 虽然不能减少保存次数，但至少保证了按顺序、异步执行。
*   **避免了不必要的重复保存（通过比较 JSON）。**

虽然 `taskQueue` 本身没有利用浏览器的空闲时间（如 `requestIdleCallback` 那样），但它通过将任务放入队列并异步执行，已经实现了基本的性能优化，避免了阻塞主线程。

在这个 `HistoryPlugin` 代码中，并没有直接使用 `requestIdleCallback`。但是，代码实现了一个任务队列 `taskQueue`，其目的和 `requestIdleCallback` 有相似之处，都是为了避免长时间运行的任务阻塞主线程。

虽然没有直接使用`requestIdleCallback`，但`historySave`方法中的逻辑可以考虑用`requestIdleCallback`进行优化。下面我将解释如何优化，以及优化后的好处：

**当前代码的问题：**

当前`historySave`方法使用了一个简单的任务队列 (`this.taskQueue`) 来处理历史记录保存的异步操作。虽然这比立即执行所有历史保存操作要好，但它仍然可能在短时间内执行多个可能比较耗时的任务（`getJSON`, `history.push` 涉及的深拷贝, 以及事件触发），尤其是在用户快速连续操作的情况下。这仍然有阻塞主线程的风险。

**使用 `requestIdleCallback` 优化 `historySave`:**

可以将 `processQueue` 函数改造为使用 `requestIdleCallback`：

```typescript
  private async processQueue() {
    if (this.taskQueue.length === 0) {
      return;
    }

    requestIdleCallback(async (deadline) => {
        while (this.taskQueue.length > 0 && deadline.timeRemaining() > 0)
        {
            const task = this.taskQueue.shift();
            if (task) {
              try {
                await task();
              } catch (error) {
                ConsoleUtil.error('Error processing task:', error);
              }
            }
        }

        if (this.taskQueue.length > 0)
        {
            this.processQueue(); // 如果还有剩余，继续用requestIdleCallback执行
        }
    });
  }
```

**优化后的好处:**

1.  **更精细的控制：**  `requestIdleCallback` 提供了 `deadline.timeRemaining()`，可以让你知道当前帧还剩多少时间。这允许你更精细地控制任务执行，例如，如果 `getJSON()` 或深拷贝操作预计会花费较长时间，你可以在 `timeRemaining()` 不足时中断任务，将剩余部分放入下一个 `requestIdleCallback`。
2.  **更好的用户体验：**  通过将历史记录保存任务分解到浏览器的空闲时间，可以最大限度地减少对用户交互的影响，避免卡顿，提供更流畅的用户体验。即使在用户连续快速操作的情况下，也不会阻塞主线程。
3. **与现有代码兼容** 不需要对historySave的调用方式进行修改.

**为什么原代码没有使用 `requestIdleCallback`：**

*   **复杂度：**  `requestIdleCallback` 的使用比简单的任务队列要复杂一些，需要处理 `deadline` 对象，并可能需要将任务分解成更小的块。
*   **兼容性：** 虽然有 polyfill，但开发者可能出于兼容性考虑，选择更简单的 `setTimeout` 方案（尽管 `setTimeout` 无法真正利用浏览器空闲时间）。
*   **历史原因/认知：** 开发者可能在编写代码时没有意识到 `requestIdleCallback` 的优势，或者项目早期版本没有使用它。

**总结:**

在这个`HistoryPlugin`中，虽然没有直接使用 `requestIdleCallback`，但是它的`taskQueue`的设计思想与`requestIdleCallback`的目标是一致的. 将 `processQueue` 函数用 `requestIdleCallback` 重构，可以显著提升性能和用户体验，特别是在处理可能有耗时操作的历史记录保存时。

这段代码实现了一个任务队列 `TaskQueue`，并利用 `requestIdleCallback` API 来在浏览器空闲时执行任务。下面是对代码的分析以及 `requestIdleCallback` 的作用、使用场景和原因：

**代码分析：**

1.  **`TaskQueue` 类:**
    *   `tasks`: 一个数组，用于存储待执行的任务。
    *   `isProcessing`: 一个布尔值，表示当前是否正在处理任务。
    *   `constructor()`: 初始化 `tasks` 数组和 `isProcessing` 标志。
    *   `addTask(task)`: 将任务添加到队列中，如果当前没有任务在处理，则立即开始处理任务。
    *   `processTasks()`: 设置 `isProcessing` 为 `true`，然后使用 `requestIdleCallback` 来调度任务执行。
        *   在 `requestIdleCallback` 的回调函数中，循环处理任务，直到任务队列为空或当前帧没有剩余时间 (`deadline.timeRemaining() > 0`)。
        *   如果任务队列不为空，则递归调用 `processTasks()`，以便在下一个空闲期继续处理任务。
        *   如果任务队列为空，则将 `isProcessing` 设置为 `false`。
    *   `executeTask(task)`: 执行任务。目前只处理函数类型的任务，直接调用该函数。

2.  **`requestIdleCallback` polyfill:**
    *   优先使用浏览器的原生 `requestIdleCallback` API。
    *   如果浏览器不支持，则使用 `setTimeout` 模拟一个简单的 `requestIdleCallback`。
        *   `start`: 记录回调函数开始执行的时间。
        *   `setTimeout` 延迟 1ms 执行回调。
        *   回调函数接收一个模拟的 `deadline` 对象，其中：
            *   `didTimeout`: 始终为 `false` (因为没有实现真正的超时)。
            *   `timeRemaining()`: 计算剩余时间，最大为 50ms (模拟浏览器每帧最大执行时间)。

3.  **`cancelIdleCallback` polyfill:**
     * 优先使用浏览器的原生 `cancelIdleCallback`
     *  如果浏览器不支持, 则使用 `clearTimeout` 来取消 `setTimeout`

**`requestIdleCallback` 的作用、使用场景和原因：**

*   **作用：** `requestIdleCallback` 是一个浏览器 API，用于在浏览器的主线程空闲时执行低优先级的后台任务。它允许开发者利用浏览器空闲时间来执行那些不紧急、不影响用户体验的任务，从而避免阻塞主线程，提高页面的响应速度和流畅度。

*   **使用场景：**
    *   **非关键任务：** 适用于那些不影响用户当前操作，可以延迟执行的任务，例如：
        *   预加载资源（图片、字体、脚本等）。
        *   数据上报和分析。
        *   后台同步数据。
        *   计算复杂但不需要立即显示结果的任务。
        *   非必要的 DOM 操作（例如，预渲染一些未来可能需要的元素）。

    *   **长列表优化：** 虚拟列表渲染时，分批次渲染列表项，避免一次性渲染大量 DOM 元素导致页面卡顿。
    *   **动画的优化：** 动画结束后执行的清理工作（例如，移除不再需要的元素或样式）。
    *  复杂计算：可以把任务分解到多个idle callback中执行

*   **为什么这么做（原因）：**
    *   **避免阻塞主线程：** 浏览器的主线程负责处理用户交互、页面渲染、JavaScript 执行等任务。如果长时间执行 JavaScript 代码，会导致主线程阻塞，页面无法响应用户操作，出现卡顿、无响应等问题。`requestIdleCallback` 将任务推迟到浏览器空闲时执行，避免了与关键任务竞争主线程资源。
    *   **提高页面性能：** 通过合理利用浏览器空闲时间，可以执行一些预加载、预计算等任务，减少用户等待时间，提高页面整体性能。
    *   **优化用户体验：** 避免因后台任务导致页面卡顿，保持页面流畅响应，提升用户体验。
    *   **充分利用浏览器资源：** 在浏览器空闲时执行任务，充分利用了浏览器资源，提高了资源利用率。

**总结：**

这段代码通过 `TaskQueue` 和 `requestIdleCallback` 实现了一个高效的任务调度机制。它将非关键任务推迟到浏览器空闲时执行，避免了阻塞主线程，提高了页面性能和用户体验。`requestIdleCallback` 的 polyfill 确保了在不支持该 API 的浏览器中也能实现类似的功能。
