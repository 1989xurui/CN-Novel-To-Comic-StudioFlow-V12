# CN-Novel-To-Comic-StudioFlow-V12

中国小说转漫画的 AI 生产流程。目标是把一章小说稳定改编成 **中国漫画**，减少 AI 味，提升角色连续性、导演分镜感和正式连载可用性。

> 核心文件：[`To-Comic-StudioFlow.md`](./To-Comic-StudioFlow.md)  
> 用户只需要上传：小说章节原文/链接 + `To-Comic-StudioFlow.md` + 基础角色图/角色基础包 + 上一期 `handoff.json`，即可继续生成下一章漫画。

---

## 产品效果预览

下图是《永生》第254章《一个一个来》的 P01-P10 测试流程预览图：

![《永生》第254章《一个一个来》的 P01-P10 测试流程预览图](./docs/assets/yongsheng_254_preview.jpg)

说明：该图用于公开展示工作流效果。正式项目建议继续按“单格无字图 → 页面合成 → 中文排版”的流程提高完成度。

---

## 1. 项目是什么

`CN-Novel-To-Comic-StudioFlow-V12` 是一个面向 **小说转漫画** 的生产型流程，不是单纯的出图提示词。

它把传统“直接整页生成漫画”的方式改为更接近人类漫画团队的流程：

```text
小说章节原文/链接
→ 章节分析
→ 角色锁定/冷启动角色基础包
→ 场景锁定
→ 导演页纲
→ 页面脚本
→ 漫画页 P01-P10
→ handoff 交接文件
```

核心目标：

- 人物一眼分清
- 人物前后稳定
- 分镜像导演安排，不像图片拼贴
- 场景连续但不过度 CG 化
- 每一期结束后生成 `handoff.json`，方便下一章继续

---

## 2. 首次使用操作流程

第一次使用时，如果没有基础角色图和上一期 `handoff.json`，按下面做：

### Step 1：准备 2 个输入

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
```

### Step 2：把下面这段发给 GPT / Agent

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md

这是首次冷启动，没有基础角色图，也没有 handoff.json。
请先根据小说生成 character_bootstrap 角色基础包，然后直接输出本章完整结果包：
- character_bootstrap/
- chapter_card.json
- director_beat_sheet.json
- character_lock.json
- scene_lock.json
- page_script.json
- comic_pages/P01-P10.png
- handoff.json
```

### Step 3：保存输出

第一次完成后，必须保存：

```text
character_bootstrap/
comic_pages/P01-P10.png
page_script.json
handoff.json
```

其中 `character_bootstrap/` 和 `handoff.json` 是下一章继续生产的关键文件。

---

## 3. 下一章继续操作流程

从第二章开始，每次上传 4 个输入：

```text
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一章 character_bootstrap
4. 上一期 handoff.json
```

然后对 GPT / Agent 说：

```text
请读取我上传的：
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一章 character_bootstrap
4. 上一期 handoff.json

按 To-Comic-StudioFlow.md 工作。
继承上一期 handoff.json 的角色、场景、道具和未解决钩子。
直接输出下一章完整结果包：
- chapter_card.json
- director_beat_sheet.json
- character_lock.json
- scene_lock.json
- page_script.json
- comic_pages/P01-P10.png
- handoff.json
```

---

## 4. 输出规格

```yaml
UNIT: 1个小说章节 = 1期漫画
PAGE_COUNT:
  default: 10
  short_chapter: 6-8
PAGE_TYPE: 中国移动端竖向漫画页面段
CANVAS:
  master_width: 1600px
  export_width: 1280px
  height: 按内容变化，不固定9:16
STYLE:
  中国漫画
  偏日系人物线稿
  赛璐璐平涂
  清晰黑线
  中低细节
  强分镜节奏
  少AI光效
  少CG电影感
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

## 6. 正式生产原则

每章默认直接输出完整结果包，不停在单页测试：

```text
character_bootstrap/（仅首次冷启动需要）
→ chapter_card.json
→ director_beat_sheet.json
→ character_lock.json
→ scene_lock.json
→ page_script.json
→ comic_pages/P01-P10.png
→ handoff.json
```

如需更高完成度，建议再做二次精修：

```text
单格无字图
→ 页面合成
→ 中文气泡排版
→ 角色一致性复查
→ 最终导出
```

---

## 7. 示例：永生 第254章

`examples/yongsheng_254/` 包含《永生》第254章《一个一个来》的测试样例：

- P01-P10 页面脚本
- handoff 示例
- 测试图说明
- 生产注意事项

公开首页展示的是预览图：`docs/assets/yongsheng_254_preview.jpg`。

---

## 8. 验收标准

```yaml
acceptance:
  complete_result:
    character_bootstrap: required_if_cold_start
    chapter_card: required
    director_beat_sheet: required
    character_lock: required
    scene_lock: required
    page_script: required
    comic_pages: required
    handoff: required

  manga_feel:
    line_art_visible: required
    cel_shading_visible: required
    no_cg_background: required
    no_ai_poster: required

  character:
    main_character_consistent: required
    support_characters_distinct: required
    child_character_stable: required
    old_men_not_same_face: required
    group_characters_not_overdrawn: required

  director:
    one_page_one_question: required
    one_page_one_answer: required
    page_end_hook: required
    panel_shape_variety: required
    camera_angle_variety: required
    reaction_panel_present: required
```

---

## 9. 版权说明

本项目是漫画生产流程与工程模板。若用于改编真实小说、商业小说或平台连载内容，正式发布前需要确认版权授权。

---

## 10. 版本信息

```yaml
version: 12.1
main_file: To-Comic-StudioFlow.md
status: production-template
owner: 1989xurui
```
