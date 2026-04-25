## sync-message-v2

> 1. **必须使用Material UI组件库**

# UI开发规范

## Material UI使用规范

### 强制要求
1. **必须使用Material UI组件库**
   - 所有UI开发必须使用Material UI组件
   - 禁止使用原生HTML元素替代Material UI组件
   - 必须从 `@mui/material` 统一导入

### 导入规范
```javascript
// ✅ 正确 - 统一从@mui/material导入
import { 
  Button, 
  TextField, 
  Dialog, 
  DialogTitle, 
  DialogContent,
  DialogActions,
  Typography,
  Box,
  Paper,
  Table,
  TableBody,
  TableCell,
  TableContainer,
  TableHead,
  TableRow,
  IconButton,
  Chip,
  Alert,
  Snackbar
} from '@mui/material';

// ❌ 错误 - 不要分别导入
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';
```

### 常用组件使用规范

#### 布局组件
```javascript
// 容器组件
import { Box, Container, Grid, Paper } from '@mui/material';

// 使用Box进行布局
<Box sx={{ p: 2, m: 1 }}>
  <Typography variant="h6">标题</Typography>
</Box>

// 使用Grid进行响应式布局
<Grid container spacing={2}>
  <Grid item xs={12} md={6}>
    <Paper sx={{ p: 2 }}>内容</Paper>
  </Grid>
</Grid>
```

#### 表单组件
```javascript
// 表单输入
<TextField
  label="用户名"
  variant="outlined"
  fullWidth
  required
  error={!!errors.username}
  helperText={errors.username}
  value={formData.username}
  onChange={(e) => setFormData({...formData, username: e.target.value})}
/>

// 按钮
<Button 
  variant="contained" 
  color="primary"
  onClick={handleSubmit}
  disabled={loading}
>
  {loading ? '提交中...' : '提交'}
</Button>
```

#### 表格组件
```javascript
// 数据表格
<TableContainer component={Paper}>
  <Table>
    <TableHead>
      <TableRow>
        <TableCell>名称</TableCell>
        <TableCell>状态</TableCell>
        <TableCell>操作</TableCell>
      </TableRow>
    </TableHead>
    <TableBody>
      {data.map((row) => (
        <TableRow key={row.id}>
          <TableCell>{row.name}</TableCell>
          <TableCell>
            <Chip 
              label={row.status} 
              color={row.status === 'active' ? 'success' : 'default'}
            />
          </TableCell>
          <TableCell>
            <IconButton onClick={() => handleEdit(row.id)}>
              <EditIcon />
            </IconButton>
          </TableCell>
        </TableRow>
      ))}
    </TableBody>
  </Table>
</TableContainer>
```

#### 对话框组件
```javascript
// 确认对话框 - 使用现有组件
import ConfirmDialog from '@/components/ConfirmDialog';

<ConfirmDialog
  open={open}
  title="确认删除"
  content="确定要删除这个项目吗？"
  onConfirm={handleDelete}
  onCancel={() => setOpen(false)}
/>
```

### 样式规范

#### 使用sx属性
```javascript
// ✅ 推荐 - 使用sx属性
<Box sx={{ 
  p: 2,           // padding: theme.spacing(2)
  m: 1,           // margin: theme.spacing(1)
  borderRadius: 1, // border-radius: theme.shape.borderRadius
  bgcolor: 'background.paper'
}}>

// ❌ 避免 - 内联样式
<Box style={{ padding: '16px', margin: '8px' }}>
```

#### 主题颜色使用
```javascript
// 使用主题颜色
<Button color="primary">主要按钮</Button>
<Button color="secondary">次要按钮</Button>
<Button color="error">错误按钮</Button>
<Button color="warning">警告按钮</Button>
<Button color="info">信息按钮</Button>
<Button color="success">成功按钮</Button>
```

### 响应式设计

#### 断点使用
```javascript
// 响应式布局
<Grid container spacing={2}>
  <Grid item xs={12} sm={6} md={4} lg={3}>
    <Paper sx={{ p: 2 }}>内容</Paper>
  </Grid>
</Grid>

// 响应式显示
<Box sx={{ 
  display: { xs: 'none', md: 'block' }  // 移动端隐藏，桌面端显示
}}>
  桌面端内容
</Box>
```

### 状态管理

#### 加载状态
```javascript
// 按钮加载状态
<Button 
  variant="contained"
  disabled={loading}
  startIcon={loading ? <CircularProgress size={20} /> : null}
>
  {loading ? '处理中...' : '提交'}
</Button>

// 页面加载状态
{loading ? (
  <Box sx={{ display: 'flex', justifyContent: 'center', p: 4 }}>
    <CircularProgress />
  </Box>
) : (
  <div>内容</div>
)}
```

#### 错误处理
```javascript
// 错误提示
<Alert severity="error" sx={{ mb: 2 }}>
  {errorMessage}
</Alert>

// 成功提示
<Snackbar 
  open={showSuccess} 
  autoHideDuration={6000} 
  onClose={() => setShowSuccess(false)}
>
  <Alert severity="success">操作成功！</Alert>
</Snackbar>
```

### 组件开发规范

#### 公共组件位置
- 所有可复用组件放在 [src/components/](mdc:src/components) 目录
- 当前已有组件：[ConfirmDialog.jsx](mdc:src/components/ConfirmDialog.jsx)

#### 组件开发原则
1. **向后兼容**：修改公共组件时考虑现有使用场景
2. **避免破坏性变更**：如需重大修改应创建新版本组件
3. **测试覆盖**：修改后需测试所有引用该组件的页面

#### 组件示例
```javascript
// 通用确认对话框组件
import { Dialog, DialogTitle, DialogContent, DialogActions, Button } from '@mui/material';

export default function ConfirmDialog({ 
  open, 
  title, 
  content, 
  onConfirm, 
  onCancel,
  confirmText = '确认',
  cancelText = '取消'
}) {
  return (
    <Dialog open={open} onClose={onCancel}>
      <DialogTitle>{title}</DialogTitle>
      <DialogContent>{content}</DialogContent>
      <DialogActions>
        <Button onClick={onCancel}>{cancelText}</Button>
        <Button onClick={onConfirm} color="primary" variant="contained">
          {confirmText}
        </Button>
      </DialogActions>
    </Dialog>
  );
}
```

### 性能优化

#### 组件懒加载
```javascript
// 页面级组件懒加载
import dynamic from 'next/dynamic';

const LazyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <CircularProgress />
});
```

#### 列表优化
```javascript
// 虚拟滚动（大数据列表）
import { FixedSizeList as List } from 'react-window';

<List
  height={400}
  itemCount={1000}
  itemSize={35}
>
  {({ index, style }) => (
    <div style={style}>
      行 {index}
    </div>
  )}
</List>
```

### 无障碍访问

#### 语义化标签
```javascript
// 使用语义化的aria标签
<Button
  aria-label="删除项目"
  aria-describedby="delete-description"
>
  删除
</Button>
<div id="delete-description">点击删除当前选中的项目</div>
```

#### 键盘导航
```javascript
// 支持键盘导航
<IconButton
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      handleClick();
    }
  }}
>
  <DeleteIcon />
</IconButton>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yaotutu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
