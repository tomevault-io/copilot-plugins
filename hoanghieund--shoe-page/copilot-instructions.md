## shoe-page

> Thêm component vào components/ui


# Hướng dẫn sử dụng shadcn/ui components

## Danh sách các component đã cài đặt

Dự án đã cài đặt các component shadcn/ui sau:

1. `button` - Nút bấm với nhiều biến thể
2. `sheet` - Panel trượt từ cạnh màn hình
3. `separator` - Đường phân cách
4. `scroll-area` - Khu vực cuộn có thanh cuộn tùy chỉnh
5. `input` - Trường nhập liệu
6. `label` - Nhãn cho form controls
7. `avatar` - Hiển thị ảnh đại diện người dùng
8. `toast` - Thông báo tạm thời
9. `select` - Dropdown cho phép chọn một giá trị
10. `dropdown-menu` - Menu xổ xuống
11. `tabs` - Tabs để chuyển đổi giữa các nội dung khác nhau

## Quy tắc sử dụng

### 1. Import components

Luôn import components từ đường dẫn `@/components/ui`:

```tsx
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
```

### 2. Tùy chỉnh style

Sử dụng `className` để tùy chỉnh style và `cn()` từ `@/lib/utils` để kết hợp các class:

```tsx
import { cn } from "@/lib/utils";

<Button className={cn("bg-primary text-white", isActive && "bg-secondary")}>
  Nút bấm
</Button>;
```

### 3. Variants và sizes

Sử dụng các variants và sizes có sẵn thay vì tạo mới:

```tsx
<Button variant="outline" size="sm">Nút nhỏ</Button>
<Button variant="destructive">Xóa</Button>
```

### 4. Form components

Khi sử dụng các form components, luôn kết hợp với `Label` để tăng tính truy cập:

```tsx
<div className="space-y-2">
  <Label htmlFor="email">Email</Label>
  <Input id="email" type="email" placeholder="Email của bạn" />
</div>
```

### 5. Toast notifications

Sử dụng hook `useToast` để hiển thị thông báo:

```tsx
import { useToast } from "@/hooks/use-toast";

const { toast } = useToast();

toast({
  title: "Thành công",
  description: "Đã thêm sản phẩm vào giỏ hàng",
  variant: "success",
});
```

### 6. Dropdown và Select

- Sử dụng `DropdownMenu` cho menu hành động
- Sử dụng `Select` cho việc chọn một giá trị từ danh sách

### 7. Sheet

Sử dụng `Sheet` cho các panel trượt từ cạnh màn hình như giỏ hàng, menu mobile:

```tsx
<Sheet>
  <SheetTrigger asChild>
    <Button variant="outline">Mở</Button>
  </SheetTrigger>
  <SheetContent>
    <SheetHeader>
      <SheetTitle>Tiêu đề</SheetTitle>
    </SheetHeader>
    <div>Nội dung</div>
  </SheetContent>
</Sheet>
```

### 8. Tabs

Sử dụng `Tabs` để chuyển đổi giữa các nội dung liên quan:

```tsx
<Tabs defaultValue="description">
  <TabsList>
    <TabsTrigger value="description">Mô tả</TabsTrigger>
    <TabsTrigger value="reviews">Đánh giá</TabsTrigger>
  </TabsList>
  <TabsContent value="description">Nội dung mô tả</TabsContent>
  <TabsContent value="reviews">Nội dung đánh giá</TabsContent>
</Tabs>
```

## Kết hợp với KokonutUI

Khi kết hợp shadcn/ui với KokonutUI:

1. Ưu tiên sử dụng các component từ KokonutUI cho các thành phần UI chính
2. Sử dụng shadcn/ui cho các form controls và các component cơ bản
3. Sử dụng `LiquidGlassCard` từ KokonutUI để bọc các form và card

```tsx
import { LiquidGlassCard } from "@/components/kokonutui/liquid-glass-card";
import { Button } from "@/components/ui/button";

<LiquidGlassCard>
  <form>
    {/* Form controls từ shadcn/ui */}
    <Button>Gửi</Button>
  </form>
</LiquidGlassCard>;
```

## Cài đặt component mới

Để cài đặt thêm component mới từ shadcn/ui:

```bash
pnpm dlx shadcn@latest add [component-name]
```

Ví dụ:

```bash
pnpm dlx shadcn@latest add dialog
```

Sau khi cài đặt, cập nhật file này để thêm component mới vào danh sách.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hoanghieund) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
