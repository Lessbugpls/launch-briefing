---
name: launch-briefing
description: 竞品情报分析工具集：竞品发布会速报撰写、网页爬虫图片提取（Scrapling/Playwright）、PDF 页面截图转飞书文档。适用于手机行业竞品分析、产品发布会总结、展会资料整理等场景，输出飞书云文档。
metadata:
  tags: competitive-intelligence, scrapling, playwright, pdf, feishu, lark, product-launch, smartphone, web-scraping
---

# 竞品情报分析工具集

手机行业竞品情报分析的完整工具链，覆盖信息采集、图片获取、文档撰写全流程。

## When to use

当用户请求涉及以下场景时，加载对应的规则文件：

- **竞品发布会速报/快报/简报**、**竞品分析报告**、**新品发布会总结** → 加载 [rules/launch-briefing.md](rules/launch-briefing.md) + [rules/scrapling.md](rules/scrapling.md)
- **从官网提取产品图片**、**webFetch 返回空壳 HTML**、**需要绕过反爬** → 加载 [rules/scrapling.md](rules/scrapling.md)
- **PDF 转飞书文档**、**PDF 截图嵌入文档**、**展会资料/产品手册整理** → 加载 [rules/pdf-to-feishu-images.md](rules/pdf-to-feishu-images.md)

## Rules

- [rules/launch-briefing.md](rules/launch-briefing.md) — 竞品发布会速报文档撰写指南（含图片集成与飞书写入策略）
- [rules/scrapling.md](rules/scrapling.md) — Scrapling 爬虫工具使用决策与操作指南（含各厂商官网实测模板）
- [rules/pdf-to-feishu-images.md](rules/pdf-to-feishu-images.md) — PDF 页面截图转飞书文档图片集成指南

## 三个模块的关系

| 模块 | 输入 | 输出 | 典型场景 |
|------|------|------|---------|
| launch-briefing | 产品发布信息 + 图片 | 飞书竞品速报文档 | 三星/华为/OPPO 新品发布 |
| scrapling | 官网 URL | CDN 图片直链 | 从产品页提取高清产品图 |
| pdf-to-feishu-images | 本地 PDF 文件 | 图床 URL + 飞书文档 | 展会 PDF、产品手册 |

**典型组合**：
1. launch-briefing + scrapling：发布会速报，从官网爬取产品图嵌入飞书
2. pdf-to-feishu-images 单独使用：展会资料 PDF 转飞书分享文档
3. 三者联合：发布会速报 + 展会 PDF 素材 + 官网产品图

## 环境依赖

```bash
# 爬虫工具
pip3 install scrapling playwright
python3 -m playwright install
python3 -m camoufox fetch

# PDF 处理
pip3 install PyPDF2 pymupdf
```

## 用户身份

默认用户所在公司为小米（Xiaomi），所有竞对分析从小米视角撰写。详见 [rules/launch-briefing.md](rules/launch-briefing.md)。
