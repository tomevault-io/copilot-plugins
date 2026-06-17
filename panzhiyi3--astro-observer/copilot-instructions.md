## astro-observer

> 深空观测条件预报助手 - 为你的天文观测和摄影提供专业级天气预报

# Astro Observer Skill

深空观测条件预报助手 - 为你的天文观测和摄影提供专业级天气预报

## 功能

- ☁️ **云量预报** - 低/中/高空云量分层数据
- 🌙 **月相信息** - 月龄、月出月落、光照比例
- 🌅 **曙暮光时间** - 天文黄昏/黎明精确时间
- 👁️ **视宁度** - 大气稳定度评估
- 💧 **透明度** - 大气透明度指数
- 🌌 **银河可见性** - 银河升起时间和位置
- 📊 **综合评分** - 0-100 分观测条件评分
- 💡 **观测建议** - 强烈推荐/推荐/一般/不推荐

## 使用方式

### 基础查询
```
"查询上海今晚的观测条件"
"北京明天晚上适合深空观测吗"
"杭州经纬度 30.27, 120.15 周末观测条件如何"
```

### 指定时间
```
"查询上海 2025-12-21 的观测条件"
"这周六晚上杭州的天气如何"
"下个月满月时的观测条件"
```

### 地点查询
```
"上海天文台的观测条件"
"青海冷湖的视宁度如何"
"西藏阿里适合观测吗"
```

## 返回数据

```json
{
  "location": {
    "name": "上海",
    "latitude": 31.23,
    "longitude": 121.47
  },
  "date": "2025-12-21",
  "weather": {
    "cloudCover": 15,
    "seeing": 2.1,
    "transparency": 8,
    "temperature": 5,
    "humidity": 45
  },
  "astronomy": {
    "moonPhase": 0.03,
    "moonFraction": 0.05,
    "astronomicalDusk": "18:45",
    "astronomicalDawn": "05:30"
  },
  "score": {
    "overall": 85,
    "recommendation": "🌟 强烈推荐 - 绝佳观测条件！"
  }
}
```

## API 来源

| 数据 | 来源 | 费用 |
|------|------|------|
| 天气/云量 | Open-Meteo | 免费 |
| 视宁度/透明度 | Open-Meteo Astronomy | 免费 |
| 月相/日出落 | SunCalc 计算 | 免费 |
| 光污染 | Light Pollution Map | 免费 |

## 安装

```bash
# Skill 会自动加载，无需额外安装
# 确保 OpenClaw Gateway 运行中
```

## 配置

无需 API Key，所有数据源均为免费服务。

## 注意事项

- 预报数据每 6 小时更新
- 最佳查询时间：观测前 1-3 天
- 山区天气变化快，建议多次查询
- 冬季注意露点温度，防止设备结露

---

*版本：1.0.0*  
*作者：Auto-generated*  
*许可：MIT*

---
> Source: [panzhiyi3/astro-observer](https://github.com/panzhiyi3/astro-observer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
