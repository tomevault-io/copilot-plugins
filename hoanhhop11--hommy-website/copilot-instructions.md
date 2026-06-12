## main

> - **Ngôn ngữ phản hồi**: AI agent phải luôn phản hồi bằng tiếng Việt (vi-VN) trong mọi tương tác, trừ khi người dùng yêu cầu ngôn ngữ khác cho một trường hợp cụ thể.

## 🗣️ 1. Communication Rules

- **Ngôn ngữ phản hồi**: AI agent phải luôn phản hồi bằng tiếng Việt (vi-VN) trong mọi tương tác, trừ khi người dùng yêu cầu ngôn ngữ khác cho một trường hợp cụ thể.

## 🏗️ 2. Code Organization Rules

### Models - Tách theo tính năng

**Nguyên tắc:**
- Mỗi model chỉ quản lý 1 domain entity chính
- File > 500 dòng → **BẮT BUỘC** tách thành nhiều files
- Methods không liên quan đến domain hiện tại → tách file mới

**Ví dụ tách ChuDuAnModel.js (1648 dòng):**

❌ **SAI**: Gom tất cả trong `ChuDuAnModel.js`
```javascript
class ChuDuAnModel {
  // Tin đăng methods
  static async layDanhSachTinDang() {}
  static async taoTinDang() {}
  
  // Dự án methods
  static async layDanhSachDuAn() {}
  static async taoDuAn() {}
  
  // Cuộc hẹn methods
  static async layDanhSachCuocHen() {}
  
  // Báo cáo methods
  static async layBaoCaoHieuSuat() {}
}
```

✅ **ĐÚNG**: Tách thành nhiều files
```
server/models/
├── TinDangModel.js           # Chỉ methods về Tin đăng
├── DuAnModel.js             # Chỉ methods về Dự án
├── CuocHenModel.js          # Chỉ methods về Cuộc hẹn
├── BaoCaoHieuSuatModel.js   # Chỉ methods về Báo cáo
└── ChinhSachCocModel.js     # Chỉ methods về Chính sách cọc (đã có)
```

**Cấu trúc model chuẩn:**
```javascript
/**
 * Model cho [Tên Domain]
 * Quản lý [Mô tả domain]
 */
const db = require('../config/db');

/**
 * @typedef {Object} EntityName
 * @property {number} EntityID
 * @property {string} PropertyName
 */

class DomainModel {
  /**
   * [Mô tả method]
   * @param {number} id ID của entity
   * @param {Object} filters Bộ lọc
   * @returns {Promise<Array>}
   */
  static async methodName(id, filters = {}) {
    try {
      // Implementation
    } catch (error) {
      throw new Error(`Lỗi [mô tả]: ${error.message}`);
    }
  }
}

module.exports = DomainModel;
```

### Controllers - Tách theo tính năng

**Nguyên tắc tương tự Models:**
- Mỗi controller chỉ xử lý 1 domain entity
- File > 500 dòng → **BẮT BUỘC** tách
- Tách theo tính năng, không theo technical layer

**Ví dụ:**
```
server/controllers/
├── TinDangController.js      # Chỉ xử lý Tin đăng
├── DuAnController.js         # Chỉ xử lý Dự án
├── CuocHenController.js      # Chỉ xử lý Cuộc hẹn
└── BaoCaoHieuSuatController.js # Chỉ xử lý Báo cáo
```

### Routes - Nhóm theo domain

**Cấu trúc:**
```
server/routes/
├── tinDangRoutes.js          # Routes cho Tin đăng
├── duAnRoutes.js             # Routes cho Dự án
├── cuocHenRoutes.js          # Routes cho Cuộc hẹn
└── baoCaoRoutes.js           # Routes cho Báo cáo
```

**Pattern chuẩn:**
```javascript
const express = require('express');
const router = express.Router();
const DomainController = require('../controllers/DomainController');
const { authenticate, authorize } = require('../middleware/auth');

router.get('/', authenticate, DomainController.list);
router.get('/:id', authenticate, DomainController.getById);
router.post('/', authenticate, authorize(['role']), DomainController.create);

module.exports = router;
```

### Components - Mỗi component có folder riêng

**Cấu trúc:**
```
client/src/components/
├── ComponentName/
│   ├── ComponentName.jsx     # Component chính
│   ├── ComponentName.css     # Styles BEM
│   └── index.js              # Barrel export (optional)
```

**Pattern:**
```jsx
// ComponentName.jsx
import './ComponentName.css';

export default function ComponentName({ prop1, prop2 }) {
  return (
    <div className="component-name">
      <div className="component-name__header">...</div>
      <div className="component-name__body">...</div>
    </div>
  );
}
```

---

## 🎨 3. CSS Rules - BEM Naming Convention

### BEM Methodology - BẮT BUỘC

**Cấu trúc:** `block__element--modifier`

- **Block**: Component chính (ví dụ: `modal-duan`, `button`)
- **Element**: Phần tử con (ví dụ: `modal-duan__header`, `button__icon`)
- **Modifier**: Trạng thái/biến thể (ví dụ: `modal-duan--open`, `button--primary`)

### Ví dụ đúng/sai

**✅ ĐÚNG:**
```css
/* Block */
.modal-duan {}

/* Element */
.modal-duan__overlay {}
.modal-duan__container {}
.modal-duan__header {}
.modal-duan__title {}
.modal-duan__close {}

/* Modifier */
.modal-duan--open {}
.modal-duan--large {}
.modal-duan__close--disabled {}
```

**❌ SAI:**
```css
/* Không dùng nested selectors phức tạp */
.modal-duan .overlay {}  /* ❌ */
.modal-duan > .header {} /* ❌ */

/* Không dùng camelCase */
.modalDuan {}  /* ❌ */
.tieuDe {}     /* ❌ */

/* Không dùng kebab-case không BEM */
.modal-duan-overlay {}  /* ❌ Phải là modal-duan__overlay */
.header {}              /* ❌ Thiếu block name */
```

### Quy tắc BEM

1. **Block name**: Dùng tên component (tiếng Việt không dấu, lowercase, hyphen-separated)
   ```css
   /* Component: ModalTaoDuAn.jsx */
   .modal-tao-duan {}
   
   /* Component: QuanLyTinDang.jsx */
   .quan-ly-tin-dang {}
   ```

2. **Element**: Dùng `__` (double underscore)
   ```css
   .modal-tao-duan__header {}
   .modal-tao-duan__body {}
   .modal-tao-duan__footer {}
   ```

3. **Modifier**: Dùng `--` (double hyphen)
   ```css
   .modal-tao-duan--open {}
   .modal-tao-duan--disabled {}
   .modal-tao-duan__button--primary {}
   ```

4. **Nested elements**: KHÔNG dùng, mỗi element phải độc lập
   ```css
   /* ❌ SAI */
   .modal-duan__header__title {}
   
   /* ✅ ĐÚNG */
   .modal-duan__header {}
   .modal-duan__title {}
   ```

5. **Mixins**: Có thể kết hợp classes
   ```jsx
   <div className="modal-duan button--primary">
     {/* Sử dụng cả 2 blocks */}
   </div>
   ```

### Design Tokens

**Đặt trong `:root` hoặc file design system:**
```css
:root {
  /* Colors */
  --color-primary: #8b5cf6;
  --color-secondary: #64748b;
  
  /* Spacing */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  
  /* Typography */
  --font-size-base: 1rem;
  --font-weight-bold: 700;
}
```

**Sử dụng trong BEM:**
```css
.modal-duan__title {
  color: var(--color-primary);
  font-size: var(--font-size-base);
  font-weight: var(--font-weight-bold);
}
```

---

## 💻 4. JavaScript/Node.js Rules

### Naming Conventions

**Variables & Functions:** `camelCase`
```javascript
const userName = 'John';
function layDanhSachTinDang() {}
```

**Classes & Components:** `PascalCase`
```javascript
class TinDangModel {}
function ModalTaoDuAn() {}
```

**Constants:** `UPPER_SNAKE_CASE`
```javascript
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.example.com';
```

**Private methods:** Prefix với `_` (nếu cần)
```javascript
class Model {
  static async publicMethod() {}
  static async _privateMethod() {}
}
```

### Import Organization

**Thứ tự imports:**
1. External libraries (React, Express, etc.)
2. Internal modules (services, utils, config)
3. Relative imports (components, types)

```javascript
// 1. External
import React, { useState, useEffect } from 'react';
import express from 'express';

// 2. Internal
import { db } from '../config/db';
import { authenticate } from '../middleware/auth';

// 3. Relative
import ComponentName from './ComponentName';
```

### Error Handling

**BẮT BUỘC** có try-catch cho async operations:
```javascript
static async methodName() {
  try {
    // Implementation
    return result;
  } catch (error) {
    // Log error với context
    console.error(`[ModelName] Error in methodName:`, error);
    throw new Error(`Lỗi [mô tả cụ thể]: ${error.message}`);
  }
}
```

### JSDoc Documentation

**BẮT BUỘC** cho public methods:
```javascript
/**
 * Lấy danh sách tin đăng theo bộ lọc
 * @param {number} chuDuAnId ID của chủ dự án
 * @param {Object} filters Bộ lọc tìm kiếm
 * @param {string} [filters.trangThai] Trạng thái tin đăng
 * @param {string} [filters.keyword] Từ khóa tìm kiếm
 * @param {number} [filters.limit] Giới hạn số lượng
 * @returns {Promise<Array<TinDang>>} Danh sách tin đăng
 * @throws {Error} Nếu có lỗi xảy ra
 */
static async layDanhSachTinDang(chuDuAnId, filters = {}) {
  // Implementation
}
```

### Type Definitions

**Sử dụng JSDoc `@typedef`:**
```javascript
/**
 * @typedef {Object} TinDang
 * @property {number} TinDangID
 * @property {string} TieuDe
 * @property {string} MoTa
 * @property {number} Gia
 * @property {string} TrangThai - Nhap|ChoDuyet|DaDuyet|DaDang
 */
```

---

## 🔍 5. Best Practices Integration (Context7)

### Tra cứu Best Practices

**Trước khi implement patterns mới, BẮT BUỘC tra cứu:**

1. **Node.js Best Practices:**
   - Library: `/goldbergyoni/nodebestpractices`
   - Topics: code organization, file structure, separation of concerns

2. **BEM Methodology:**
   - Library: `/bem/bem-react`
   - Topics: BEM naming convention, component structure

3. **React Patterns:**
   - Library: `/websites/nodejs_api` hoặc React official docs
   - Topics: component patterns, hooks, state management

**Workflow:**
```markdown
1. Xác định pattern cần implement
2. Tra cứu Context7 với library/topic phù hợp
3. Áp dụng best practices từ kết quả tra cứu
4. Đảm bảo tuân thủ patterns đã học
```

### Áp dụng Patterns

**Code Organization:**
- ✅ Component-based structure (theo domain, không theo technical layer)
- ✅ 3-layer architecture: entry-points (routes/controllers), domain (services), data-access (models)
- ✅ Separation of concerns: mỗi file chỉ làm 1 việc

**File Structure:**
```
component-a/
├── entry-points/    # controllers, routes
├── domain/          # services, business logic
└── data-access/    # models, database
```

**Module Interface:**
- ✅ Sử dụng `index.js` để export public interface
- ✅ Không require trực tiếp từ sub-files

---

## 🔄 6. Workflow Rules

### Trước khi tạo file mới

**BƯỚC 1: Tìm kiếm**
```bash
# Tìm file tương tự trong codebase
- Tìm theo tên tính năng
- Tìm theo pattern tương tự
- Kiểm tra imports hiện có
```

**BƯỚC 2: Đánh giá**
```markdown
- File hiện có có thể extend không?
- Có thể refactor để tái sử dụng không?
- Có thể tạo shared utility không?
- File hiện có có quá lớn (>500 dòng) không?
```

**BƯỚC 3: Quyết định**
```markdown
✅ Nếu có thể tái sử dụng: Refactor/extend file hiện có
✅ Nếu không thể: Tạo file mới với naming convention chuẩn
❌ KHÔNG tạo file mới nếu đã có file tương tự có thể dùng được
```

### Trước khi thêm method vào model/controller lớn

**Đánh giá:**
1. Method này thuộc domain nào?
2. File hiện tại đã > 500 dòng chưa?
3. Method có liên quan đến domain hiện tại không?

**Quyết định:**
- ✅ Nếu file < 500 dòng và method cùng domain → Thêm vào file hiện tại
- ✅ Nếu file > 500 dòng → Tách file mới
- ✅ Nếu method khác domain → Tách file mới

**Ví dụ:**
```javascript
// ❌ SAI: Thêm method về TinDang vào DuAnModel.js
class DuAnModel {
  static async layDanhSachDuAn() {}
  static async layDanhSachTinDang() {} // ❌ Khác domain
}

// ✅ ĐÚNG: Tách thành TinDangModel.js
// DuAnModel.js chỉ giữ methods về Dự án
// TinDangModel.js chỉ giữ methods về Tin đăng
```

### Refactoring Strategy

**Khi tách file lớn:**
1. Xác định các nhóm methods theo domain
2. Tạo files mới cho từng domain
3. Di chuyển methods sang files tương ứng
4. Cập nhật imports trong controllers/routes
5. Xóa file cũ sau khi đã migrate xong

**Kiểm tra sau khi tách:**
- ✅ Tất cả imports đã được cập nhật
- ✅ Không có circular dependencies
- ✅ Tests vẫn pass (nếu có)
- ✅ Không có breaking changes

---

## ✅ 7. Testing & Quality Rules

### Error Handling

**BẮT BUỘC có error handling:**
```javascript
// ✅ ĐÚNG
try {
  const result = await db.execute(query, params);
  return result;
} catch (error) {
  console.error(`[Context] Error:`, error);
  throw new Error(`Lỗi [mô tả]: ${error.message}`);
}

// ❌ SAI: Không có error handling
const result = await db.execute(query, params);
return result;
```

### Validation

**Validate input parameters:**
```javascript
static async methodName(id, data) {
  if (!id || typeof id !== 'number') {
    throw new Error('ID không hợp lệ');
  }
  
  if (!data || typeof data !== 'object') {
    throw new Error('Dữ liệu không hợp lệ');
  }
  
  // Implementation
}
```

### Business Logic

**Tuân thủ nghiêm ngặt `docs/use-cases-v1.2.md`:**
- ✅ Đọc use case liên quan trước khi implement
- ✅ Tuân thủ state transitions
- ✅ Tuân thủ validation rules
- ✅ Tuân thủ audit logging requirements

**Ví dụ từ use cases:**
```javascript
// UC-PROJ-01: Đăng tin Cho thuê
// - Tiền điều kiện: ChuDuAn đã KYC hoặc cho phép tạo trước KYC
// - Ràng buộc: Bắt buộc có ít nhất 1 ảnh
// - Hậu điều kiện: Tạo TinĐăng ở trạng thái ChoDuyet

static async taoTinDang(chuDuAnId, tinDangData) {
  // Kiểm tra KYC (theo use case)
  // Validate ảnh (theo use case)
  // Tạo với trạng thái ChoDuyet (theo use case)
}
```

### Audit Logging

**Ghi log cho các hành động quan trọng:**
```javascript
// Sử dụng NhatKyHeThongService
const NhatKyHeThongService = require('../services/NhatKyHeThongService');

async function importantAction(userId, action, data) {
  // Business logic
  
  // Audit log
  await NhatKyHeThongService.ghiNhatKy({
    NguoiDungID: userId,
    HanhDong: action,
    DoiTuong: 'EntityName',
    DoiTuongID: entityId,
    GiaTriTruoc: oldValue,
    GiaTriSau: newValue
  });
}
```

---

## 📚 8. Reference Documents

**BẮT BUỘC tham chiếu:**
- `docs/use-cases-v1.2.md` - Đặc tả use cases và business rules
- `docs/CHU_DU_AN_ACTUAL_STATUS_2025.md` - Trạng thái hiện tại
- `client/src/pages/ChuDuAn/README.md` - Frontend guidelines

**Best Practices Libraries:**
- `/goldbergyoni/nodebestpractices` - Node.js best practices
- `/bem/bem-react` - BEM methodology
- `/janishar/nodejs-backend-architecture-typescript` - Backend architecture

---

## 🚨 9. Anti-Patterns - TRÁNH

### ❌ Không làm

1. **Gom nhiều tính năng trong 1 file**
   ```javascript
   // ❌ SAI: ChuDuAnModel.js có 29 methods về nhiều domain
   ```

2. **Tạo file mới mà không kiểm tra file hiện có**
   ```javascript
   // ❌ SAI: Tạo TinDangController.js mới khi đã có trong ChuDuAnController.js
   ```

3. **Không dùng BEM cho CSS**
   ```css
   /* ❌ SAI */
   .modal-duan-overlay {}
   .header {}
   ```

4. **File quá lớn không tách**
   ```javascript
   // ❌ SAI: File > 500 dòng không tách
   ```

5. **Không có error handling**
   ```javascript
   // ❌ SAI: Không có try-catch
   const result = await db.execute(query);
   ```

6. **Không có JSDoc cho public methods**
   ```javascript
   // ❌ SAI: Không có documentation
   static async methodName() {}
   ```

---

## 📝 10. Checklist trước khi commit

**Trước khi commit code, đảm bảo:**

- [ ] File mới đã được kiểm tra xem có thể tái sử dụng không
- [ ] CSS đã tuân thủ BEM naming convention
- [ ] Models/Controllers đã được tách theo tính năng (không > 500 dòng)
- [ ] Error handling đã được thêm vào async operations
- [ ] JSDoc đã được thêm cho public methods
- [ ] Business logic tuân thủ `docs/use-cases-v1.2.md`
- [ ] Audit logging đã được thêm cho các hành động quan trọng
- [ ] Imports đã được tổ chức đúng thứ tự
- [ ] Không có circular dependencies
- [ ] Code đã được test (nếu có)

---

## 🔄 11. Migration Strategy

### Migrating existing code

**CSS Migration to BEM:**
1. Xác định block name từ component name
2. Đổi tên classes theo BEM pattern
3. Cập nhật JSX sử dụng classes mới
4. Test UI không bị break

**Model/Controller Splitting:**
1. Xác định các nhóm methods theo domain
2. Tạo files mới cho từng domain
3. Di chuyển methods sang files tương ứng
4. Cập nhật imports
5. Test APIs vẫn hoạt động

---

**Lưu ý:** File này được Cursor IDE đọc tự động. Các rules sẽ được áp dụng cho mọi AI agent interactions. Cập nhật file này khi có thay đổi quy tắc mới.

![1762774520729](image/main/1762774520729.png)![1762774529038](image/main/1762774529038.png)![1762774541120](image/main/1762774541120.png)

---
> Source: [HoanhHop11/Hommy-Website](https://github.com/HoanhHop11/Hommy-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
