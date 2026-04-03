# VERIFY_ALPHAEAR_NEWS

## 1）安装步骤

```bash
# 进入仓库
cd Awesome-finance-skills

# 创建并激活虚拟环境
python -m venv .venv
source .venv/bin/activate

# 安装最小依赖
pip install -U pip
pip install requests loguru
```

## 2）验证步骤

```bash
# 运行内置最小测试（仅验证初始化）
python -m unittest skills/alphaear-news/tests/test_news.py

# 手工验证抓取热点新闻
python - <<'PY'
import sys
sys.path.append('skills/alphaear-news')

from scripts.database_manager import DatabaseManager
from scripts.news_tools import NewsNowTools

db = DatabaseManager(':memory:')
tool = NewsNowTools(db)
items = tool.fetch_hot_news('cls', count=3)

print('count=', len(items))
if items:
    print('first_title=', items[0].get('title', ''))
PY
```

## 3）预期输出

- 单元测试命令返回 `OK`（1 个测试通过）。
- 手工验证命令输出：
  - `count=` 后跟一个大于等于 `0` 的整数；
  - 当网络可用且接口正常时，通常 `count > 0`，并打印 `first_title=`。
- 若网络不可用或接口限流，可能出现 `count=0`，但脚本应能正常结束，不应崩溃。
