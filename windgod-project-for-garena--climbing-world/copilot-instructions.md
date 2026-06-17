## generate-widget

> Inspector 添加新组件智能提示


# Overview

# Inspector Widget 组件生成知识库

## 简介

本知识库用于指导AI和开发者自动生成Inspector Widget组件。所有新的组件添加流程都遵循相同的模式。

## 目录结构

```
docs/knowledge_base/
├── README.md                          # 本文件（包含所有生成规范）
├── generate-widget.js                 # 组件生成脚本（可选工具）
└── templates/                         # 代码模板目录
    ├── widget_vue_template.vue        # Vue组件模板
    └── widget_interface_template.ts   # TypeScript接口模板
```

## 组件结构

每个Inspector Widget组件由以下部分组成：

1. **Vue组件文件** (`{ComponentName}Editor.vue`)
   - 位置：`src/components/Inspector/widget/{ComponentName}/{ComponentName}Editor.vue`
   - 实现组件的UI和交互逻辑

2. **接口定义文件** (`interface.ts`)
   - 位置：`src/components/Inspector/widget/{ComponentName}/interface.ts`
   - 定义组件的TypeScript类型和接口

3. **注册到系统**
   - 在 `src/ts/common/property/Inspector.ts` 的 `CompName` 枚举中添加组件名
   - 在 `src/components/Inspector/widget/index.ts` 中导入和导出类型

## 生成步骤（AI和开发者通用）

### 步骤1: 理解需求

提取以下信息：
- **组件名称**（ComponentName）：必须是PascalCase格式，如 `MyWidget`
- **值类型**（ValueType）：如 `string`、`number`、`boolean`、`Array<string>` 等
- **组件功能**：了解组件的作用，以便实现正确的UI

### 步骤2: 创建组件目录和文件

**目录结构：**
```
src/components/Inspector/widget/{ComponentName}/
├── {ComponentName}Editor.vue
└── interface.ts
```

**文件1: {ComponentName}Editor.vue**
- 基于模板：`docs/knowledge_base/templates/widget_vue_template.vue`
- 替换变量：
  - `{{ComponentName}}` → 实际组件名（PascalCase）
  - `{{componentNameLower}}` → 组件名首字母小写（camelCase）
  - `{{ValueType}}` → 值类型

**文件2: interface.ts**
- 基于模板：`docs/knowledge_base/templates/widget_interface_template.ts`
- 替换相同的变量

### 步骤3: 更新注册文件

**文件3: Inspector.ts**
- 位置：`src/ts/common/property/Inspector.ts`
- 操作：在 `CompName` 枚举中添加新值
- 格式：`{ComponentName} = '{ComponentName}Editor',`

**文件4: widget/index.ts**
- 位置：`src/components/Inspector/widget/index.ts`
- 操作：
  1. 在文件顶部添加导入：`import type { {ComponentName} } from './{ComponentName}/interface';`
  2. 在 `Widget` 类型联合中添加：`| {ComponentName}`

### 步骤4: 实现组件逻辑

根据组件功能，在生成的Vue文件中实现：
- UI组件（使用项目中的组件库）
- 事件处理
- 数据绑定

## 标准组件模板

### Vue组件模板结构

参考 `templates/widget_vue_template.vue`，基本结构如下：

```vue
<template>
    <div class="inspector-property-{componentNameLower}">
        <!-- 根据组件功能实现UI -->
    </div>
</template>

<script lang="ts">
import InspectorItemChangeCommand from '@/ts/unredo/commands/InspectorItemChangeCommand';
import { Vue, Component, Prop, Emit } from 'vue-property-decorator';
import { I{ComponentName} } from './interface';
import { IWidgetProps } from '@/components/Inspector/widget/IWidgetProps';
import { ECustomBG } from '@/ts/common/property/Inspector';

@Component({
    name: '{ComponentName}Editor',
})
export default class {ComponentName}Editor extends Vue implements I{ComponentName}, IWidgetProps {
    @Prop() readonly value: {ValueType};
    @Prop({ default: (): boolean => false }) readonly disabled: boolean;
    @Prop() customBG: ECustomBG;

    @Emit('notify')
    notify(val: {ValueType}) {
        return { path: '', val, command: InspectorItemChangeCommand.Create() };
    }
}
</script>

<style lang="scss" scoped>
@import '@/styles/common.scss';
</style>
```

### Interface文件模板结构

参考 `templates/widget_interface_template.ts`，基本结构如下：

```typescript
import { CompName, ECustomBG } from '@/ts/common/property/Inspector';
import { IEditor, Component, Input, Output } from '../index';

export interface {ComponentName}Input extends Input<{ValueType}> {
    disabled?: boolean;
    customBG?: ECustomBG;
    // 根据组件需求添加其他自定义属性
}

export interface {ComponentName}Output extends Output {
    
}

export type I{ComponentName} = IEditor<{ComponentName}Input, {ComponentName}Output>;

export type {ComponentName} = Component<CompName.{ComponentName}, Omit<{ComponentName}Input, 'value'>>;

export const {ComponentName}Comparer = (val1: {ValueType}, val2: {ValueType}) => {
    if(val1 === undefined || val2 === undefined) {
        return '—';
    } else {
        return val1 === val2 ? val1 : '—';
    }
}
```

## 常见组件类型参考

### 字符串输入组件
- 值类型：`string`
- UI：使用 `<Input>` 组件
- 参考：`src/components/Inspector/widget/Input/InputEditor.vue`

### 数字输入组件
- 值类型：`number`
- UI：使用 `<InputNumber>` 组件
- 参考：`src/components/Inspector/widget/InputNumber/InputNumberEditor.vue`

### 布尔值组件
- 值类型：`boolean`
- UI：使用 `<checkbox>` 组件
- 参考：`src/components/Inspector/widget/Checkbox/CheckboxEditor.vue`

### 下拉选择组件
- 值类型：`string | number`
- UI：使用 `<Select>` 或自定义下拉组件
- 参考：`src/components/Inspector/widget/DropDown/DropDownEditor.vue`

### 列表组件
- 值类型：`Array<T>`
- UI：使用列表编辑器
- 参考：`src/components/Inspector/widget/List/ListEditor.vue`

## 命名规范

1. **组件名**：使用PascalCase，如 `MyComponent`
2. **文件名**：组件文件夹和文件使用PascalCase
3. **接口名**：使用 `I{ComponentName}` 格式
4. **类型名**：使用 `{ComponentName}Input`、`{ComponentName}Output` 格式

## 注意事项

1. **必须实现的接口**：
   - `IWidgetProps`：所有组件必须实现
   - `I{ComponentName}`：组件特定的接口

2. **必须实现的方法**：
   - `notify(val: ValueType)`：用于通知值变化
   - 返回格式：`{ path: '', val, command: InspectorItemChangeCommand.Create() }`

3. **多选支持**：
   - 实现 `{ComponentName}Comparer` 函数
   - 用于多选模式下比较值

4. **样式**：
   - 使用项目中的SCSS变量和mixins
   - 导入 `@/styles/common.scss`

5. **其他要求**：
   - 支持 `disabled` 和 `customBG` 属性
   - 使用 `InspectorItemChangeCommand.Create()` 创建命令对象

## 完整示例：生成 "MyWidget" 组件

假设要创建一个新的widget组件 "MyWidget"，用于编辑字符串值：

### 1. 创建目录
```
src/components/Inspector/widget/MyWidget/
```

### 2. 生成 MyWidgetEditor.vue

```vue
<template>
    <div class="inspector-property-myWidget">
        <Input
            :value="value"
            :disabled="disabled"
            @change="notify"
        />
    </div>
</template>

<script lang="ts">
import InspectorItemChangeCommand from '@/ts/unredo/commands/InspectorItemChangeCommand';
import { Vue, Component, Prop, Emit } from 'vue-property-decorator';
import { IMyWidget } from './interface';
import { IWidgetProps } from '@/components/Inspector/widget/IWidgetProps';
import { ECustomBG } from '@/ts/common/property/Inspector';

@Component({
    name: 'MyWidgetEditor',
})
export default class MyWidgetEditor extends Vue implements IMyWidget, IWidgetProps {
    @Prop() readonly value: string;
    @Prop({ default: (): boolean => false }) readonly disabled: boolean;
    @Prop() customBG: ECustomBG;

    @Emit('notify')
    notify(val: string) {
        return { path: '', val, command: InspectorItemChangeCommand.Create() };
    }
}
</script>

<style lang="scss" scoped>
@import '@/styles/common.scss';

.inspector-property-myWidget {
    // 组件样式
}
</style>
```

### 3. 生成 interface.ts

```typescript
import { CompName, ECustomBG } from '@/ts/common/property/Inspector';
import { IEditor, Component, Input, Output } from '../index';

export interface MyWidgetInput extends Input<string> {
    disabled?: boolean;
    customBG?: ECustomBG;
}

export interface MyWidgetOutput extends Output {
    
}

export type IMyWidget = IEditor<MyWidgetInput, MyWidgetOutput>;

export type MyWidget = Component<CompName.MyWidget, Omit<MyWidgetInput, 'value'>>;

export const MyWidgetComparer = (val1: string, val2: string) => {
    if(val1 === undefined || val2 === undefined) {
        return '—';
    } else {
        return val1 === val2 ? val1 : '—';
    }
}
```

### 4. 更新 Inspector.ts

在 `CompName` 枚举中添加：
```typescript
MyWidget = 'MyWidgetEditor',
```

### 5. 更新 widget/index.ts

添加导入：
```typescript
import type { MyWidget } from './MyWidget/interface';
```

在 `Widget` 类型中添加：
```typescript
export type Widget = (
    // ... 其他组件
    | MyWidget
);
```

## 验证清单

生成完成后，检查：

- [ ] 组件目录已创建
- [ ] Vue文件已生成并实现基本功能
- [ ] interface.ts文件已生成
- [ ] Inspector.ts中已添加枚举值
- [ ] widget/index.ts中已添加导入和类型导出
- [ ] 代码符合TypeScript规范
- [ ] 代码符合Vue规范
- [ ] 样式已正确导入

## 使用生成脚本（可选，仅限开发者）

知识库提供了生成脚本 `docs/knowledge_base/generate-widget.js`，可以快速生成基础文件：

```bash
node docs/knowledge_base/generate-widget.js MyWidget string
```

**注意：**
- 此脚本主要用于开发者快速生成基础文件框架
- **AI应该直接按照本知识库规范生成文件**，而不是使用脚本，以便更好地控制生成的内容和实现细节
- 脚本生成的文件需要手动完善UI逻辑和自定义属性

---
> Source: [WindGod-Project-For-Garena/Climbing-World](https://github.com/WindGod-Project-For-Garena/Climbing-World) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
