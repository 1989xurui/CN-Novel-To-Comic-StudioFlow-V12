# To-Comic-StudioFlow V12.2

> 单文件版中国小说转漫画生产流程。  
> V12.2 的核心是 **硬门槛**：首次没有基础角色图和 handoff 时，必须先生成 `character_bootstrap/` 与 `scene_bootstrap/`，再生成 `comic_pages/P01.png` 到 `P10.png`。  
> 不允许用简笔图、线框图、SVG/Python 示意图、脚本卡片页或“可阅读测试页”冒充漫画成品页。

---

## 0. 一句话目标

```text
输入一章小说，直接输出中国漫画 P01-P10、角色基础图包、场景基础图包和 handoff.json；下一章继续上传本文件 + 基础角色图/角色基础包 + 上一期 handoff + 新章节原文/链接，即可连续生产。
```

---

## 1. 版本守卫

```yaml
version_guard:
  required_file_name: To-Comic-StudioFlow.md
  required_version: V12.2
  deprecated_names:
    - SKIIS_V12.md
    - SKIIS_V12(1).md
    - SKIIS_V12(2).md
  if_uploaded_old_file:
    action:
      - warn_user: 请使用 To-Comic-StudioFlow.md V12.2
      - continue_only_if_content_contains: To-Comic-StudioFlow V12.2
```

用户上传旧版 `SKIIS_V12.md` 时，系统必须提醒版本风险。若旧文件没有 V12.2 的硬门槛规则，不得按旧规则生成漫画页。

---

## 2. 三种启动模式

### A. 首次冷启动：没有基础角色图，也没有 handoff

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
```

系统必须先生成：

```text
character_bootstrap/
scene_bootstrap/
```

然后才允许生成：

```text
comic_pages/P01.png ... P10.png
handoff.json
```

### B. 首次标准启动：有基础角色图，没有 handoff

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
3. 基础角色图
```

系统自动新建空白 `handoff.json`，并以用户上传的基础角色图作为最高优先级视觉锚点。仍需生成或补全 `scene_bootstrap/`，用于保证场景美术连续。

### C. 下一章继续：有上一期 handoff

用户输入：

```text
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一期 character_bootstrap
4. 上一期 handoff.json
```

系统必须继承上一期 `handoff.json` 的角色外观、道具状态、场景状态和未解决钩子，然后直接生成下一章 P01-P10 与新的 `handoff.json`。

---

## 3. 正文读取门槛

如果用户提供的是小说链接，系统必须先读取章节正文。只读到标题、只读到 URL、只读到网页说明，不得生成漫画页。

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

---

## 4. 冷启动角色基础图硬门槛

当用户没有上传基础角色图，且没有上一期 `handoff.json` 时，系统必须进入“冷启动角色建档模式”。

```yaml
cold_start_trigger:
  no_base_character_images: true
  no_previous_handoff: true
```

冷启动禁止项：

```yaml
cold_start_forbid:
  - 不允许直接生成 comic_pages/P01-P10
  - 不允许输出无角色漫画页
  - 不允许输出简笔占位角色
  - 不允许生成章节总览图冒充漫画页
  - 不允许只输出 JSON 而不生成角色基础图
  - 不允许只输出角色说明而不生成角色基础图
```

必须先输出真实图片文件和 JSON：

```yaml
cold_start_required_outputs:
  - character_bootstrap/cast_master_sheet.png
  - character_bootstrap/char_A_main_01.png
  - character_bootstrap/char_B_support_01.png
  - character_bootstrap/char_C_enemy_01.png
  - character_bootstrap/character_bootstrap.json
```

文件级硬门槛：

```yaml
character_bootstrap_file_gate:
  cold_start_must_have_actual_files:
    - character_bootstrap/cast_master_sheet.png
    - character_bootstrap/char_A_main_01.png
    - character_bootstrap/character_bootstrap.json

  fail_if:
    - only_text_description_without_images
    - only_character_plan_without_downloadable_files
    - only_json_without_visual_character_sheet
    - placeholder_character_sheet
    - stick_figure_character_sheet
```

角色图生成来源：

```yaml
character_bootstrap_sources:
  text_explicit:
    use: 小说中明确写出的外貌、身份、服装、法宝、称谓
  text_inferred:
    use: 从角色行为、语气、地位、战斗方式推断气质和视觉特征
  genre_inferred:
    use: 当小说没有明确外貌时，根据题材、门派、修为、身份生成临时视觉方案
  user_base_image:
    use: 如果用户后来上传基础角色图，必须覆盖系统推断图
```

`character_bootstrap.json` 必须标记每个角色信息来源：

```yaml
source_type:
  - text_explicit
  - text_inferred
  - genre_inferred
  - user_base_image
```

---

## 5. 场景基础图硬门槛

首次冷启动不仅要有角色，还必须有场景美术锚点。否则会出现空白背景、低美感、无场景连续的问题。

```yaml
scene_bootstrap_required_when:
  - first_cold_start
  - chapter_has_new_key_location
  - previous_handoff_has_no_scene_state
```

必须输出：

```text
scene_bootstrap/
  scene_master_sheet.png
  scene_key_location_01.png
  prop_key_item_01.png
  scene_bootstrap.json
```

场景基础图必须包含：

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
```

禁止：

```yaml
scene_bootstrap_forbid:
  - blank_background
  - only_color_blocks
  - only_text_description
  - no_landmark
  - no_prop_visual
```

---

## 6. 图像能力门槛

如果当前环境无法生成真实图像，系统必须停止，不得用程序绘图或占位图冒充漫画页。

```yaml
image_generation_capability_gate:
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

---

## 7. 生成漫画页前置门槛

```yaml
pre_page_gate:
  before_generating_comic_pages:
    must_have:
      - chapter_main_text
      - page_script.json
    must_have_one_of:
      - base_character_images
      - previous_handoff_with_character_state
      - generated_character_bootstrap
    must_have_one_of_scene:
      - previous_handoff_with_scene_state
      - generated_scene_bootstrap

  if_missing_character_anchor:
    action: create_character_bootstrap_first

  if_missing_scene_anchor:
    action: create_scene_bootstrap_first

  if_missing_real_image_generation:
    action: stop
```

---

## 8. 默认输出结果包

默认不要停在 P01 测试页，也不要停在 P08 测试页。除非用户明确说“先测试”，否则直接输出完整结果包。

```yaml
default_outputs:
  - character_bootstrap/            # 冷启动时必须输出
  - scene_bootstrap/                # 冷启动或新场景时必须输出
  - chapter_card.json
  - director_beat_sheet.json
  - character_lock.json
  - scene_lock.json
  - page_script.json
  - comic_pages/
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
  - handoff.json
  - qc_report.json
```

严禁把 `P01-P10.png` 当成一个总览图。必须是 10 个独立页面文件：`P01.png` 到 `P10.png`。

---

## 9. 漫画页质量门槛

删除“可阅读测试页”概念。允许的是正式视觉页或生产预览漫画页，不允许脚本示意页。

```yaml
comic_page_quality_gate:
  allowed:
    - rendered_comic_page
    - production_preview_comic_page

  forbidden:
    - readable_test_page
    - placeholder_page
    - wireframe_page
    - stick_figure_page
    - blank_layout_page
    - script_card_page
    - python_drawn_schema
    - svg_diagram
    - chapter_summary_sheet
```

每页必须满足：

```yaml
page_qc:
  each_page:
    must_contain:
      - at_least_one_visible_story_character_if_script_requires
      - clear_story_action
      - clear_relationship_or_conflict
      - visual_continuity_with_character_lock
      - scene_or_background_when_required
      - panel_shape_variety

  fail_conditions:
    - no_character_when_character_should_appear
    - character_not_matching_bootstrap
    - background_blank_when_scene_required
    - all_panels_same_angle
    - all_panels_same_distance
    - no_page_hook
```

---

## 10. 角色基础图标准

### cast_master_sheet.png

作用：让用户一眼看到本章主要角色关系和视觉差异。

```yaml
cast_master_sheet:
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
```

### 单角色图

每个 A/B/C 级角色至少一张：

```yaml
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
```

首次不要做过重设定：

```yaml
first_bootstrap_do_not:
  - 不做复杂三视图
  - 不做完整表情包
  - 不做武器拆解页
  - 不做服装结构爆炸图
```

---

## 11. 角色优先级

```yaml
character_bootstrap_priority:
  A_main:
    required: true
    target: 主角
    must_define:
      - silhouette
      - face_shape
      - eye_shape
      - hair_shape
      - outfit_shape
      - color_block
      - prop_or_mark
      - speech_style

  B_key_support:
    required: if_present
    max_count: 2
    target: 器灵、同伴、幼态角色、重要辅助角色

  C_named_pressure:
    required: if_present
    max_count: 3
    target: 主要敌方、长老、对手、压迫者

  D_crowd:
    required: if_present
    output: 阵营色块与群像规则，不单独生成复杂角色图
```

基础角色图优先级：

```yaml
character_anchor_priority:
  1: 用户上传的基础角色图
  2: 上一期 handoff.json 中的角色状态
  3: 本期自动生成的 character_bootstrap
  4: 小说文字推断
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
      - 检查是否为 V12.2
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
      - 若缺失，停止或补齐，不得生成占位页

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

  step_11_qc_report:
    output:
      - qc_report.json

  step_12_output_handoff:
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

## 16. 画风锁定

```yaml
STYLE_LOCK:
  name: 中国漫画_日系线稿_赛璐璐平涂_导演分镜版

  line:
    - 清晰黑色漫画线稿
    - 主轮廓略粗
    - 内部线条较细
    - 允许少量手绘粗糙感

  color:
    - 赛璐璐平涂
    - 平涂色块
    - 1-2层硬边阴影
    - 少渐变
    - 少法宝光
    - 少材质纹理

  character:
    - 偏日系人物脸型
    - 鼻口简化
    - 眼睛有表现力
    - 表情漫画化
    - 中国玄幻/中国漫画服装轮廓

  background:
    - 背景简化但不能空白
    - 只保留关键地标
    - 不要CG电影感
    - 不要游戏场景概念图

  panel:
    - 黑边分镜
    - 竖向滚动
    - 大中小切块变化
    - 留白节奏
    - 速度线和拟声字
```

统一风格提示词：

```text
中国漫画，偏日系人物线稿，清晰黑色漫画线，主轮廓略粗，内部线条较细，赛璐璐平涂，平涂色块，1-2层硬边阴影，少量局部法宝光，背景简化但不空白，分镜黑边清楚，竖向滚动漫画，人物表情漫画化，强镜头切换，留白节奏，像人类漫画工作室连载页。
```

负面提示：

```text
AI poster, cinematic CG, game concept art, glossy painting, painterly rendering, oil painting, realistic skin, 3D face, excessive glow, volumetric light, depth of field, hyper detailed background, same face syndrome, character sheet only, infographic, summary page, thumbnails, readable test page, stick figure, wireframe, placeholder layout, python drawing, svg diagram
```

---

## 17. handoff.json 必须记录

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

## 18. 用户启动提示词

### 首次无角色图

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md

这是首次冷启动，没有基础角色图，也没有 handoff.json。
请先根据小说生成 character_bootstrap 角色基础包和 scene_bootstrap 场景基础包。
然后直接输出本章完整结果包：
- character_bootstrap/
- scene_bootstrap/
- chapter_card.json
- director_beat_sheet.json
- character_lock.json
- scene_lock.json
- page_script.json
- comic_pages/P01.png 到 P10.png
- handoff.json
- qc_report.json

如果无法生成真实图片，不要用简笔图、线框图、SVG 或 Python 示意图替代。
```

### 首次有角色图

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图

这是第一期，没有上一期 handoff.json。
请自动新建 handoff，并补全 scene_bootstrap，然后直接输出本章完整结果包。
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

## 19. V12.2 防 BUG 审计清单

```yaml
audit_cases:
  case_1_only_url_and_flow_file:
    expected:
      - read_chapter_text_or_stop
      - generate_character_bootstrap
      - generate_scene_bootstrap
      - generate_P01_to_P10
      - generate_handoff

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
      - require_To_Comic_StudioFlow_V12_2

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
```

---

## 20. 验收标准

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
    comic_pages_P01_to_P10: required
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
