# To-Comic-StudioFlow V12.4.1

> 单文件版中国小说转漫画生产流程。  
> V12.4.1 在 V12.4 基础上重点修复两个执行层 BUG：  
> 1）模型不能一次完成时，不能停在中间还说完成；  
> 2）模型不能输出生产看板/资产总览图并当作最终漫画结果。  
>
> 最终主交付必须是 `comic_pages/P01.png` 到 `comic_pages/P10.png` 十张独立中国漫画页；`character_bootstrap/`、`scene_bootstrap/`、JSON、QC、preview_sheet 都只是辅助交付，不得混入漫画页，不得替代十张独立页面。

---

## 0. 一句话目标

```text
输入一章小说，最终输出十张独立中国漫画页 P01.png 到 P10.png，并附带角色基础包、场景基础包、脚本文件、handoff.json、qc_report.json 与 output_manifest.json。不得把所有内容合成一张大图，不得用预览图、看板图、脚本包或 render_manifest 替代漫画页。
```

---

## 1. 版本守卫

```yaml
version_guard:
  required_file_name: To-Comic-StudioFlow.md
  required_version: V12.4.1
  deprecated_names:
    - SKIIS_V12.md
    - SKIIS_V12(1).md
    - SKIIS_V12(2).md
    - To-Comic-StudioFlow V12.1
    - To-Comic-StudioFlow V12.2
    - To-Comic-StudioFlow V12.3
    - To-Comic-StudioFlow V12.4
  if_uploaded_old_file:
    action:
      - warn_user: 请使用 To-Comic-StudioFlow.md V12.4.1
      - continue_only_if_content_contains: To-Comic-StudioFlow V12.4.1
```

旧版文件不得继续执行漫画生成。若文件不是 V12.4.1，必须提醒用户换文件。

---

## 2. 三种启动模式

### A. 首次冷启动：没有基础角色图，也没有 handoff

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
```

系统必须内部先生成：

```text
character_bootstrap/
scene_bootstrap/
```

然后必须继续生成：

```text
comic_pages/P01.png ... P10.png
handoff.json
qc_report.json
output_manifest.json
```

不得停在角色包、场景包、脚本包、prompt 包、render_manifest 或 preview_sheet。

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

## 3. 正文读取门槛

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
    status: BLOCKED
    action:
      - stop
      - ask_user_to_paste_chapter_text
      - do_not_generate_comic_pages
    next_step_must_say:
      - 请粘贴小说当前章节正文，或上传可读取的章节文本文件。
```

如果只读取到标题或链接，不得生成漫画页。

---

## 4. 交付层级隔离：最重要规则

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
      - output_manifest.json
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
      - 双页合图冒充单页

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
    4: qc_report.json 与 output_manifest.json
    5: preview_sheet.jpg 如果存在
```

如果只生成 `preview_sheet.jpg`、一张合成总览图、脚本包、prompt包或 render_manifest，必须判定失败。

---

## 7. 单页独立规则

```yaml
single_page_file_rule:
  each_comic_page:
    must_contain:
      - only_one_page
      - 3_to_6_panels
      - one_page_story_beat
    must_not_contain:
      - other_page_thumbnails
      - character_sheet
      - scene_sheet
      - json_table
      - qc_table
      - production_board

  forbid:
    - P01和P02合在一张图
    - 一张图放多页缩略图
    - 一张图放完整章节
    - preview_sheet冒充单页
```

---

## 8. 中文后期排版规则

```yaml
lettering_layer_rule:
  preferred_flow:
    - 先生成无字或少字漫画页 comic_pages_raw/P01_no_text.png 到 P10_no_text.png
    - 再用 lettering_data.json 添加中文对白、旁白、拟声词
    - 最终输出带字版 comic_pages/P01.png 到 P10.png

  required_outputs_when_possible:
    - lettering_data.json

  forbid:
    - 让图像模型直接生成大量中文小字
    - 中文乱码页直接通过
    - 气泡遮挡人物脸
    - 台词与角色关系不一致
```

如图像生成工具无法稳定生成中文，必须采用无字图 + 后期排版；不得让乱码中文进入最终成品页。

---

## 9. 最终文件验证门槛

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
      - output_manifest.json
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

## 10. output_manifest.json 必须输出

```yaml
output_manifest:
  purpose: 防止口头说完成但文件不存在
  must_include:
    comic_pages:
      P01.png: exists/type/validation_status
      P02.png: exists/type/validation_status
      P03.png: exists/type/validation_status
      P04.png: exists/type/validation_status
      P05.png: exists/type/validation_status
      P06.png: exists/type/validation_status
      P07.png: exists/type/validation_status
      P08.png: exists/type/validation_status
      P09.png: exists/type/validation_status
      P10.png: exists/type/validation_status
    bootstrap:
      character_bootstrap: exists/validation_status
      scene_bootstrap: exists/validation_status
    metadata:
      handoff_json: exists
      qc_report_json: exists
      lettering_data_json: exists_if_used
    final_status: PASS_or_IN_PROGRESS_or_BLOCKED_or_FAIL
```

`output_manifest.json` 中如果 `final_status` 不是 `PASS`，最终答复不得写“完成”。

---

## 11. 分批继续协议：不能一次完成时怎么做

V12.4.1 允许因为工具限制分批生成，但不允许把中间状态说成完成。

```yaml
batch_continuation_protocol:
  when_cannot_finish_in_one_response:
    status: IN_PROGRESS
    must_output:
      - production_state.json
      - output_manifest.json
      - next_batch_instruction.txt
    must_not_say:
      - 完成
      - 已交付完整结果包
      - P01-P10已生成
    final_result_invalid_until:
      - comic_pages/P01.png_to_P10.png_all_exist
      - handoff.json_exists
      - qc_report.json_exists
      - output_manifest_final_status_PASS

  next_batch_instruction_must_include:
    - 已完成文件列表
    - 缺失文件列表
    - 下一批必须生成的文件
    - 继续口令

  continue_prompt_template:
    text: 继续按 production_state.json 从缺失文件开始生成，不要重做已完成文件；直到 comic_pages/P01.png 到 P10.png、handoff.json、qc_report.json、output_manifest.json 全部存在，才允许说完成。
```

如果被迫停止，必须明确告诉用户下一步操作，例如：

```text
当前不是最终结果，状态为 IN_PROGRESS。下一步请回复“继续”，系统必须从 production_state.json 记录的缺失文件继续生成，不得重做已完成文件。
```

如果因为缺少图像生成能力被阻断，必须明确告诉用户：

```text
当前环境无法生成真实漫画图。下一步请切换到支持真实图像生成的环境，或上传基础角色图/场景图后继续。不得用简笔图、SVG、Python示意图替代。
```

---

## 12. 图像能力门槛

```yaml
image_generation_capability_gate:
  if_real_image_generation_available:
    action:
      - generate_character_bootstrap
      - generate_scene_bootstrap
      - generate_all_comic_pages
      - do_not_stop_at_prompts

  if_real_image_generation_unavailable:
    status: BLOCKED
    action:
      - stop
      - explain_unable_to_generate_images
      - output_script_pack_only_if_user_accepts
      - provide_next_step_instruction

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

## 13. 旧 V12 漫画风味继承锁

V12.4.1 的目标不是改成暗黑资产图，而是在旧 V12 中国漫画连载风味基础上补齐冷启动资产。

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

## 14. 角色基础图标准

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

## 15. 场景基础图标准

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

## 16. 自动工作流

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
      - 检查是否为 V12.4.1
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

  step_10_generate_comic_pages_raw:
    preferred_output:
      - comic_pages_raw/P01_no_text.png
      - comic_pages_raw/P02_no_text.png
      - comic_pages_raw/P03_no_text.png
      - comic_pages_raw/P04_no_text.png
      - comic_pages_raw/P05_no_text.png
      - comic_pages_raw/P06_no_text.png
      - comic_pages_raw/P07_no_text.png
      - comic_pages_raw/P08_no_text.png
      - comic_pages_raw/P09_no_text.png
      - comic_pages_raw/P10_no_text.png

  step_11_lettering:
    output:
      - lettering_data.json
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

  step_12_optional_preview_sheet:
    condition: all_P01_to_P10_exist
    output:
      - preview_sheet.jpg
    note: 仅作为附加缩略图，不得替代漫画页

  step_13_qc_report:
    output:
      - qc_report.json

  step_14_output_manifest:
    output:
      - output_manifest.json

  step_15_output_handoff:
    output:
      - handoff.json
```

---

## 17. P01-P10 结构

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

## 18. 用户启动提示词

### 首次无角色图

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md

这是首次冷启动，没有基础角色图，也没有 handoff.json。
请按 V12.4.1 工作：先生成 character_bootstrap 角色基础包和 scene_bootstrap 场景基础包，但不要把这些资产混进漫画页。
然后继续生成完整结果包：
- character_bootstrap/
- scene_bootstrap/
- chapter_card.json
- director_beat_sheet.json
- character_lock.json
- scene_lock.json
- page_script.json
- comic_pages_raw/P01_no_text.png 到 P10_no_text.png（如可行）
- lettering_data.json
- comic_pages/P01.png 到 P10.png 十张独立漫画页
- handoff.json
- qc_report.json
- output_manifest.json
- preview_sheet.jpg（可选，不能替代 P01-P10）

如果不能一次完成，必须输出 production_state.json、output_manifest.json、next_batch_instruction.txt，并标记 IN_PROGRESS，不得说完成。
如果无法生成真实图片，不要用简笔图、线框图、SVG 或 Python 示意图替代，也不要把脚本包说成完成。
```

---

## 19. V12.4.1 防 BUG 审计清单

```yaml
audit_cases:
  case_1_only_url_and_flow_file:
    expected:
      - read_chapter_text_or_stop
      - generate_character_bootstrap
      - generate_scene_bootstrap
      - generate_P01_to_P10_individual_pages
      - generate_handoff
      - generate_output_manifest
      - do_not_stop_at_script_pack
      - do_not_output_single_production_board

  case_2_url_unreadable:
    expected:
      - stop
      - ask_user_to_paste_chapter_text
      - provide_next_step_instruction
      - do_not_generate_pages

  case_3_no_image_generation_capability:
    expected:
      - stop
      - explain_next_step
      - do_not_create_stick_figure_pages
      - do_not_create_svg_or_python_pages

  case_4_output_single_summary_image:
    expected:
      - fail
      - require_P01_png_to_P10_png_separate_files

  case_5_partial_page_generation:
    expected:
      - status_IN_PROGRESS
      - list_missing_pages
      - provide_next_batch_instruction
      - do_not_say_complete

  case_6_asset_board_mixed_with_comic_pages:
    expected:
      - fail
      - separate_assets_from_comic_pages

  case_7_chinese_text_garbled:
    expected:
      - use_lettering_data
      - regenerate_or_postprocess_text_layer
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
    comic_pages_P01_to_P10_individual_files: required
    handoff: required
    qc_report: required
    output_manifest: required

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
    - no_output_manifest_json
    - final_output_is_script_pack_only
    - final_output_is_partial_pages_only
    - asset_board_mixed_with_comic_pages_as_single_image
    - preview_sheet_replaces_individual_pages
    - production_board_presented_as_final_comic
    - final_status_is_not_PASS_but_answer_says_complete

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

## 21. 验收测试方法

```yaml
acceptance_test_method:
  static_audit:
    pass_conditions:
      - has_delivery_layer_separation
      - has_final_file_gate
      - has_preview_sheet_rule
      - has_visual_style_regression_lock
      - has_no_placeholder_policy
      - has_batch_continuation_protocol
      - has_output_manifest
      - has_lettering_layer_rule

  dry_run_audit:
    cases:
      - 首次只上传小说链接和本文件
      - 首次上传小说链接和基础角色图
      - 下一章上传小说链接、角色包和 handoff
      - 链接不可读
      - 图像生成能力不可用
      - 输出只有一张总览图
      - 只能生成部分页面

  real_output_audit:
    must_check:
      - 是否存在 comic_pages/P01.png 到 P10.png
      - 是否存在 character_bootstrap 与 scene_bootstrap
      - 是否存在 handoff.json、qc_report.json、output_manifest.json
      - P01-P10 是否为独立漫画页
      - 是否没有资产板混入漫画页
      - 是否保留中国漫画连载风味
      - 如果 final_status 不是 PASS，是否没有说完成
```
