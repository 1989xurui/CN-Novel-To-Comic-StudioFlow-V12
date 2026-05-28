# CN-Novel-To-Comic-StudioFlow-V13

中国小说转漫画的 AI 生产流程。当前主版本为 **To-Comic-StudioFlow V13.0：漫画导演型 SKIIS**。

目标：输入一章小说，固定输出 **10 张独立中国漫画页 P01.png 到 P10.png**，并附带角色基础包、场景基础包、剧情锁定文件、脚本文件、handoff.json、qc_report.json。整体要求是：剧情保真、角色鲜明、分镜足够、台词有内容、画风接近中国主流彩色漫画 / 国漫连载页，禁止 CG 概念图、剧情摘要页、生产看板图。

> 核心文件：[`To-Comic-StudioFlow.md`](./To-Comic-StudioFlow.md)  
> 推荐用途：个人娱乐、非商业化、不公开发布、内部流程测试。

---

## 产品效果预览

下图是《永生》第254章《一个一个来》的 P01-P10 测试流程预览图：

<img width="1414" height="812" alt="Yongsheng Chapter 254 P01-P10 Preview" src="https://github.com/user-attachments/assets/12d11f92-a064-4726-ab6b-04506c1ae8f6" />

说明：该图用于公开展示工作流效果。V13.0 要求最终交付为 `comic_pages/P01.png` 到 `P10.png` 十张独立漫画页，不允许用一张总览图替代 P01-P10。

---

## 1. 项目是什么

`CN-Novel-To-Comic-StudioFlow` 是一个面向 **中国小说转漫画** 的生产型流程，不是单纯的出图提示词。

V13.0 的核心不是“只防出错”，而是把流程升级为 **漫画导演型工作流**：

```text
小说章节原文/链接
→ source_event_lock.json 剧情保真锁
→ 角色基础包 character_bootstrap/
→ 场景基础包 scene_bootstrap/
→ 漫画导演页纲 director_beat_sheet.json
→ page_script.json
→ comic_pages/P01.png 到 P10.png
→ handoff.json
→ qc_report.json
```

核心目标：

- 固定 10 页，不再摇摆
- 每页 5 到 7 个分镜，低于 5 格判失败
- 先锁剧情，再做漫画
- 禁止剧情摘要式脚本
- 台词要表达内容，不为了防乱码缩水成空话
- 角色基础包要鲜明、有记忆点
- 场景服务人物与动作，禁止 CG 概念图
- 个人娱乐 / 非商业 / 不公开发布只做记录，不干扰剧情

---

## 2. 首次冷启动使用流程

第一次使用时，如果没有基础角色图和上一期 `handoff.json`，只需要准备：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
```

### 可直接复制的首次冷启动提示词

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md

这是个人娱乐用途，非商业化，不公开发布，仅用于内部流程测试。
这是首次冷启动，没有基础角色图，也没有 handoff.json。
请按 To-Comic-StudioFlow V13.0 工作。

要求：
- 固定输出 10 页漫画：P01.png 到 P10.png
- 每页分镜不能少于 5 格，标准 6 格，最多 7 格
- 禁止把漫画做成剧情摘要
- 台词要能表达内容，不要为了防乱码而缩水成空话
- 必须先生成 source_event_lock.json，严格保留主角、敌人、关键法宝、事件顺序、冲突关系和结尾钩子
- 角色基础包必须鲜明、有记忆点，接近中国主流漫画角色包水平
- 禁止 CG 概念图、游戏宣传图、海报化结果
- 必须输出：
  source_event_lock.json
  character_bootstrap/
  scene_bootstrap/
  page_script.json
  comic_pages/P01.png 到 P10.png
  handoff.json
  qc_report.json

如果不能忠实改编，请直接停止说明，不要静默改写剧情。
```

首次完成后，必须保存：

```text
source_event_lock.json
character_bootstrap/
scene_bootstrap/
comic_pages/P01.png 到 P10.png
page_script.json
handoff.json
qc_report.json
```

其中 `character_bootstrap/`、`scene_bootstrap/` 和 `handoff.json` 是下一章继续生产的关键资产。

---

## 3. 下一章继续使用流程

从第二章开始，每次上传：

```text
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一期 character_bootstrap/
4. 上一期 handoff.json
```

### 可直接复制的下一章继续提示词

```text
请读取我上传的：
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一期 character_bootstrap
4. 上一期 handoff.json

这是个人娱乐用途，非商业化，不公开发布，仅用于内部流程测试。
请按 To-Comic-StudioFlow V13.0 工作，继续输出本章 P01.png 到 P10.png。

要求继续保持：
- 固定10页
- 每页5到7格
- 剧情保真
- 角色稳定
- 禁止剧情摘要
- 禁止 CG 概念图
```

---

## 4. V13.0 输出规格

```yaml
UNIT: 1个小说章节 = 1期漫画
PAGE_COUNT: 固定10页
PAGE_FILES:
  - comic_pages/P01.png
  - comic_pages/P02.png
  - comic_pages/P03.png
  - comic_pages/P04.png
  - comic_pages/P05.png
  - comic_pages/P06.png
  - comic_pages/P07.png
  - comic_pages/P08.png
  - comic_pages/P09.png
  - comic_pages/P10.png
PANELS_PER_PAGE:
  min: 5
  target: 6
  max: 7
STYLE:
  中国主流彩色漫画连载页
  偏日系清线
  清楚黑色线稿
  赛璐璐平涂
  1到2层硬边阴影
  禁止CG概念图
  禁止游戏宣传图
  禁止剧情摘要页
```

---

## 5. 目录结构

```text
README.md
To-Comic-StudioFlow.md
docs/
  assets/
    yongsheng_254_preview.jpg
prompts/
  style_prompt.md
  system_prompt.md
templates/
  chapter_card.template.json
  director_beat_sheet.template.json
  character_lock.template.json
  scene_lock.template.json
  page_script.template.json
  handoff.template.json
examples/
  yongsheng_254/
    README.md
    page_script.example.json
    handoff.example.json
    metadata/manifest.json
```

---

## 6. 生产原则

V13.0 不允许把流程文件、角色设定表、场景设定表、QC 表、JSON 表格画进漫画页。

正确交付：

```text
source_event_lock.json
character_bootstrap/
scene_bootstrap/
page_script.json
director_beat_sheet.json
comic_pages/P01.png
comic_pages/P02.png
comic_pages/P03.png
comic_pages/P04.png
comic_pages/P05.png
comic_pages/P06.png
comic_pages/P07.png
comic_pages/P08.png
comic_pages/P09.png
comic_pages/P10.png
handoff.json
qc_report.json
```

错误交付：

```text
一张总览图代替 P01-P10
一张生产看板代替漫画页
一张缩略合集代替十张独立页
用剧情摘要页代替漫画
用 CG 概念图代替漫画分镜
```

---

## 7. 验收标准

### 剧情验收

```yaml
source_fidelity_acceptance:
  required:
    - main_character_correct
    - antagonist_correct
    - key_props_correct
    - event_order_correct
    - ending_hook_correct
  hard_fail_if:
    - story_silently_rewritten
    - renamed_core_characters
    - replaced_key_props
    - changed_conflict_axis
```

### 漫画导演验收

```yaml
director_acceptance:
  required:
    - exactly_10_pages
    - every_page_has_5_to_7_panels
    - every_page_has_conflict
    - every_page_has_turn
    - every_page_has_hook
    - no_plot_summary_pages
```

### 角色验收

```yaml
character_acceptance:
  required:
    - main_characters_distinct
    - support_characters_distinct
    - child_or_mascot_stable
    - no_same_face_old_men
```

### 视觉验收

```yaml
style_acceptance:
  required:
    - chinese_manhua_serial_feel
    - japanese_influenced_clean_line
    - cel_shading
    - no_cg_concept_art
    - no_poster_like_rendering
    - background_not_stealing_focus
```

### 交付验收

```yaml
delivery_acceptance:
  required:
    - source_event_lock_exists
    - character_bootstrap_exists
    - scene_bootstrap_exists
    - P01_to_P10_all_exist
    - handoff_exists
    - qc_report_exists
```

---

## 8. 版权与使用说明

本项目是漫画生产流程与工程模板。若用于改编真实小说、商业小说或平台连载内容，正式公开发布前需要确认版权授权。

V13.0 支持“个人娱乐、非商业化、不公开发布、仅用于内部流程测试”的使用声明。该声明只用于减少反复打断，不代表授予公开发布或商业使用权。

---

## 9. 版本信息

```yaml
version: 13.0
main_file: To-Comic-StudioFlow.md
status: director-first-production-template
owner: 1989xurui
```
