# Scrapling 爬虫工具 - 使用决策与操作指南

## ⚡ 触发判别规则（何时必须使用 Scrapling）

遇到以下场景时，**必须使用 Scrapling** 而非 web_search 或 webFetch：

1. **需要从官网提取产品图片直链**：手机厂商官网（mi.com、apple.com、huawei.com、oppo.com 等）的产品图片通常是 JS 动态渲染的，webFetch 只能拿到空壳 HTML，必须用 StealthyFetcher
2. **webFetch 返回内容为空/只有导航/JS 框架壳**：说明目标页面依赖 JS 渲染，需要切换到 Scrapling
3. **需要绕过反爬保护**：Cloudflare、WAF、人机验证等场景
4. **需要提取懒加载内容**：页面需要滚动或等待才能加载的图片、数据
5. **需要批量抓取多个页面的结构化数据**：如竞品参数对比、价格监控
6. **用户明确要求使用爬虫/抓取/Scrapling**

### 不需要使用 Scrapling 的场景
- 纯文本信息搜索 → 用 web_search
- 静态页面内容获取 → 用 webFetch
- API 接口调用 → 直接用 bash curl

## 环境信息

- 已安装版本：0.2.99
- Python 路径：python3
- Camoufox 浏览器：已安装（v135.0.1-beta.24）
- 可用 Fetcher：`Fetcher`、`AsyncFetcher`、`StealthyFetcher`、`PlayWrightFetcher`
- ⚠️ 注意：此版本没有 `DynamicFetcher`，动态页面用 `PlayWrightFetcher` 或 `StealthyFetcher`
- ⚠️ API 差异：`Fetcher` 使用 `.get()` / `.post()` 方法（无 `.fetch()`）；`StealthyFetcher` 和 `PlayWrightFetcher` 使用 `.fetch()` 方法
- ⚠️ 元素访问：返回的 Response 对象用 `page.css('selector')` 返回列表，用 `[0]` 取第一个元素（无 `css_first()` 方法）；元素文本用 `.text` 属性（非方法调用）

## Fetcher 选择策略

| 场景 | 推荐 Fetcher | 原因 |
|------|-------------|------|
| 手机厂商官网（华为/苹果/小米/OPPO） | `StealthyFetcher` | 这些站点有反爬+JS渲染，实测验证可用 |
| 普通静态网站 | `Fetcher`（用 `.get()` 方法） | 轻量快速 |
| 有 Cloudflare 保护的站点 | `StealthyFetcher` | Camoufox 隐身浏览器可绕过 |
| 需要 JS 渲染但无反爬 | `PlayWrightFetcher` | Playwright 引擎 |
| 批量并发抓取 | `AsyncFetcher` | 异步高效 |

## 实测验证的手机官网爬取模板

### 华为官网产品页（已验证 ✅）

```python
from scrapling.fetchers import StealthyFetcher

page = StealthyFetcher.fetch(
    'https://consumer.huawei.com/cn/phones/mate80/',
    headless=True,
    network_idle=False,  # 华为官网持续有请求，必须设为 False
    timeout=60000
)
# 提取产品图片
imgs = page.css('img')
for img in imgs:
    src = img.attrib.get('src', '')
    if src and '/content/dam/' in src:
        full_url = f'https://consumer.huawei.com{src}' if src.startswith('/') else src
        print(full_url)
```

华为官网图片路径规律：`/content/dam/huawei-cbg-site/cn/mkt/pdp/phones/{型号}/img/...`

### 苹果中国官网（已验证 ✅）

```python
page = StealthyFetcher.fetch(
    'https://www.apple.com.cn/iphone-17/',
    headless=True,
    network_idle=False,
    timeout=60000
)
# 苹果官网图片在 img 和 source 标签中
imgs = page.css('img')
sources = page.css('source')
for img in imgs:
    src = img.attrib.get('src', '')
    alt = img.attrib.get('alt', '')
    if src and '/images/' in src:
        full_url = f'https://www.apple.com.cn{src}' if src.startswith('/') else src
        print(f'[{alt}] {full_url}')
```

苹果官网图片路径规律：`/iphone-17/images/overview/...`，有 `_large` 和 `_large_2x` 两种分辨率

### 小米商城产品页（已验证 ✅）

```python
page = StealthyFetcher.fetch(
    'https://www.mi.com/shop/buy/detail?product_id=20458',
    headless=True,
    network_idle=False,
    timeout=60000
)
imgs = page.css('img')
for img in imgs:
    src = img.attrib.get('src', '') or img.attrib.get('data-src', '')
    if src and ('mifile.cn' in src or 'mi-img.com' in src) and 'placeholder' not in src:
        url = f'https:{src}' if src.startswith('//') else src
        print(url)
```

小米图片 CDN 域名：`i8.mifile.cn`、`cdn.cnbj1.fds.api.mi-img.com`
⚠️ 小米官网 `mi.com/xiaomi-17` 这类路径会 302 到 404，必须用商城详情页 URL

### 荣耀官网产品页（⚠️ 需注意）

```python
# 荣耀CN官网产品页URL不稳定，可能返回404
# honor.com/cn/smartphones/honor-magic-v6/ → 404
# hihonor.com/cn/smartphones/honor-magic-v6/ → 301 → 404
# 
# 解决方案：不依赖爬取产品页，直接用CDN路径模式探测图片
```

荣耀图片 CDN 域名：`www-file.honor.com`
荣耀图片路径规律：`/content/dam/honor/cn/product/smartphone/{型号}/product/imgs/section-{功能}/{型号}-{功能}-bg@2x.jpg`

**⚠️ 荣耀CDN命名规则不统一**：
- `section-kv/honormagicv6-kv-bg@2x.jpg` ✅
- `section-cmf/honormagicv6-cmf-bg-red@2x.jpg` ✅（注意 `bg-red` 不是 `red`）
- `section-cmf/honormagicv6-cmf-red@2x.jpg` ❌ 404
- `section-zoom/honormagicv6-zoom-sample1@2x.jpg` ❌ 404（zoom图片可能不存在）

**推荐策略**：用 Python `urllib.request` HEAD 请求批量探测候选URL，而非猜测：

```python
import urllib.request, urllib.error

BASE = "https://www-file.honor.com/content/dam/honor/cn/product/smartphone/honor-magic-v6/product/imgs"

# 生成候选路径列表，用HEAD请求逐一验证
candidates = [
    f"section-kv/honormagicv6-kv-bg@2x.jpg",
    f"section-cmf/honormagicv6-cmf-bg-red@2x.jpg",
    f"section-cmf/honormagicv6-cmf-red@2x.jpg",  # 备选命名
    # ... 更多候选
]

for p in candidates:
    url = f"{BASE}/{p}"
    try:
        req = urllib.request.Request(url, method='HEAD')
        req.add_header('User-Agent', 'Mozilla/5.0')
        resp = urllib.request.urlopen(req, timeout=5)
        cl = resp.headers.get('Content-Length', '?')
        print(f"200 {p} ({cl} bytes)")
    except urllib.error.HTTPError:
        pass
```

### OPPO 官网产品页（⚠️ 特殊处理）

**⚠️ OPPO 官网（如 Find N6）使用视频帧渲染，页面无静态 `<img>` 产品图**。StealthyFetcher 爬取 DOM 只能拿到视频元素和占位符，无法提取产品图片 URL。

**方案1：Playwright 网络拦截（推荐）**

通过拦截浏览器网络请求，捕获页面加载过程中请求的 CDN 图片 URL：

```python
from playwright.sync_api import sync_playwright
import time

captured = []
def on_response(response):
    url = response.url
    ct = response.headers.get('content-type', '')
    if any(ext in url for ext in ['.png', '.jpg', '.jpeg', '.webp']) or 'image/' in ct:
        captured.append(url)

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page(viewport={"width": 1920, "height": 1080})
    page.on("response", on_response)
    page.goto('https://www.oppo.com/cn/smartphones/series-find-n/find-n6/',
              wait_until="domcontentloaded", timeout=60000)
    total = page.evaluate("document.body.scrollHeight")
    for pos in range(0, total, 500):
        page.evaluate(f"window.scrollTo(0, {pos})")
        time.sleep(0.15)
    time.sleep(3)
    browser.close()

# 过滤：去重、排除 < 200px 图标、验证 HEAD 200
```

捕获后需用 DOM Y坐标验证图片-section 映射（CDN 文件名不可信，如 `ksp-img-0` ≠ 芯片图）。

**方案2：StealthyFetcher DOM 爬取（仅对有 `<img>` 的 OPPO 页面有效）**

```python
page = StealthyFetcher.fetch(
    'https://www.oppo.com/cn/smartphones/series-find-x/find-x9/',
    headless=True, network_idle=False, timeout=60000
)
imgs = page.css('img')
for img in imgs:
    src = img.attrib.get('src', '') or img.attrib.get('data-src', '')
    if src and len(src) > 30 and 'placeholder' not in src:
        if src.endswith('.png.webp'):
            src = src[:-5]  # 去掉 .webp，飞书不兼容
        full_url = f'https://www.oppo.com{src}' if src.startswith('/') else src
        print(full_url)
```

OPPO 图片 CDN 域名：`image.oppo.com` 或 `oppo.com` 域名下
⚠️ OPPO 图片常见 `.png.webp` 双后缀，飞书不兼容 webp，需去掉 `.webp` 后缀变成 `.png`
⚠️ **判断用哪个方案**：先用 StealthyFetcher 爬取，如果 `<img>` 全是占位符或无产品图，立即切换到 Playwright 网络拦截

### vivo 官网产品页（参考模板）

```python
page = StealthyFetcher.fetch(
    'https://www.vivo.com.cn/vivo/xfold5/',
    headless=True,
    network_idle=False,
    timeout=60000
)
imgs = page.css('img')
for img in imgs:
    src = img.attrib.get('src', '') or img.attrib.get('data-src', '')
    if src and len(src) > 30 and 'placeholder' not in src:
        full_url = f'https://www.vivo.com.cn{src}' if src.startswith('/') else src
        print(full_url)
```

vivo 图片通常在 `vivo.com.cn` 域名下，格式为 jpg/png，无需转换

### 三星中国官网产品页（参考模板）

```python
page = StealthyFetcher.fetch(
    'https://www.samsung.com/cn/smartphones/galaxy-z-fold7/',
    headless=True,
    network_idle=False,
    timeout=60000
)
imgs = page.css('img')
for img in imgs:
    src = img.attrib.get('src', '') or img.attrib.get('data-src', '')
    if src and len(src) > 30 and 'placeholder' not in src:
        full_url = f'https://www.samsung.com{src}' if src.startswith('/') else src
        print(full_url)
```

三星图片 CDN 域名：`images.samsung.com` 或 `image-us.samsung.com`
三星官网图片通常为 jpg/png/webp，优先选 jpg/png
⚠️ 三星官网有较强的反爬，必须用 StealthyFetcher

## 通用图片提取模板

```python
from scrapling.fetchers import StealthyFetcher
import re

def extract_all_images(url):
    """从任意页面提取所有图片URL"""
    page = StealthyFetcher.fetch(
        url,
        headless=True,
        network_idle=False,
        timeout=60000
    )
    images = set()
    
    # 1. img 标签
    for img in page.css('img'):
        for attr in ['src', 'data-src', 'data-lazy-src', 'data-original']:
            src = img.attrib.get(attr, '')
            if src and len(src) > 20 and 'placeholder' not in src and 'data:image' not in src:
                images.add(src)
    
    # 2. source 标签 (picture 元素)
    for source in page.css('source'):
        srcset = source.attrib.get('srcset', '')
        if srcset and 'data:image' not in srcset:
            # srcset 可能包含多个URL和分辨率描述符
            for part in srcset.split(','):
                url_part = part.strip().split(' ')[0]
                if url_part and len(url_part) > 20:
                    images.add(url_part)
    
    # 3. 内联样式背景图
    for el in page.css('[style]'):
        style = el.attrib.get('style', '')
        urls = re.findall(r'url\(["\']?(https?://[^"\')]+)["\']?\)', style)
        images.update(urls)
    
    return images
```

## Scrapling vs Playwright 直接使用的区别

| 工具 | 用途 | 何时用 |
|------|------|--------|
| `StealthyFetcher`（Scrapling） | 爬取页面 DOM，提取 CDN 图片直链 | 需要从页面提取 `<img>` src、分析 DOM 结构 |
| `PlayWrightFetcher`（Scrapling） | 爬取 JS 渲染页面，无反爬场景 | 需要 DOM 但目标站无 Cloudflare 等防护 |
| `playwright` 直接使用 | **网络拦截** 或 **Section 截图** | 视频渲染站点捕获 CDN URL / CDN 不可用时截图 |

**关键区别**：Scrapling Fetcher 返回可解析 DOM 对象（提取数据），Playwright 直接使用可做网络拦截（`page.on("response", ...)` 捕获 CDN 图片 URL）和 `el.screenshot()` 截图。三者互补：
1. 先用 Scrapling 爬取 DOM → 分析 section 结构和 `<img>` 图片
2. 如果无 `<img>`（视频渲染站点如 OPPO）→ Playwright 网络拦截捕获 CDN URL
3. 如果 CDN 图片仍不可用 → Playwright 截图（`device_scale_factor=2` 高清）

**Playwright 截图和网络拦截的详细流程见 [launch-briefing.md](launch-briefing.md) 第二部分**。

## 关键注意事项

1. **必须用 `network_idle=False`**：大多数现代官网有持续的网络请求（分析、心跳等），设为 True 会导致 30s 超时
2. **timeout 建议 60000ms**：复杂页面加载需要时间
3. **headless=True**：无头模式，不弹出浏览器窗口
4. **相对路径处理**：华为和苹果官网返回的图片路径通常是相对路径，需要拼接域名
5. **StealthyFetcher 依赖 Camoufox**：首次使用需运行 `python3 -m camoufox fetch`
6. **PlayWrightFetcher 对反爬能力弱**：小米官网用 PlayWrightFetcher 会被 302 重定向，必须用 StealthyFetcher
7. **产品页 404 时的降级策略**：如果官网产品页返回 404（如荣耀 CN 官网），不要反复尝试不同 URL 变体。直接切换到 CDN 路径探测（HEAD 请求）或 Playwright 截图方案
8. **视频渲染站点（如 OPPO）**：如果 StealthyFetcher 爬取后 `<img>` 全是占位符，说明站点用视频渲染。立即切换到 Playwright 网络拦截方案（见 OPPO 模板）