# PDF 页面截图转飞书文档图片集成指南

## ⚡ 触发规则

当以下**任一条件**满足时，必须遵循本指南：

### 强触发（必须立即激活）
1. 用户消息中提到 **PDF 文件名**（如 `xxx.pdf`），且要求输出飞书文档/分析/报告/总结
2. 工作区中存在 `.pdf` 文件，且用户要求基于其内容写文档
3. 用户明确要求"把 PDF 里的图放到文档里""配上 PDF 里的图""引用 PDF 原图"或类似表述
4. 用户要求"参考 PDF 写分析/报告/分享"——此时 PDF 中的图表是核心素材，必须截图嵌入

### 中触发（检查后激活）
5. 用户提到"展会资料""产品手册""PPT""演示文稿""presentation"等关键词，且工作区有对应 PDF
6. 需要将 PDF 中的图表、数据可视化、产品页以截图形式嵌入飞书文档
7. 处理 PPT 导出的 PDF、产品手册、展会资料等图文混排文档

### 判别要点
- **PDF ≠ 纯文本**：PDF 中的图表、排版、数据可视化无法通过 PyPDF2 文本提取还原，必须用 pymupdf 渲染为图片
- **"配图"指的是 PDF 原始页面截图**：当用户说"配上图""放图""引用图"时，如果上下文涉及 PDF，指的是 PDF 页面截图，不是随意找的网络图片
- **飞书文档需要公开 URL**：PDF 页面截图必须上传到图床获取 HTTPS URL 后才能嵌入飞书 `<image>` 标签
- **与 scrapling.md 的区别**：[scrapling.md](scrapling.md) 处理的是从网页爬取图片；本 skill 处理的是本地 PDF 文件的页面截图。两者互补，不冲突

## 核心思路

PDF 文件无法直接以文本方式完美还原其排版和图表。最佳方案是：**将 PDF 每一页渲染为高清 PNG 图片，上传到图床获取公开 URL，再以 `<image>` 标签嵌入飞书文档**。这样可以完整保留 PDF 原始的排版、图表、配色和视觉效果。

## 核心原则

1. **pymupdf 是首选 PDF 渲染库**：比 pdf2image（依赖 poppler）更易安装，纯 Python 实现
2. **2x 缩放是最佳平衡点**：清晰度足够，文件大小可控（通常 100KB-300KB/页）
3. **不要尝试用 readFile 读取 PDF**：PDF 是二进制文件，readFile 会失败或返回乱码
4. **PyPDF2 只能提取文本**：用于获取文字内容做总结，但无法保留排版和图表
5. **图床上传是必要步骤**：飞书 `<image>` 标签需要公开可访问的 HTTPS URL

## 标准工作流程（4步）

### Step 1：安装依赖并提取文本

```bash
pip3 install PyPDF2 pymupdf
```

先用 PyPDF2 提取纯文本，用于理解内容和撰写文字总结：

```python
import PyPDF2
reader = PyPDF2.PdfReader('document.pdf')
for i, page in enumerate(reader.pages):
    text = page.extract_text()
    if text and text.strip():
        print(f'--- Page {i+1} ---')
        print(text)
```

### Step 2：将 PDF 页面渲染为 PNG 图片

```python
import fitz  # pymupdf
import os

os.makedirs("pdf_images", exist_ok=True)
doc = fitz.open("document.pdf")

for i in range(len(doc)):
    page = doc[i]
    mat = fitz.Matrix(2, 2)  # 2x 缩放，清晰度与文件大小的最佳平衡
    pix = page.get_pixmap(matrix=mat)
    pix.save(f"pdf_images/page_{i+1}.png")

doc.close()
```

**缩放倍率选择**：
| 倍率 | 适用场景 | 典型文件大小 |
|------|---------|------------|
| 1x | 快速预览，对清晰度要求不高 | 50-150KB |
| 2x | **推荐默认值**，清晰且文件大小可控 | 100-300KB |
| 3x | 需要放大查看细节（如电路图、小字表格） | 300-800KB |

**选择性渲染**：如果 PDF 页数很多（>20页），不需要全部渲染，只渲染与报告相关的关键页面：

```python
key_pages = [1, 3, 5, 8, 12]  # 只渲染需要的页面
for i in key_pages:
    page = doc[i-1]  # 注意：fitz 页码从 0 开始
    # ...
```

### Step 3：上传图片到图床获取公开 URL

**首选方案：freeimage.host**（免费、无需注册、稳定）：

```python
import subprocess, json

def upload_image(path):
    """上传图片到 freeimage.host，返回公开 URL"""
    cmd = [
        "curl", "-s", "-X", "POST",
        "https://freeimage.host/api/1/upload",
        "-F", f"source=@{path}",
        "-F", "key=6d207e02198a847aa98d0a2a901485a5"
    ]
    r = subprocess.run(cmd, capture_output=True, text=True)
    data = json.loads(r.stdout)
    return data["image"]["url"]  # 返回类似 https://iili.io/xxxxx.png
```

批量上传：

```python
urls = {}
for i in range(1, total_pages + 1):
    path = f"pdf_images/page_{i}.png"
    url = upload_image(path)
    urls[i] = url
    print(f"Page {i}: {url}")
```

**备选图床方案（按优先级）**：

| 图床 | 优点 | 缺点 | API |
|------|------|------|-----|
| freeimage.host | 免费、稳定、无需注册 | 公共 API key | `POST /api/1/upload` |
| imgbb.com | 免费、API 简单 | 需要注册获取 key | `POST /1/upload` |
| sm.ms | 国内访问快 | 有每日限额 | `POST /api/v2/upload` |

**踩坑记录 — 不可用的方案**：
- ❌ `file.io`：返回 301 重定向，curl -L 后返回 HTML 页面而非 JSON，不适合程序化上传
- ❌ `0x0.st`：会封禁部分网络段（如云服务器 IP）
- ❌ `telegra.ph/upload`：返回 "Unknown error"，不稳定
- ❌ ams-osram.com 等厂商官网直链：有防盗链，飞书后端无法下载

### Step 4：在飞书文档中插入图片

```markdown
<image url="https://iili.io/xxxxx.png" width="800" align="center" caption="PDF原文：产品名称/页面标题"/>
```

**caption 写法**：统一使用 `PDF原文：{页面实际标题}` 格式，让读者知道这是 PDF 截图而非独立配图。

**排版建议**：图片放在对应章节的文字总结之前或之后，形成"原文截图 + 文字解读"的配对结构。

## 完整场景示例：展会/发布会 PDF → 飞书报告

典型流程：

1. **PyPDF2 提取全部文本** → 理解内容，规划报告结构
2. **pymupdf 渲染关键页面为 PNG** → 每个产品/功能对应的页面
3. **freeimage.host 批量上传** → 获取所有图片的公开 URL
4. **飞书 create-doc + update-doc** → 分段写入，每段包含截图 + 文字解读

文档结构模板：

```
## 产品概述
<image>  ← 公司介绍页截图
文字总结

## 产品详解
### 产品1
<image>  ← 产品1的PDF页面截图
文字解读 + 参数表格 + 对我方的价值

### 产品2
<image>  ← 产品2的PDF页面截图
文字解读 + 参数表格 + 对我方的价值
...

## 关键收获与行动建议
```

## 与其他规则的关系

- 本规则处理的是**本地 PDF 文件**的图片提取，与 [scrapling.md](scrapling.md)（处理网页爬取图片）互补
- 如果同时需要 PDF 截图和网页产品图，两个规则可以同时生效
- 飞书文档的 `<image>` 标签用法与 [launch-briefing.md](launch-briefing.md) 中的规则一致

## ⚠️ 常见遗漏场景（必须避免）

| 遗漏 | 正确做法 |
|------|---------|
| 用户说"参考 PDF 写分析，配上图"，直接用 urls.json 里的旧图或网络图 | 必须从目标 PDF 渲染页面截图，上传图床后使用新 URL |
| PDF 已有部分页面截图（如 urls.json），就不再渲染其他页面 | 检查哪些关键页面缺失，补充渲染和上传 |
| 只用 PyPDF2 提取文本就开始写文档，不截图 | PyPDF2 提取文本用于理解内容，但图表必须用 pymupdf 渲染为截图 |
| 图片 URL 用了其他 PDF 的截图（如 corp_page 的图用到了 Canalys 报告里） | 每个 PDF 的截图独立管理，不混用 |
| 飞书文档中 `<image>` 的 caption 写得模糊 | 统一用 `PDF原文：{页面实际标题}` 格式 |

## 常见问题

### Q: PDF 太大（>50页）怎么办？
A: 只渲染关键页面。先用 PyPDF2 提取文本，确定哪些页面包含重要图表，再选择性渲染。

### Q: 图片上传失败怎么办？
A: 换一个图床。freeimage.host 的公共 key 有频率限制，如果短时间上传太多（>30张），可能被限流。等几分钟再试，或换 imgbb。

### Q: 飞书显示图片失败怎么办？
A: 检查 URL 是否是 HTTPS 且可公开访问。在浏览器中直接打开 URL 确认图片可见。飞书后端需要能直接下载该 URL。

### Q: pymupdf 安装失败怎么办？
A: `pip3 install pymupdf`。如果系统 Python 版本太旧（<3.7），考虑用 `pdf2image` + `poppler`（需要 `brew install poppler`）。