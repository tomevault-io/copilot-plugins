## web-crypto-proxy

> 全自动分析网站加密逻辑，自动提取密钥并生成 mitmproxy 脚本，实现 BurpSuite 与目标服务器之间的**双向透明加解密**。

# Web Crypto Proxy Skill

## 描述

全自动分析网站加密逻辑，自动提取密钥并生成 mitmproxy 脚本，实现 BurpSuite 与目标服务器之间的**双向透明加解密**。

## 核心架构

```
目标 URL ──→ AI 自动分析 ──→ 提取密钥 ──→ 生成脚本 ──→ mitmproxy 运行
                                                          ↓
浏览器 → BurpSuite (:8080) → mitmproxy (:8083) → 目标服务器
              ↑                    ↓
         [明文操作]          [自动加解密]
```

## 双向加解密机制

```
┌─────────────────────────────────────────────────────────────────┐
│                      双向透明加解密                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  【请求方向】BurpSuite 明文 → mitmproxy 加密 → 服务器收到密文     │
│                                                                  │
│    BurpSuite 发送:                                               │
│    {"username": "admin", "password": "123456"}                   │
│                         ↓                                        │
│    mitmproxy 检测明文字段，自动加密:                              │
│    {"username": "Z09xqdN...", "password": "Yq+prW..."}           │
│                         ↓                                        │
│    服务器收到加密数据                                            │
│                                                                  │
│  【响应方向】服务器密文 → mitmproxy 解密 → BurpSuite 显示明文     │
│                                                                  │
│    服务器返回:                                                   │
│    {"data": "encrypted_base64_string..."}                        │
│                         ↓                                        │
│    mitmproxy 检测加密数据，自动解密:                              │
│    {"data": {"user": "admin", "role": "admin"}}                  │
│                         ↓                                        │
│    BurpSuite 显示明文数据                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 使用方法

```
skill: web-crypto-proxy https://target.com
```

AI 将**自动执行**以下步骤：

1. 获取网站 HTML 源码
2. 提取并下载所有 JS 文件
3. 分析加密算法和提取密钥
4. 生成完整的 mitmproxy 脚本
5. 测试加解密功能
6. 提供启动命令

## AI 自动执行流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI 自动化分析流程                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [Step 1] 获取目标 URL，解析域名                                 │
│       ↓                                                          │
│  [Step 2] curl 获取 HTML 源码                                    │
│       ↓                                                          │
│  [Step 3] 正则提取所有 <script src="..."> JS 文件链接           │
│       ↓                                                          │
│  [Step 4] 批量下载 JS 文件到本地                                 │
│       ↓                                                          │
│  [Step 5] 搜索加密关键词:                                        │
│           - encrypt / decrypt                                    │
│           - CryptoJS / JSEncrypt / sm2 / sm4                     │
│           - setPublicKey / setPrivateKey                         │
│           - AES / DES / RSA / SM2 / SM4                          │
│       ↓                                                          │
│  [Step 6] 正则提取密钥:                                          │
│           - RSA 公钥: MFwwDQYJKoZIhvcNAQEB...                    │
│           - RSA 私钥: MIIBVAIBADANBgkqhkiG9w...                  │
│           - AES 密钥/IV: CryptoJS.enc.Utf8.parse(...)            │
│       ↓                                                          │
│  [Step 7] 识别加密算法、模式、填充方式                           │
│       ↓                                                          │
│  [Step 8] 生成 Python 加解密函数代码                             │
│       ↓                                                          │
│  [Step 9] 生成完整 mitmproxy 脚本（包含双向加解密逻辑）          │
│       ↓                                                          │
│  [Step 10] 测试加解密功能是否正常                                │
│       ↓                                                          │
│  [Step 11] 输出分析报告和启动命令                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## AI 执行的具体命令

### 获取 HTML

```bash
curl -s -k "https://target.com/" -o index.html
```

### 提取 JS 文件链接

```bash
grep -oE 'src="[^"]+\.js[^"]*"' index.html
grep -oE '<script[^>]+src="[^"]+"' index.html
```

### 下载 JS 文件

```bash
mkdir -p js
# 对每个 JS URL
curl -s -k "$JS_URL" -o "js/$(basename $JS_URL)"
```

### 搜索加密关键词

```bash
grep -rn "encrypt\|decrypt\|CryptoJS\|JSEncrypt" js/
grep -rn "setPublicKey\|setPrivateKey\|AES\|RSA\|SM2\|SM4" js/
```

### 提取 RSA 密钥

```bash
# RSA 公钥 (512-bit)
grep -o 'MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJB[A-Za-z0-9+/=\n]*' js/*.js

# RSA 公钥 (1024-bit)
grep -o 'MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC[A-Za-z0-9+/=\n]*' js/*.js

# RSA 公钥 (2048-bit)
grep -o 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA[A-Za-z0-9+/=\n]*' js/*.js

# RSA 私钥
grep -o 'MIIB[A-Za-z0-9+/=\n]*' js/*.js | grep -E '(PRIVATE|AgEAAkEA)'
```

### 提取 AES 密钥

```bash
# CryptoJS parse 形式
grep -oE "CryptoJS\.enc\.Utf8\.parse\(['\"][^'\"]+['\"]\)" js/*.js

# 直接定义形式
grep -oE "(key|iv)\s*[:=]\s*['\"][A-Za-z0-9]{16}['\"]" js/*.js
```

## 密钥识别正则模式

```python
# RSA 公钥模式
RSA_PATTERNS = {
    '512bit': r'MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJB[A-Za-z0-9+/=\n\\]+',
    '1024bit': r'MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC[A-Za-z0-9+/=\n\\]+',
    '2048bit': r'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA[A-Za-z0-9+/=\n\\]+',
}

# RSA 私钥模式
RSA_PRIVATE_PATTERN = r'MII[B-Za-z0-9+/=\n\\]+'

# AES 密钥模式
AES_PATTERNS = {
    'parse': r"CryptoJS\.enc\.Utf8\.parse\(['\"]([^'\"]+)['\"]\)",
    'direct': r"(?:key|KEY)\s*[:=]\s*['\"]([A-Za-z0-9]{16,32})['\"]",
    'iv': r"(?:iv|IV)\s*[:=]\s*['\"]([A-Za-z0-9]{16})['\"]",
}

# 国密模式
SM_PATTERNS = {
    'sm2_pub': r'sm2Public[Kk]ey\s*[:=]\s*["\']([A-Fa-f0-9]{64,130})["\']',
    'sm4_key': r'sm4[Kk]ey\s*[:=]\s*["\']([A-Fa-f0-9]{32})["\']',
}
```

## 加密算法识别

### RSA (JSEncrypt)

**识别特征:**
- `JSEncrypt` 或 `new JSEncrypt()`
- `setPublicKey()` / `setPrivateKey()`
- 公钥以 `MFwwDQYJKoZI` 开头 (512-bit) 或 `MIGfMA0GCSqGSIb3` 开头 (1024-bit)

**生成的 Python 代码:**
```python
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5
import base64

def encrypt(self, plaintext: str) -> str:
    rsa_key = RSA.import_key(self.public_key)
    cipher = PKCS1_v1_5.new(rsa_key)
    encrypted = cipher.encrypt(plaintext.encode('utf-8'))
    return base64.b64encode(encrypted).decode()

def decrypt(self, ciphertext: str) -> str:
    rsa_key = RSA.import_key(self.private_key)
    cipher = PKCS1_v1_5.new(rsa_key)
    decrypted = cipher.decrypt(base64.b64decode(ciphertext), None)
    return decrypted.decode('utf-8') if decrypted else ciphertext
```

### AES (CryptoJS)

**识别特征:**
- `CryptoJS.AES.encrypt` / `CryptoJS.AES.decrypt`
- `CryptoJS.mode.CBC` / `CryptoJS.mode.ECB`
- `CryptoJS.pad.Pkcs7`

**生成的 Python 代码:**
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import base64

def encrypt(self, plaintext: str) -> str:
    cipher = AES.new(self.key, AES.MODE_CBC, self.iv)
    encrypted = cipher.encrypt(pad(plaintext.encode(), 16))
    return base64.b64encode(encrypted).decode()

def decrypt(self, ciphertext: str) -> str:
    cipher = AES.new(self.key, AES.MODE_CBC, self.iv)
    decrypted = cipher.decrypt(base64.b64decode(ciphertext))
    return unpad(decrypted, 16).decode('utf-8')
```

### 国密 SM2/SM4

**识别特征:**
- `sm2Encrypt` / `sm2Decrypt` / `SM2`
- `sm4Encrypt` / `sm4Decrypt` / `SM4`
- `sm-crypto` 库

**生成的 Python 代码:**
```python
from gmssl.sm4 import CryptSM4, SM4_ENCRYPT, SM4_DECRYPT
import base64

def encrypt(self, plaintext: str) -> str:
    crypt = CryptSM4()
    crypt.set_key(self.key, SM4_ENCRYPT)
    return base64.b64encode(crypt.crypt_ecb(plaintext.encode())).decode()

def decrypt(self, ciphertext: str) -> str:
    crypt = CryptSM4()
    crypt.set_key(self.key, SM4_DECRYPT)
    return crypt.crypt_ecb(base64.b64decode(ciphertext)).decode('utf-8')
```

## 通用 mitmproxy 脚本模板

```python
#!/usr/bin/env python3
"""
{TARGET_DOMAIN} 加密流量处理脚本

自动分析结果:
- 加密算法: {ALGORITHM}
- 加密字段: {ENCRYPT_FIELDS}

使用方法:
    mitmdump -s {SCRIPT_NAME} -p 8083

BurpSuite 配置:
    Settings → Proxy → Upstream Proxy Servers
    添加: *{TARGET_DOMAIN}* → 127.0.0.1:8083
"""

from mitmproxy import http
import json
import base64
import re


class CryptoProxy:
    """加密代理插件 - 支持双向加解密"""

    def __init__(self):
        # 目标域名
        self.target_domains = ['{TARGET_DOMAIN}']

        # 需要加密的字段 (AI 识别)
        self.encrypt_fields = ['username', 'password']

        # 加密密钥 (AI 提取)
        # self.public_key = "..."
        # self.private_key = "..."
        # self.key = b'...'
        # self.iv = b'...'

        self._init_crypto()

    def _init_crypto(self):
        """初始化加解密器"""
        # AI 根据算法生成初始化代码
        pass

    def encrypt(self, plaintext: str) -> str:
        """加密函数 - AI 生成"""
        # AI 生成的加密代码
        raise NotImplementedError()

    def decrypt(self, ciphertext: str) -> str:
        """解密函数 - AI 生成"""
        # AI 生成的解密代码
        raise NotImplementedError()

    def is_base64(self, s: str) -> bool:
        """判断是否为 Base64 编码的加密数据"""
        if not s or len(s) < 20:
            return False
        if not re.match(r'^[A-Za-z0-9+/]+=*$', s):
            return False
        try:
            decoded = base64.b64decode(s)
            # RSA 加密结果长度固定
            if len(decoded) in [64, 128, 256]:
                return True
        except:
            pass
        return False

    def is_target(self, url: str) -> bool:
        return any(d in url for d in self.target_domains)

    def request(self, flow: http.HTTPFlow) -> None:
        """
        请求拦截 - 自动加密明文字段
        关键：检测明文字段，自动加密后发送给服务器
        """
        if not self.is_target(flow.request.pretty_url):
            return

        url = flow.request.pretty_url
        print(f"\n[*] {flow.request.method} {url}")

        try:
            content_type = flow.request.headers.get("Content-Type", "")

            # 处理 JSON 请求
            if "application/json" in content_type:
                body = flow.request.text
                if not body:
                    return

                data = json.loads(body)
                modified = False

                for field in self.encrypt_fields:
                    if field in data:
                        value = data[field]
                        # 关键：如果是明文，加密它
                        if not self.is_base64(value):
                            encrypted = self.encrypt(value)
                            print(f"    [加密] {field}: {value} -> {encrypted[:30]}...")
                            data[field] = encrypted
                            modified = True
                        else:
                            # 已是加密数据，可选拒绝或解密显示
                            decrypted = self.decrypt(value)
                            print(f"    [已加密] {field} -> {decrypted}")

                if modified:
                    flow.request.text = json.dumps(data, ensure_ascii=False)
                    print("[+] 请求已加密")

            # 处理 form 表单
            elif "application/x-www-form-urlencoded" in content_type:
                form = dict(flow.request.urlencoded_form) if flow.request.urlencoded_form else {}
                modified = False

                for field in self.encrypt_fields:
                    if field in form and not self.is_base64(form[field]):
                        encrypted = self.encrypt(form[field])
                        print(f"    [加密] {field}: {form[field]} -> {encrypted[:30]}...")
                        form[field] = encrypted
                        modified = True

                if modified:
                    flow.request.urlencoded_form = form.items()

        except Exception as e:
            print(f"[-] 请求处理错误: {e}")

    def response(self, flow: http.HTTPFlow) -> None:
        """
        响应拦截 - 自动解密加密数据
        关键：检测加密数据，自动解密后显示在 BurpSuite
        """
        if not self.is_target(flow.request.pretty_url):
            return

        try:
            content_type = flow.response.headers.get("Content-Type", "")
            if "application/json" not in content_type:
                return

            body = flow.response.text
            if not body:
                return

            data = json.loads(body)
            modified = self._decrypt_fields(data)

            if modified:
                flow.response.text = json.dumps(data, ensure_ascii=False, indent=2)
                print("[+] 响应已解密")

        except:
            pass

    def _decrypt_fields(self, data):
        """递归解密嵌套数据"""
        modified = False
        if isinstance(data, dict):
            for k, v in data.items():
                if isinstance(v, str) and self.is_base64(v):
                    decrypted = self.decrypt(v)
                    if decrypted != v:
                        try:
                            data[k] = json.loads(decrypted)
                        except:
                            data[k] = decrypted
                        print(f"    [解密] {k}")
                        modified = True
                elif isinstance(v, (dict, list)):
                    if self._decrypt_fields(v):
                        modified = True
        elif isinstance(data, list):
            for i, v in enumerate(data):
                if isinstance(v, str) and self.is_base64(v):
                    decrypted = self.decrypt(v)
                    if decrypted != v:
                        data[i] = decrypted
                        modified = True
                elif isinstance(v, (dict, list)):
                    if self._decrypt_fields(v):
                        modified = True
        return modified


addons = [CryptoProxy()]
```

## 输出文件结构

```
{target_domain}/
├── index.html              # 网站源码
├── js/                     # JS 文件目录
│   ├── app.js
│   ├── chunk-libs.js
│   └── ...
├── ANALYSIS_REPORT.md      # 分析报告
└── {domain}_proxy.py       # mitmproxy 脚本 (可直接运行)
```

## 启动命令

```bash
# 1. 安装依赖
pip install mitmproxy pycryptodome

# 国密支持 (如需要)
pip install gmssl

# 2. 启动 mitmproxy
cd {output_dir}
mitmdump -s {domain}_proxy.py -p 8083

# 3. 配置 BurpSuite
# Settings → Proxy → Upstream Proxy Servers
# 添加: *{domain}* → 127.0.0.1:8083

# 4. 浏览器代理指向 BurpSuite (:8080)
```

## 使用场景

### 登录爆破

```
1. BurpSuite 拦截登录请求
2. 发送到 Intruder
3. 设置 username/password 为 payload 位置
4. 填入明文 payload (如: admin, root, test)
5. 启动攻击 → 自动加密发送
```

### SQL 注入测试

```
1. BurpSuite 拦截请求
2. 修改 username 为: admin' OR '1'='1
3. 放行 → 自动加密发送
4. 观察响应判断注入点
```

### 越权测试

```
1. 拦截用户信息查询请求
2. 修改加密的用户 ID 为其他用户 ID
3. 放行 → 自动加密发送
4. 检查是否返回其他用户数据
```

## 日志输出示例(一定要输出详细实例加密内容)

```
[*] POST https://api.target.com/login
    [加密] username: admin -> ZkeeMKHSfeofC+B83GEYMQsH...
    [加密] password: 123456 -> eQa0WyHx5H1yO9jWqipCPtBY...
[+] 请求已加密

[*] GET https://api.target.com/user/info
    [解密] data
[+] 响应已解密
```

## 故障排除

### 未检测到加密

- JS 文件可能动态加载 → 检查 Network 面板
- 代码可能高度混淆 → 尝试反混淆工具
- 密钥可能通过接口获取 → 分析网络请求

### 密钥提取失败

- 密钥可能通过接口动态获取 → 分析 API 响应
- 密钥可能硬编码在 HTML 中 → 检查源码
- 密钥可能在 Webpack chunk 中 → 搜索所有 JS 文件

### 加解密失败

- 检查编码格式 (UTF-8 / GBK / GB2312)
- 检查 Base64 变体 (标准 / URL-safe)
- 检查填充方式 (PKCS7 / ZeroPadding)
- RSA 检查填充模式 (PKCS1_v1_5 / OAEP)

### BurpSuite 无法连接

- 确认 mitmproxy 已启动: `lsof -i :8083`
- 检查上游代理配置格式: `*domain*`
- 检查防火墙设置

## 安全注意事项

⚠️ **仅用于授权的安全测试**

- 仅在获得授权的系统上使用
- 安全存储提取的密钥
- 测试后清除日志和密钥文件
- 不要将密钥提交到版本控制

## 依赖

```bash
pip install mitmproxy pycryptodome
pip install gmssl  # 国密支持
```

---

*本 Skill 提供全自动化加密流量分析。输入 URL 即可自动完成信息收集、密钥提取、脚本生成，实现双向透明加解密。*

---
> Source: [Arg3Sea/web-crypto-proxy](https://github.com/Arg3Sea/web-crypto-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
