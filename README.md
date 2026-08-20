# LLMNEX 文档

对外接入文档的**唯一源头**。用 [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) 构建,
推到 `main` 后由 GitHub Actions 自动发布到 GitHub Pages。

## 本地预览

```bash
pip install -r requirements.txt
mkdocs serve          # http://127.0.0.1:8000
```

## 结构

```
docs/
├── index.md                       首页(卡片导航)
├── guide/getting-started.md       快速开始
└── image-generation/
    ├── overview.md                生图介绍(模型清单与选型)
    └── gemini.md                  Gemini 接入指南
```

新增页面后记得在 `mkdocs.yml` 的 `nav` 里挂上,否则不会出现在导航里。

## 注意

- 构建用 `--strict`,坏链接会直接让 CI 失败
- `pymdownx.highlight` 的 `pygments_lang_class: true` **不能删** —— 少了它代码块拿不到
  `language-*` 类,语法高亮和复制按钮都会受影响
