## scm-frontend

> handleSearch() {

# SCM前端项目输入控件库规则

## 1. 基础输入控件

### 1.1 普通文本输入框
```vue
<!-- 无需import，Element UI原生组件 -->
<el-input
  v-model.trim="dataJson.tempJson.name"
  clearable
  placeholder="请输入"
  :maxlength="dataJson.inputSettings.maxLength.name"
  @keyup.enter.native="handleSearch"
/>
```
- **Import**: 无需import (Element UI)
- **绑定字段**: `dataJson.searchForm.*` 或 `dataJson.tempJson.*`
- **校验**: 通过 `el-form-item` 的 `prop` 属性
- **常用属性**: `clearable`, `placeholder`, `maxlength`, `@keyup.enter.native`

### 1.2 多行文本输入框
```vue
<!-- 无需import，Element UI原生组件 -->
<el-input
  v-model.trim="dataJson.tempJson.remark"
  type="textarea"
  clearable
  show-word-limit
  placeholder="请输入"
  :maxlength="500"
/>
```
- **Import**: 无需import (Element UI)
- **特殊属性**: `type="textarea"`, `show-word-limit`

## 2. 数字/金额输入控件

### 2.1 数字输入组件
```vue
<!-- Import -->
import numeric from '@/components/40_input/numeric'

<!-- 使用 -->
<numeric
  v-model="dataJson.tempJson.amount"
  :positive-percentage="true"
  :decimal-places="4"
  :minimum-value="0"
  :maximum-value="10000000000"
  style="width: 100%"
  @change.native="handleAmountChange"
/>
```
- **Import**: `import numeric from '@/components/40_input/numeric'`
- **Components注册**: `components: { numeric }`
- **主要属性**:
  - `positive-percentage`: 正百分比模式
  - `percentage`: 百分比模式
  - `decimal-places`: 小数位数
  - `currency-symbol`: 货币符号 (默认¥)
  - `minimum-value`: 最小值
  - `maximum-value`: 最大值

## 3. 字典选择控件

### 3.1 字典下拉选择框
```vue
<!-- Import -->
import SelectDict from '@/components/00_dict/select/SelectDict.vue'

<!-- 使用 -->
<select-dict
  v-model="dataJson.tempJson.type"
  :para="CONSTANTS.DICT_B_PROJECT_TYPE"
  init-placeholder="请选择"
  style="width:100%"
  @change="handleTypeChange"
/>
```
- **Import**: `import SelectDict from '@/components/00_dict/select/SelectDict.vue'`
- **Components注册**: `components: { SelectDict }`
- **必需属性**: `:para` (字典常量)
- **可选属性**: `init-placeholder`, `disabled`, `filter-para`

### 3.2 字典多选下拉框
```vue
<!-- Import -->
import SelectDicts from '@/components/00_dict/select/SelectDicts.vue'

<!-- 使用 -->
<select-dicts
  v-model="dataJson.searchForm.status_list"
  :para="CONSTANTS.DICT_B_PROJECT_STATUS"
  init-placeholder="请选择单据状态"
  :disabled="false"
/>
```
- **Import**: `import SelectDicts from '@/components/00_dict/select/SelectDicts.vue'`
- **Components注册**: `components: { SelectDicts }`
- **返回值**: 数组类型

### 3.3 字典单选按钮组
```vue
<!-- Import -->
import RadioDict from '@/components/00_dict/redio'

<!-- 使用 -->
<radio-dict
  v-model="dataJson.tempJson.type"
  :value="dataJson.tempJson.type"
  :para="CONSTANTS.DICT_B_PROJECT_TYPE"
  :disabled="false"
  @change="handleTypeChange"
/>
```
- **Import**: `import RadioDict from '@/components/00_dict/redio'`
- **Components注册**: `components: { RadioDict }`
- **注意**: 路径是 `redio` 不是 `radio`

## 4. 业务搜索选择控件

### 4.1 通用搜索输入框
```vue
<!-- Import -->
import InputSearch from '@/components/40_input/inputSearch'

<!-- 使用 -->
<input-search
  v-model.trim="dataJson.tempJson.supplier_name"
  :disabled="false"
  @onInputSearch="handleSupplierDialog"
/>
```
- **Import**: `import InputSearch from '@/components/40_input/inputSearch'`
- **Components注册**: `components: { InputSearch }`
- **事件**: `@onInputSearch` (点击选择按钮触发)

### 4.2 供应商选择控件
```vue
<!-- Import -->
import SelectCpSupplier from '@/components/xx/supplier/SelectCpSupplier.vue'

<!-- 使用 -->
<select-cp-supplier
  v-model.trim="dataJson.searchForm.supplier_name"
  :placeholder="isPlaceholderShow('请选择供应商')"
  placement="left"
  @keyup.enter.native="handleSearch"
  @onReturnData="handleSupplierReturnDataName"
/>
```
- **Import**: 根据实际路径 (业务专用组件)
- **事件**: `@onReturnData` (选择完成后触发)

### 4.3 客户选择控件
```vue
<!-- Import -->
import SelectSeCustomer from '@/components/xx/customer/SelectSeCustomer.vue'

<!-- 使用 -->
<select-se-customer
  v-model.trim="dataJson.searchForm.purchaser_name"
  :placeholder="isPlaceholderShow('请选择主体企业')"
  placement="left"
  @keyup.enter.native="handleSearch"
  @onReturnData="handleCustomerReturnDataName"
/>
```
- **Import**: 根据实际路径 (业务专用组件)

### 4.4 仓库选择控件
```vue
<!-- Import -->
import SelectWarehouse from '@/components/xx/warehouse/SelectWarehouse.vue'

<!-- 使用 -->
<select-warehouse
  v-model.trim="dataJson.searchForm.warehouse_name"
  placeholder="请选择仓库"
  placement="left"
  @keyup.enter.native="handleSearch"
  @onReturnData="handleWarehouseReturnDataName"
/>
```
- **Import**: 根据实际路径 (业务专用组件)

## 5. 日期时间控件

### 5.1 日期选择器
```vue
<!-- 无需import，Element UI原生组件 -->
<el-date-picker
  v-model="dataJson.tempJson.plan_time"
  value-format="yyyy-MM-dd"
  type="date"
  clearable
  placeholder="选择日期"
  style="width: 100%"
/>
```
- **Import**: 无需import (Element UI)
- **格式**: `value-format="yyyy-MM-dd"`

### 5.2 日期时间选择器
```vue
<!-- 无需import，Element UI原生组件 -->
<el-date-picker
  v-model="dataJson.tempJson.sign_date"
  value-format="yyyy-MM-dd HH:mm:ss"
  type="datetime"
  clearable
  placeholder="选择日期"
  style="width: 100%"
/>
```
- **Import**: 无需import (Element UI)
- **格式**: `value-format="yyyy-MM-dd HH:mm:ss"`
- **类型**: `type="datetime"`

### 5.3 日期范围选择器
```vue
<!-- 无需import，Element UI原生组件 -->
<el-date-picker
  v-model="dataJson.searchForm.plan_times"
  type="datetimerange"
  range-separator="至"
  start-placeholder="计划时间开始"
  end-placeholder="计划时间结束"
  format="yyyy-MM-dd HH:mm:ss"
  value-format="yyyy-MM-dd HH:mm:ss"
/>
```
- **Import**: 无需import (Element UI)
- **类型**: `type="datetimerange"`
- **返回值**: 数组 `[startTime, endTime]`

## 6. 文件上传控件

### 6.1 多文件上传组件
```vue
<!-- Import -->
import SimpleUploadMutilFile from '@/components/10_file/SimpleUploadMutilFile/index.vue'

<!-- 使用 -->
<Simple-upload-mutil-file
  :accept="'*'"
  @upload-success="handleOtherUploadFileSuccess"
  @upload-error="handleFileError"
/>
```
- **Import**: `import SimpleUploadMutilFile from '@/components/10_file/SimpleUploadMutilFile/index.vue'`
- **Components注册**: `components: { SimpleUploadMutilFile }`
- **事件**: `@upload-success`, `@upload-error`

### 6.2 单文件上传组件
```vue
<!-- Import -->
import SimpleUpload from '@/components/10_file/SimpleUpload/index.vue'

<!-- 使用 -->
<SimpleUpload />
```
- **Import**: `import SimpleUpload from '@/components/10_file/SimpleUpload/index.vue'`
- **Components注册**: `components: { SimpleUpload }`

## 7. 预览组件

### 7.1 文件预览卡片
```vue
<!-- Import -->
import PreviewCard from '@/components/50_preview_card/preview_card.vue'

<!-- 使用 -->
<previewCard
  :file-name="item.fileName"
  :url="item.url"
  :time="item.timestamp"
  @removeFile="removeOtherFile"
/>
```
- **Import**: `import PreviewCard from '@/components/50_preview_card/preview_card.vue'`
- **Components注册**: `components: { PreviewCard }`

### 7.2 预览描述组件
```vue
<!-- Import -->
import PreviewDescription from '@/components/51_preview_description/index.vue'

<!-- 使用 -->
<PreviewDescription
  v-if="dataJson.doc_att_files && dataJson.doc_att_files.length > 0"
  :attachment-files="dataJson.doc_att_files"
/>
```
- **Import**: `import PreviewDescription from '@/components/51_preview_description/index.vue'`
- **Components注册**: `components: { PreviewDescription }`

## 8. 表单校验规则

### 8.1 表单声明模板
```vue
<el-form
  ref="dataSubmitForm"
  :rules="settings.rules"
  :model="dataJson.tempJson"
  label-position="right"
  label-width="150px"
  status-icon
>
```

### 8.2 表单项校验模板
```vue
<el-form-item
  prop="name"
  label-width="0"
>
  <div slot="label" class="required-mark">字段名称</div>
  <!-- 输入控件 -->
</el-form-item>
```

### 8.3 校验规则定义模板
```javascript
settings: {
  rules: {
    name: [
      { required: true, message: '请输入名称', trigger: 'blur' }
    ],
    type: [
      { required: true, message: '请选择类型', trigger: 'change' }
    ],
    amount: [
      { required: true, message: '请输入金额', trigger: 'blur' },
      { type: 'number', message: '金额必须为数字', trigger: 'blur' }
    ]
  }
}
```

## 9. 数据绑定规范

### 9.1 数据结构模板
```javascript
dataJson: {
  // 查询表单数据
  searchForm: {
    code: '',
    name: '',
    type: '',
    status_list: [],
    supplier_name: '',
    purchaser_name: ''
  },
  // 编辑表单数据
  tempJson: {
    id: null,
    code: '',
    name: '',
    type: '',
    remark: '',
    // 附件相关
    doc_att: [],
    doc_att_files: []
  },
  // 输入限制设置
  inputSettings: {
    maxLength: {
      name: 100,
      code: 50,
      remark: 500
    }
  }
}
```

### 9.2 常用方法模板
```javascript
methods: {
  // 通用查询方法
  handleSearch() {
    // 查询逻辑
  },
  
  // 字典变更处理
  handleTypeChange(val) {
    this.dataJson.tempJson.type = val
    // 其他逻辑
  },
  
  // 搜索选择处理
  handleSupplierDialog() {
    // 打开供应商选择对话框
  },
  
  // 文件上传成功处理
  handleOtherUploadFileSuccess(data) {
    this.dataJson.doc_att.push(data)
  },
  
  // 文件删除处理
  removeOtherFile(data) {
    const index = this.dataJson.doc_att.findIndex(item => item.fileName === data.fileName)
    if (index !== -1) {
      this.dataJson.doc_att.splice(index, 1)
    }
  }
}
```

## 10. 常量使用规范

### 10.1 字典常量使用
```javascript
// 在Vue中直接使用，无需import
this.CONSTANTS.DICT_B_PROJECT_TYPE
this.CONSTANTS.DICT_B_PROJECT_STATUS
this.CONSTANTS.DICT_B_PO_CONTRACT_TYPE
```

### 10.2 格式化函数使用
```javascript
// 在Vue中直接使用，无需import
this.formatCurrency(amount, true) // 金额格式化
this.formatNumber(number, true, 4) // 数字格式化
this.showLoading('正在处理，请稍后...') // 显示loading
this.closeLoading() // 关闭loading
```

## 11. 组件导入模板

### 11.1 完整导入示例
```javascript
// 输入组件
import InputSearch from '@/components/40_input/inputSearch'
import numeric from '@/components/40_input/numeric'

// 字典组件
import SelectDict from '@/components/00_dict/select/SelectDict.vue'
import SelectDicts from '@/components/00_dict/select/SelectDicts.vue'
import RadioDict from '@/components/00_dict/redio'

// 文件组件
import SimpleUploadMutilFile from '@/components/10_file/SimpleUploadMutilFile/index.vue'
import PreviewCard from '@/components/50_preview_card/preview_card.vue'
import PreviewDescription from '@/components/51_preview_description/index.vue'

// 业务组件 (根据实际情况调整路径)
import SupplierDialog from '@/views/xx/supplier/dialog/index.vue'
import CustomerDialog from '@/views/xx/customer/dialog/index.vue'

export default {
  components: {
    InputSearch,
    numeric,
    SelectDict,
    SelectDicts,
    RadioDict,
    SimpleUploadMutilFile,
    PreviewCard,
    PreviewDescription,
    SupplierDialog,
    CustomerDialog
  }
}
```

## 12. 页面通用Watch

### 12.1 Loading状态监听
```javascript
watch: {
  // 全屏loading监听 (每个页面都要添加，弹出页面除外)
  'settings.loading': {
    handler (newVal, oldVal) {
      switch (newVal) {
        case true:
          this.showLoading('正在处理，请稍后...')
          break
        case false:
          this.closeLoading()
          break
      }
    }
  }
}
```

## 13. 作废页面特殊处理

### 13.1 作废对话框调用
```javascript
// 在作废API调用的finally中调用
cancelApi(tempData).finally(() => {
  this.$emit('closeMeOk', tempData)
})
```

## 14. 审批流组件注意事项

### 14.1 BPM组件使用
```vue
<!-- 注意：是bpm不是bmp -->
<bpm-dialog />
```

### 14.2 审批页面结构
- **左边**: 复制该模块的查看页
- **右边**: 审批流 (bpm组件)
- **下边**: 审批操作区域 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zxhbloomer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
