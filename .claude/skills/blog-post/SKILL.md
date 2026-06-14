---
name: blog-post
description: |
  This skill should be used when the user wants to create a new blog post, write an article,
  publish a post, or upload blog content for their Hugo-based silentmoon blog. Triggers on
  phrases like "write a blog post", "new article", "publish", "upload to blog", "create post",
  "写文章", "发布", "新建博客", or any mention of managing blog content. Also triggers when
  the user says "upload this" referring to blog content, or asks to "commit and push" blog changes.
---

# Blog Post Manager — Silentmoon 博客文章管理

博客基于 Hugo + Congo 主题，项目路径：`e:\Program Develop\silentmoon\New silentmoon\`。
所有文章存放在 `content/posts/` 下，GitHub 远端 `https://github.com/mistymoonz/silentmoon-blog.git`。

## 工作流程 1：新建文章

### 1.1 收集信息

向用户询问以下内容（每次只问缺口的信息）：

- **标题**（必填）：文章标题，中英文均可
- **标签**（可选）：逗号分隔，如 `["Go","后端","教程"]`。如果用户没给，先留空，后续发布时根据内容补充。
- **分类**（可选）：如 `编程`、`前端`、`碎碎念` 等。如果不给，根据标题和内容推断一个合适的分类。

不要一次问太多问题。如果用户只说了「新建文章」，先问标题和标签；分类可以推断。

### 1.2 生成 slug

从标题生成 URL slug：
- 英文标题：转小写，空格换连字符，去除非字母数字字符
- 中文标题：用拼音或英文关键词概括，长度 3-8 个词
- 如果标题本身就是英文（如 "Understanding LLMs"），直接转 slug：`understanding-llms`

### 1.3 创建文件

文件名格式：`YYYY-MM-DD-slug.md`，日期用 `{{date}}` 占位符（由 Claude 替换为当前日期）。

在 `content/posts/` 下创建文件，填入如下 frontmatter 模板：

```yaml
---
title: "{{用户输入的标题}}"
date: {{当前日期 YYYY-MM-DD}}
tags: [{{用户输入的标签}}]
category: "{{推断或用户输入的分类}}"
featuredImage: ""
description: ""
draft: false
---

正文开始...
```

- `date` 必须用 `YYYY-MM-DD` 格式，月和日始终两位（如 `2026-06-14`，不是 `2026-6-14`）
- `draft` 默认 `false`；如果用户说「草稿」，则设为 `true`
- `description` 先留空，发布时根据正文填充
- `featuredImage` 留空，如需封面图后续手动填

### 1.4 确认

创建文件后，将文件路径和 frontmatter 展示给用户确认。用户可以现在写正文，也可以之后再来写。

---

## 工作流程 2：发布 / 上传文章

用户会说「发布」「上传」「push」「提交」或直接给一个已有文章路径让你发布。

### 2.1 读取文章

读入目标 Markdown 文件的 frontmatter 和正文。确认以下几点：

- `date` 格式正确（`YYYY-MM-DD`，月日两位）
- `title` 非空
- 正文非空

### 2.2 智能补充 frontmatter

**生成描述（description）**：
阅读正文内容，提炼核心观点，生成一句中文摘要（15-40 字）。
- 要简洁准确，概括文章主旨
- 不包含「本文」「这篇文章」等冗余词语
- 直接陈述文章讲了什么

**补充标签（tags）**：
根据正文内容分析：
- 提取技术关键词（如出现的框架名、语言名、工具名）
- 提取主题词（如「性能优化」「入门教程」「源码分析」）
- 与用户已有的标签合并去重
- 建议 2-5 个标签，展示给用户确认后再写入

### 2.3 更新文件

将 `description` 和 `tags` 写回文件的 frontmatter。保持其他字段不变。展示最终 frontmatter 给用户确认。

### 2.4 Git 提交并推送

确认后执行：

```bash
cd "e:/Program Develop/silentmoon/New silentmoon"
git add content/posts/{{文件名}}.md
git commit -m "post: {{文章标题}}"
git push origin main
```

提交信息格式：`post: 文章标题`（用中文）。

### 2.5 完成通知

推送成功后告知用户文章已上线，并给出 GitHub 上的文件链接。

---

## 示例

**用户**：「新建文章，标题：Go 泛型实战指南」
**Claude**：
1. 询问标签和分类
2. 用户：标签 `["Go","泛型"]`，分类 `编程`
3. 生成 slug：`go-generics-guide`
4. 创建 `content/posts/2026-06-14-go-generics-guide.md`
5. 填入 frontmatter，展示给用户

**用户**：「把刚才那篇发布了」
**Claude**：
1. 读取 `content/posts/2026-06-14-go-generics-guide.md`
2. 分析正文，生成描述：「Go 1.18 泛型特性的完整实战教程，涵盖类型约束、泛型函数与性能对比。」
3. 建议补充标签：`["Go","泛型","类型约束","性能"]`
4. 用户确认后写入，commit，push

---

## 注意事项

- 日期格式严格 `YYYY-MM-DD`，月份和日期必须两位数字
- 标签使用 YAML 数组格式：`tags: ["标签1", "标签2"]`
- 分类和标签中的中文字符不需要转义
- 推送前始终让用户确认最终 frontmatter
- 如果 git push 失败（如网络问题），告知用户并保留本地更改
