## project-rules

> ======================================

======================================
JJYB_AI智剪 项目规则
请全选复制此文件内容，粘贴到 Cursor 设置 > Project Rules
======================================

# JJYB_AI智剪 项目规则

## 项目概述
JJYB_AI智剪 是一个本地优先的 AI 视频剪辑应用，集成多种配音、字幕、混剪和项目管理能力。

**当前实际技术栈**：
- 后端: Python 3.x, Flask, Flask-SocketIO, 原生 sqlite3 + 自定义 DatabaseManager
- 前端: Flask 模板 + HTML5/CSS3/JavaScript, Layui, jQuery, Socket.IO
- 数据库: SQLite（统一使用 `database/jjyb_ai.db`）
- AI模型:
  - TTS: Edge-TTS、gTTS、Azure TTS、本地 Voice Clone
  - ASR: Whisper / faster-whisper
  - 场景检测: PySceneDetect + 内置分析逻辑
- 视频处理: FFmpeg（主引擎）+ 部分 OpenCV/辅助工具

## 项目结构
```
JJYB_AI智剪/
├── frontend/               # 前端与应用入口
│   ├── app.py             # Flask 应用入口 + SocketIO
│   ├── templates/         # 页面模板（编辑器/项目/素材/配音/解说/混剪/设置等）
│   └── static/            # 前端静态资源
├── backend/                # 后端增强模块
│   ├── api/               # 业务 API（项目/素材/Remix/字幕/导出/测试等）
│   ├── services/          # 服务层（TaskService、RemixService、VoiceoverService 等）
│   ├── engine/            # 引擎层（VoiceCloneEngine、VideoProcessor 等）
│   └── config/            # AI/路径等配置
├── 开发文档/               # 完整开发文档（架构/实现说明）
├── .cursor/                # Cursor 项目规则与辅助文件
├── database/               # SQLite 数据文件
├── logs/                   # 运行日志
├── resource/               # 第三方资源（如 ImageMagick 等）
├── tests/                  # 测试脚本
├── requirements.txt        # Python 依赖
└── 启动应用.bat             # Windows 启动入口
```

## 开发工作流 (6A)
1. **Align**: 明确需求和技术方案
2. **Architect**: 设计实现方案和架构
3. **Atomize**: 拆解为可执行小任务 (`.tasks/YYYY-MM-DD_N_<task>.md`)
4. **Approve**: 自我审查计划完整性
5. **Automate**: 按计划实施开发
6. **Assess**: 验收功能并总结

配合 RIPER-5 五模式工作流：RESEARCH → INNOVATE → PLAN → EXECUTE → REVIEW

## Python代码规范
- 遵循 PEP 8
- 强制使用类型注解
- 类名大驼峰，函数/变量小写+下划线
- 完整的文档字符串 (Args, Returns, Raises)
- 明确的异常处理和日志记录
- 使用上下文管理器管理资源

示例：
```python
from typing import Optional, List
from pathlib import Path

class ProjectService:
    """项目管理服务"""
    
    def create_project(self, name: str, description: str = "") -> Project:
        """创建新项目
        
        Args:
            name: 项目名称
            description: 项目描述
            
        Returns:
            Project: 创建的项目对象
            
        Raises:
            ValueError: 名称为空或已存在
        """
        if not name:
            raise ValueError("项目名称不能为空")
        # 实现逻辑...
```

## Flask API规范
- 按功能模块组织路由 (`backend/api/`)
- 使用Blueprint组织API
- 统一响应格式：`{code, success, message, data}`
- 完整的API文档注释 (Query Parameters, Request Body, Returns)
- WebSocket用于实时进度推送

## 前端开发规范 (Layui)
- 使用Layui标准布局结构
- 统一封装API请求处理
- WebSocket实时通信
- 模块化组织JavaScript代码

## 数据库操作规范
- 使用自定义 `DatabaseManager` + sqlite3 进行持久化
- 后端 API 通过服务层/管理器访问数据库，不在视图中直接写 SQL
- 统一解析/序列化 JSON 字段（如 config、input_data、output_data）
- 需要时为热点查询添加索引，保持查询性能
- 操作失败必须回滚事务并记录日志

## AI模型集成规范
- 尽量延迟加载重量级模型（Whisper/大模型等），按需初始化
- 通过配置集中管理模型开关与 API Key（参见 backend/config/ai_config.py）
- 统一封装 TTS/ASR/场景检测接口，前端只关心业务含义
- 所有推理过程必须有详细日志和异常处理

TTS 引擎（当前实际实现）：
- Edge-TTS: 在线免费，通用中文推荐
- gTTS: 轻量多语言
- Azure TTS: 高质量，需用户自行配置 Key
- Voice Clone: 本地语音克隆，可配置模型与可执行文件

ASR 引擎：
- Whisper / faster-whisper：采用后端配置的模型，统一通过字幕 API 调用

场景检测与智能剪辑：
- 基于 PySceneDetect + 自有逻辑实现 `/api/ai/smart-clip*` 系列接口

## FFmpeg视频处理规范
- 封装FFmpegEngine类统一操作
- 支持视频信息获取、剪辑、合并、音频处理
- 区分快速复制和重新编码模式
- 完整的错误处理和进度回调

## 异步任务处理
- 使用TaskQueue管理异步任务
- ThreadPoolExecutor处理耗时操作
- 实时进度通过WebSocket推送
- 任务状态追踪 (pending/running/completed/failed)

## Git提交规范
分支命名：`task/[功能]_YYYY-MM-DD_N`

提交格式：`<type>(<scope>): <description>`
- feat: 新功能
- fix: Bug修复
- docs: 文档更新
- refactor: 代码重构
- perf: 性能优化
- test: 测试相关
- chore: 构建工具变动

示例：
```bash
feat(tts): 添加Edge TTS引擎支持
fix(video): 修复视频导出编码错误
docs(api): 更新API接口文档
```

## 测试规范
- 使用pytest编写单元测试
- 测试文件：`tests/test_*.py`
- 完整的测试覆盖关键功能
- Mock外部依赖

## 日志规范
- 使用统一的logger工具
- 日志轮换 (10MB)
- 格式：时间 - 模块 - 级别 - 消息
- 关键操作必须记录日志

## 安全规范
- 敏感配置使用环境变量
- 所有用户输入必须验证
- 使用Pydantic验证请求数据
- SQL参数化查询防注入
- 完整的错误处理，避免信息泄露

## 性能优化
- 异步处理耗时任务
- 数据库查询优化（索引、批量操作）
- GPU加速AI模型推理
- 资源及时释放

## 完整文档位置
- 详细项目规则: `.cursor/rules/project-rules.md`
- AI视频处理规范: `.cursor/rules/ai-video-standards.md`
- 配置指南: `.cursor/RULES_SETUP_GUIDE.md`
- 快速开始: `.cursor/QUICK_START.md`
- 开发文档: `开发文档/` 目录

---

**版本**: 2.0.0  
**更新**: 2025-11-08  
**维护**: JJYB_AI智剪开发团队

---
> Source: [jianjieyiban/JJYB_AI_VideoAutoCut](https://github.com/jianjieyiban/JJYB_AI_VideoAutoCut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
