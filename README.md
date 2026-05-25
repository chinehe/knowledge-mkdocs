# ChineHe Knowledge

基于 [MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) 构建的个人技术知识库。

## 项目用途与目标

本项目是一个个人技术博客 / 知识库站点，用于系统性地整理和沉淀学习笔记，涵盖开发者需要涉及的各个技术方向。

目标是将零散的学习内容结构化，形成可检索、可分享的在线文档站点。

## 技术栈

* 文档语言：Markdown
* 网站工具：[MkDocs](https://www.mkdocs.org/)
* 网站主题：[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)

## 快速开始

### 环境要求

- Python 3.x

### 安装依赖

```bash
python3 -m pip install mkdocs mkdocs-material
```

### 本地预览

启动开发服务器，支持实时热重载：

```bash
mkdocs serve
```

默认访问地址：http://127.0.0.1:8000

### 构建静态站点

```bash
mkdocs build
```

构建产物输出到 `site/` 目录，可直接部署到任意静态托管服务。

## 项目结构

```
.
├── mkdocs.yml          # MkDocs 配置文件（站点名称、导航、主题）
├── docs/               # Markdown 文档源文件
│   ├── Golang/         # Golang 语言 & Gin 框架笔记
│   ├── Python/         # Python 基础笔记
│   ├── ai/            # AI 大模型 & MCP 相关笔记
│   ├── 数据库与缓存/    # Redis 系列笔记
│   ├── DevOps/        # DevOps 相关（K8s 等）
│   └── about.md       # 关于页面
├── site/               # 构建产物（git 忽略）
└── README.md           # 本文件
```

## 新增文档

1. 在 `docs/` 对应目录下创建 `.md` 文件
2. 在 `mkdocs.yml` 的 `nav` 配置中添加对应条目
3. 运行 `mkdocs serve` 预览效果
