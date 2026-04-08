# LOCAL_RUN_ALPHAEAR_NEWS

> 说明：本文仅基于仓库静态分析整理，适用于“本地网络与 pip 正常可用”的 Python 环境。

## 1）alphaear-news 运行流程（静态梳理）

### A. 初始化阶段
1. 创建 `DatabaseManager`（默认数据库路径为 `data/signal_flux.db`，会自动建表 `daily_news`）。
2. 创建 `NewsNowTools(db)`，内部初始化 `ContentExtractor` 与简单内存缓存。
3. 如需预测市场数据，可创建 `PolymarketTools(db)`。

### B. 新闻获取流程（`fetch_hot_news`）
1. 先检查 5 分钟缓存（`source_id + count`）。
2. 未命中缓存时，调用 NewsNow 接口：
   - `GET https://newsnow.busiyi.world/api/s?id=<source_id>`
3. 解析 `items`，组装标准化字段：`id/source/rank/title/url/content/publish_time/meta_data`。
4. 如果 `fetch_content=True` 且有 URL，调用 `ContentExtractor.extract_with_jina(url)` 抓正文。
5. 将结果写入本地 SQLite（`save_daily_news`），并返回 `List[Dict]`。
6. 失败时返回空列表；若有旧缓存则会回退到旧缓存。

### C. 热点汇总流程（`get_unified_trends`）
1. 遍历多个来源（默认 `weibo/zhihu/wallstreetcn`）。
2. 逐个调用 `fetch_hot_news` 拉取热点。
3. 生成 Markdown 报告字符串返回。

### D. 预测市场流程（`PolymarketTools`）
1. 调用 `GET https://gamma-api.polymarket.com/markets` 获取活跃市场。
2. 解析字段（`question/outcomePrices/volume` 等）。
3. 可返回结构化列表（`get_active_markets`）或格式化摘要（`get_market_summary`）。

---

## 2）输入、输出、依赖文件

### 输入
- `fetch_hot_news(source_id, count=15, fetch_content=False)`
  - `source_id`：新闻源 ID（如 `cls`、`wallstreetcn`、`weibo` 等）
  - `count`：返回条数
  - `fetch_content`：是否抓取正文
- `get_unified_trends(sources=None)`
  - `sources`：来源列表（可为空，使用默认值）
- `fetch_news_content(url)`
  - `url`：网页地址
- `get_active_markets(limit=20)` / `get_market_summary(limit=10)`
  - `limit`：市场条数

### 输出
- `fetch_hot_news`：`List[Dict]`（每条含 `id/source/rank/title/url/content/publish_time/meta_data`）
- `fetch_news_content`：`str | None`
- `get_unified_trends`：Markdown 文本（`str`）
- `get_active_markets`：`List[Dict]`
- `get_market_summary`：格式化报告（`str`）

### 依赖文件（alphaear-news 内）
- `skills/alphaear-news/SKILL.md`：能力说明与依赖声明
- `skills/alphaear-news/scripts/news_tools.py`：主流程（新闻 + Polymarket）
- `skills/alphaear-news/scripts/content_extractor.py`：Jina 正文提取与限流
- `skills/alphaear-news/scripts/database_manager.py`：SQLite 存储
- `skills/alphaear-news/references/sources.md`：支持的 `source_id` 列表
- `skills/alphaear-news/tests/test_news.py`：最小初始化测试

---

## 3）在正常本地 Python 环境中的安装与验证

### 3.1 安装步骤

```bash
# 1) 进入仓库
cd Awesome-finance-skills

# 2) 创建虚拟环境
python -m venv .venv
source .venv/bin/activate

# 3) 升级 pip
pip install -U pip

# 4) 安装最小依赖
pip install requests loguru
```

> 可选：若你要频繁做正文抓取，可设置 `JINA_API_KEY`（不设也可运行）。

```bash
export JINA_API_KEY="<your_key>"
```

### 3.2 验证步骤

#### 步骤 A：运行内置最小测试

```bash
python -m unittest skills/alphaear-news/tests/test_news.py
```

#### 步骤 B：手工烟雾验证（拉取 3 条新闻）

```bash
python - <<'PY'
import sys
sys.path.append('skills/alphaear-news')

from scripts.database_manager import DatabaseManager
from scripts.news_tools import NewsNowTools

# 使用内存数据库，避免污染本地文件
db = DatabaseManager(':memory:')
news = NewsNowTools(db)

items = news.fetch_hot_news('cls', count=3)
print('count=', len(items))
if items:
    print('first_title=', items[0].get('title', ''))
PY
```

#### 步骤 C（可选）：验证综合汇总输出

```bash
python - <<'PY'
import sys
sys.path.append('skills/alphaear-news')

from scripts.database_manager import DatabaseManager
from scripts.news_tools import NewsNowTools

report = NewsNowTools(DatabaseManager(':memory:')).get_unified_trends(['cls','weibo'])
print(report[:500])
PY
```

---

## 4）预期结果（本地环境正常时）

- 步骤 A：单元测试输出 `OK`（1 test）。
- 步骤 B：打印 `count= <整数>`；通常网络正常时 `count > 0`，并看到 `first_title=`。
- 步骤 C：输出以 `# 实时全网热点汇总` 开头的 Markdown 内容。
- 若外部接口临时异常，`fetch_hot_news` 可能返回空列表，但程序应可正常退出。
