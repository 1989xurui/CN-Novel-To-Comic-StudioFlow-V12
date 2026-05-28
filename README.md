# CN-Novel-To-Comic-StudioFlow-V12

中国小说转漫画的 AI 生产流程 SKIIS。目标是把一章小说稳定改编成 **中国漫画**，减少 AI 味，提升角色连续性、导演分镜感和正式连载可用性。

## 1. 项目是什么

`CN-Novel-To-Comic-StudioFlow-V12` 是一个面向 **小说转漫画** 的生产型 SKIIS，不是单纯的出图提示词。

它把传统“直接整页生成漫画”的方式改为更接近人类漫画团队的流程：

```text
小说章节原文
→ 章节分析
→ 角色锁定
→ 场景锁定
→ 导演页纲
→ 页面脚本
→ 单格无字图
→ 页面合成
→ 中文后期排版
→ handoff 交接文件
```

核心目标：

- 人物一眼分清
- 人物前后稳定
- 分镜像导演安排，不像图片拼贴
- 场景连续但不过度 CG 化
- 中文气泡后期排版，避免 AI 乱码
- 每一期结束后生成 handoff，方便下一章继续

## 2. 适用场景

适合：

- 中国玄幻 / 修真 / 武侠 / 都市异能 / 奇幻小说改漫画
- 移动端竖向条漫
- 连载型漫画项目
- 需要长期保持角色、场景、画风连续的生产流程
- 需要降低 AI 味、减少角色漂移、提升正式发布感的漫画项目

不适合：

- 一次性生成一张海报
- 一页塞完整章
- 直接复刻未授权商业漫画
- 纯 CG 概念图或电影分镜

## 3. 输出规格

```yaml
UNIT: 1个小说章节 = 1期漫画
PAGE_COUNT:
  default: 8
  dense_chapter: 10
  max: 10
PAGE_TYPE: 中国移动端竖向条漫页面段
CANVAS:
  master_width: 1600px
  export_width: 1280px
  height: 按内容变化，不固定9:16
STYLE:
  国漫彩条漫
  偏日系人物线稿
  赛璐璐平涂
  清晰黑线
  中低细节
  强分镜节奏
  少AI光效
  少CG电影感
```

## 4. 目录结构

```text
README.md
SKIIS_V12.md
docs/
  OPERATION_GUIDE.md
  NEXT_STEPS.md
  GITHUB_UPLOAD_GUIDE.md
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
    test_pages/
      README.md
```

## 5. 快速开始

每次新开 GPT / Agent 项目时，准备：

```text
1. 小说当前章节原文
2. SKIIS_V12.md
3. 基础角色图
4. 上一期 handoff.json
```

第一期没有 handoff 时，可以使用：

```text
templates/handoff.template.json
```

启动提示词见：`prompts/system_prompt.md`。

## 6. 正式生产流程

每页按这个流程：

```text
页面脚本
→ 单格无字图
→ 页面合成
→ 中文气泡排版
→ 检查角色和场景连续性
```

不要直接让 AI 一次生成完整最终页。

## 7. 示例

`examples/yongsheng_254/` 包含《永生》第254章《一个一个来》的测试样例：

- P01-P10 页面脚本
- handoff 示例
- 测试图说明
- 生产注意事项

测试图原始文件保存在本地交付包中。GitHub 仓库目前先保留脚本、模板和操作文档。

## 8. 验收标准

```yaml
acceptance:
  production:
    single_panel_generation: required
    page_compose: required
    lettering_after_generation: required
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

## 9. 版权说明

本项目是漫画生产流程与工程模板。若用于改编真实小说、商业小说或平台连载内容，正式发布前需要确认版权授权。

## 10. 版本信息

```yaml
version: 12.0
name: CN_Novel_To_Comic_StudioFlow_V12
status: production-template
owner: 1989xurui
```
