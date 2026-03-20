# 竞品发布会速报文档撰写指南（含图片集成）

## ⚡ 触发规则

当用户请求涉及以下关键词时，必须遵循本指南：
- "发布会简报/速报/快报"、"竞品分析报告"（涉及新品发布）、"新品发布会总结"
- 提到某品牌刚发布的新产品，要求输出分析文档
- 需要在飞书文档中插入产品图片

**联动规则**：本规则触发时，必须同时加载 [scrapling.md](scrapling.md)（爬虫工具）。

## 用户身份

- **用户所在公司：小米（Xiaomi）**
- 所有竞对分析从小米视角撰写
- 进攻点 = 小米可攻击竞品之处（我方优势）；防守点 = 竞品做得好、小米需跟进之处（我方劣势）
- 小米折叠屏：MIX Fold 系列；直板旗舰：小米数字系列
- 如果小米同代产品未发布，用上代产品对比并标注"当代信息待补充"
- 部门名称默认"手机产品部"

## 核心原则

1. **竞品情报分析，不是产品说明书**：视角是"我们怎么看这个产品"
2. **信息交叉验证**：关键数据至少 2 个独立来源确认，不搬运厂商宣传口径
3. **数据全文统一**：同一参数全文只出现一个数值
4. **批判性视角**：每条"做的好的/不好的"必须有竞品数据对比
5. **不唯参数论**：参数对比结合实际体验（如苹果小电池≠差续航）
6. **图片是必须的**：每篇速报必须有图片，CDN 失败必须切换截图方案
7. **绝不凭文件名猜测图片内容**：必须通过 DOM 上下文验证
8. **不确定就不标注**：图片 caption/描述只在 100% 确认内容时才写，否则留空。错误标注比无标注更损害文档可信度

---

## 第一部分：文档结构与写作规范

### 文档头部

```markdown
手机产品部 2026.3.19 {align="right"}
```
- 部门+日期**同一行**，末尾 `{align="right"}`
- 日期格式 `YYYY.M.DD`（不补零）
- ⚠️ 禁止：分两行、加 `---` 分隔线

### 1. 执行摘要（callout，可选）

适用于内容多、需快速浏览的长文档。结构清晰时可省略。

```html
<callout emoji="bulb" background-color="light-blue">
### 产品策略
- **产品定位**：{2-3条}
- **发布节奏**：{时间、与竞品关系}
### 核心卖点
- {电报体，斜杠分隔关键参数}
### 定价
- **{趋势判断}**：{全SKU价格}
### 做的好的（2-4条，宁少勿滥）
- **{要点}**：{竞品数据对比}
### 做的不好的（1-3条，只写高影响短板）
- **{要点}**：{竞品数据对比}
### 竞对分析
- **{我司产品 vs 竞品}**
  - 进攻点：{具体理由+数据}
  - 防守点：{紧迫程度标注}
</callout>
```

**规则**：
- `background-color` 统一 `"light-blue"`（飞书 fetch_doc 返回的始终是 light-blue，用 blue 会导致 replace_range 定位失败）
- "做的不好的"禁止鸡蛋挑骨头：差距小/不可操作/行业普遍的问题不写
- "发布会观感"仅当发布会有显著差异时才写，常规发布跳过

### 2. 产品策略（## 产品策略）

- 发布时间/地点/节奏（与上代对比，判断提速/延后）
- 产品线定位、配色策略、定价策略
- 定价对比用 `<quote-container>` + `<text color="gray">` 标灰上代价格
- 涨价异常 SKU 用 `<text bgcolor="light-red">` 高亮
- 配一张产品主视觉 KV 图（width="1260"）

### 3. 核心卖点（## 核心卖点）

**子标题格式**：`### <text color="blue">**{类别}——{一句话概括}**</text>`

**排序**：按厂商发布会传播重点排序，不按固定模板。

每个子章节：bullet points 参数 + 对应产品图（至少1张）+ 分析最多一句话。

**必写**：外观设计、屏幕、影像、SoC/性能、电池充电、可靠性
**可选**：AI系统、通讯、材质架构、隐私安全（同行业常规水平且未重点宣传则不写）

**图片规则**：
- 单图 width="800"，grid 双栏 width="1000"，KV hero width="1260"
- 必须按实际像素比例缩放，禁止统一固定值

### 4. 竞争分析（## 竞争分析）

- 竞品选 3-5 款头部品牌（三星/华为/小米/OPPO/vivo），不选小众品牌
- 对比表含：处理器、屏幕、摄像头、电池充电、尺寸重量、核心差异化参数、防护等级、起售价
- 表后三色 callout：✅ light-green 竞争优势 / ⚠️ light-yellow 需要关注 / 🔴 light-red 对我司影响
- 进攻/防守点独立章节，每条加粗标题+理由+数据，防守紧迫方向标注"P0方向"

### 5. 发布会卖点图组（## 发布会卖点图组）

- 12-16 张产品图，`<grid cols="2">` 双栏
- 按类别：外观配色 → 设计工艺 → 铰链防护 → 屏幕性能 → 影像样张 → 系统AI
- **caption 规则**：只有当图片内容可以 100% 确认时才写 caption。如果图片来自第三方媒体文章（文件名无语义、DOM 上下文推断不确定），默认不写 caption，纯图片展示。错误的 caption 比没有 caption 更糟糕。

### 写作风格

- 锐利判断开头，不做说明书
- 数据说话，每个判断跟具体数字对比
- bullet points 为主，避免长段落
- 进攻/防守视角清晰，每条标注紧迫程度
- 价格涨跌必须量化到元

---

## 第二部分：图片获取与集成

### 图片获取策略选择

**图片来源优先级**（SU7 实测总结）：
1. **官网产品页**（最优）：section 结构清晰，图片-内容映射可靠，可写 caption
2. **官方新闻稿/媒体素材包**：图片有明确标题和描述，映射可靠
3. **第三方媒体文章**（次优）：图片文件名无语义，DOM 上下文推断有不确定性，默认不写 caption
4. **社交媒体/论坛**（最后手段）：图片质量和内容不可控

| 策略 | 适用场景 |
|------|---------|
| **方案A：CDN 直链** | 图片以 `<img>` 存在，CDN URL 可直接访问 |
| **方案B：Section 截图** | 视频/Canvas 渲染，或 CDN 图内容无法确认 |
| **方案C：Playwright 网络拦截** | 整站视频渲染无 `<img>`（如 OPPO），通过拦截网络请求捕获 CDN 图片 |

**决策流程**：
1. 爬取产品页检查 `<img>` → 有真实 src → 方案A
2. 全是 placeholder / 无 `<img>` → 检查是否有网络请求中的图片资源 → 方案C
3. 方案A/C 均失败 → 方案B（截图）

### 方案A：CDN 直链（5步）

**Step 1**：用 Scrapling StealthyFetcher 爬取产品页，提取 `<img>` src（详见 [scrapling.md](scrapling.md)）

**Step 2**：下载检测尺寸，过滤无效图

| 过滤条件 | 处理 |
|---------|------|
| 宽或高 < 200px | 丢弃（图标） |
| 文件 < 5KB | 丢弃 |
| 宽高比 > 3.5:1 或 < 1:3.5 | 丢弃（横幅/竖图） |
| 文件名含 prev/next/arrow/close/play | 丢弃（控件） |
| 宽 > 800 且高 > 400 且比例 0.5~2.5 | 保留 |

用 Python struct 解析图片头部获取尺寸（**不要用 sips 处理 webp，会卡死**）。

**Step 3**：DOM 上下文分析（最关键，不可跳过）

⚠️ **CDN 路径中的 section 名称不可信**（案例：`section-diamond` ≠ 铰链，实际是黑钻屏；`ksp-img-0` ≠ 芯片图）

必须输出映射表：`图片文件名 → DOM section标题 → 实际尺寸 → 文档用途`

**DOM Y坐标验证法**（OPPO N6 实测有效）：当 CDN 文件名无法判断内容时，通过图片在页面中的 Y 坐标位置与 section 标题的 Y 坐标对比，确定图片属于哪个 section。

**第三方媒体文章的特殊困难**（SU7 实测教训）：
- 媒体文章（如 carz.com.my、carnewschina.com）的图片文件名通常是编辑随意编号（如 `Facelift-1` ~ `Facelift-17`），与图片实际内容完全无关
- DOM 上下文分析只能推断"这张图大概在讲什么 section"，但同一 section 可能有多张图，且文章结构不如官网规整
- **置信度分级**：
  - ✅ 高置信：官网产品页，section 结构清晰，每个 section 1-2 张图 → 可写 caption
  - ⚠️ 中置信：媒体文章，DOM 上下文可推断大致类别 → 只按类别分组，不写具体 caption
  - ❌ 低置信：图片密集、section 边界模糊、文件名无语义 → 不写 caption，纯图片展示
- **核心卖点 section 的图片**：即使置信度中等，也只用于"大致匹配"（如外观图放外观 section），不要在 caption 中编造具体描述（如"集成毫米波雷达的重新设计格栅"）

**Step 4**：格式兼容性

| 格式 | 飞书支持 |
|------|---------|
| PNG/JPG/GIF | ✅ 完全支持 |
| WebP | ⚠️ 大概率可用，优先用 jpg/png |
| AVIF | ❌ 不支持 |

CDN 转换：OPPO `.png.webp` → 去掉 `.webp`；小米 `//` 开头 → 加 `https:`；苹果优先 `_large_2x.jpg`

**Step 5**：批量 HEAD 验证 → 写入飞书

所有 URL 必须先 HEAD 验证返回 200（飞书下载失败不报错，只静默跳过）。

### 方案B：Section 截图

适用：CDN 不可用 / 视频渲染 / 内容不匹配时。

1. **Playwright + device_scale_factor=2**（推荐，3840px 宽高清）截取 section
2. 必须滚动全页触发懒加载，用 `el.screenshot()` 而非 `page.screenshot(clip=...)`
3. 截图前关闭 cookies 弹窗（JS 移除 `[class*="cookie"]` 等元素）
4. 裁剪只去纯背景空白（2-3%），禁止裁掉有效信息
5. 缩放到 2000px 宽，转 JPG quality 90
6. 上传 `freeimage.host` 获取 HTTPS URL
7. HEAD 验证后写入飞书

Camoufox 仅在 Playwright 被反爬拦截时使用（不支持 device_scale_factor，分辨率受限）。

### 方案C：Playwright 网络拦截（OPPO 等视频渲染站点）

部分厂商（如 OPPO Find N6）整站使用视频帧渲染，无任何静态 `<img>` 产品图。此时通过 Playwright 拦截网络请求捕获 CDN 图片 URL：

```python
from playwright.sync_api import sync_playwright
import time, json

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
    page.goto(url, wait_until="domcontentloaded", timeout=60000)
    # 滚动全页触发所有图片加载
    total = page.evaluate("document.body.scrollHeight")
    for pos in range(0, total, 500):
        page.evaluate(f"window.scrollTo(0, {pos})")
        time.sleep(0.15)
    time.sleep(3)
    browser.close()

# 过滤+去重后，用 DOM Y坐标验证图片-section 映射
```

捕获后仍需 Step 2（尺寸过滤）+ Step 3（DOM 上下文验证）+ Step 5（HEAD 验证）。

### 全文图片去重清单（写入前必做）

写入飞书前，列出所有章节要用的图片 URL，检查：
- 同一卖点是否用了两个不同图源（CDN + 截图）
- 核心卖点和卖点图组是否重复展示同一内容
- 卖点图组可复用核心卖点的图（同 URL），但不能用不同 URL 展示同一内容

### 图片数量与分布

| 章节 | 图片数 | 尺寸策略 |
|------|-------|---------|
| 产品策略 | 1 张 KV | width="1260" |
| 每个核心卖点 | 1-2 张 | 单图 800，双图 grid 1000 |
| 竞争分析 | 0 张 | 纯表格文字 |
| 卖点图组 | 12-16 张 | grid 双栏 1000 |
| 全文合计 | 16-24 张 | |

---

## 第三部分：信息采集与飞书写入

### 信息采集流程

1. web_search 搜索产品 specs/launch + 中文参数价格
2. gsmarena.com 标准化参数 + 厂商官网官方参数
3. 确定 3-5 款头部竞品，逐一搜索最新参数
4. 交叉验证关键数据（电池/屏幕/尺寸重量/定价/处理器/相机）
5. 建立参数清单，统一全文数据

### 飞书写入策略

```
Step 1: create_doc（只写标题）
Step 2: update_doc mode="overwrite"（头部 + 摘要 + 产品策略 + 核心卖点文字）
Step 3: update_doc mode="append"（竞争分析：表格 + 三色callout + 进攻防守）
Step 4: update_doc mode="append"（卖点图组，图片密集单独写入）
```

**异步处理**：返回 task_id 时，等 3-5 秒后轮询 `update_doc(task_id=xxx)`，最多 5 次。

**模式选择**：

| 场景 | 模式 |
|------|------|
| 首次写入 | overwrite |
| 追加章节 | append |
| 修改章节 | replace_range + selection_by_title |
| 修改文字 | replace_range + selection_with_ellipsis |
| 修复 1-2 张图片 | replace_range 精确定位（用周围文字定位，不用 token 值） |
| 修复 3+ 张图片 | **直接 overwrite 重建全文**（避免累积腐败） |
| 用户反馈图片错误 | overwrite 重建，所有图片用 url 重新写入 |

### replace_range 累积腐败问题

多次 replace_range 后飞书 Markdown 结构会腐败（callout 截断、标签错乱、列表嵌套错位、残留碎片）。

**SU7 实测教训**：3 次 replace_range 后即出现 `- \n  -` 嵌套错乱、grid 后残留 `" width="1000"...` 碎片。阈值比预期更低。

| 修改次数 | 策略 |
|---------|------|
| 1-2 次 | replace_range 可用 |
| 3 次 | 每次后 fetch_doc 检查，发现任何格式异常立即 overwrite |
| 4+ 次 | **停止 replace_range，直接 overwrite 重建** |

**overwrite 重建流程**：
1. fetch_doc 获取当前全文
2. 在本地修复所有问题（格式、图片、内容）
3. overwrite 写入头部 + 摘要 + 产品策略 + 核心卖点（含图片 URL）
4. **立即** append 竞争分析（表格 + callout）
5. **立即** append 卖点图组 + 信息来源
6. fetch_doc 验证最终结果

⚠️ overwrite 会清空全文，必须在同一轮对话中完成所有 append，否则内容丢失。

### 飞书图片 token 不可逆

飞书 `<image token="xxx">` 中的 token 是上传时生成的内部标识，**无法从 token 反推原始 URL**。这意味着：
- 修复图片时必须用 `<image url="...">` 重新写入，不能基于 token 操作
- 如果需要保留某张已上传的图片，只能通过 token 引用，但无法验证其内容
- **最可靠的修复方式**：overwrite 重建全文，所有图片用 url 重新写入

---

## 常见错误速查

| 错误 | 正确做法 |
|------|---------|
| 照搬"同级最大""行业首创" | 搜索验证+竞品数据对比 |
| 凭 CDN 文件名猜图片内容 | DOM 上下文/Y坐标验证 |
| 图片统一 width="1200" height="675" | 按实际像素比例缩放 |
| sips 处理 webp | Python struct 解析 |
| URL 未 HEAD 验证就写飞书 | 先验证 200 再写入 |
| callout 用 `"blue"` | 统一 `"light-blue"` |
| 头部分两行+分隔线 | 同一行 `{align="right"}`，无分隔线 |
| 多次 replace_range 后格式乱 | 3+ 次改用 overwrite 重建 |
| CDN 失败就跳过图片 | 必须切换截图方案 |
| OPPO 等视频站尝试 CDN 直链 | 直接用网络拦截（方案C）或截图（方案B） |
| 同一卖点核心卖点+图组重复不同图源 | 全文图片去重清单 |
| "做的不好的"鸡蛋挑骨头 | 只写高影响短板 |
| 竞品选小众品牌 | 只选头部品牌 |
| overwrite 后忘 append 剩余章节 | overwrite 后立即 append |
| 第三方媒体图片写具体 caption | 置信度不够就不写 caption，纯图片展示 |
| 基于 token 修复图片 | 必须用 url 重新写入，token 不可逆 |
| replace_range 定位到 image token 值 | 用周围文字内容定位，不要用 token 值作为定位锚点 |
| 图片修复用多次 replace_range | 涉及 3+ 张图片修复时直接 overwrite 重建全文 |