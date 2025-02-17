<img width="477" alt="image" src="https://github.com/user-attachments/assets/4cb0d5a0-d473-43f4-b0ae-1089e5a71ee8" />
<img width="936" alt="image" src="https://github.com/user-attachments/assets/4dedee49-6fc1-4ee2-b611-36838c6a6583" />



### 一、组件功能概述
这是一个2D编辑器左侧面板的项目管理组件，主要功能包括：
1. 展示用户项目列表（支持分页加载）
2. 项目缩略图展示与管理
3. 项目选择/切换/删除操作
4. 多选批量删除功能
5. 与后端API的数据同步
6. 本地缓存管理(DataCache)
7. 响应式布局（Masonry vs Flex）

### 二、核心功能实现分析

#### 1. 数据获取与缓存
```typescript
const getProjectListData = () => {
  const request = { /* 分页参数 */ }
  setLoading(true);
  getProjectList(request).then((res) => {
    // 处理分页逻辑
    DataCache.getInstance().setCacheItem('project', ...)
  })
}
```
- 使用分页请求(PAGE_SIZE=20)
- 通过DataCache实现本地缓存
- 滚动加载更多（ScrollMoreView2d组件），Masonary实现瀑布流布局

#### 2. 项目操作逻辑
```typescript
const clickChangeProject = (project_id: string) => {
  ProjectManager.getInstance().changeProject(project_id, (ret) => {
    // 更新URL状态
    history.replaceState(...)
  })
}

const handleDeleteSelected = () => {
  deleteProjectList({ project_ids }).then(() => {
    // 过滤已删除项目
    setProjectList(newProjectList)
  })
}
```
- 项目切换通过ProjectManager处理
- 批量删除使用API请求
- 同步更新URL参数

#### 3. 状态管理
```typescript
const [selectedItems, setSelectedItems] = useState<string[]>([]);
const [selectAll, setSelectAll] = useState(false);
const [isEditing, setIsEditing] = useState(false);
```
- 使用多个状态管理选择状态
- 派生状态计算：
  ```typescript
  useEffect(() => {
    setSelectAll(selectedItems.length === projectList.length - 1)
  }, [selectedItems])
  ```

#### 4. UI交互
```jsx
<CommonImage src={imagerUrl} />
<CustomCheckbox checked={selectedItems.includes(project_id)} />
<Popover>
  <div onClick={handleDelete}>删除</div>
</Popover>
```
- 使用自定义图片组件优化图片加载
- 实现hover状态显示复选框
- Popover实现右键菜单操作

### 三、关键代码结构

#### 1. 组件参数
```typescript
interface Props {
  allRight_state: { state: boolean }; // 布局状态
}
```

#### 2. 核心Hook
```typescript
const {
  useCanvasEditor,      // 编辑器上下文
  useProjectData,       // 项目数据
  useDataCache,         // 缓存管理
  useEvent,             // 事件总线
  useDispatch           // Redux
} = props;
```

#### 3. 事件处理
```typescript
useEffect(() => {
  event.on('EventUpdateProjectThumbnail', updateThumbnail);
  return () => event.off('EventUpdateProjectThumbnail');
}, [])
```




### 六、关键流程图解
```
组件挂载
  → 初始化数据（优先读取缓存）
  → 订阅事件（缩略图更新等）
  → 渲染项目列表

用户滚动
  → 触发加载更多
  → API请求新数据
  → 更新缓存和状态

用户操作项目：
  - 单击：切换当前项目
  - 多选：进入编辑模式
  - 删除：API请求 → 更新列表
  - 右键菜单：弹出操作选项
```

### 七、总结
该组件是一个典型的CRUD列表管理组件，主要特点包括：
- 结合本地缓存的分页加载
- 复杂的交互状态管理
- 响应式布局支持
- 与编辑器核心的深度集成

