# To-Comic-StudioFlow V12.4

> 单文件版中国小说转漫画生产流程。  
> V12.4 的核心是：**交付层级隔离 + 漫画页优先 + 旧V12漫画风味回归 + 冷启动资产硬门槛**。  
> 首次没有基础角色图和 `handoff.json` 时，必须先生成 `character_bootstrap/` 与 `scene_bootstrap/`，但这些只是独立资产包，不得混入漫画页。最终主交付必须是 `comic_pages/P01.png` 到 `comic_pages/P10.png` 十张独立漫画页。

---

## 0. 一句话目标

```text
输入一章小说，最终输出十张独立中国漫画页 P01.png 到 P10.png，并附带角色基础包、场景基础包、脚本文件、handoff.json 与 qc_report.json；不得把所有内容合成一张大图，不得用预览图替代漫画页。
```

---

## 1. 版本守卫

```yaml
version_guard:
  required_file_name: To-Comic-StudioFlow.md
  required_version: V12.4
  deprecated_names:
    - SKIIS_V12.md
    - SKIIS_V12(1).md
    - SKIIS_V12(2).md
    - To-Comic-StudioFlow V12.1
    - To-Comic-StudioFlow V12.2
    - To-Comic-StudioFlow V12.3
  if_uploaded_old_file:
    action:
      - warn_user: 请使用 To-Comic-StudioFlow.md V12.4
      - continue_only_if_content_contains: To-Comic-StudioFlow V12.4
```

旧版文件不得继续执行漫画生成。若文件不是 V12.4，必须提示用户换文件。

---

## 2. 交付层级隔离：最重要规则

```yaml
delivery_layer_separation:
  principle:
    - 生产资产、脚本文件、QC文件、漫画页必须分开交付
    - 不允许把所有文件内容拼成一张图片
    - 不允许把 character_bootstrap、scene_bootstrap、JSON表格和漫画页混在同一张图里

  character_bootstrap:
    output_type: 独立资产文件夹
    allowed_files:
      - cast_master_sheet.png
      - char_A_main_01.png
      - char_B_support_01.png
      - char_C_enemy_01.png
      - character_bootstrap.json
    forbid:
      - 混入 comic_pages
      - 画进 P01-P10
      - 合成到最终漫画页
      - 合成到 preview_sheet 主图

  scene_bootstrap:
    output_type: 独立资产文件夹
    allowed_files:
      - scene_master_sheet.png
      - scene_key_location_01.png
      - prop_key_item_01.png
      - scene_bootstrap.json
    forbid:
      - 混入 comic_pages
      - 画进 P01-P10
      - 合成到最终漫画页
      - 合成到 preview_sheet 主图

  metadata:
    output_type: 独立文本文件
    allowed_files:
      - chapter_card.json
      - director_beat_sheet.json
      - character_lock.json
      - scene_lock.json
      - page_script.json
      - handoff.json
      - qc_report.json
    forbid:
      - 画进 comic_pages
      - 画进 preview_sheet
      - 变成图片表格总览

  comic_pages:
    output_type: 独立漫画页文件夹
    required_files:
      - P01.png
      - P02.png
      - P03.png
      - P04.png
      - P05.png
      - P06.png
      - P07.png
      - P08.png
      - P09.png
      - P10.png
    forbid:
      - 资产总览图
      - 流程看板
      - JSON表格
      - 角色设定栏
      - 场景设定栏
      - QC报告栏
      - 一张图塞完整结果包
      - P01-P10缩略图合集冒充独立页

  preview_sheet:
    output_type: 可选缩略图
    allowed:
      - 仅将 P01.png 到 P10.png 缩略图排成总览
    generate_after:
      - all_individual_comic_pages_exist
    forbid:
      - 替代 P01-P10
      - 混入 JSON 表格
      - 混入 character_bootstrap
      - 混入 scene_bootstrap
      - 作为最终唯一图片
```

---

## 3. 三种启动模式

### A. 首次冷启动：没有基础角色图，也没有 handoff

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
```

系统内部必须先生成：

```text
character_bootstrap/
scene_bootstrap/
```

然后必须继续生成：

```text
comic_pages/P01.png ... P10.png
handoff.json
qc_report.json
```

不得停在角色包、场景包、脚本包或 prompt 包。

### B. 首次标准启动：有基础角色图，没有 handoff

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
3. 基础角色图
```

系统自动新建空白 `handoff.json`，以用户上传的基础角色图作为最高优先级视觉锚点，并补齐 `scene_bootstrap/`。

### C. 下一章继续：有上一期 handoff

用户输入：

```text
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一期 character_bootstrap
4. 上一期 handoff.json
```

系统必须继承上一期 `handoff.json` 的角色、道具、场景、未解决钩子，继续生成下一章 P01-P10 与新的 `handoff.json`。

---

## 4. 正文读取门槛

```yaml
source_text_gate:
  before_adaptation:
    must_have:
      - chapter_title
      - chapter_main_text
      - enough_plot_events

  fail_if:
    - only_url_without_content
    - only_title_detected
    - source_page_unreadable
    - chapter_text_too_short

  if_fail:
    action:
      - stop
      - ask_user_to_paste_chapter_text
      - do_not_generate_comic_pages
```

如果只读取到标题或链接，不得生成漫画页。

---

## 5. 冷启动资产硬门槛

```yaml
cold_start_trigger:
  no_base_character_images: true
  no_previous_handoff: true
```

冷启动必须先输出真实图片文件与 JSON：

```text
character_bootstrap/
  cast_master_sheet.png
  char_A_main_01.png
  char_B_support_01.png
  char_C_enemy_01.png
  character_bootstrap.json

scene_bootstrap/
  scene_master_sheet.png
  scene_key_location_01.png
  prop_key_item_01.png
  scene_bootstrap.json
```

但这些资产包不得变成最终主图，也不得与漫画页混在一张图。

```yaml
bootstrap_file_gate:
  fail_if:
    - only_text_description_without_images
    - only_json_without_visual_sheet
    - placeholder_character_sheet
    - placeholder_scene_sheet
    - stick_figure_character_sheet
    - blank_scene_sheet
    - asset_board_used_as_final_result
```

---

## 6. 漫画页优先级

```yaml
comic_page_first_rule:
  primary_delivery:
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

  secondary_delivery:
    - character_bootstrap/
    - scene_bootstrap/
    - metadata json files
    - preview_sheet.jpg

  final_answer_order:
    1: comic_pages/P01.png 到 P10.png
    2: handoff.json
    3: character_bootstrap 与 scene_bootstrap
    4: qc_report.json
    5: preview_sheet.jpg 如果存在
```

如果只生成 `preview_sheet.jpg` 或一张合成总览图，必须判定失败。

---

## 7. 最终文件验证门槛

```yaml
final_file_gate:
  before_final_answer:
    must_verify_files_exist:
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
      - handoff.json
      - qc_report.json
    cold_start_extra_required:
      - character_bootstrap/cast_master_sheet.png
      - scene_bootstrap/scene_master_sheet.png

  fail_if:
    - only_one_composite_image_exists
    - P01_to_P10_are_only_thumbnails_inside_preview_sheet
    - no_individual_page_files
    - final_answer_contains_only_preview_sheet
    - final_answer_contains_only_script_pack
    - final_answer_contains_only_render_manifest
```

---

## 8. 图像能力门槛

```yaml
image_generation_capability_gate:
  if_real_image_generation_available:
    action:
      - generate_character_bootstrap
      - generate_scene_bootstrap
      - generate_all_comic_pages
      - do_not_stop_at_prompts

  if_real_image_generation_unavailable:
    action:
      - stop
      - explain_unable_to_generate_images
      - output_script_pack_only_if_user_accepts

  forbid_as_comic_pages:
    - python_drawn_pages
    - svg_diagrams
    - html_canvas_layouts
    - stick_figure_pages
    - wireframe_storyboards
    - blank_layout_pages
    - script_card_pages
    - readable_test_pages
```

不得用程序绘图、简笔图、线框图冒充漫画成品页。

---

## 9. 旧 V12 漫画风味继承锁

V12.4 的目标不是改成暗黑资产图，而是在旧 V12 中国漫画连载风味基础上补齐冷启动资产。

```yaml
visual_style_regression_lock:
  reference_target:
    - 旧V12 P01-P10 中国漫画连载页风味

  preserve:
    - 独立漫画页结构
    - 每页 3-6 个分镜
    - 清晰角色近景
    - 战场/场景远景
    - 明确阅读顺序
    - 赛璐璐线稿感
    - 中国漫画连载感
    - 明亮或中性场景光线
    - 法宝光效服务剧情，不压过人物

  avoid:
    - 暗黑概念图
    - 游戏宣传图
    - 电影海报
    - 资产总览板
    - 全流程拼图
    - 把角色设定图和漫画页混在一起
    - 把场景设定图和漫画页混在一起
```

统一风格提示词：

```text
中国漫画连载页，偏日系人物线稿，清晰黑色漫画线，赛璐璐平涂，平涂色块，1-2层硬边阴影，背景有地标和空间美感但不电影化，分镜黑边清楚，竖向滚动漫画，人物表情漫画化，强镜头切换，留白节奏，像人类漫画工作室连载页，不是暗黑概念图，不是资产总览图。
```

负面提示：

```text
AI poster, cinematic CG, game concept art, glossy painting, painterly rendering, oil painting, realistic skin, 3D face, excessive glow, volumetric light, depth of field, hyper detailed background, same face syndrome, character sheet only, infographic, summary page, thumbnails, readable test page, stick figure, wireframe, placeholder layout, python drawing, svg diagram, asset board mixed with comic pages, dark concept art, production board, json table in image
```

---

## 10. 角色基础图标准

```yaml
character_bootstrap_standard:
  cast_master_sheet:
    purpose: 展示本章主要角色关系和视觉差异
    must_show:
      - 主角
      - 关键配角
      - 主要敌方
      - 群像阵营色块
      - 身高/体型对比
      - 发型差异
      - 脸型差异
      - 服装主色块
      - 标志性道具

  single_character_card:
    must_show:
      - 3/4角度半身或全身
      - 正脸识别
      - 发型
      - 脸型
      - 服装主轮廓
      - 主色块
      - 标志性道具
      - 基础表情

  first_bootstrap_do_not:
    - 不做复杂三视图
    - 不做完整表情包
    - 不做武器拆解页
    - 不做服装结构爆炸图
```

角色优先级：

```yaml
character_anchor_priority:
  1: 用户上传的基础角色图
  2: 上一期 handoff.json 中的角色状态
  3: 本期自动生成的 character_bootstrap
  4: 小说文字推断
```

---

## 11. 场景基础图标准

```yaml
scene_bootstrap_standard:
  scene_master_sheet:
    must_show:
      - 本章主场景地标
      - 主要空间关系
      - 色彩基调
      - 场景氛围
      - 可复用背景角度

  key_location_card:
    must_show:
      - 地标轮廓
      - 前景/中景/远景关系
      - 角色站位参考
      - 动作页可简化版本

  key_prop_card:
    must_show:
      - 本章关键道具外观
      - 道具发光/状态变化
      - 道具归属

  forbid:
    - blank_background
    - only_color_blocks
    - only_text_description
    - no_landmark
    - no_prop_visual
```

---

## 12. 自动工作流

```yaml
workflow:
  step_1_read_inputs:
    action:
      - 读取小说当前章节原文或链接内容
      - 读取基础角色图，如果有
      - 读取上一期 handoff.json，如果有
      - 读取本 To-Comic-StudioFlow.md

  step_2_version_and_source_gate:
    action:
      - 检查是否为 V12.4
      - 检查是否读到章节正文
      - 如果未读到正文，停止并要求用户粘贴正文

  step_3_start_mode_detection:
    action:
      - 判断是否有基础角色图
      - 判断是否有上一期 handoff.json
      - 如果二者都没有，进入 cold_start_character_bootstrap 与 scene_bootstrap

  step_4_bootstrap_assets:
    output:
      - character_bootstrap/
      - scene_bootstrap/
    note: 生成后不得停顿，不得把资产表拼入漫画页，必须继续后续步骤

  step_5_chapter_analysis:
    output:
      - chapter_card.json

  step_6_director_beat:
    output:
      - director_beat_sheet.json

  step_7_lock_assets:
    output:
      - character_lock.json
      - scene_lock.json

  step_8_page_script:
    output:
      - page_script.json
    action:
      - 生成 P01-P10 页面脚本
      - 每页 3-6 个分镜
      - 每页一个剧情变化点
      - 每页有 hook_line

  step_9_pre_page_gate:
    action:
      - 检查角色图锚点
      - 检查场景图锚点
      - 检查图像生成能力
      - 若缺失，补齐后继续；若无法补齐，停止，不得生成占位页

  step_10_generate_comic_pages:
    output:
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

  step_11_optional_preview_sheet:
    condition: all_P01_to_P10_exist
    output:
      - preview_sheet.jpg
    note: 仅作为附加缩略图，不得替代漫画页

  step_12_qc_report:
    output:
      - qc_report.json

  step_13_output_handoff:
    output:
      - handoff.json
```

---

## 13. P01-P10 结构

```yaml
page_structure:
  P01: 地点建立 + 主角发现异常
  P02: 关键物/关键规则识别
  P03: 主角行动，触发核心变化
  P04: 外部压力逼近
  P05: 敌方大场面登场
  P06: 正面对峙，敌方提出压力
  P07: 主角亮身份或亮底牌，暂时稳局
  P08: 旧怨/新冲突升级
  P09: 主角反问或反制，夺回节奏
  P10: 本章标题落点 + 下一期钩子
```

每页必须有：

```yaml
page_required:
  - page_goal
  - reader_question
  - page_answer
  - new_question
  - key_visual
  - hook_line
  - panels
```

---

## 14. 页面脚本格式

```yaml
page_script:
  page_no:
  page_goal:
  reader_hook:
  conflict_line:
  turn_or_reveal:
  end_hook:
  layout_plan:
  panels:
    - panel_id:
      panel_role: 建立/推进/反应/反转/钩子
      size: 大/中/小/窄长/斜切/无边框
      shot: 大远景/中景/近景/特写/极近特写/背影/俯视/仰视
      camera_intent: 神秘/压迫/速度/停顿/爽点
      image_prompt:
      no_text_image: true
      lettering:
        text:
        bubble_type:
        position:
      sfx:
```

---

## 15. 每页分镜规则

```yaml
page_panel_rule:
  panels_per_page: 3-6
  max_focus_faces: 3
  max_dialogue_bubbles: 5
  one_page_one_change: true

must_have_per_page:
  - 1个主视觉大格
  - 1个特写格
  - 1个反应格或静默格
  - 至少1个非普通横格
```

---

## 16. handoff.json 必须记录

```yaml
handoff_must_include:
  - 角色外观锚点
  - 角色当前状态
  - 角色图来源
  - 场景地标
  - 场景图来源
  - 道具归属与状态
  - 本章已解决事件
  - 下一章未解决钩子
  - 下次不能改变的内容
```

冷启动时 handoff 还必须记录：

```yaml
handoff_bootstrap_source:
  character_anchor_source:
    mode: auto_bootstrap
    generated_character_pack:
      - character_bootstrap/cast_master_sheet.png
      - character_bootstrap/char_A_main_01.png
      - character_bootstrap/char_B_support_01.png
      - character_bootstrap/char_C_enemy_01.png

  scene_anchor_source:
    mode: auto_bootstrap
    generated_scene_pack:
      - scene_bootstrap/scene_master_sheet.png
      - scene_bootstrap/scene_key_location_01.png
      - scene_bootstrap/prop_key_item_01.png
```

---

## 17. 用户启动提示词

### 首次无角色图

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md

这是首次冷启动，没有基础角色图，也没有 handoff.json。
请按 V12.4 工作：先生成 character_bootstrap 角色基础包和 scene_bootstrap 场景基础包，但不要把这些资产混进漫画页。
然后继续生成完整结果包：
- character_bootstrap/
- scene_bootstrap/
- chapter_card.json
- director_beat_sheet.json
- character_lock.json
- scene_lock.json
- page_script.json
- comic_pages/P01.png 到 P10.png 十张独立漫画页
- handoff.json
- qc_report.json
- preview_sheet.jpg（可选，不能替代 P01-P10）

如果无法生成真实图片，不要用简笔图、线框图、SVG 或 Python 示意图替代，也不要把脚本包说成完成。
```

### 首次有角色图

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图

这是第一期，没有上一期 handoff.json。
请自动新建 handoff，并补全 scene_bootstrap，然后继续输出本章完整结果包。
```

### 下一章继续

```text
请读取我上传的：
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一章 character_bootstrap
4. 上一期 handoff.json

请继承上一期 handoff，直接输出下一章 P01.png 到 P10.png 与新的 handoff.json。
```

---

## 18. V12.4 防 BUG 审计清单

```yaml
audit_cases:
  case_1_only_url_and_flow_file:
    expected:
      - read_chapter_text_or_stop
      - generate_character_bootstrap
      - generate_scene_bootstrap
      - generate_P01_to_P10_individual_pages
      - generate_handoff
      - do_not_stop_at_script_pack
      - do_not_output_single_production_board

  case_2_url_unreadable:
    expected:
      - stop
      - ask_user_to_paste_chapter_text
      - do_not_generate_pages

  case_3_no_image_generation_capability:
    expected:
      - stop
      - do_not_create_stick_figure_pages
      - do_not_create_svg_or_python_pages

  case_4_old_skiis_uploaded:
    expected:
      - warn_version_risk
      - require_To_Comic_StudioFlow_V12_4

  case_5_existing_handoff_no_character_image:
    expected:
      - use_handoff_character_state
      - if_visual_anchor_insufficient_generate_character_bootstrap_patch

  case_6_new_scene_in_next_chapter:
    expected:
      - generate_scene_bootstrap_patch
      - update_scene_lock

  case_7_output_single_summary_image:
    expected:
      - fail
      - require_P01_png_to_P10_png_separate_files

  case_8_script_card_pages:
    expected:
      - fail
      - regenerate_as_comic_pages

  case_9_partial_page_generation:
    expected:
      - fail_until_missing_pages_are_generated
      - do_not_finish_with_only_P08_P09

  case_10_style_drift_to_dark_concept_art:
    expected:
      - fail
      - return_to_chinese_comic_serial_page_style

  case_11_asset_board_mixed_with_comic_pages:
    expected:
      - fail
      - separate_assets_from_comic_pages
```

---

## 19. 验收标准

```yaml
acceptance:
  cold_start:
    if_no_base_character_images_and_no_handoff:
      character_bootstrap: required
      scene_bootstrap: required
      cast_master_sheet: required
      scene_master_sheet: required
      character_bootstrap_json: required
      scene_bootstrap_json: required
      no_placeholder_pages: required

  complete_result:
    chapter_card: required
    director_beat_sheet: required
    character_lock: required
    scene_lock: required
    page_script: required
    comic_pages_P01_to_P10_individual_files: required
    handoff: required
    qc_report: required

  hard_fail_if:
    - comic_pages_generated_without_character_bootstrap_when_cold_start
    - comic_pages_generated_without_scene_bootstrap_when_cold_start
    - pages_are_stick_figures
    - pages_are_wireframes
    - pages_are_svg_or_python_diagrams
    - pages_are_script_cards
    - no_downloadable_character_bootstrap_pack
    - no_downloadable_scene_bootstrap_pack
    - no_handoff_json
    - final_output_is_script_pack_only
    - final_output_is_partial_pages_only
    - asset_board_mixed_with_comic_pages_as_single_image
    - preview_sheet_replaces_individual_pages
    - production_board_presented_as_final_comic

  manga_feel:
    line_art_visible: required
    cel_shading_visible: required
    no_cg_background: required
    no_ai_poster: required
    chinese_comic_serial_page_feel: required

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

## 20. 验收测试方法

```yaml
acceptance_test_method:
  static_audit:
    purpose: 检查 To-Comic-StudioFlow.md 是否包含关键防 BUG 规则
    pass_conditions:
      - has_delivery_layer_separation
      - has_final_file_gate
      - has_preview_sheet_rule
      - has_visual_style_regression_lock
      - has_no_placeholder_policy

  dry_run_audit:
    purpose: 用典型输入场景推演是否会走正确流程
    cases:
      - 首次只上传小说链接和本文件
      - 首次上传小说链接和基础角色图
      - 下一章上传小说链接、角色包和 handoff
      - 链接不可读
      - 图像生成能力不可用
      - 输出只有一张总览图

  real_output_audit:
    purpose: 对实际生成结果包验收
    must_check:
      - 是否存在 comic_pages/P01.png 到 P10.png
      - 是否存在 character_bootstrap 与 scene_bootstrap
      - 是否存在 handoff.json 与 qc_report.json
      - P01-P10 是否为独立漫画页
      - 是否没有资产板混入漫画页
      - 是否保留中国漫画连载风味
```

仅做 `static_audit` 和 `dry_run_audit` 不能证明最终图片一定达标；必须对实际生成的输出包执行 `real_output_audit` 才能最终验收。
```