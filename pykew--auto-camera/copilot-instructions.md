## auto-camera

> 相机定位至拍照位置的功能实现指南

# 相机定位功能

## 概述

移动到拍照位是指控制XY轴移动到预设的拍照位置，确保图像在视野中居中。这个位置可以来自两个来源：

1. PLC传递的坐标点位
2. 手动绑定XY轴后调整的点位

无论来源如何，最终目标都是确保被拍摄物体在相机视野中居中，提高图像质量和后续处理效果。

## 功能实现要点

### 1. 坐标点位的获取和存储

```javascript
// PLC传递的点位示例
function receivePlcCoordinates(data) {
  const { x, y } = parseCoordinates(data);
  savePhotoPosition(x, y, 'plc');
}

// 手动调整的点位示例
function saveManualPosition() {
  const currentPos = getCurrentAxisPosition();
  savePhotoPosition(currentPos.x, currentPos.y, 'manual');
}

// 存储点位
function savePhotoPosition(x, y, source) {
  const position = {
    x: parseFloat(x),
    y: parseFloat(y),
    source: source,
    timestamp: Date.now()
  };
  
  // 存储到本地或数据库
  storePosition(position);
}
```

### 2. 移动到拍照位功能

```javascript
// 移动到拍照位
async function moveToPhotoPosition(positionId) {
  try {
    // 获取目标位置
    const position = getStoredPosition(positionId);
    
    // 显示正在移动的状态
    updateUIStatus('moving');
    
    // 发送移动指令给控制器
    await sendMoveCommand(position.x, position.y);
    
    // 等待移动完成
    await waitForMovementComplete();
    
    // 更新UI状态
    updateUIStatus('positioned');
    
    return true;
  } catch (error) {
    handleMovementError(error);
    return false;
  }
}
```

### 3. 位置调整模拟

为了模拟相机移动到拍照位的效果，可以实现以下功能：

```javascript
// 模拟相机位置调整效果
function simulateCameraPositioning(startPos, targetPos) {
  // 创建模拟视图
  const simulationView = createSimulationView();
  
  // 获取真实相机当前图像
  const currentImage = getCameraImage();
  
  // 计算起始位置和目标位置的偏移
  const offset = {
    x: targetPos.x - startPos.x,
    y: targetPos.y - startPos.y
  };
  
  // 根据偏移量模拟图像位置变化
  animateImageOffset(simulationView, currentImage, offset, {
    duration: 1500, // 毫秒
    onComplete: () => {
      showPositioningResult(simulationView);
    }
  });
}
```

### 4. 多个拍照位的管理

```javascript
// 多拍照位管理示例
const photoPositions = {
  add: function(name, x, y, source) {
    // 添加新拍照位
  },
  remove: function(positionId) {
    // 删除拍照位
  },
  update: function(positionId, newX, newY) {
    // 更新拍照位坐标
  },
  list: function() {
    // 列出所有拍照位
    return storedPositions;
  }
};
```

## 用户界面设计建议

1. 提供可视化的坐标调整界面
2. 显示当前位置和目标位置之间的偏差
3. 提供实时预览，模拟调整后的效果
4. 保存历史位置记录，便于快速切换

## 模拟实现建议

模拟相机定位效果应当实现以下功能：

1. 显示当前相机视图
2. 使用动画展示从当前位置到目标位置的移动过程
3. 预览调整后的图像位置效果
4. 提供虚拟控制器供用户交互测试

这种模拟既可以用于开发测试，也可以作为用户培训工具，帮助操作人员理解系统行为。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PykeW) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
