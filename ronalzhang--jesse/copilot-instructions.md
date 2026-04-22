## jesse

> 1. 服务器ip：43.134.38.231密码：Pr971V3j，

1. 服务器ip：43.134.38.231密码：Pr971V3j，
2. 登陆使用sshpass命令
3. 应用在这个目录下：/ubuntu/Jesse+

5. 用pm2管理应用，应用都运行在各自的虚拟环境下，确保每次安装和运行都要在虚拟环境中进行
6. 服务器上运行的操作要用sshpass 登陆命令+具体命令，比如登陆命令+ “pm2 status”
7. 命令里有--lines 50的时候一定加上 --nostream才完整。
类似：sshpass -p "Pr971V3j" ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ubuntu@43.134.38.231

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ronalzhang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
