# START HERE

## 1）这个仓库是什么

`Awesome-finance-skills` 是一个可插拔的金融分析技能集合，用于给 AI Agent 增加财经能力，例如：实时新闻聚合、股票数据查询、情绪分析、逻辑链路可视化与市场预测。

## 2）alphaear-news 怎么安装

### 方式 A：一键安装（推荐）

```bash
npx skills add RKiding/Awesome-finance-skills@alphaear-news
```

### 方式 B：手动安装

```bash
# 1. 克隆仓库
git clone https://github.com/RKiding/Awesome-finance-skills.git

# 2. 复制 skill 到 Codex 技能目录
mkdir -p ~/.codex/skills
cp -r Awesome-finance-skills/skills/alphaear-news ~/.codex/skills/
```

## 3）alphaear-news 怎么验证是否工作

### 第一步：安装最小依赖

```bash
python -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install requests loguru
```

### 第二步：运行内置测试

```bash
python -m unittest skills/alphaear-news/tests/test_news.py
```

### 第三步：手工验证获取新闻

```bash
python - <<'PY'
import sys
sys.path.append('skills/alphaear-news')
from scripts.database_manager import DatabaseManager
from scripts.news_tools import NewsNowTools

db = DatabaseManager(':memory:')
tools = NewsNowTools(db)
items = tools.fetch_hot_news('cls', count=3)
print('新闻条数:', len(items))
print('首条标题:', items[0]['title'] if items else '无数据')
PY
```

如果能看到“新闻条数 > 0”或测试正常通过，即说明 `alphaear-news` 基本可用。
