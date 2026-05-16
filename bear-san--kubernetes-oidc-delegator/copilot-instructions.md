## kubernetes-oidc-delegator

> - これは、Kubernetes の API Server に提供されている ServiceAccount Token Signing Keys を JWKs 形式で公開する API サーバーです

# このプロジェクトについて
- これは、Kubernetes の API Server に提供されている ServiceAccount Token Signing Keys を JWKs 形式で公開する API サーバーです
- 通常、Kubernetes API はローカルでのみ公開されていますが、この API を通して、AWS のような外部 IdP から、Kubernetes の ServiceAccount に対して発行された token を検証することができます

# 前提
- Kubernetes クラスタは、ServiceAccount に対して token を発行するための署名鍵を持っています
- 起動時の引数で、署名鍵のファイルパスを指定します
  - 鍵は PEM 形式で保存されている必要があります
- 鍵ペアは、1つのみ指定することができます

# 動作フロー
- `/.well-known/openid-configuration` にアクセスすると、JWKs の URL が返却されます
- `/keys` にアクセスすると、JWKs が返却されます

## ファイルフォーマット
### OpenID Configuration
```json
{
    "issuer": "https://kubernetes.default.svc.cluster.local",
    "jwks_uri": "https://10.194.230.72:6443/openid/v1/jwks",
    "response_types_supported": [
      "id_token"
    ],
    "subject_types_supported": [
      "public"
    ],
    "id_token_signing_alg_values_supported": [
      "RS256"
    ]
}
```

### JWKs
```json
{
    "keys": [
        {
            "use": "sig",
            "kty": "RSA",
            "kid": "XBRQklg6V4uMi9zGXrC1d_gqrT4tKWKyM6iZzXKiYhQ",
            "alg": "RS256",
            "n": "1jfO8HAAowQhhmrJLKOZPMuKPTZlaruCCTbKURpViqbltcXUwJzdgkgobu5yi3H_I4l9aqCvyjzNnEiP3oux3l6oP49D-5VBoS7PuifP8ZCV6fO8_4O-2h9rbwh1TaGfSIAoJw3CydF4DWAdN4rqyaDL82suX2HOAmDgZs8Lz7eBeQS2ztE9Lhh-YGfsMwIskd_3rvzbFZrY7L_rDYh-W0Zsvt-7twlQwjqoudC7gQMILe6zEP8MB3MmQKhd1ZPeqD8esbMYcwO2409SSxxHg48t_j3Uh1bCS08kFRMOgybk0luLzwx6sqruUUrf9OgEPyZCiFZB8nWlGUC7IqA4aQ",
            "e": "AQAB"
        }
    ]
}
```

---
> Source: [bear-san/kubernetes-oidc-delegator](https://github.com/bear-san/kubernetes-oidc-delegator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
