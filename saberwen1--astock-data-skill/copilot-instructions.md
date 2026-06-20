## astock-data-skill

> A股量化数据分析工具，基于AkShare库获取A股行情、财务数据、板块信息等。用于回答关于A股股票查询、行情数据、财务分析、选股等问题。


# A股量化 - AkShare 数据接口

## 快速开始

安装依赖：
```bash
pip install akshare
```


# AKShare 股票数据接口

本文档为 AKShare 股票数据接口的渐进式披露文档，按接口功能分类组织。

---

## A股

### 股票市场总貌

#### 上海证券交易所

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 上海证券交易所-股票数据总貌 | 上海证券交易所-股票数据总貌 | [stock_sse_summary.md](./stock_metadata/stock_sse_summary.md) |

#### 深圳证券交易所

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 深圳证券交易所-市场总貌-证券类别统计 | 深圳证券交易所-市场总貌-证券类别统计 | [stock_szse_summary.md](./stock_metadata/stock_szse_summary.md) |
| 深圳证券交易所-市场总貌-地区交易排序 | 深圳证券交易所-市场总貌-地区交易排序 | [stock_szse_area_summary.md](./stock_metadata/stock_szse_area_summary.md) |
| 深圳证券交易所-股票行业成交 | 深圳证券交易所-统计资料-股票行业成交数据 | [stock_szse_sector_summary.md](./stock_metadata/stock_szse_sector_summary.md) |

### 个股信息查询

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-个股-股票信息 | 东方财富-个股-股票信息 | [stock_individual_info_em.md](./stock_metadata/stock_individual_info_em.md) |
| 雪球财经-个股-公司概况 | 雪球财经-个股-公司概况-公司简介 | [stock_individual_basic_info_xq.md](./stock_metadata/stock_individual_basic_info_xq.md) |

### 行情报价

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-行情报价 | 东方财富-行情报价 | [stock_bid_ask_em.md](./stock_metadata/stock_bid_ask_em.md) |

### 实时行情数据

#### 实时行情数据-东财

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-沪深京A股-实时行情数据 | 东方财富网-沪深京A股-实时行情数据 | [stock_zh_a_spot_em.md](./stock_metadata/stock_zh_a_spot_em.md) |
| 东方财富网-沪A股-实时行情数据 | 东方财富网-沪A股-实时行情数据 | [stock_sh_a_spot_em.md](./stock_metadata/stock_sh_a_spot_em.md) |
| 东方财富网-深A股-实时行情数据 | 东方财富网-深A股-实时行情数据 | [stock_sz_a_spot_em.md](./stock_metadata/stock_sz_a_spot_em.md) |
| 东方财富网-京A股-实时行情数据 | 东方财富网-京A股-实时行情数据 | [stock_bj_a_spot_em.md](./stock_metadata/stock_bj_a_spot_em.md) |
| 东方财富网-新股-实时行情数据 | 东方财富网-新股-实时行情数据 | [stock_new_a_spot_em.md](./stock_metadata/stock_new_a_spot_em.md) |
| 东方财富网-创业板-实时行情 | 东方财富网-创业板-实时行情 | [stock_cy_a_spot_em.md](./stock_metadata/stock_cy_a_spot_em.md) |
| 东方财富网-科创板-实时行情 | 东方财富网-科创板-实时行情 | [stock_kc_a_spot_em.md](./stock_metadata/stock_kc_a_spot_em.md) |
| 东方财富网-AB股比价 | 东方财富网-行情中心-沪深京个股-AB股比价-全部AB股比价 | [stock_zh_ab_comparison_em.md](./stock_metadata/stock_zh_ab_comparison_em.md) |

#### 实时行情数据-新浪

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-沪深京A股数据 | 新浪财经-沪深京A股数据,重复运行本函数会被新浪暂时封IP | [stock_zh_a_spot.md](./stock_metadata/stock_zh_a_spot.md) |

#### 实时行情数据-雪球

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 雪球-行情中心-个股 | 雪球-行情中心-个股 | [stock_individual_spot_xq.md](./stock_metadata/stock_individual_spot_xq.md) |

### 历史行情数据

#### 历史行情数据-东财

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-沪深京A股日频率数据 | 东方财富-沪深京A股日频率数据;历史数据按日频率更新,当日收盘价请在收盘后获取 | [stock_zh_a_hist.md](./stock_metadata/stock_zh_a_hist.md) |

#### 历史行情数据-新浪

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-沪深京A股的数据 | 新浪财经-沪深京A股的数据,历史数据按日频率更新 | [stock_zh_a_daily.md](./stock_metadata/stock_zh_a_daily.md) |

#### 历史行情数据-腾讯

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 腾讯证券-日频-股票历史数据 | 腾讯证券-日频-股票历史数据;历史数据按日频率更新,当日收盘价请在收盘后获取 | [stock_zh_a_hist_tx.md](./stock_metadata/stock_zh_a_hist_tx.md) |

### 分时数据

#### 分时数据-新浪

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-沪深京A股分时数据 | 新浪财经-沪深京A股股票或者指数的分时数据,目前可以获取1,5,15,30,60分钟的数据频率 | [stock_zh_a_minute.md](./stock_metadata/stock_zh_a_minute.md) |

#### 分时数据-东财

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-沪深京A股-每日分时行情 | 东方财富网-行情首页-沪深京A股-每日分时行情;该接口只能获取近期的分时数据 | [stock_zh_a_hist_min_em.md](./stock_metadata/stock_zh_a_hist_min_em.md) |

### 日内分时数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-分时数据 | 东方财富-分时数据 | [stock_intraday_em.md](./stock_metadata/stock_intraday_em.md) |
| 新浪财经-日内分时数据 | 新浪财经-日内分时数据;只能获取近期的数据,此处仅返回大单数据 | [stock_intraday_sina.md](./stock_metadata/stock_intraday_sina.md) |
| 东方财富-股票行情-盘前数据 | 东方财富-股票行情-盘前数据 | [stock_zh_a_hist_pre_min_em.md](./stock_metadata/stock_zh_a_hist_pre_min_em.md) |

### 历史分笔数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 腾讯财经-历史分笔数据 | 每个交易日16:00提供当日数据 | [stock_zh_a_tick_tx.md](./stock_metadata/stock_zh_a_tick_tx.md) |

### 同行比较

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-行情中心-同行比较-成长性比较 | 东方财富-行情中心-同行比较-成长性比较 | [stock_zh_growth_comparison_em.md](./stock_metadata/stock_zh_growth_comparison_em.md) |
| 东方财富-行情中心-同行比较-估值比较 | 东方财富-行情中心-同行比较-估值比较 | [stock_zh_valuation_comparison_em.md](./stock_metadata/stock_zh_valuation_comparison_em.md) |
| 东方财富-行情中心-同行比较-杜邦分析比较 | 东方财富-行情中心-同行比较-杜邦分析比较 | [stock_zh_dupont_comparison_em.md](./stock_metadata/stock_zh_dupont_comparison_em.md) |
| 东方财富-行情中心-同行比较-公司规模 | 东方财富-行情中心-同行比较-公司规模 | [stock_zh_scale_comparison_em.md](./stock_metadata/stock_zh_scale_comparison_em.md) |

---

## A股-CDR

### 历史行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 上海证券交易所-科创板-CDR | 上海证券交易所-科创板-CDR | [stock_zh_a_cdr_daily.md](./stock_metadata/stock_zh_a_cdr_daily.md) |

---

## B股

### 实时行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-实时行情数据 | 东方财富网-实时行情数据 | [stock_zh_b_spot_em.md](./stock_metadata/stock_zh_b_spot_em.md) |
| B股数据-新浪财经 | B股数据是从新浪财经获取的数据 | [stock_zh_b_spot.md](./stock_metadata/stock_zh_b_spot.md) |

### 历史行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| B股数据-新浪财经历史行情 | B股数据是从新浪财经获取的数据,历史数据按日频率更新 | [stock_zh_b_daily.md](./stock_metadata/stock_zh_b_daily.md) |
| 新浪财经B股分时数据 | 新浪财经B股股票或者指数的分时数据 | [stock_zh_b_minute.md](./stock_metadata/stock_zh_b_minute.md) |

---

## 次新股

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-行情中心-沪深股市-次新股 | 新浪财经-行情中心-沪深股市-次新股 | [stock_zh_a_new.md](./stock_metadata/stock_zh_a_new.md) |

---

## 股市日历

### 公司动态

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-数据中心-股市日历-公司动态 | 东方财富网-数据中心-股市日历-公司动态 | [stock_gsrl_gsdt_em.md](./stock_metadata/stock_gsrl_gsdt_em.md) |

---

## 风险警示板

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-行情中心-沪深个股-风险警示板 | 东方财富网-行情中心-沪深个股-风险警示板 | [stock_zh_a_st_em.md](./stock_metadata/stock_zh_a_st_em.md) |

---

## 新股

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-行情中心-沪深个股-新股 | 东方财富网-行情中心-沪深个股-新股 | [stock_zh_a_new_em.md](./stock_metadata/stock_zh_a_new_em.md) |

### 新股上市首日

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 同花顺-数据中心-新股数据-新股上市首日 | 同花顺-数据中心-新股数据-新股上市首日 | [stock_xgsr_ths.md](./stock_metadata/stock_xgsr_ths.md) |

### IPO受益股

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 同花顺-数据中心-新股数据-IPO受益股 | 同花顺-数据中心-新股数据-IPO受益股 | [stock_ipo_benefit_ths.md](./stock_metadata/stock_ipo_benefit_ths.md) |

---

## 两网及退市

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-行情中心-沪深个股-两网及退市 | 东方财富网-行情中心-沪深个股-两网及退市 | [stock_zh_a_stop_em.md](./stock_metadata/stock_zh_a_stop_em.md) |

---

## 科创板

### 实时行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-科创板股票实时行情数据 | 新浪财经-科创板股票实时行情数据 | [stock_zh_kcb_spot.md](./stock_metadata/stock_zh_kcb_spot.md) |

### 历史行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-科创板股票历史行情数据 | 新浪财经-科创板股票历史行情数据 | [stock_zh_kcb_daily.md](./stock_metadata/stock_zh_kcb_daily.md) |

### 科创板公告

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-科创板报告数据 | 东方财富-科创板报告数据 | [stock_zh_kcb_report_em.md](./stock_metadata/stock_zh_kcb_report_em.md) |

---

## A+H股

### 实时行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-行情中心-沪深港通-AH股比价-实时行情 | 东方财富网-行情中心-沪深港通-AH股比价-实时行情,延迟15分钟更新 | [stock_zh_ah_spot_em.md](./stock_metadata/stock_zh_ah_spot_em.md) |
| A+H股数据-腾讯财经 | A+H股数据是从腾讯财经获取的数据,延迟15分钟更新 | [stock_zh_ah_spot.md](./stock_metadata/stock_zh_ah_spot.md) |

### 历史行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 腾讯财经-A+H股数据 | 腾讯财经-A+H股数据 | [stock_zh_ah_daily.md](./stock_metadata/stock_zh_ah_daily.md) |

### A+H股票字典

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| A+H股数据字典-腾讯财经 | A+H股数据是从腾讯财经获取的数据,历史数据按日频率更新 | [stock_zh_ah_name.md](./stock_metadata/stock_zh_ah_name.md) |

---

## 美股

### 实时行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-美股-实时行情 | 东方财富网-美股-实时行情 | [stock_us_spot_em.md](./stock_metadata/stock_us_spot_em.md) |
| 新浪财经-美股 | 新浪财经-美股;获取的数据有15分钟延迟 | [stock_us_spot.md](./stock_metadata/stock_us_spot.md) |

### 历史行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-行情-美股-每日行情 | 东方财富网-行情-美股-每日行情 | [stock_us_hist.md](./stock_metadata/stock_us_hist.md) |

### 个股信息查询

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 雪球-个股-公司概况-公司简介 | 雪球-个股-公司概况-公司简介 | [stock_individual_basic_info_us_xq.md](./stock_metadata/stock_individual_basic_info_us_xq.md) |

### 分时数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富网-行情首页-美股-每日分时行情 | 东方财富网-行情首页-美股-每日分时行情 | [stock_us_hist_min_em.md](./stock_metadata/stock_us_hist_min_em.md) |

---

## 美股-历史数据(新浪)

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-美股历史行情数据 | 美股历史行情数据，设定 adjust="qfq" 则返回前复权后的数据，默认 adjust="", 则返回未复权的数据 | [stock_us_daily.md](./stock_metadata/stock_us_daily.md) |

### 粉单市场

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-美股粉单市场实时行情 | 美股粉单市场的实时行情数据 | [stock_us_pink_spot_em.md](./stock_metadata/stock_us_pink_spot_em.md) |

### 知名美股

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-知名美股实时行情 | 美股-知名美股的实时行情数据 | [stock_us_famous_spot_em.md](./stock_metadata/stock_us_famous_spot_em.md) |

---

## 港股

### 实时行情数据-东财

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-港股实时行情 | 所有港股的实时行情数据; 该数据有 15 分钟延时 | [stock_hk_spot_em.md](./stock_metadata/stock_hk_spot_em.md) |
| 东方财富-港股主板实时行情 | 港股主板的实时行情数据; 该数据有 15 分钟延时 | [stock_hk_main_board_spot_em.md](./stock_metadata/stock_hk_main_board_spot_em.md) |
| 东方财富-知名港股实时行情 | 东方财富网-行情中心-港股市场-知名港股实时行情数据 | [stock_hk_famous_spot_em.md](./stock_metadata/stock_hk_famous_spot_em.md) |

### 实时行情数据-新浪

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-港股实时行情 | 获取所有港股的实时行情数据 15 分钟延时 | [stock_hk_spot.md](./stock_metadata/stock_hk_spot.md) |

### 个股信息查询

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 雪球-港股公司概况 | 雪球-个股-公司概况-公司简介 | [stock_individual_basic_info_hk_xq.md](./stock_metadata/stock_individual_basic_info_hk_xq.md) |

### 历史分时数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-港股每日分时行情 | 东方财富网-行情首页-港股-每日分时行情 | [stock_hk_hist_min_em.md](./stock_metadata/stock_hk_hist_min_em.md) |

### 历史行情数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-港股历史行情数据 | 港股-历史行情数据, 可以选择返回复权后数据, 更新频率为日频 | [stock_hk_hist.md](./stock_metadata/stock_hk_hist.md) |
| 新浪财经-港股历史行情数据 | 港股-历史行情数据, 可以选择返回复权后数据,更新频率为日频 | [stock_hk_daily.md](./stock_metadata/stock_hk_daily.md) |

### 公司资料

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-港股证券资料 | 东方财富-港股-证券资料 | [stock_hk_security_profile_em.md](./stock_metadata/stock_hk_security_profile_em.md) |
| 东方财富-港股公司资料 | 东方财富-港股-公司资料 | [stock_hk_company_profile_em.md](./stock_metadata/stock_hk_company_profile_em.md) |
| 东方财富-港股财务指标 | 东方财富-港股-核心必读-最新指标 | [stock_hk_financial_indicator_em.md](./stock_metadata/stock_hk_financial_indicator_em.md) |
| 东方财富-港股分红派息 | 东方财富-港股-核心必读-分红派息 | [stock_hk_dividend_payout_em.md](./stock_metadata/stock_hk_dividend_payout_em.md) |

### 行业对比

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-港股成长性对比 | 东方财富-港股-行业对比-成长性对比 | [stock_hk_growth_comparison_em.md](./stock_metadata/stock_hk_growth_comparison_em.md) |
| 东方财富-港股估值对比 | 东方财富-港股-行业对比-估值对比 | [stock_hk_valuation_comparison_em.md](./stock_metadata/stock_hk_valuation_comparison_em.md) |
| 东方财富-港股规模对比 | 东方财富-港股-行业对比-规模对比 | [stock_hk_scale_comparison_em.md](./stock_metadata/stock_hk_scale_comparison_em.md) |

---

## 机构调研

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-机构调研统计 | 东方财富网-数据中心-特色数据-机构调研-机构调研统计 | [stock_jgdy_tj_em.md](./stock_metadata/stock_jgdy_tj_em.md) |
| 东方财富-机构调研详细 | 东方财富网-数据中心-特色数据-机构调研-机构调研详细 | [stock_jgdy_detail_em.md](./stock_metadata/stock_jgdy_detail_em.md) |

---

## 主营介绍

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 同花顺-主营介绍 | 同花顺-主营介绍 | [stock_zyjs_ths.md](./stock_metadata/stock_zyjs_ths.md) |
| 东方财富-主营构成 | 东方财富网-个股-主营构成 | [stock_zygc_em.md](./stock_metadata/stock_zygc_em.md) |

---

## 股票质押

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-股权质押市场概况 | 东方财富网-数据中心-特色数据-股权质押-股权质押市场概况 | [stock_gpzy_profile_em.md](./stock_metadata/stock_gpzy_profile_em.md) |
| 东方财富-上市公司质押比例 | 东方财富网-数据中心-特色数据-股权质押-上市公司质押比例 | [stock_gpzy_pledge_ratio_em.md](./stock_metadata/stock_gpzy_pledge_ratio_em.md) |
| 东方财富-重要股东股权质押明细 | 东方财富网-数据中心-特色数据-股权质押-重要股东股权质押明细 | [stock_gpzy_pledge_ratio_detail_em.md](./stock_metadata/stock_gpzy_pledge_ratio_detail_em.md) |
| 东方财富-个股股权质押明细 | 东方财富网-数据中心-股权质押-个股 | [stock_gpzy_individual_pledge_ratio_detail_em.md](./stock_metadata/stock_gpzy_individual_pledge_ratio_detail_em.md) |
| 东方财富-质押分布统计-证券公司 | 东方财富网-数据中心-特色数据-股权质押-质押分布统计-证券公司 | [stock_gpzy_distribute_statistics_company_em.md](./stock_metadata/stock_gpzy_distribute_statistics_company_em.md) |
| 东方财富-质押分布统计-银行 | 东方财富网-数据中心-特色数据-股权质押-质押分布统计-银行 | [stock_gpzy_distribute_statistics_bank_em.md](./stock_metadata/stock_gpzy_distribute_statistics_bank_em.md) |
| 东方财富-股权质押行业数据 | 东方财富网-数据中心-特色数据-股权质押-上市公司质押比例-行业数据 | [stock_gpzy_industry_data_em.md](./stock_metadata/stock_gpzy_industry_data_em.md) |

---

## 商誉专题

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-A股商誉市场概况 | 东方财富网-数据中心-特色数据-商誉-A股商誉市场概况 | [stock_sy_profile_em.md](./stock_metadata/stock_sy_profile_em.md) |
| 东方财富-商誉减值预期明细 | 东方财富网-数据中心-特色数据-商誉-商誉减值预期明细 | [stock_sy_yq_em.md](./stock_metadata/stock_sy_yq_em.md) |
| 东方财富-个股商誉减值明细 | 东方财富网-数据中心-特色数据-商誉-个股商誉减值明细 | [stock_sy_jz_em.md](./stock_metadata/stock_sy_jz_em.md) |
| 东方财富-个股商誉明细 | 东方财富网-数据中心-特色数据-商誉-个股商誉明细 | [stock_sy_em.md](./stock_metadata/stock_sy_em.md) |
| 东方财富-行业商誉 | 东方财富网-数据中心-特色数据-商誉-行业商誉 | [stock_sy_hy_em.md](./stock_metadata/stock_sy_hy_em.md) |

---

## 股票账户统计

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-股票账户统计 | 东方财富网-数据中心-特色数据-股票账户统计 | [stock_account_statistics_em.md](./stock_metadata/stock_account_statistics_em.md) |

---

## 分析师指数

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-分析师指数 | 东方财富网-数据中心-研究报告-东方财富分析师指数 | [stock_analyst_rank_em.md](./stock_metadata/stock_analyst_rank_em.md) |
| 东方财富-分析师详情 | 东方财富网-数据中心-研究报告-东方财富分析师指数-分析师详情 | [stock_analyst_detail_em.md](./stock_metadata/stock_analyst_detail_em.md) |

---

## 千股千评

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-千股千评 | 东方财富网-数据中心-特色数据-千股千评 | [stock_comment_em.md](./stock_metadata/stock_comment_em.md) |
| 东方财富-千股千评-主力控盘 | 东方财富网-数据中心-特色数据-千股千评-主力控盘-机构参与度 | [stock_comment_detail_zlkp_jgcyd_em.md](./stock_metadata/stock_comment_detail_zlkp_jgcyd_em.md) |
| 东方财富-千股千评-综合评价 | 东方财富网-数据中心-特色数据-千股千评-综合评价-历史评分 | [stock_comment_detail_zhpj_lspf_em.md](./stock_metadata/stock_comment_detail_zhpj_lspf_em.md) |
| 东方财富-千股千评-用户关注指数 | 东方财富网-数据中心-特色数据-千股千评-市场热度-用户关注指数 | [stock_comment_detail_scrd_focus_em.md](./stock_metadata/stock_comment_detail_scrd_focus_em.md) |
| 东方财富-千股千评-市场参与意愿 | 东方财富网-数据中心-特色数据-千股千评-市场热度-市场参与意愿 | [stock_comment_detail_scrd_desire_em.md](./stock_metadata/stock_comment_detail_scrd_desire_em.md) |

---

## 沪深港通资金流向

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-沪深港通资金流向 | 东方财富网-数据中心-资金流向-沪深港通资金流向 | [stock_hsgt_fund_flow_summary_em.md](./stock_metadata/stock_hsgt_fund_flow_summary_em.md) |

---

## 停复牌信息

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-停复牌信息 | 东方财富网-数据中心-特色数据-停复牌信息 | [stock_tfp_em.md](./stock_metadata/stock_tfp_em.md) |
| 百度股市通-停复牌 | 百度股市通-交易提醒-停复牌 | [news_trade_notify_suspend_baidu.md](./stock_metadata/news_trade_notify_suspend_baidu.md) |
| 百度股市通-分红派息 | 百度股市通-交易提醒-分红派息 | [news_trade_notify_dividend_baidu.md](./stock_metadata/news_trade_notify_dividend_baidu.md) |

---

## 个股新闻

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-个股新闻资讯 | 东方财富指定个股的新闻资讯数据 | [stock_news_em.md](./stock_metadata/stock_news_em.md) |

---

## 新股数据

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 同花顺-新股申购与中签 | 同花顺-数据中心-新股申购与中签 | [stock_ipo_ths.md](./stock_metadata/stock_ipo_ths.md) |

---

## 年报季报

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-业绩报表 | 东方财富-数据中心-年报季报-业绩报表 | [stock_yjbb_em.md](./stock_metadata/stock_yjbb_em.md) |
| 巨潮资讯-预约披露 | 巨潮资讯-数据-预约披露的数据 | [stock_report_disclosure.md](./stock_metadata/stock_report_disclosure.md) |

---

## 上海证券交易所-每日概况

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 上海证券交易所-数据-股票数据-成交概况-股票成交概况-每日股票情况 | 上海证券交易所-数据-股票数据-成交概况-股票成交概况-每日股票情况 | [stock_sse_deal_daily.md](./stock_metadata/stock_sse_deal_daily.md) |

---

## 巨潮资讯

### 信息披露

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 信息披露公告-巨潮资讯 | 巨潮资讯-首页-公告查询-信息披露公告-沪深京 | [stock_zh_a_disclosure_report_cninfo.md](./stock_metadata/stock_zh_a_disclosure_report_cninfo.md) |
| 信息披露调研-巨潮资讯 | 巨潮资讯-首页-公告查询-信息披露调研-沪深京 | [stock_zh_a_disclosure_relation_cninfo.md](./stock_metadata/stock_zh_a_disclosure_relation_cninfo.md) |
| 行业分类数据-巨潮资讯 | 巨潮资讯-数据-行业分类数据 | [stock_industry_category_cninfo.md](./stock_metadata/stock_industry_category_cninfo.md) |
| 上市公司行业归属的变动情况-巨潮资讯 | 巨潮资讯-数据-上市公司行业归属的变动情况 | [stock_industry_change_cninfo.md](./stock_metadata/stock_industry_change_cninfo.md) |
| 公司股本变动-巨潮资讯 | 巨潮资讯-数据-公司股本变动 | [stock_share_change_cninfo.md](./stock_metadata/stock_share_change_cninfo.md) |
| 配股实施方案-巨潮资讯 | 巨潮资讯-个股-配股实施方案 | [stock_allotment_cninfo.md](./stock_metadata/stock_allotment_cninfo.md) |
| 公司概况-巨潮资讯 | 巨潮资讯-个股-公司概况 | [stock_profile_cninfo.md](./stock_metadata/stock_profile_cninfo.md) |
| 上市相关-巨潮资讯 | 巨潮资讯-个股-上市相关 | [stock_ipo_summary_cninfo.md](./stock_metadata/stock_ipo_summary_cninfo.md) |

---

## 财务报表

### 资产负债表

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 资产负债表-沪深 | 东方财富-数据中心-年报季报-业绩快报-资产负债表 | [stock_zcfz_em.md](./stock_metadata/stock_zcfz_em.md) |
| 资产负债表-北交所 | 东方财富-数据中心-年报季报-业绩快报-资产负债表 | [stock_zcfz_bj_em.md](./stock_metadata/stock_zcfz_bj_em.md) |

### 利润表

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 利润表 | 东方财富-数据中心-年报季报-业绩快报-利润表 | [stock_lrb_em.md](./stock_metadata/stock_lrb_em.md) |

### 现金流量表

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 现金流量表 | 东方财富-数据中心-年报季报-业绩快报-现金流量表 | [stock_xjll_em.md](./stock_metadata/stock_xjll_em.md) |

---

## 分红配送

### 股东增减持

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 股东增减持 | 东方财富网-数据中心-特色数据-高管持股 | [stock_ggcg_em.md](./stock_metadata/stock_ggcg_em.md) |

### 分红配送

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 分红配送-东财 | 东方财富-数据中心-年报季报-分红配送 | [stock_fhps_em.md](./stock_metadata/stock_fhps_em.md) |
| 分���配送详情-东财 | 东方财富网-数据中心-分红送配-分红送配详情 | [stock_fhps_detail_em.md](./stock_metadata/stock_fhps_detail_em.md) |
| 分红情况-同花顺 | 同花顺-分红情况 | [stock_fhps_detail_ths.md](./stock_metadata/stock_fhps_detail_ths.md) |
| 分红配送详情-港股-同花顺 | 同花顺-港股-分红派息 | [stock_hk_fhpx_detail_ths.md](./stock_metadata/stock_hk_fhpx_detail_ths.md) |

---

## 资金流向

### 同花顺

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 个股资金流-同花顺 | 同花顺-数据中心-资金流向-个股资金流 | [stock_fund_flow_individual.md](./stock_metadata/stock_fund_flow_individual.md) |
| 概念资金流-同花顺 | 同花顺-数据中心-资金流向-概念资金流 | [stock_fund_flow_concept.md](./stock_metadata/stock_fund_flow_concept.md) |
| 行业资金流-同花顺 | 同花顺-数据中心-资金流向-行业资金流 | [stock_fund_flow_industry.md](./stock_metadata/stock_fund_flow_industry.md) |
| 大单追踪-同花顺 | 同花顺-数据中心-资金流向-大单追踪 | [stock_fund_flow_big_deal.md](./stock_metadata/stock_fund_flow_big_deal.md) |

### 东方财富

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 个股资金流-东方财富 | 东方财富网-数据中心-个股资金流向 | [stock_individual_fund_flow.md](./stock_metadata/stock_individual_fund_flow.md) |
| 个股资金流排名-东方财富 | 东方财富网-数据中心-资金流向-排名 | [stock_individual_fund_flow_rank.md](./stock_metadata/stock_individual_fund_flow_rank.md) |
| 大盘资金流-东方财富 | 东方财富网-数据中心-资金流向-大盘 | [stock_market_fund_flow.md](./stock_metadata/stock_market_fund_flow.md) |
| 板块资金流排名-东方财富 | 东方财富网-数据中心-资金流向-板块资金流-排名 | [stock_sector_fund_flow_rank.md](./stock_metadata/stock_sector_fund_flow_rank.md) |
| 主力净流入排名-东方财富 | 东方财富网-数据中心-资金流向-主力净流入排名 | [stock_main_fund_flow.md](./stock_metadata/stock_main_fund_flow.md) |
| 行业个股资金流-东方财富 | 东方财富网-数据中心-资金流向-行业资金流-xx行业个股资金流 | [stock_sector_fund_flow_summary.md](./stock_metadata/stock_sector_fund_flow_summary.md) |
| 行业历史资金流-东方财富 | 东方财富网-数据中心-资金流向-行业资金流-行业历史资金流 | [stock_sector_fund_flow_hist.md](./stock_metadata/stock_sector_fund_flow_hist.md) |
| 概念历史资金流-东方财富 | 东方财富网-数据中心-资金流向-概念资金流-概念历史资金流 | [stock_concept_fund_flow_hist.md](./stock_metadata/stock_concept_fund_flow_hist.md) |

---

## 筹码分布

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 筹码分布-东方财富 | 东方财富网-概念板-行情中心-日K-筹码分布 | [stock_cyq_em.md](./stock_metadata/stock_cyq_em.md) |

---

## 基本面数据

### 股东大会

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 股东大会-东方财富 | 东方财富网-数据中心-股东大会 | [stock_gddh_em.md](./stock_metadata/stock_gddh_em.md) |

### 重大合同

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 重大合同-东方财富 | 东方财富网-数据中心-重大合同-重大合同明细 | [stock_zdhtmx_em.md](./stock_metadata/stock_zdhtmx_em.md) |

### 个股研报

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 个股研报-东方财富 | 东方财富网-数据中心-研究报告-个股研报 | [stock_research_report_em.md](./stock_metadata/stock_research_report_em.md) |

### 沪深京A股公告

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 沪深京A股公告-��方财富 | 东方财富网-数据中心-公告大全-沪深京A股公告 | [stock_notice_report.md](./stock_metadata/stock_notice_report.md) |
| 沪深京A股个股公告-东方财富 | 东方财富网-数据中心-公告大全-个股 | [stock_individual_notice_report.md](./stock_metadata/stock_individual_notice_report.md) |

---

## 互动平台

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 互动易-提问 | 互动易-提问 | [stock_irm_cninfo.md](./stock_metadata/stock_irm_cninfo.md) |
| 互动易-回答 | 互动易-回答 | [stock_irm_ans_cninfo.md](./stock_metadata/stock_irm_ans_cninfo.md) |
| 上证e互动 | 上证e互动-提问与回答 | [stock_sns_sseinfo.md](./stock_metadata/stock_sns_sseinfo.md) |

---

## 股票热度

### 个股人气榜-实时变动

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 个股人气榜-实时变动A股 | 东方财富网-个股人气榜-实时变动 | [stock_hot_rank_detail_realtime_em.md](./stock_metadata/stock_hot_rank_detail_realtime_em.md) |
| 个股人气榜-实时变动港股 | 东方财富网-个股人气榜-实时变动 | [stock_hk_hot_rank_detail_realtime_em.md](./stock_metadata/stock_hk_hot_rank_detail_realtime_em.md) |

### 个股人气榜-最新排名

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 个股人气榜-最新排名A股 | 东方财富-个股人气榜-最新排名 | [stock_hot_rank_latest_em.md](./stock_metadata/stock_hot_rank_latest_em.md) |
| 个股人气榜-最新排名港股 | 东方财富-个股人气榜-最新排名 | [stock_hk_hot_rank_latest_em.md](./stock_metadata/stock_hk_hot_rank_latest_em.md) |

### 热门关键词

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 热门关键词 | 东方财富-个股人气榜-热门关键词 | [stock_hot_keyword_em.md](./stock_metadata/stock_hot_keyword_em.md) |

### 相关股票

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 相关股票 | 东方财富-个股人气榜-相关股票 | [stock_hot_rank_relate_em.md](./stock_metadata/stock_hot_rank_relate_em.md) |

### 热搜股票

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 热搜股票 | 百度股市通-热搜股票 | [stock_hot_search_baidu.md](./stock_metadata/stock_hot_search_baidu.md) |

### 内部交易

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 内部交易 | 雪球-行情中心-沪深股市-内部交易 | [stock_inner_trade_xq.md](./stock_metadata/stock_inner_trade_xq.md) |

---

## 盘口异动

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 盘口异动 | 东方财富-行情中心-盘口异动数据 | [stock_changes_em.md](./stock_metadata/stock_changes_em.md) |
| 板块异动详情 | 东方财富-行情中心-当日板块异动详情 | [stock_board_change_em.md](./stock_metadata/stock_board_change_em.md) |

---

## 涨停板行情

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 涨停股池 | 东方财富网-行情中心-涨停板行情-涨停股池 | [stock_zt_pool_em.md](./stock_metadata/stock_zt_pool_em.md) |
| 昨日涨停股池 | 东方财富网-行情中心-涨停板行情-昨日涨停股池 | [stock_zt_pool_previous_em.md](./stock_metadata/stock_zt_pool_previous_em.md) |
| 强势股池 | 东方财富网-行情中心-涨停板行情-强势股池 | [stock_zt_pool_strong_em.md](./stock_metadata/stock_zt_pool_strong_em.md) |
| 次新股池 | 东方财富网-行情中心-涨停板行情-次新股池 | [stock_zt_pool_sub_new_em.md](./stock_metadata/stock_zt_pool_sub_new_em.md) |
| 炸板股池 | 东方财富网-行情中心-涨停板行情-炸板股池 | [stock_zt_pool_zbgc_em.md](./stock_metadata/stock_zt_pool_zbgc_em.md) |
| 跌停股池 | 东方财富网-行情中心-涨停板行情-跌停股池 | [stock_zt_pool_dtgc_em.md](./stock_metadata/stock_zt_pool_dtgc_em.md) |

---

## 赚钱效应分析

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 赚钱效应分析 | 乐咕乐股网-赚钱效应分析数据 | [stock_market_activity_legu.md](./stock_metadata/stock_market_activity_legu.md) |

---

## 资讯数据

### 财经早餐

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 财经早餐-东财财富 | 东方财富-财经早餐 | [stock_info_cjzc_em.md](./stock_metadata/stock_info_cjzc_em.md) |

### 全球财经快讯

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 全球财经快讯-东财财富 | 东方财富-全球财经快讯 | [stock_info_global_em.md](./stock_metadata/stock_info_global_em.md) |
| 全球财经快讯-新浪财经 | 新浪财经-全球财经快讯 | [stock_info_global_sina.md](./stock_metadata/stock_info_global_sina.md) |
| 快讯-富途牛牛 | 富途牛牛-快讯 | [stock_info_global_futu.md](./stock_metadata/stock_info_global_futu.md) |
| 全球财经直播-同花顺 | 同花顺财经-全球财经直播 | [stock_info_global_ths.md](./stock_metadata/stock_info_global_ths.md) |
| 电报-财联社 | 财联社-电报 | [stock_info_global_cls.md](./stock_metadata/stock_info_global_cls.md) |

---

## 技术指标

### 创新高/创新低

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 创新高 | 同花顺-数据中心-技术选股-创新高 | [stock_rank_cxg_ths.md](./stock_metadata/stock_rank_cxg_ths.md) |
| 创新低 | 同花顺-数据中心-技术选股-创新低 | [stock_rank_cxd_ths.md](./stock_metadata/stock_rank_cxd_ths.md) |

### 连续上涨/下跌

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 连续上涨 | 同花顺-数据中心-技术选股-连续上涨 | [stock_rank_lxsz_ths.md](./stock_metadata/stock_rank_lxsz_ths.md) |
| 连续下跌 | 同花顺-数据中心-技术选股-连续下跌 | [stock_rank_lxxd_ths.md](./stock_metadata/stock_rank_lxxd_ths.md) |

### 持续放量/缩量

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 持续放量 | 同花顺-数据中心-技术选股-持续放量 | [stock_rank_cxfl_ths.md](./stock_metadata/stock_rank_cxfl_ths.md) |
| 持续缩量 | 同花顺-数据中心-技术选股-持续缩量 | [stock_rank_cxsl_ths.md](./stock_metadata/stock_rank_cxsl_ths.md) |

### 突破

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 向上突破 | 同花顺-数据中心-技术选股-向上突破 | [stock_rank_xstp_ths.md](./stock_metadata/stock_rank_xstp_ths.md) |
| 向下突破 | 同花顺-数据中心-技术选股-向下突破 | [stock_rank_xxtp_ths.md](./stock_metadata/stock_rank_xxtp_ths.md) |

### 量价关系

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 量价齐升 | 同花顺-数据中心-技术选股-量价齐升 | [stock_rank_ljqs_ths.md](./stock_metadata/stock_rank_ljqs_ths.md) |
| 量价齐跌 | 同花顺-数据中心-技术选股-量价齐跌 | [stock_rank_ljqd_ths.md](./stock_metadata/stock_rank_ljqd_ths.md) |

### 险资举牌

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 险资举牌 | 同花顺-数据中心-技术选股-险资举牌 | [stock_rank_xzjp_ths.md](./stock_metadata/stock_rank_xzjp_ths.md) |

---

## ESG评级

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| ESG评级数据 | 新浪财经-ESG评级中心-ESG评级-ESG评级数据 | [stock_esg_rate_sina.md](./stock_metadata/stock_esg_rate_sina.md) |
| MSCI ESG评级 | 新浪财经-ESG评级中心-ESG评级-MSCI | [stock_esg_msci_sina.md](./stock_metadata/stock_esg_msci_sina.md) |
| 路孚特 ESG评级 | 新浪财经-ESG评级中心-ESG评级-路孚特 | [stock_esg_rft_sina.md](./stock_metadata/stock_esg_rft_sina.md) |
| 秩鼎 ESG评级 | 新浪财经-ESG评级中心-ESG评级-秩鼎 | [stock_esg_zd_sina.md](./stock_metadata/stock_esg_zd_sina.md) |
| 华证指数 ESG评级 | 新浪财经-ESG评级中心-ESG评级-华证指数 | [stock_esg_hz_sina.md](./stock_metadata/stock_esg_hz_sina.md) |


---

## 基金持股明细

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-基金持股明细 | 东方财富网-数据中心-主力数据-基金持仓-基金持仓明细表 | [stock_report_fund_hold_detail.md](./stock_metadata/stock_report_fund_hold_detail.md) |

---

## 龙虎榜

### 龙虎榜-东财

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-龙虎榜详情 | 东方财富网-数据中心-龙虎榜单-龙虎榜详情 | [stock_lhb_detail_em.md](./stock_metadata/stock_lhb_detail_em.md) |
| 东方财富-龙虎榜个股上榜统计 | 东方财富网-数据中心-龙虎榜单-个股上榜统计 | [stock_lhb_stock_statistic_em.md](./stock_metadata/stock_lhb_stock_statistic_em.md) |
| 东方财富-龙虎榜机构买卖每日统计 | 东方财富网-数据中心-龙虎榜单-机构买卖每日统计 | [stock_lhb_jgmmtj_em.md](./stock_metadata/stock_lhb_jgmmtj_em.md) |
| 东方财富-龙虎榜机构席位追踪 | 东方财富网-数据中心-龙虎榜单-机构席位追踪 | [stock_lhb_jgstatistic_em.md](./stock_metadata/stock_lhb_jgstatistic_em.md) |
| 东方财富-龙虎榜每日活跃营业部 | 东方财富网-数据中心-龙虎榜单-每日活跃营业部 | [stock_lhb_hyyyb_em.md](./stock_metadata/stock_lhb_hyyyb_em.md) |
| 东方财富-龙虎榜营业部详情数据 | 东方财富网-数据中心-龙虎榜单-营业部历史交易明细-营业部交易明细 | [stock_lhb_yyb_detail_em.md](./stock_metadata/stock_lhb_yyb_detail_em.md) |
| 东方财富-龙虎榜营业部排行 | 东方财富网-数据中心-龙虎榜单-营业部排行 | [stock_lhb_yybph_em.md](./stock_metadata/stock_lhb_yybph_em.md) |
| 东方财富-龙虎榜营业部统计 | 东方财富网-数据中心-龙虎榜单-营业部统计 | [stock_lhb_traderstatistic_em.md](./stock_metadata/stock_lhb_traderstatistic_em.md) |
| 东方财富-龙虎榜个股龙虎榜详情 | 东方财富网-数据中心-龙虎榜单-个股龙虎榜详情 | [stock_lhb_stock_detail_em.md](./stock_metadata/stock_lhb_stock_detail_em.md) |

### 龙虎榜-同花顺

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 同花顺-龙虎榜营业部排行-上榜次数最多 | 龙虎榜-营业部排行-上榜次数最多 | [stock_lh_yyb_most.md](./stock_metadata/stock_lh_yyb_most.md) |
| 同花顺-龙虎榜营业部排行-资金实力最强 | 龙虎榜-营业部排行-资金实力最强 | [stock_lh_yyb_capital.md](./stock_metadata/stock_lh_yyb_capital.md) |
| 同花顺-龙虎榜营业部排行-抱团操作实力 | 龙虎榜-营业部排行-抱团操作实力 | [stock_lh_yyb_control.md](./stock_metadata/stock_lh_yyb_control.md) |

### 龙虎榜-新浪

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 新浪财经-龙虎榜每日详情 | 新浪财经-龙虎榜-每日详情 | [stock_lhb_detail_daily_sina.md](./stock_metadata/stock_lhb_detail_daily_sina.md) |
| 新浪财经-龙虎榜个股上榜统计 | 新浪财经-龙虎榜-个股上榜统计 | [stock_lhb_ggtj_sina.md](./stock_metadata/stock_lhb_ggtj_sina.md) |
| 新浪财经-龙虎榜营业上榜统计 | 新浪财经-龙虎榜-营业上榜统计 | [stock_lhb_yytj_sina.md](./stock_metadata/stock_lhb_yytj_sina.md) |
| 新浪财经-龙虎榜机构席位追踪 | 新浪财经-龙虎榜-机构席位追踪 | [stock_lhb_jgzz_sina.md](./stock_metadata/stock_lhb_jgzz_sina.md) |
| 新浪财经-龙虎榜机构席位成交明细 | 新浪财经-龙虎榜-机构席位成交明细 | [stock_lhb_jgmx_sina.md](./stock_metadata/stock_lhb_jgmx_sina.md) |

---

## IPO数据

### 首发申报

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-首发申报信息 | 东方财富网-数据中心-新股申购-首发申报信息-首发申报企业信息 | [stock_ipo_declare_em.md](./stock_metadata/stock_ipo_declare_em.md) |

### IPO审核信息

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-IPO审核信息-全部 | 东方财富网-数据中心-新股数据-IPO审核信息-全部 | [stock_register_all_em.md](./stock_metadata/stock_register_all_em.md) |
| 东方财富-IPO审核信息-科创板 | 东方财富网-数据中心-新股数据-IPO审核信息-科创板 | [stock_register_kcb.md](./stock_metadata/stock_register_kcb.md) |
| 东方财富-IPO审核信息-创业板 | 东方财富网-数据中心-新股数据-IPO审核信息-创业板 | [stock_register_cyb.md](./stock_metadata/stock_register_cyb.md) |
| 东方财富-IPO审核信息-上海主板 | 东方财富网-数据中心-新股数据-IPO审核信息-上海主板 | [stock_register_sh.md](./stock_metadata/stock_register_sh.md) |
| 东方财富-IPO审核信息-深圳主板 | 东方财富网-数据中心-新股数据-IPO审核信息-深圳主板 | [stock_register_sz.md](./stock_metadata/stock_register_sz.md) |
| 东方财富-IPO审核信息-北交所 | 东方财富网-数据中心-新股数据-IPO审核信息-北交所 | [stock_register_bj.md](./stock_metadata/stock_register_bj.md) |
| 东方财富-注册制审核-达标企业 | 东方财富网-数据中心-新股数据-注册制审核-达标企业 | [stock_register_db.md](./stock_metadata/stock_register_db.md) |

---

## 增发配股

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-全部增发 | 东方财富网-数据中心-新股数据-增发-全部增发 | [stock_qbzf_em.md](./stock_metadata/stock_qbzf_em.md) |
| 东方财富-配股 | 东方财富网-数据中心-新股数据-配股 | [stock_pg_em.md](./stock_metadata/stock_pg_em.md) |
| 东方财富-股票回购数据 | 东方财富网-数据中心-股票回购-股票回购数据 | [stock_repurchase_em.md](./stock_metadata/stock_repurchase_em.md) |

---

## 大宗交易

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-大宗交易市场统计 | 东方财富网-数据中心-大宗交易-市场统计 | [stock_dzjy_sctj.md](./stock_metadata/stock_dzjy_sctj.md) |
| 东方财富-大宗交易每日明细 | 东方财富网-数据中心-大宗交易-每日明细 | [stock_dzjy_mrmx.md](./stock_metadata/stock_dzjy_mrmx.md) |
| 东方财富-大宗交易每日统计 | 东方财富网-数据中心-大宗交易-每日统计 | [stock_dzjy_mrtj.md](./stock_metadata/stock_dzjy_mrtj.md) |
| 东方财富-大宗交易活跃A股统计 | 东方财富网-数据中心-大宗交易-活跃A股统计 | [stock_dzjy_hygtj.md](./stock_metadata/stock_dzjy_hygtj.md) |
| 东方财富-大宗交易活跃营业部统计 | 东方财富网-数据中心-大宗交易-活跃营业部统计 | [stock_dzjy_hyyybtj.md](./stock_metadata/stock_dzjy_hyyybtj.md) |
| 东方财富-大宗交易营业部排行 | 东方财富网-数据中心-大宗交易-营业部排行 | [stock_dzjy_yybph.md](./stock_metadata/stock_dzjy_yybph.md) |

---

## 股本结构

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-A股股本结构 | 东方财富-A股数据-股本结构 | [stock_zh_a_gbjg_em.md](./stock_metadata/stock_zh_a_gbjg_em.md) |

---

## 一致行动人

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-一致行动人 | 东方财富网-数据中心-特色数据-一致行动人 | [stock_yzxdr_em.md](./stock_metadata/stock_yzxdr_em.md) |

---

## 融资融券

### 融资融券账户

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 融资融券-标的证券名单及保证金比例查询 | 融资融券-标的证券名单及保证金比例查询 | [stock_margin_ratio_pa.md](./stock_metadata/stock_margin_ratio_pa.md) |
| 东方财富-两融账户信息 | 东方财富网-数据中心-融资融券-融资融券账户统计-两融账户信息 | [stock_margin_account_info.md](./stock_metadata/stock_margin_account_info.md) |

### 上海证券交易所

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 上海证券交易所-融资融券汇总 | 上海证券交易所-融资融券数据-融资融券汇总数据 | [stock_margin_sse.md](./stock_metadata/stock_margin_sse.md) |
| 上海证券交易所-融资融券明细 | 上海证券交易所-融资融券数据-融资融券明细数据 | [stock_margin_detail_sse.md](./stock_metadata/stock_margin_detail_sse.md) |

### 深圳证券交易所

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 深圳证券交易所-融资融券汇总 | 深圳证券交易所-融资融券数据-融资融券汇总数据 | [stock_margin_szse.md](./stock_metadata/stock_margin_szse.md) |
| 深圳证券交易所-融资融券明细 | 深证证券交易所-融资融券数据-融资融券交易明细数据 | [stock_margin_detail_szse.md](./stock_metadata/stock_margin_detail_szse.md) |
| 深圳证券交易所-标的证券信息 | 深圳证券交易所-融资融券数据-标的证券信息 | [stock_margin_underlying_info_szse.md](./stock_metadata/stock_margin_underlying_info_szse.md) |

---

## 盈利预测

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-盈利预测 | 东方财富网-数据中心-研究报告-盈利预测 | [stock_profit_forecast_em.md](./stock_metadata/stock_profit_forecast_em.md) |
| 经济通-港股盈利预测 | 经济通-公司资料-盈利预测 | [stock_hk_profit_forecast_et.md](./stock_metadata/stock_hk_profit_forecast_et.md) |
| 同花顺-盈利预测 | 同花顺-盈利预测 | [stock_profit_forecast_ths.md](./stock_metadata/stock_profit_forecast_ths.md) |

---

## 概念板块

### 同花顺

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 同花顺-概念板块指数 | 同花顺-板块-概念板块-指数日频率数据 | [stock_board_concept_index_ths.md](./stock_metadata/stock_board_concept_index_ths.md) |
| 同花顺-概念板块简介 | 同花顺-板块-概念板块-板块简介 | [stock_board_concept_info_ths.md](./stock_metadata/stock_board_concept_info_ths.md) |

### 东方财富

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-概念板块 | 东方财富网-行情中心-沪深京板块-概念板块 | [stock_board_concept_name_em.md](./stock_metadata/stock_board_concept_name_em.md) |
| 东方财富-概念板块实时行情 | 东方财富网-行情中心-沪深京板块-概念板块-实时行情 | [stock_board_concept_spot_em.md](./stock_metadata/stock_board_concept_spot_em.md) |
| 东方财富-概念板块成份股 | 东方财富-沪深板块-概念板块-板块成份 | [stock_board_concept_cons_em.md](./stock_metadata/stock_board_concept_cons_em.md) |
| 东方财富-概念板块指数历史 | 东方财富-沪深板块-概念板块-历史行情数据 | [stock_board_concept_hist_em.md](./stock_metadata/stock_board_concept_hist_em.md) |
| 东方财富-概念板块指数分时 | 东方财富-沪深板块-概念板块-分时历史行情数据 | [stock_board_concept_hist_min_em.md](./stock_metadata/stock_board_concept_hist_min_em.md) |

### 富途牛牛

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 富途牛牛-美股概念成分股 | 富途牛牛-主题投资-概念板块-成分股 | [stock_concept_cons_futu.md](./stock_metadata/stock_concept_cons_futu.md) |

---

## 行业板块

### 同花顺

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 同花顺-行业板块一览表 | 同花顺-同花顺行业一览表 | [stock_board_industry_summary_ths.md](./stock_metadata/stock_board_industry_summary_ths.md) |
| 同花顺-行业板块指数 | 同花顺-板块-行业板块-指数日频率数据 | [stock_board_industry_index_ths.md](./stock_metadata/stock_board_industry_index_ths.md) |

### 东方财富

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-行业板块 | 东方财富-沪深京板块-行业板块 | [stock_board_industry_name_em.md](./stock_metadata/stock_board_industry_name_em.md) |
| 东方财富-行业板块实时行情 | 东方财富网-沪深板块-行业板块-实时行情 | [stock_board_industry_spot_em.md](./stock_metadata/stock_board_industry_spot_em.md) |
| 东方财富-行业板块成份股 | 东方财富-沪深板块-行业板块-板块成份 | [stock_board_industry_cons_em.md](./stock_metadata/stock_board_industry_cons_em.md) |
| 东方财富-行业板块指数历史 | 东方财富-沪深板块-行业板块-历史行情数据 | [stock_board_industry_hist_em.md](./stock_metadata/stock_board_industry_hist_em.md) |
| 东方财富-行业板块指数分时 | 东方财富-沪深板块-行业板块-分时历史行情数据 | [stock_board_industry_hist_min_em.md](./stock_metadata/stock_board_industry_hist_min_em.md) |

---

## 股票热度

### 雪球

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 雪球-股票热度关注排行榜 | 雪球-沪深股市-热度排行榜-关注排行榜 | [stock_hot_follow_xq.md](./stock_metadata/stock_hot_follow_xq.md) |
| 雪球-股票热度讨论排行榜 | 雪球-沪深股市-热度排行榜-讨论排行榜 | [stock_hot_tweet_xq.md](./stock_metadata/stock_hot_tweet_xq.md) |
| 雪球-股票热度交易排行榜 | 雪球-沪深股市-热度排行榜-交易排行榜 | [stock_hot_deal_xq.md](./stock_metadata/stock_hot_deal_xq.md) |

### 东方财富

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 东方财富-股票热度人气榜A股 | 东方财富网站-股票热度 | [stock_hot_rank_em.md](./stock_metadata/stock_hot_rank_em.md) |
| 东方财富-股票热度飙升榜A股 | 东方财富-个股人气榜-飙升榜 | [stock_hot_up_em.md](./stock_metadata/stock_hot_up_em.md) |
| 东方财富-股票热度人气榜港股 | 东方财富-个股人气榜-人气榜-港股市场 | [stock_hk_hot_rank_em.md](./stock_metadata/stock_hk_hot_rank_em.md) |
| 东方财富-股票热度历史趋势及粉丝特征A股 | 东方财富网-股票热度-历史趋势及粉丝特征 | [stock_hot_rank_detail_em.md](./stock_metadata/stock_hot_rank_detail_em.md) |
| 东方财富-股票热度历史趋势港股 | 东方财富网-股票热度-历史趋势 | [stock_hk_hot_rank_detail_em.md](./stock_metadata/stock_hk_hot_rank_detail_em.md) |

### 互动平台

| 接口标题 | 接口描述 | 详细元数据 |
|---------|---------|-----------|
| 互动易-提问 | 互动易-提问 | [stock_irm_cninfo.md](./stock_metadata/stock_irm_cninfo.md) |

---
> Source: [saberwen1/astock-data-skill](https://github.com/saberwen1/astock-data-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
