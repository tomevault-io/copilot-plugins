## hyprland-elite-desktop

> 这个文件提供Claude在工作时的全局指导原则和特殊功能。

# Claude全局配置

这个文件提供Claude在工作时的全局指导原则和特殊功能。

## 系统操作指导

### sudo权限操作
当需要执行需要sudo权限的操作时（如安装软件包、修改系统文件等）：

1. **不要直接执行sudo命令**
   - Claude不应该直接运行需要sudo的命令
   - 避免潜在的安全风险

2. **提供命令让用户执行**：
   - 给出完整的命令行指令
   - 解释命令的作用和目的
   - 让用户自己决定是否执行

3. **示例格式**：
   ```bash
   # 安装软件包
   sudo pacman -S package-name
   
   # 或者从AUR安装
   yay -S package-name
   ```

4. **后续处理**：
   - 提醒用户安装完成后通知Claude继续
   - 准备好后续的配置步骤

## 注意事项

- 保持专业和客观的态度
- 确保信息准确传递
- 避免重复无效的协商
- 优先用户需求和实际解决方案
- 涉及sudo操作时给出命令而非直接执行

---
> Source: [laofahai/hyprland-elite-desktop](https://github.com/laofahai/hyprland-elite-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
