## jobcatcher

> 1. **目录错误**: 终端在 `project` 目录，但backend在 `project/JobCatcher/backend`

# Service Startup Rules - 服务启动规则

## 重要提醒 - Critical Reminders

### 🚨 常见错误避免 - Common Mistakes to Avoid
1. **目录错误**: 终端在 `project` 目录，但backend在 `project/JobCatcher/backend`
2. **端口检查忽略**: 启动前必须检查端口占用情况
3. **虚拟环境未激活**: 必须先激活虚拟环境
4. **PYTHONPATH未设置**: 环境变量设置错误
5. **⚠️ 最关键错误**: 不在backend目录下执行uvicorn命令导致ModuleNotFoundError

### 📍 正确目录结构 - Correct Directory Structure
```
project/                    # 终端当前位置
└── JobCatcher/
    ├── backend/            # 后端服务目录 (正确的工作目录)
    │   ├── app/
    │   ├── requirements.txt
    │   └── main.py
    └── frontend/           # 前端服务目录
        ├── index.html
        ├── css/
        ├── js/
        └── assets/
```

---

# 🔧 后端服务启动 - Backend Service Startup

## ✅ 后端正确启动流程 - Backend Startup Process

### 1. 检查端口占用 - Check Port Usage
```bash
# 检查8000端口占用
netstat -tulpn | grep :8000
# 或者
lsof -i :8000

# 如果端口被占用，杀死进程
kill -9 <PID>
# 或者强制杀死所有占用8000端口的进程
sudo fuser -k 8000/tcp
# 或者杀死所有uvicorn进程
pkill -f uvicorn
```

### 2. 进入正确目录 - Navigate to Correct Directory
```bash
# 从project目录开始，必须进入backend目录
cd JobCatcher/backend
pwd  # 确认当前在 /home/devbox/project/JobCatcher/backend
```

### 3. 激活虚拟环境 - Activate Virtual Environment
```bash
# 激活虚拟环境 (路径相对于project根目录)
source ../../bin/activate
# 确认虚拟环境已激活
which python
which pip
```

### 4. 设置PYTHONPATH - Set PYTHONPATH
```bash
# 设置正确的PYTHONPATH
export PYTHONPATH=/home/devbox/project/JobCatcher/backend:$PYTHONPATH
echo $PYTHONPATH  # 确认设置正确
```

### 5. 测试模块导入 - Test Module Import
```bash
# 在backend目录下测试模块导入
python -c "from app.main import app; print('✅ 应用模块导入成功')"
```

### 6. 启动后端服务 - Start Backend Service
```bash
# ⚠️ 关键：必须在backend目录下执行此命令
# 当前目录必须是 /home/devbox/project/JobCatcher/backend
pwd  # 再次确认目录
cd JobCathcer/backend & python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

---

# 🌐 前端服务启动 - Frontend Service Startup

## ✅ 前端正确启动流程 - Frontend Startup Process

### 1. 检查端口占用 - Check Port Usage
```bash
# 检查7860端口占用（前端默认端口）
netstat -tulpn | grep :7860
# 或者
lsof -i :7860

# 如果端口被占用，杀死进程
kill -9 <PID>
# 或者强制杀死所有占用7860端口的进程
sudo fuser -k 7860/tcp
```

### 2. 进入前端目录 - Navigate to Frontend Directory
```bash
# 从project目录开始，进入frontend目录
cd JobCatcher/frontend
pwd  # 确认当前在 /home/devbox/project/JobCatcher/frontend
```

### 3. 检查前端文件 - Check Frontend Files
```bash
# 检查前端文件结构
ls -la
# 确认主要文件存在
ls -la index.html css/ js/ assets/
```

### 4. 启动前端服务 - Start Frontend Service

#### 方法1: 使用Python内置HTTP服务器 (推荐)
```bash
# 在frontend目录下启动HTTP服务器
python -m http.server 7860
# 或者使用Python3
python3 -m http.server 7860
```

#### 方法2: 使用Node.js服务器 (如果安装了Node.js)
```bash
# 检查是否安装了Node.js
node --version
npm --version

# 如果安装了Node.js，可以使用http-server
npx http-server -p 7860 -c-1
```

#### 方法3: 使用Live Server (如果需要热重载)
```bash
# 安装live-server (如果需要)
npm install -g live-server

# 启动live-server
live-server --port=7860 --host=0.0.0.0
```

### 5. 验证前端服务 - Verify Frontend Service
```bash
# 检查服务是否启动
curl -s http://localhost:7860/ | head -10

# 或者在浏览器中访问
echo "前端服务已启动，请访问: http://localhost:7860"
```

---

# 🔧 完整启动脚本 - Complete Startup Scripts

## 后端启动脚本 - Backend Startup Script
```bash
#!/bin/bash
# 从project目录执行此脚本

echo "🚀 启动JobCatcher后端服务..."

# 1. 检查和清理端口
echo "🔍 检查端口8000占用情况..."
if lsof -i :8000; then
    echo "⚠️  端口8000被占用，正在清理..."
    sudo fuser -k 8000/tcp
    sleep 2
fi

# 2. 进入backend目录 - 关键步骤
echo "📁 进入backend目录..."
cd JobCatcher/backend || exit 1
pwd
echo "✅ 当前目录: $(pwd)"

# 3. 激活虚拟环境
echo "🐍 激活虚拟环境..."
source ../../bin/activate
which python

# 4. 设置PYTHONPATH
echo "🔧 设置PYTHONPATH..."
export PYTHONPATH=/home/devbox/project/JobCatcher/backend:$PYTHONPATH

# 5. 检查依赖
echo "📦 检查依赖..."
pip list | grep fastapi
pip list | grep uvicorn

# 6. 测试模块导入
echo "🧪 测试模块导入..."
python -c "from app.main import app; print('✅ 模块导入成功')"

# 7. 启动服务 - 在backend目录下执行
echo "🚀 在backend目录下启动JobCatcher后端服务..."
echo "当前工作目录: $(pwd)"
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

## 前端启动脚本 - Frontend Startup Script
```bash
#!/bin/bash
# 从project目录执行此脚本

echo "🌐 启动JobCatcher前端服务..."

# 1. 检查和清理端口
echo "🔍 检查端口7860占用情况..."
if lsof -i :7860; then
    echo "⚠️  端口7860被占用，正在清理..."
    sudo fuser -k 7860/tcp
    sleep 2
fi

# 2. 进入frontend目录
echo "📁 进入frontend目录..."
cd JobCatcher/frontend || exit 1
pwd
echo "✅ 当前目录: $(pwd)"

# 3. 检查前端文件
echo "📄 检查前端文件..."
if [ ! -f "index.html" ]; then
    echo "❌ 错误：未找到index.html文件"
    exit 1
fi

# 4. 启动前端服务
echo "🚀 启动前端HTTP服务器..."
echo "前端服务将在 http://localhost:7860 启动"
python -m http.server 7860
```

## 同时启动前后端脚本 - Start Both Services Script
```bash
#!/bin/bash
# 从project目录执行此脚本

echo "🚀 同时启动JobCatcher前后端服务..."

# 启动后端服务（后台运行）
echo "📡 启动后端服务..."
cd JobCatcher/backend
source ../../bin/activate
export PYTHONPATH=/home/devbox/project/JobCatcher/backend:$PYTHONPATH
nohup python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 > ../../logs/backend.log 2>&1 &
BACKEND_PID=$!
echo "✅ 后端服务已启动 (PID: $BACKEND_PID)"

# 等待后端启动
sleep 5

# 启动前端服务（后台运行）
echo "🌐 启动前端服务..."
cd ../frontend
nohup python -m http.server 7860 > ../../logs/frontend.log 2>&1 &
FRONTEND_PID=$!
echo "✅ 前端服务已启动 (PID: $FRONTEND_PID)"

# 保存PID到文件
echo $BACKEND_PID > ../../logs/backend.pid
echo $FRONTEND_PID > ../../logs/frontend.pid

echo "🎉 JobCatcher服务启动完成！"
echo "📡 后端API: http://localhost:8000"
echo "🌐 前端界面: http://localhost:7860"
echo "📋 查看日志: tail -f logs/backend.log 或 tail -f logs/frontend.log"
```

---

# 🐛 故障排除 - Troubleshooting

## 后端故障排除 - Backend Troubleshooting

### 端口问题 - Port Issues
```bash
# 查找占用端口的进程
sudo ss -tulpn | grep :8000
# 强制释放端口
sudo kill -9 $(sudo lsof -t -i:8000)
# 杀死所有uvicorn进程
pkill -f uvicorn
```

### 模块导入错误 - Module Import Errors
```bash
# 检查当前目录 - 必须在backend目录下
pwd
# 应该显示: /home/devbox/project/JobCatcher/backend

# 检查Python路径
python -c "import sys; print('\n'.join(sys.path))"

# 测试导入
python -c "from app.main import app; print('Import successful')"

# 如果导入失败，确认目录
ls -la app/
```

### 虚拟环境问题 - Virtual Environment Issues
```bash
# 检查虚拟环境
which python
python --version
pip --version
# 重新创建虚拟环境（如果需要）
python -m venv venv
source venv/bin/activate
```

## 前端故障排除 - Frontend Troubleshooting

### 端口问题 - Port Issues
```bash
# 查找占用7860端口的进程
sudo ss -tulpn | grep :7860
# 强制释放端口
sudo kill -9 $(sudo lsof -t -i:7860)
```

### 文件访问问题 - File Access Issues
```bash
# 检查前端文件权限
ls -la JobCatcher/frontend/
# 检查index.html是否存在
cat JobCatcher/frontend/index.html | head -10
```

### 服务器问题 - Server Issues
```bash
# 如果Python HTTP服务器失败，尝试其他端口
python -m http.server 8080
# 或者使用其他服务器
php -S localhost:7860  # 如果安装了PHP
```

---

# 📝 检查清单 - Checklist

## 后端启动检查清单 - Backend Startup Checklist
启动前必须确认:
- [ ] 端口8000未被占用
- [ ] **当前目录为 `JobCatcher/backend`** (最关键)
- [ ] 虚拟环境已激活
- [ ] PYTHONPATH已正确设置
- [ ] 依赖包已安装
- [ ] app.main模块可正常导入
- [ ] pwd命令显示: `/home/devbox/project/JobCatcher/backend`

## 前端启动检查清单 - Frontend Startup Checklist
启动前必须确认:
- [ ] 端口7860未被占用
- [ ] **当前目录为 `JobCatcher/frontend`**
- [ ] index.html文件存在
- [ ] CSS和JS文件目录存在
- [ ] Python或Node.js可用于启动HTTP服务器
- [ ] pwd命令显示: `/home/devbox/project/JobCatcher/frontend`

---

# ⚡ 快速命令 - Quick Commands

## 后端快速命令 - Backend Quick Commands
```bash
# 一键检查后端环境 (从project目录)
cd JobCatcher/backend && source ../../bin/activate && python -c "from app.main import app; print('✅ 后端环境正常')"

# 一键启动后端 (从project目录)
cd JobCatcher/backend && source ../../bin/activate && export PYTHONPATH=$PWD:$PYTHONPATH && pwd && python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

## 前端快速命令 - Frontend Quick Commands
```bash
# 一键检查前端环境 (从project目录)
cd JobCatcher/frontend && ls -la index.html && echo "✅ 前端环境正常"

# 一键启动前端 (从project目录)
cd JobCatcher/frontend && pwd && python -m http.server 7860
```

## 服务管理命令 - Service Management Commands
```bash
# 停止所有服务
pkill -f uvicorn && pkill -f "http.server"

# 检查服务状态
ps aux | grep -E "(uvicorn|http.server)"

# 检查端口占用
netstat -tulpn | grep -E ":(8000|7860)"
```

---

# 🚨 关键警告 - Critical Warning

## 后端警告 - Backend Warning
**绝对不要在project根目录下执行uvicorn命令！**
**Always execute uvicorn command in the backend directory!**

错误示例 (会导致ModuleNotFoundError):
```bash
# ❌ 错误 - 在project目录下执行
(jobcatcher_env) devbox@jobcatcher:~/project$ python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
# 结果: ModuleNotFoundError: No module named 'app'
```

正确示例:
```bash
# ✅ 正确 - 在backend目录下执行
(jobcatcher_env) devbox@jobcatcher:~/project/JobCatcher/backend$ python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
# 结果: 服务正常启动
```

## 前端警告 - Frontend Warning
**确保在frontend目录下启动前端服务！**
**Always start frontend service in the frontend directory!**

错误示例:
```bash
# ❌ 错误 - 在错误目录下执行
(jobcatcher_env) devbox@jobcatcher:~/project$ python -m http.server 7860
# 结果: 无法正确访问前端文件
```

正确示例:
```bash
# ✅ 正确 - 在frontend目录下执行
(jobcatcher_env) devbox@jobcatcher:~/project/JobCatcher/frontend$ python -m http.server 7860
# 结果: 前端服务正常启动
```

---

**重要**: 每次启动都必须严格按照此流程执行，特别是确保在正确的目录下执行相应的服务命令！

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/freesoulinempty) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
