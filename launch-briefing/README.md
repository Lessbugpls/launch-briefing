# launch-briefing

竞品情报分析 Kiro Skill 工具集，面向手机行业产品经理。

## 功能

- **竞品发布会速报** — 从信息采集到飞书文档输出的完整流程，含图片集成
- **网页爬虫图片提取** — 基于 Scrapling/Playwright，支持各主流手机厂商官网（华为/苹果/小米/OPPO/vivo/三星/荣耀）
- **PDF 转飞书文档** — 将 PDF 页面渲染为高清截图，上传图床后嵌入飞书文档

## 安装

在 Kiro 中通过 GitHub 安装此 skill。

## 环境依赖

```bash
# 爬虫工具
pip3 install scrapling playwright
python3 -m playwright install
python3 -m camoufox fetch

# PDF 处理
pip3 install PyPDF2 pymupdf
```

## 项目结构

```
launch-briefing/
├── SKILL.md                        # 主入口
├── rules/
│   ├── launch-briefing.md          # 竞品速报撰写指南
│   ├── scrapling.md                # 爬虫工具操作指南
│   └── pdf-to-feishu-images.md     # PDF 截图转飞书指南
├── README.md
└── LICENSE
```

## License

MIT
