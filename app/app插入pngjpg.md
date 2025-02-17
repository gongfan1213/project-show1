### **添加图片素材到 Fabric 画布**

素材的获取是在 Flutter 端完成，将素材添加到画布时，会通过 Flutter 到 Fabric 画布的数据传递过程。

---

#### **Fabric 画布插入 PNG/JPG 图片**

**流程：**
1. **Flutter 端选择图片**：用户在 Flutter 端选择需要插入的图片。
2. **图片转换为 Base64 格式**：将图片数据转换为 Base64 格式，便于传输。
3. **通过 JS 传输到 Fabric 画布**：使用 JS Bridge 将图片数据传递到 Fabric 画布。

**流程图：**
```
Flutter 端选择图片 → 图片转换成 Base64 → 通过 JS 传输 → Fabric 画布
```

---

#### **具体实现代码**
```javascript
// 计算画布缩放比例，按正常图片大小进行计算
imageSaveData.current[guid()] = url;

fabric.Image.fromURL(url, function (oImg: any) {
    let scale = CommonTools.getInsertImageScale(oImg.width, oImg.height, width, height);
    console.log("[fabric] insertElement scale = ", scale, oImg.width * scale);

    oImg.scale(1).set({
        left: (width - oImg.width * scale) / 2.0,  // 图片水平居中
        top: (height - oImg.height * scale) / 2.0, // 图片垂直居中
        angle: 0,                                  // 图片角度
        scaleX: scale,                             // X 轴缩放比例
        scaleY: scale,                             // Y 轴缩放比例
    });

    canvasRef.current.add(oImg).setActiveObject(oImg); // 添加图片到画布并设置为活动对象
});
```

---

### **总结**
- **核心流程**：
  1. Flutter 端选择图片。
  2. 图片数据转换为 Base64 格式。
  3. 通过 JS Bridge 将图片传递到 Fabric 画布。
  4. 在画布中插入图片并设置缩放、居中等属性。

- **关键点**：
  - 计算图片的缩放比例，确保图片适配画布大小。
  - 图片插入后设置为活动对象，便于后续操作。
