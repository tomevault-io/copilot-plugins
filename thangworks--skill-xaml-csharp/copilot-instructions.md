## skill-xaml-csharp

> Áp dụng các rules này cho **mọi file XAML** trong project (WPF / UWP / MAUI).

# XAML Design Rules — GitHub Copilot Instructions

Áp dụng các rules này cho **mọi file XAML** trong project (WPF / UWP / MAUI).

---

## Core Mental Model

```
Chiều DỌC  → StackPanel (hoặc ItemsControl)
Chiều NGANG → Grid với ColumnDefinitions
```

**Không bao giờ** dùng cả `RowDefinitions` lẫn `ColumnDefinitions` trong cùng một Grid nếu có thể tránh được.  
Mỗi Grid chỉ nên giải quyết **một chiều**.

---

## Rule 1 — StackPanel thay thế RowDefinitions

❌ Sai — Grid vừa Row vừa Column:
```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="150"/>
        <ColumnDefinition Width="*"/>
    </Grid.ColumnDefinitions>
    <TextBlock Text="Name" />
    <TextBlock Grid.Column="1" Text="{Binding Name}" />
    <TextBlock Grid.Row="1" Text="Email" />
    <TextBlock Grid.Row="1" Grid.Column="1" Text="{Binding Email}" />
</Grid>
```

✅ Đúng — StackPanel + Grid thuần cột:
```xml
<StackPanel>
    <Grid Margin="0,0,0,8">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="150"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <TextBlock Text="Name" />
        <TextBlock Grid.Column="1" Text="{Binding Name}" />
    </Grid>
    <Grid Margin="0,0,0,8">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="150"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <TextBlock Text="Email" />
        <TextBlock Grid.Column="1" Text="{Binding Email}" />
    </Grid>
</StackPanel>
```

---

## Rule 2 — Không hard-code Row/Column index

❌ Sai — index rải rác, dễ vỡ khi thêm dòng:
```xml
<TextBlock Grid.Row="2" Grid.Column="1" Text="..." />
<Button    Grid.Row="2" Grid.Column="2" Content="Copy" />
```

✅ Đúng — mỗi Grid là một đơn vị độc lập, không cần index Row.

---

## Rule 3 — Pattern lặp ≥ 3 lần → ItemsControl + DataTemplate

Không copy-paste cùng một cấu trúc Label/Value/Action nhiều lần.  
Dùng `ItemsControl` với `DataTemplate`:

```xml
<ItemsControl ItemsSource="{Binding Fields}" Margin="10">
    <ItemsControl.ItemTemplate>
        <DataTemplate>
            <Grid Margin="0,0,0,8">
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="150"/>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="Auto"/>
                </Grid.ColumnDefinitions>
                <TextBlock Text="{Binding Label}" />
                <TextBlock Grid.Column="1" Text="{Binding Value}" TextWrapping="Wrap" />
                <Button Grid.Column="2" Content="Copy"
                        Command="{Binding CopyCommand}"
                        CommandParameter="{Binding Value}"
                        Padding="10,2" Margin="8,0,0,0"/>
            </Grid>
        </DataTemplate>
    </ItemsControl.ItemTemplate>
</ItemsControl>
```

---

## Rule 4 — SharedSizeGroup để căn cột đồng nhau

```xml
<StackPanel Grid.IsSharedSizeScope="True">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition SharedSizeGroup="Label"/>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition SharedSizeGroup="Action"/>
        </Grid.ColumnDefinitions>
        ...
    </Grid>
    <!-- Grid khác cùng group tự căn theo -->
</StackPanel>
```

`Grid.IsSharedSizeScope="True"` đặt trên **phần tử cha**, không phải từng Grid con.

---

## Rule 5 — UserControl cho pattern phức tạp

Khi một row có logic (tooltip, validation, visibility binding) và dùng ≥ 2 nơi:

```xml
<!-- FieldRow.xaml -->
<UserControl x:Class="App.Controls.FieldRow"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             x:Name="root">
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="150"/>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="Auto"/>
        </Grid.ColumnDefinitions>
        <TextBlock Text="{Binding Label, ElementName=root}" VerticalAlignment="Center"/>
        <TextBlock Grid.Column="1" Text="{Binding Value, ElementName=root}"
                   TextWrapping="Wrap" VerticalAlignment="Center"/>
        <Button Grid.Column="2" Content="Copy"
                Command="{Binding CopyCommand, ElementName=root}"
                CommandParameter="{Binding Value, ElementName=root}"
                Padding="10,2" Margin="8,0,0,0"/>
    </Grid>
</UserControl>
```

---

## Rule 6 — Thứ tự ưu tiên layout

```
1. StackPanel    → xếp chồng đơn giản, không cần căn cột
2. Grid (column) → căn nhiều cột trong một dòng
3. Grid (2D)     → lưới thực sự: calendar, dashboard tile, table
4. UniformGrid   → lưới đều, không cần binding
5. WrapPanel     → tag cloud, chip list
6. DockPanel     → shell layout (header / footer / sidebar)
```

---

## Rule 7 — Spacing tập trung ở container

❌ Sai — margin rải rác từng element:
```xml
<TextBlock Margin="0,0,8,8" />
<TextBlock Margin="0,0,0,8" />
<Button Margin="8,0,0,8" />
```

✅ Đúng — gom vào `ItemContainerStyle`:
```xml
<ItemsControl>
    <ItemsControl.ItemContainerStyle>
        <Style TargetType="ContentPresenter">
            <Setter Property="Margin" Value="0,0,0,8"/>
        </Style>
    </ItemsControl.ItemContainerStyle>
</ItemsControl>
```

---

## Rule 8 — Quy tắc DataGrid

- **Luôn tắt tạo cột tự động**: `AutoGenerateColumns="False"`.
- **Kết hợp Fix width và `*` width**:
  - Cột dữ liệu cố định (ID, Status, Date...) dùng width cứng hoặc `Auto`.
  - Cột nội dung dài (Name, Note, Description...) dùng width `*` để DataGrid luôn hiển thị full chiều ngang màn hình.

---

## Rule 9 — Khi dự án dùng MahApps.Metro

Nếu dự án có sự hiện diện của MahApps, kết hợp các tiện ích của nó:
- **TextBox**:
  - Thêm `Controls:TextBoxHelper.Watermark="..."` cho text.
  - Thêm `Controls:TextBoxHelper.ClearTextButton="True"` nếu cần thiết.
  - Thêm `Controls:TextBoxHelper.SelectAllOnFocus="True"` (Auto select khi focus) nếu hợp lý.
- **Icon / Menu**: Nếu có `MahApps.Metro.IconPacks` đi kèm, Menu chuột phải cho DataGrid cần thêm icon, button có thể dùng thêm icon hoặc `Style="{DynamicResource MahApps.Styles.Button.Circle}"` ở những vị trí hợp lý.
- **Dialog Box**: Thay thế `MessageBox.Show` mặc định bằng các dialog tích hợp sẵn của MahApps.

---

## Rule 10 — Fody / PropertyChanged cho MVVM

Kiểm tra project có sự hiện diện của `Fody/PropertyChanged` khi thiết kế MVVM không:
- Nếu có, chỉ cần có `public event PropertyChangedEventHandler PropertyChanged;` là đủ dùng cho nhiều trường hợp.

---

## Rule 11 — Ưu tiên d:DataContext cho UserControl

Để IDE (Visual Studio, Rider) có thể sử dụng tính năng **Go to Definition (F12)** trên các thuộc tính trong XAML, hãy khai báo `DataContext` (hoặc `d:DataContext` cho design-time) rõ ràng.

Cách làm đơn giản và dễ hiểu nhất:
```xml
<UserControl ...
             xmlns:vm="clr-namespace:YourApp.ViewModels"
             x:Name="root">
    <UserControl.DataContext>
        <vm:TestViewModel />
    </UserControl.DataContext>

    <!-- Lúc này binding sẽ hỗ trợ IntelliSense và F12 (Go to Definition) -->
    <TextBlock Text="{Binding PropertyInViewModel}" />
</UserControl>
```

---

## Checklist — áp dụng khi generate hoặc review XAML

- [ ] Grid dùng cả Row lẫn Column? → Tách StackPanel + Grid column
- [ ] Pattern lặp ≥ 3 lần? → ItemsControl + DataTemplate
- [ ] Row index hard-code? → Cảnh báo, refactor
- [ ] ColumnDefinitions copy-paste? → SharedSizeGroup hoặc UserControl
- [ ] Margin rải rác? → Gom vào ItemContainerStyle
- [ ] UserControl cần binding từ ngoài? → Tạo DependencyProperty, dùng ElementName
- [ ] DataGrid đã thiết lập AutoGenerateColumns="False" và mix fix/* width chưa?
- [ ] Nếu có MahApps, đã dùng TextBoxHelper, IconPacks và Dialog của MahApps chưa?
- [ ] Đã khai báo d:DataContext cho UserControl để hỗ trợ Go to Definition chưa?
- [ ] Đã tận dụng Fody/PropertyChanged đúng cách chưa?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thangworks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
