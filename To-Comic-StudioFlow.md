# To-Comic-StudioFlow V12.5

> 单文件版中国小说转漫画生产流程。  
> V12.5 = V12.4.2 的交付硬门槛 + 旧 V12 漫画风味 + Creative Director Core。  
> 核心目标：输入一章小说，稳定输出接近人类中国漫画连载感的独立漫画页；角色要分明，场景要漫画化，分镜要有导演节奏，脚本文案要有冲突和钩子，同时继续防止火柴人、流程看板、资产总览图、脚本包冒充成品。

---

## 0. 一句话目标

```text
输入一章小说，按剧情密度输出 6 / 8 / 10 张独立中国漫画页，并附带角色基础包、场景基础包、脚本文件、handoff.json、qc_report.json 与 output_manifest.json；不得把所有内容合成一张大图，不得用预览图、看板图、脚本包或 render_manifest 替代漫画页。
```

---

## 1. 版本守卫

```yaml
version_guard:
  required_file_name: To-Comic-StudioFlow.md
  required_version: V12.5
  deprecated_names:
    - SKIIS_V12.md
    - SKIIS_V12(1).md
    - SKIIS_V12(2).md
    - To-Comic-StudioFlow V12.1
    - To-Comic-StudioFlow V12.2
    - To-Comic-StudioFlow V12.3
    - To-Comic-StudioFlow V12.4
    - To-Comic-StudioFlow V12.4.1
    - To-Comic-StudioFlow V12.4.2
  if_uploaded_old_file:
    action:
      - warn_user: 请使用 To-Comic-StudioFlow.md V12.5
      - continue_only_if_content_contains: To-Comic-StudioFlow V12.5
```

旧版文件不得继续执行漫画生成。若文件不是 V12.5，必须提醒用户换文件。

---

## 2. 使用模式：私人非商业流程测试

本流程支持用户声明为“个人娱乐、非商业化、不公开发布、仅流程测试”的使用场景。该模式用于避免系统在每次任务中反复打断询问版权/改编权问题，但不代表授予任何公开发布、商业发行或平台连载权利。

```yaml
private_noncommercial_mode:
  trigger:
    all_required:
      - 用户明确声明个人娱乐
      - 用户明确声明非商业化
      - 用户明确声明不公开发布
      - 用户明确声明仅流程测试或内部测试

  accepted_user_phrases:
    - 这是个人娱乐用途
    - 非商业化
    - 不公开发布
    - 仅用于内部流程测试
    - 只做个人研究和流程测试
    - 不上传平台、不公开传播、不商业发行

  action_when_triggered:
    - 不再重复询问版权/改编权问题
    - 继续执行 To-Comic-StudioFlow 主流程
    - 不因版权确认问题中断 character_bootstrap、scene_bootstrap、comic_pages 生成
    - 在 qc_report.json 中记录 rights_mode
    - 在 output_manifest.json 中记录 publication_mode 与 commercial_mode

  record_fields:
    rights_mode: user_declared_private_noncommercial
    publication_mode: private_only
    commercial_mode: no
    public_release: no
    workflow_blocked_by_rights: false
    user_declaration_required: already_declared
```

公开发布、商业化发行、上传漫画平台、对外连载、商业售卖或宣传推广时，必须确认改编授权；未确认授权前不得输出面向公开发布的复刻式商业成品。

---

## 3. 启动模式

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
comic_pages/P01.png ... Pxx.png
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

系统必须继承上一期 `handoff.json` 的角色、道具、场景、未解决钩子，继续生成下一章漫画页与新的 `handoff.json`。

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

## 5. V12.5 创作核心：漫画优先，而不是流程优先

```yaml
creative_director_core:
  core_goal:
    - 保留旧 V12 的中国漫画连载页风味
    - 保留 V12.4.2 的交付层级隔离和硬门槛
    - 强化角色差异、导演分镜、场景漫画化、脚本文案张力
    - 降低 AI 味、CG 感、同脸、资产板倾向

  priority_order:
    1: 独立漫画页 P01-Pxx 的阅读体验
    2: 角色稳定与角色差异
    3: 页面导演节奏与钩子
    4: 场景漫画化与空间清楚
    5: 交付完整性
    6: 单张图精细度

  forbid:
    - 把工作流说明画成最终图
    - 把角色设定表画进漫画页
    - 把场景设定表画进漫画页
    - 把 JSON / QC / manifest 画进漫画页
    - 为了展示流程牺牲漫画阅读感
```

---

## 6. 页数策略：少页高质量，最多 10 页

不再默认追求 13 页。页数越多，角色漂移和场景断裂风险越高。

```yaml
page_count_policy:
  default_pages: 8
  dense_action_chapter: 10
  short_dialogue_chapter: 6
  hard_max_pages: 10

  use_6_pages_when:
    - 单场景
    - 单冲突
    - 低角色密度
    - 低动作密度

  use_8_pages_when:
    - 正常章节
    - 有完整起承转合
    - 有2到4个关键角色
    - 有1个主要场景

  use_10_pages_when:
    - 高密度战斗
    - 多方势力对峙
    - 关键法宝登场
    - 章节高潮或标题落点强

  rule:
    - 页数服务质量和稳定性，不服务数量
    - 如果用户明确要求 P01-P10，则输出 10 页
    - 如果用户没有指定，按章节密度自动选择 6 / 8 / 10 页
```

输出文件必须与实际页数一致，例如：

```text
6页: comic_pages/P01.png 到 P06.png
8页: comic_pages/P01.png 到 P08.png
10页: comic_pages/P01.png 到 P10.png
```

---

## 7. 漫画导演页纲：每页必须有问题、答案、情绪和钩子

```yaml
director_page_beat_required:
  each_page_must_have:
    - reader_question: 本页开始读者想知道什么
    - page_answer: 本页回答什么
    - emotional_turn: 本页情绪如何变化
    - visual_memory: 本页最大视觉记忆点
    - page_end_hook: 本页结尾让读者继续看的钩子

  forbid:
    - 只按小说段落平均切页
    - 只做剧情总结
    - 每页只有说明，没有情绪变化
    - 页面之间突然跳转，没有因果桥
    - 每页都是同样的近景对话
```

每页必须是一个小戏剧单元：

```yaml
page_drama_unit:
  start: 引出问题或压力
  middle: 给出动作/反应/冲突
  end: 留下视觉钩子或态度钩子
```

---

## 8. 小说改漫画提炼逻辑

```yaml
adaptation_logic:
  extract:
    - 主角本章目标
    - 对手本章压力
    - 关键道具/法宝
    - 转折点
    - 爽点/压迫点
    - 章节标题落点

  compress:
    - 删除重复解释
    - 删除低视觉价值旁白
    - 删除同义设定堆叠
    - 把长心理活动改成眼神/手部/停顿

  convert:
    internal_monologue_to:
      - eye_closeup
      - hand_prop_closeup
      - silent_gap
    worldbuilding_to:
      - prop_closeup
      - short_caption
      - reaction_panel
    crowd_pressure_to:
      - formation
      - color_block
      - reaction_stack
```

---

## 9. 角色差异系统：每个命名角色必须一眼分清

```yaml
character_distinction_rule:
  each_named_character_must_have:
    - silhouette
    - face_shape
    - hair_or_beard
    - posture
    - color_block
    - prop_or_mark
    - speech_style

  old_men_rule:
    forbid:
      - 所有老者同脸
      - 只靠衣服颜色区分
    must_differentiate_by:
      - 胡须形状
      - 脸型宽窄
      - 眉眼角度
      - 身体姿态
      - 手势习惯

  antagonist_rule:
    each_antagonist_must_have:
      - 独立体型
      - 独立眼神
      - 独立服装轮廓
      - 独立法宝或武器
      - 独立说话气质

  crowd_rule:
    - 群像不精画脸
    - 群像靠阵型、服色、旗帜、武器、站位区分
    - 群像不能抢主角和关键敌方
```

角色基础图优先级：

```yaml
character_anchor_priority:
  1: 用户上传的基础角色图
  2: 上一期 handoff.json 中的角色状态
  3: 本期自动生成的 character_bootstrap
  4: 小说文字推断
```

---

## 10. 特殊角色稳定规则：儿童/萌角色/器灵不乱跑

```yaml
special_character_budget:
  child_or_mascot_character:
    max_pages_per_issue: 2
    allowed_panel_type:
      - reaction_panel
      - identity_support_panel
      - emotional_relief_panel
    fixed_traits:
      - small_body
      - round_face
      - fixed_hair_shape
      - fixed_color_block
      - fixed_prop
    forbid:
      - random_background_appearance
      - adult_body
      - dark_clothing_swap
      - long_hair_girl_version
      - unplanned_crowd_mix
      - 每页都出现

  spirit_or_artifact_character:
    must_define:
      - 固定轮廓
      - 固定发光形态
      - 固定表情范围
      - 固定与主角的空间关系
```

星云宝宝、器灵、幼态角色必须作为“稀缺表现资源”，不能随意塞进每一页。

---

## 11. 场景漫画化规则：不要 CG，场景服务人物和动作

```yaml
scene_comic_style:
  establishing_panel:
    detail: medium
    purpose: 建立空间和地标

  dialogue_panel:
    detail: low
    purpose: 人物关系优先

  action_panel:
    detail: low_to_medium
    purpose: 速度线、气浪、碎石、色块背景强化动作

  key_location_card:
    must_show:
      - 地标轮廓
      - 前景/中景/远景关系
      - 角色站位参考
      - 动作页可简化版本

  forbid:
    - cinematic_volumetric_light
    - game_concept_art_background
    - excessive_dark_golden_particles
    - hyperreal_texture
    - background_stealing_character_focus
    - 全页都像电影海报
```

---

## 12. 镜头语法库：每页要有漫画切块

```yaml
panel_language_library:
  wide_establishing:
    use: 建立空间、地点、阵营关系

  face_closeup:
    use: 表情压力、心理转折

  eye_closeup:
    use: 杀意、判断、恐惧、识破

  hand_prop_closeup:
    use: 灵符、刀、碑、法宝、关键动作

  reaction_stack:
    use: 群众、敌方、同伴反应

  diagonal_action:
    use: 冲击、斩击、飞行、法术爆发

  silent_gap:
    use: 停顿、压迫、悬念

  page_turn_splash:
    use: 页尾钩子、高潮落点
```

每页必须至少包含：

```yaml
page_panel_minimum:
  - one_main_visual_panel
  - one_character_closeup
  - one_prop_or_eye_closeup
  - one_reaction_or_silent_panel
```

每页分镜规则：

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

## 13. 脚本文案规则：短、狠、有态度

```yaml
dialogue_rule:
  bubble_text:
    max_length: 16_chinese_chars_per_bubble
    preferred_length: 6_to_12_chinese_chars

  line_style:
    - 短
    - 狠
    - 有态度
    - 有角色口吻
    - 少解释
    - 多冲突

  forbid:
    - 大段小说原文照搬
    - 旁白解释过多
    - 所有人说话一个语气
    - 每个气泡都在解释设定
    - 用小说摘要代替漫画台词
```

台词要像人物说出来，不像 AI 总结：

```yaml
speech_identity:
  protagonist:
    style: 克制、狠、反问、压迫回击
  antagonist:
    style: 高位、冷、命令、威胁
  crowd:
    style: 短促、震惊、议论，不长篇解释
  mascot_child:
    style: 短句、反应、辅助情绪，不承担设定说明
```

---

## 14. 旧 V12 漫画风味继承锁

```yaml
old_v12_style_memory:
  preserve:
    - 独立漫画页
    - 彩色中国漫画连载感
    - 偏日系人物线稿
    - 清楚黑线
    - 赛璐璐平涂
    - 人物脸部清楚
    - 场景有地标但不电影化
    - 分镜清楚
    - 动作关系清楚

  avoid:
    - 暗黑概念图
    - 电影 CG
    - 游戏宣传图
    - 资产总览板
    - JSON 表格入图
    - P01-P10 缩略总览代替独立页
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

## 15. 交付层级隔离

```yaml
delivery_layer_separation:
  principle:
    - 生产资产、脚本文件、QC文件、漫画页必须分开交付
    - 不允许把所有文件内容拼成一张图片
    - 不允许把 character_bootstrap、scene_bootstrap、JSON表格和漫画页混在同一张图里

  character_bootstrap:
    output_type: 独立资产文件夹
    forbid:
      - 混入 comic_pages
      - 画进 P01-Pxx
      - 合成到最终漫画页
      - 合成到 preview_sheet 主图

  scene_bootstrap:
    output_type: 独立资产文件夹
    forbid:
      - 混入 comic_pages
      - 画进 P01-Pxx
      - 合成到最终漫画页
      - 合成到 preview_sheet 主图

  metadata:
    output_type: 独立文本文件
    forbid:
      - 画进 comic_pages
      - 画进 preview_sheet
      - 变成图片表格总览

  comic_pages:
    output_type: 独立漫画页文件夹
    forbid:
      - 资产总览图
      - 流程看板
      - JSON表格
      - 角色设定栏
      - 场景设定栏
      - QC报告栏
      - 一张图塞完整结果包
      - 多页缩略图合集冒充独立页
      - 双页合图冒充单页

  preview_sheet:
    output_type: 可选缩略图
    allowed:
      - 仅将已生成的独立漫画页缩略图排成总览
    generate_after:
      - all_individual_comic_pages_exist
    forbid:
      - 替代 comic_pages
      - 混入 JSON 表格
      - 混入 character_bootstrap
      - 混入 scene_bootstrap
      - 作为最终唯一图片
```

---

## 16. 中文后期排版规则

```yaml
lettering_layer_rule:
  preferred_flow:
    - 先生成无字或少字漫画页 comic_pages_raw/P01_no_text.png 到 Pxx_no_text.png
    - 再用 lettering_data.json 添加中文对白、旁白、拟声词
    - 最终输出带字版 comic_pages/P01.png 到 Pxx.png

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

## 17. 最终文件验证门槛与 output_manifest

```yaml
final_file_gate:
  before_final_answer:
    must_verify_files_exist:
      - comic_pages/P01.png 到 Pxx.png
      - handoff.json
      - qc_report.json
      - output_manifest.json
    cold_start_extra_required:
      - character_bootstrap/cast_master_sheet.png
      - scene_bootstrap/scene_master_sheet.png

  fail_if:
    - only_one_composite_image_exists
    - pages_are_only_thumbnails_inside_preview_sheet
    - no_individual_page_files
    - final_answer_contains_only_preview_sheet
    - final_answer_contains_only_script_pack
    - final_answer_contains_only_render_manifest
```

```yaml
output_manifest:
  purpose: 防止口头说完成但文件不存在
  must_include:
    page_count:
      expected: 6_or_8_or_10
      actual: number
    comic_pages:
      P01.png: exists/type/validation_status
      P02.png: exists/type/validation_status
      Pxx.png: exists/type/validation_status
    bootstrap:
      character_bootstrap: exists/validation_status
      scene_bootstrap: exists/validation_status
    metadata:
      handoff_json: exists
      qc_report_json: exists
      lettering_data_json: exists_if_used
    usage_mode:
      rights_mode: user_declared_private_noncommercial_if_declared
      publication_mode: private_only_if_declared
      commercial_mode: no_if_declared
      workflow_blocked_by_rights: false_if_private_noncommercial_declared
    final_status: PASS_or_IN_PROGRESS_or_BLOCKED_or_FAIL
```

`output_manifest.json` 中如果 `final_status` 不是 `PASS`，最终答复不得写“完成”。

---

## 18. 分批继续协议

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
      - P01-Pxx已生成
    final_result_invalid_until:
      - all_expected_comic_pages_exist
      - handoff.json_exists
      - qc_report.json_exists
      - output_manifest_final_status_PASS

  next_batch_instruction_must_include:
    - 已完成文件列表
    - 缺失文件列表
    - 下一批必须生成的文件
    - 继续口令

  continue_prompt_template:
    text: 继续按 production_state.json 从缺失文件开始生成，不要重做已完成文件；直到全部预期 comic_pages、handoff.json、qc_report.json、output_manifest.json 全部存在，才允许说完成。
```

如果被迫停止，必须明确告诉用户下一步操作。

---

## 19. 图像能力门槛

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

## 20. 自动工作流

```yaml
workflow:
  step_1_read_inputs:
    action:
      - 读取小说当前章节原文或链接内容
      - 读取基础角色图，如果有
      - 读取上一期 handoff.json，如果有
      - 读取本 To-Comic-StudioFlow.md

  step_2_mode_rights_and_source_gate:
    action:
      - 检查是否为 V12.5
      - 检查是否触发 private_noncommercial_mode
      - 检查是否读到章节正文
      - 如果未读到正文，停止并要求用户粘贴正文

  step_3_page_count_and_director_beat:
    output:
      - page_count_decision
      - director_beat_sheet.json
    action:
      - 按 page_count_policy 决定 6 / 8 / 10 页
      - 为每页设计 reader_question、page_answer、emotional_turn、visual_memory、page_end_hook

  step_4_start_mode_detection:
    action:
      - 判断是否有基础角色图
      - 判断是否有上一期 handoff.json
      - 如果二者都没有，进入 cold_start_character_bootstrap 与 scene_bootstrap

  step_5_bootstrap_assets:
    output:
      - character_bootstrap/
      - scene_bootstrap/
    note: 生成后不得停顿，不得把资产表拼入漫画页，必须继续后续步骤

  step_6_chapter_analysis:
    output:
      - chapter_card.json

  step_7_lock_assets:
    output:
      - character_lock.json
      - scene_lock.json

  step_8_page_script:
    output:
      - page_script.json

  step_9_pre_page_gate:
    action:
      - 检查角色图锚点
      - 检查场景图锚点
      - 检查图像生成能力
      - 若缺失，补齐后继续；若无法补齐，停止，不得生成占位页

  step_10_generate_comic_pages_raw:
    preferred_output:
      - comic_pages_raw/P01_no_text.png 到 Pxx_no_text.png

  step_11_lettering:
    output:
      - lettering_data.json
      - comic_pages/P01.png 到 Pxx.png

  step_12_optional_preview_sheet:
    condition: all_individual_comic_pages_exist
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

## 21. 用户启动提示词

### 首次无角色图

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md

这是个人娱乐用途，非商业化，不公开发布，仅用于内部流程测试。
这是首次冷启动，没有基础角色图，也没有 handoff.json。
请按 V12.5 工作：先生成 character_bootstrap 角色基础包和 scene_bootstrap 场景基础包，但不要把这些资产混进漫画页。
然后继续生成完整结果包：
- character_bootstrap/
- scene_bootstrap/
- chapter_card.json
- director_beat_sheet.json
- character_lock.json
- scene_lock.json
- page_script.json
- comic_pages_raw/P01_no_text.png 到 Pxx_no_text.png（如可行）
- lettering_data.json
- comic_pages/P01.png 到 Pxx.png 独立漫画页
- handoff.json
- qc_report.json
- output_manifest.json
- preview_sheet.jpg（可选，不能替代独立漫画页）

如果不能一次完成，必须输出 production_state.json、output_manifest.json、next_batch_instruction.txt，并标记 IN_PROGRESS，不得说完成。
如果无法生成真实图片，不要用简笔图、线框图、SVG 或 Python 示意图替代，也不要把脚本包说成完成。
```

---

## 22. 验收标准

```yaml
acceptance:
  creative_director:
    page_count_selected_by_density: required
    director_page_beat_per_page: required
    character_distinction: required
    special_character_budget: required
    scene_comic_style: required
    panel_language_variety: required
    dialogue_short_and_characterful: required

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
    comic_pages_individual_files: required
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
```
