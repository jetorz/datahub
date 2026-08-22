# DataHub

项目资料、业务数据、分析结果与公开静态内容的集中仓库。

## Repository Role

DataHub 与代码仓库分离：

- `jetorz/scripts` 保存可维护的软件源码、工具和自动化；
- `jetorz/datahub` 保存项目级资料、数据、分析产物和用于网页访问的公开副本。

本仓库当前为 **public**。进入 `main` 的内容应视为可被公开读取，并可能通过 GitHub Pages 对外访问。

## Structure

仓库按项目或数据集组织，顶层目录原则上保持相互独立，例如：

```text
DataHub/
├─ <project-a>/
│  ├─ source/ or existing source files
│  ├─ converted/ or derived data
│  └─ ChatGPT/
├─ <project-b>/
├─ skill-samples/
├─ index.html
└─ .nojekyll
```

实际项目不要求机械套用统一子目录；已有业务目录结构应优先保持稳定。

约定：

- 一个项目尽量使用一个稳定的顶层目录。
- 客户原始文件、OfficeTools 转换文件以及程序采集/运行产物保留在项目原有位置或既有目录。
- ChatGPT 在 DataHub 项目中新增或修改的文件，默认写入该项目的 `ChatGPT/` 子目录；除非任务明确要求覆盖原文件。
- `index.html` 为项目导航入口，可由 GitTools 等工具生成或维护。
- `.nojekyll` 用于保持 GitHub Pages 对静态目录和文件的直接发布行为。

## Data Management

- 文件名应尽量体现项目、用途、日期或版本，避免 `new`、`final2`、`临时` 等不可追溯命名长期存在。
- 原始数据与派生结果应能从目录或文件名上区分。
- CSV、Excel、PDF、图片、HTML、Markdown 等格式可以按实际分析和交付需求共存。
- 可重新生成的中间产物不要求永久保存；需要长期复现的关键输入、规则和结果应保留。
- 大规模替换或清洗数据时，应优先保留原始输入，不直接覆盖唯一原件。

## Git Workflow

- `main`：当前有效的数据与项目资料基线，也是公开访问的主分支。
- 日常新增资料和小型分析产物：检查目标目录后可以直接提交到 `main`。
- 大规模数据迁移、批量重命名或目录重构：建议在独立分支完成，并在合并前检查链接、Pages 路径和项目引用。
- Commit 应聚焦单一项目或单一批次变更，避免在一次提交中混合多个无关项目。

## Public Repository Boundary

由于本仓库为公开仓库：

- 版本库中的文件和历史提交应按公开内容管理；
- Token、密码、私钥、API Secret 等访问凭证不得提交；
- 需要保留但不适合进入 Git 的本地内容，应由 `.gitignore` 或仓库外目录管理。

## Related Repositories

| Repository | Purpose |
| --- | --- |
| `jetorz/datahub` | 项目资料、业务数据、分析结果与公开页面 |
| `jetorz/scripts` | 开发代码、工具和自动化 |
| `jetorz/obsidian` | Obsidian 第二大脑与文章知识库 |
| `jetorz/codex` | Codex / Agent 配置、Skills 与自动化环境 |
