# 通过 bangumi.pro 反代访问（免代理方案）

## 背景

原插件硬编码 `api.bgm.tv` 与 `bgm.tv`。在中国大陆部分网络环境下，这两个域名会遭遇
**SNI 阻断**（TLS 握手中的 SNI 字段被重置），典型表现：

- DNS 解析正常，甚至能拿到正确的 IP
- 换成正确 IP 直连依然超时或被 reset
- 只有挂代理才能访问

本分支将数据源切换到社区维护的反代站点 `bangumi.pro`，从而**无需任何代理**即可使用。

## 改动

| 文件 | 原值 | 现值 |
| --- | --- | --- |
| `bangumi_api.py` | `API_BASE = "https://api.bgm.tv/v0"` | `API_BASE = "https://api.bangumi.pro/v0"` |
| `bangumi_html.py` | `BGM_BASE = "https://bgm.tv"` | `BGM_BASE = "https://bangumi.pro"` |

共两行，均在行尾保留了原值注释。

## 注意：网页侧需要浏览器 UA

`bangumi.pro` 的**网页**有反爬校验，非浏览器 UA 会返回 `403`；
API 侧（`api.bangumi.pro`）**不校验 UA**。

因此使用本分支时，请在 `config.toml` 中把 `user_agent` 改为浏览器 UA：

```toml
[request]
timeout = 20
user_agent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36"
proxy = ""
```

（`config.toml` 在本仓库的 `.gitignore` 中，属用户本地配置，不入库。）

## 覆盖范围

| 能力 | 数据源 | 状态 |
| --- | --- | --- |
| 搜索 / 详情 / 制作阵容 / 剧集 / 关联作品 / 分类浏览 | `api.bangumi.pro` | ✅ |
| 每日放送 / 单集吐槽 / 长评列表 | `bangumi.pro` 网页 | ✅（需浏览器 UA） |

## 验证方式

```bash
# API 侧
curl https://api.bangumi.pro/v0/subjects/1

# 网页侧（注意 UA）
curl -A "Mozilla/5.0 ... Chrome/122.0.0.0 Safari/537.36" https://bangumi.pro/calendar
```

## 上游更新

本分支基于上游 `main`。上游更新时若 `git pull` 覆盖了这两行，
从上表改回即可（改动仅两行）。
