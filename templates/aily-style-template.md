# 飞书文档输出模板（aily风格）

> 版本：v20260620 | 基于aily实际文档逆向校准

## 格式规范

### 1. 文档结构
```
<title>文档标题</title>
<blockquote>来源标注</blockquote>
<callout emoji="💡">核心发现</callout>
<h2>章节1</h2> → <h2>章节2</h2> → ... → <callout emoji="🚀">趋势总结</callout>
```

### 2. 开头：核心发现高亮框
```
<callout emoji="💡">
**核心发现**：一句话概括核心结论，关键数据**加粗**标注。
</callout>
```
⚠️ 不使用 background-color / border-color 属性，只用 emoji。

### 3. 来源标注（blockquote）
```
<blockquote><p>本文档由 <a href="https://aily.feishu.cn/?&open-from=feishu_doc">飞书 aily</a> 创建</p></blockquote>
```
自定义时替换为 helloworld 智能体来源。

### 4. 章节配图
每个大章节至少配一张图（封面图/示意图/数据图），使用image_generate生成后通过docs +media-insert插入。
图片标签格式：
```
<img name="文件名.jpg" alt="图片描述文字" caption="图片标题" href="下载URL" mime="image/jpeg" scale="1.000000" src="文件token"/>
```

### 5. 数据对比用grid双栏
```
<grid>
  <column width-ratio="0.500000">
    **标题A**
    - 要点1
    - 要点2
  </column>
  <column width-ratio="0.500000">
    **标题B**
    - 要点1
    - 要点2
  </column>
</grid>
```
⚠️ 用 `width-ratio`（小数0-1），不用 `width`（整数百分比）。grid标签不需要 `cols` 属性。

### 6. 关键数据用表格
标准HTML表格格式：
```
<table>
  <colgroup><col/><col/></colgroup>
  <thead><tr><th><p>列名</p></th></tr></thead>
  <tbody><tr><td><p>数据</p></td></tr></tbody>
</table>
```
表头用 `<b>` 加粗。

### 7. 结尾：趋势总结高亮框
```
<callout emoji="🚀">
**核心趋势**：
1. 趋势1
2. 趋势2
3. 趋势3
</callout>
```
⚠️ 同样不使用 background-color / border-color 属性。

### 8. 引用脚注
行内引用格式：
```
<a href="https://来源URL">[1]</a>
```
多个引用连续排列：`<a href="url1">[1]</a><a href="url2">[7]</a>`

### 9. 画板（思维导图/流程图）——必须对齐aily
aily报告的核心特色之一是画板，必须包含以下两类：

**思维导图**（至少1个）：
- 适用场景：行业概览、技术分类体系、竞争格局梳理、市场细分结构
- 实现方式：在文档中插入 `<whiteboard token="画板token"></whiteboard>` 标签引用已创建的画板
- 关键要求：层级清晰（3层以上）、分支标注关键词、配色统一

**流程图**（至少1个）：
- 适用场景：技术演进路径、产业价值链、商业化落地流程、决策链路
- 实现方式：同上，用whiteboard技能绘制方框+箭头流程
- 关键要求：流向明确（从左到右或从上到下）、节点命名简洁、关键路径高亮

**位置建议**：
- 行业概览章节后 → 思维导图（梳理领域结构）
- 技术演进/趋势章节 → 流程图（展示演进路径）
- 市场格局章节 → 思维导图（竞争态势）
- 总计至少2个画板，推荐3个

### 10. 加粗格式
- 段落内关键词加粗用 `<b>` 标签（飞书原生渲染最稳定）
- 也可用 `**` markdown格式，但aily实际文档以 `<b>` 为主

## 生成流程
1. 收集内容素材（搜索/分析）
2. 按模板结构撰写markdown（含所有标签）
3. 生成封面图（image_generate，科技风格16:9）
4. 生成章节配图（每个大章节至少1张，image_generate）
5. 创建飞书文档（docs +create，content中含`<title>`标签）
6. 写入内容（docs +update，stdin管道方式，带callout/grid/table/whiteboard标签）
7. 插入封面图和章节配图（docs +media-insert）
8. 编辑画板内容：使用whiteboard技能绘制思维导图和流程图

## 对齐aily的关键要素（2026-06-20校准）
- ✅ 标题用 `<title>` 标签（非 --title 参数）
- ✅ 来源用 `<blockquote>` + `<a>` 链接（非 quote-container）
- ✅ callout只用 `emoji` 属性，不加 background-color / border-color
- ✅ grid用 `width-ratio`（小数），不用 `width`（整数）
- ✅ 表格用标准HTML `<table>` 格式
- ✅ 脚注用 `<a href="url">[n]</a>` 格式
- ✅ 加粗优先用 `<b>` 标签
- ✅ 画板用 `<whiteboard token="xxx"></whiteboard>` 引用
- ✅ 每章配图（含 alt / caption / scale 属性）
- ✅ 开头💡callout + 结尾🚀callout
- ✅ grid双栏对比
- ✅ 画板≥2个（思维导图+流程图）

## 历史变更
- v20260620：基于aily实际文档逆向校准，修正6处标签差异
- v20260501：初版创建，基于aily文档对比结论
