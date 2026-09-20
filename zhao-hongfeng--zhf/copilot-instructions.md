## zhf

> 个人网址收藏网站:点击卡片在新标签页跳转,数据存 txt,无数据库、无前端构建工具。

# LinkHub 网址收藏

个人网址收藏网站:点击卡片在新标签页跳转,数据存 txt,无数据库、无前端构建工具。

## 技术栈与结构

- 后端:FastAPI(Python 3.13),入口 `python main.py`,默认 http://127.0.0.1:8000
- 前端:原生 HTML / CSS / JS,位于 `static/`,由 FastAPI 静态托管
- 存储:`data/bookmarks.txt`,每行一个 JSON(字段:id/title/url/category/note/created_at)

```
main.py                    # 启动入口
app/main.py                # 应用工厂(注册路由、挂载静态文件)
app/config.py              # 路径与常量
app/models.py              # Pydantic 模型
app/storage.py             # 数据访问层(带线程锁,读-改-写需在 storage.lock 内)
app/routers/bookmarks.py   # /api/bookmarks 增删改查
static/                    # index.html + css/ + js/
```

## 约定

- 新增接口:在 `app/routers/` 下建模块(APIRouter),在 `app/main.py` 注册
- 数据读写只走 `app/storage.py`,路由层不直接碰文件
- 前端无构建工具,改完刷新页面即可;接口统一 `/api/` 前缀
- 接口文档:http://127.0.0.1:8000/docs

---
> Source: [zhao-hongfeng/zhf](https://github.com/zhao-hongfeng/zhf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
