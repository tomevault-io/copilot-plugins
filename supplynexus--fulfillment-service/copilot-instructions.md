## production-deployment

> 部署/SSH/服务器时参考——目录、Dev vs Prod、敏感文件


# 服务器部署（按需参考）

- Git：走 PR，见 git-workflow.mdc。SSH：ubuntu@133.242.179.110。仓库：/home/ubuntu/project/fulfillment-service；部署目录：/opt/supplynexus（rsync 后 docker-compose，无 .git）。
- **Dev**：`./scripts/dev/deploy-from-git.sh`；postgres_dev/backend_dev/frontend_dev，Backend 8001、Frontend 3001。**Prod**：`./scripts/prod/deploy-from-git.sh`；端口 8000/3000 仅 127.0.0.1。脚本含拉分支、rsync、up、alembic upgrade head。
- 敏感文件不提交：env.prod、docker-compose.prod.yml；scp + chmod 600。容器异常：按环境 `docker rm -f` 对应 supplynexus 容器后 `docker-compose -f docker-compose.{dev|prod}.yml up -d --remove-orphans`。详见 scripts/dev/DEPLOYMENT.md、scripts/prod/DEPLOYMENT_STEPS.md。

---
> Source: [supplynexus/fulfillment-service](https://github.com/supplynexus/fulfillment-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
