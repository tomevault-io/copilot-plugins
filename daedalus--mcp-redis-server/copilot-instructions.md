## mcp-redis-server

> MCP server exposing Redis API to AI models.

# MCP Redis Server

MCP server exposing Redis API to AI models.

## When to use this skill

Use this skill when you need to:
- Interact with Redis databases
- Store/cache data
- Use pub/sub messaging
- Manage Redis keys

## Tools

**Connection:**
- `redis_connect`, `redis_disconnect`, `redis_ping`

**Strings:**
- `redis_get`, `redis_set`, `redis_mget`, `redis_mset`, `redis_del`
- `redis_incr`, `redis_decr`

**Lists:**
- `redis_lpush`, `redis_rpush`, `redis_lrange`, `redis_llen`

**Sets:**
- `redis_sadd`, `redis_srem`, `redis_smembers`, `redis_sismember`, `redis_scard`

**Hashes:**
- `redis_hset`, `redis_hget`, `redis_hmset`, `redis_hmget`, `redis_hgetall`

**Sorted Sets:**
- `redis_zadd`, `redis_zscore`, `redis_zrange`, `redis_zrevrange`, `redis_zcard`

**Keys:**
- `redis_exists`, `redis_expire`, `redis_ttl`, `redis_type`, `redis_scan`

**Server:**
- `redis_dbsize`, `redis_flushdb`, `redis_flushall`, `redis_info`
- `redis_config_get`, `redis_config_set`

## Install

```bash
pip install mcp-redis-server
```

---
> Source: [daedalus/mcp-redis-server](https://github.com/daedalus/mcp-redis-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
