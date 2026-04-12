## liuyao-desktop-tauri2

> * @Author: mwlt_sanodia mwlt@163.com

<!--
 * @Author: mwlt_sanodia mwlt@163.com
 * @Date: 2025-06-25 23:59:29
 * @LastEditors: mwlt_sanodia mwlt@163.com
 * @LastEditTime: 2025-06-26 00:00:08
 * @FilePath: \liuyao_desktop_tauri\.cursor\rules\cc.mdc
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
# Cursor Project Rule for cc方案

1. 前端所有网络请求（iframe、fetch、XHR、图片、视频等）必须自动走本地 Rust 代理服务器，无需前端特殊处理。
2. 禁止大幅度修改现有前端 UI 结构，仅允许在 ProxySettingsPopover、Settings、NavBar 等组件基础上增强功能。
3. 代理服务器必须支持智能分流（如 core333.com 直连，其他走系统代理），并暴露配置接口供前端调用。
4. Tauri WebView 启动参数必须设置 `--proxy-server=127.0.0.1:端口`，确保全流量可控。
5. 必须兼容 Windows、Linux、macOS，macOS 下如无法自动代理需有明确用户提示。
6. 代理配置、状态、系统代理信息等必须在前端 UI 直观展示，并支持一键测试连接。
7. 代码实现需注重安全性，禁止本地代理监听非 127.0.0.1。

8. 任何涉及证书信任、系统代理等敏感操作，需有详细用户指引和异常处理。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mwlt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mwlt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
