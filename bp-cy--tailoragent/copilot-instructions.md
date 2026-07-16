## tailoragent

> 一个 Windows 桌面应用骨架：**Spring Boot（Java 21）后端** + **Vue 3 SPA**前端，通过 **JCEF** 在嵌入式 Chromium 窗口中显示。整个应用打包为单个 fat jar（或 jpackage `.msi`/app-image），完全离线运行 —— Chromium native 文件已捆绑在 jar 中，无需下载。

## 这是什么

一个 Windows 桌面应用骨架：**Spring Boot（Java 21）后端** + **Vue 3 SPA**前端，通过 **JCEF** 在嵌入式 Chromium 窗口中显示。整个应用打包为单个 fat jar（或 jpackage `.msi`/app-image），完全离线运行 —— Chromium native 文件已捆绑在 jar 中，无需下载。
## 架构

**启动顺序和前后端边界**是不太直观的部分 —— 需要同时阅读以下几个文件才能理解：

- `TailorAgentApplication.main()` —— 并行启动：JCEF 初始化（后台线程）与 Spring Boot（主线程）同时进行，启动耗时 ≈ max(Spring, JCEF) 而非两者之和。`setHeadless(false)`（必须，否则 AWT/JCEF 无法创建窗口）。后端绑定 `server.port=0`（由操作系统分配空闲端口，仅回环地址 `127.0.0.1`）。启动后从 `local.server.port` 读取实际端口。两者都就绪后创建浏览器窗口。
- `desktop/BrowserWindow` —— 创建一个 Swing `JFrame`，其中包含指向 `http://127.0.0.1:<port>/` 的 `CefBrowser`。窗口关闭手动处理：先关闭 Spring 上下文，再 `CefApp.dispose()`。
- `desktop/JcefSetup` —— 初始化 JCEF。`resolveInstallDir()` 很关键：打包运行时（通过 `jpackage.app-path` 系统属性检测）会将 native 文件解压到 `%LOCALAPPDATA%\TailorAgent\jcef-bundle`（可写目录，因为应用可能安装在只读的 `Program Files` 下）；开发时使用 `./jcef-bundle`。在 `TERMINATED` 状态下调用 `System.exit(0)` 以终止所有 Chromium 辅助进程。
- **前端 → 后端分发**：`frontend/`（Vite）构建时设置 `outDir: ../src/main/resources/static` 且 `emptyOutDir: true`，因此构建产物直接落入后端的静态资源目录并被打包进 jar。Spring Boot 提供 SPA 服务；生产环境下同源，前端使用相对路径调用 `/api/...`。
- **Maven 构建会运行前端构建**：通过 `frontend-maven-plugin`（安装 node/npm、`npm install`、`npm run build`）在 `generate-resources` 阶段执行。因此 `mvnw package` 即可生成完整 jar，无需单独的前端构建步骤。

### 开发 vs. 打包路由

- **打包/jar 运行**：前后端在随机端口上同源。相对路径 `/api` 请求直接到达后端。
- **开发模式**：后端运行在固定端口 8080（`application-dev.yml`，`profiles=dev`）；Vite 开发服务器运行在 5173，并将 `/api` 代理到 `127.0.0.1:8080`（参见 `vite.config.ts`）。作为两个独立进程运行。
- Vue Router 使用 **hash 历史模式**（`createWebHashHistory`），以便在 WebView 中深层链接/刷新无需后端回退路由即可正常工作。

## 常用命令

使用 Maven Wrapper `./mvnw`（bash）或 `mvnw.cmd`（cmd），也可直接用 `mvn`（已在 PATH）。

```bash
# 完整构建（同时构建前端） → target/TailorAgent-0.0.1-SNAPSHOT.jar
mvn clean package

# AOT 优化构建（Spring AOT 编译，启动快 40-60%）
mvn clean package -Paot

# 仅后端迭代 —— 跳过 node/npm/前端构建
mvn clean package -Dfrontend.skip=true

# 运行桌面应用（启动 JCEF 窗口）
mvn spring-boot:run          # 或：java -jar target/TailorAgent-0.0.1-SNAPSHOT.jar

# 开发模式后端，固定端口 8080（配合下方 Vite 开发服务器使用）
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# 测试
mvn test
mvn test -Dtest=TailorAgentApplicationTests#contextLoads   # 单个测试
```

```bash
# 前端开发服务器（Vite，端口 :5173，/api 代理到 :8080）
cd frontend && npm run dev
cd frontend && npm run build     # vue-tsc 类型检查 + vite 构建 → ../src/main/resources/static
```

```bat
REM Windows 原生打包（在 cmd 中运行，需要 JDK 21 且已设置 JAVA_HOME）
package.bat            REM .msi 安装包 —— 需要 WiX Toolset v3 在 PATH 中
package.bat app-image  REM 便携文件夹，无需 WiX
```

## 约定与注意事项

- **注释和 UI 文字使用中文。** 修改已有文件时请保持一致。
- **`pom.xml` 中的版本号是有意锁定的**：`commons-lang3` 为修复 CVE 而覆盖了版本；`jcef.natives.version` 必须与 `jcefmaven` 中嵌入的 `cef-*` 版本完全匹配（命名规则不同 —— 直接复制完整字符串）。请勿随意升级这些版本。
- **中国网络镜像是有意配置的**：node 从华为云镜像下载（`pom.xml`），npm registry 使用 npmmirror（`frontend/.npmrc`）。除非在其他地区工作，否则请保留。
- `src/main/resources/static/` 是**生成的**（已 gitignore）—— 切勿手动编辑；修改 `frontend/` 后重新构建。
- **修改前端后自动构建**：每次完成任务时，如果本次会话中有修改过 `frontend/` 下的任何文件，在任务结束后自动执行 `cd frontend && npm run build`，确保 `src/main/resources/static/` 与前端源码保持同步。
- 本项目仅支持 Windows：native 依赖为 `jcef-natives-windows-amd64`，打包方式为 `jpackage` + WiX。
- Java 包名为 `com.changy.tailoragent`（注意：Maven `groupId` 为 `com.Changy`）。

---
> Source: [BP-cy/TailorAgent](https://github.com/BP-cy/TailorAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
