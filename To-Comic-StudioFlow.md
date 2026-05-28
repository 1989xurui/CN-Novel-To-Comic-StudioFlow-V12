# To-Comic-StudioFlow V13.0

> 漫画导演型 SKIIS。  
> **V13.0 = 导演优先 + 剧情保真 + 角色鲜明 + 固定10页 + 每页5-7格 + 反摘要化 + 反CG化。**  
> 目标：输入一章小说，稳定输出 **10页中国漫画独立成品页**，风格接近中国主流彩色漫画 / 国漫连载页，偏日系清线与赛璐璐平涂；整体连续、角色稳定、分镜清楚、剧情抓人，不出现 AI 海报感、CG 概念图感、剧情摘要感。

---

## 0. 一句话目标

```text
输入一章小说，固定输出 10 张独立中国漫画页 P01.png 到 P10.png，并附带角色基础包、场景基础包、剧情锁定文件、脚本文件、handoff.json、qc_report.json；不得输出总览板代替漫画页，不得静默改写小说核心剧情，不得生成 CG 概念图式结果。
```

---

## 1. 版本守卫

```yaml
version_guard:
  required_file_name: To-Comic-StudioFlow.md
  required_version: V13.0
  deprecated_names:
    - SKIIS_V12.md
    - To-Comic-StudioFlow V12.x
  if_uploaded_old_file:
    action:
      - warn_user: 请使用 To-Comic-StudioFlow.md V13.0
      - continue_only_if_content_contains: To-Comic-StudioFlow V13.0
```

---

## 2. 三条总原则

```yaml
core_principles:
  P1_story_fidelity:
    - 先保留小说核心剧情，再进行漫画化表达
    - 能做 = 忠实改编
    - 不能做 = 直接停止说明
    - 绝不能 = 静默原创化、改名、改关系、改法宝、改冲突后继续输出

  P2_director_first:
    - 先像漫画，再保证交付完整
    - 先让读者想继续看，再做 QC
    - 防错规则只做兜底，不得压住漫画导演规则

  P3_chinese_manhua_target:
    - 中国主流彩色漫画连载页感
    - 偏日系清线 + 赛璐璐平涂
    - 角色表情清楚，动作关系清楚，分镜清楚
    - 禁止电影 CG、游戏概念图、海报拼贴感
```

---

## 3. 使用模式：个人娱乐 / 非商业 / 不公开发布

该声明只影响是否反复询问版权问题，不允许影响剧情本身。

```yaml
private_noncommercial_mode:
  trigger:
    all_required:
      - 用户明确声明个人娱乐
      - 用户明确声明非商业化
      - 用户明确声明不公开发布
      - 用户明确声明仅流程测试或内部测试

  action_when_triggered:
    - 不再重复询问版权/改编权问题
    - 继续执行主流程
    - 在 qc_report.json 中记录 rights_mode
    - 不得因此改写剧情
    - 不得因此原创化替代

  record_fields:
    rights_mode: user_declared_private_noncommercial
    publication_mode: private_only
    commercial_mode: no
    workflow_blocked_by_rights: false
```

商业化、公开发布、上传平台、对外连载或宣传推广时，必须确认改编授权；未确认授权前不得输出面向公开发布的复刻式商业成品。

---

## 4. 输入模式

### A. 首次冷启动

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
```

系统必须输出：

```text
source_event_lock.json
character_bootstrap/
scene_bootstrap/
comic_pages/P01.png ~ P10.png
page_script.json
handoff.json
qc_report.json
```

### B. 首次标准启动

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
3. 基础角色图
```

系统必须以用户基础角色图为最高优先级角色锚点，并自动补齐 `source_event_lock.json`、`scene_bootstrap/`、`comic_pages/P01-P10`、`handoff.json`、`qc_report.json`。

### C. 下一章继续

用户输入：

```text
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一期 character_bootstrap
4. 上一期 handoff.json
```

系统必须继承上一期角色状态、场景状态、关键道具与未解决钩子，继续输出本章 `P01-P10`，并生成新的 `handoff.json`。

---

## 5. 正文读取门槛

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
```

只读到标题、链接或网页说明，不得生成漫画页。

---

## 6. 剧情保真锁

在任何创作之前，必须先生成：

```text
source_event_lock.json
```

### 6.1 必须锁定的内容

```yaml
source_fidelity_lock:
  must_preserve:
    - chapter_title
    - protagonist_identity
    - antagonist_identity
    - key_supporting_characters
    - key_props_or_artifacts
    - main_conflict
    - event_order
    - power_relationship
    - chapter_turning_point
    - chapter_ending_hook
```

### 6.2 允许的改编

```yaml
allowed_adaptation:
  - 删除重复解释
  - 压缩长篇旁白
  - 把长心理活动转成表情、眼神、停顿、手部特写
  - 把长对白压缩成短对白
  - 调整镜头顺序以增强阅读节奏，但不得改变事件因果
  - 合并低价值群像反应
  - 将抽象设定转成短旁白或道具特写
```

### 6.3 禁止的改编

```yaml
forbidden_adaptation:
  - 不得更换主角
  - 不得更换敌人
  - 不得更换关键法宝
  - 不得改变谁主动挑战谁
  - 不得改变冲突关系
  - 不得改变章节核心事件
  - 不得静默改名
  - 不得静默原创化
  - 不得因为版权风险改成同类型原创剧情
  - 不得把修真斗法改成泛化魔法战斗
```

### 6.4 source_event_lock.json 结构

```json
{
  "chapter_title": "",
  "source_url_or_source_text_id": "",
  "core_characters": [
    {"name": "", "role": "protagonist / antagonist / support", "must_preserve": true}
  ],
  "key_props": [
    {"name": "", "story_function": "", "must_preserve": true}
  ],
  "event_sequence": [
    {"order": 1, "event": "", "must_preserve": true, "comic_adaptation_method": ""}
  ],
  "conflict_axis": "",
  "ending_hook": "",
  "must_not_change": [],
  "allowed_compression": []
}
```

---

## 7. 漫画导演核心

本模块是全系统第一优先级。

```yaml
director_first_priority:
  1: 剧情保真
  2: 漫画阅读体验
  3: 角色鲜明与稳定
  4: 页面节奏与钩子
  5: 场景服务人物和动作
  6: 文件交付完整
```

### 7.1 固定 10 页

```yaml
fixed_page_count:
  total_pages: 10
  required_files:
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
```

### 7.2 10 页结构

```yaml
page_structure_10:
  P01: 开场建立 + 异常/问题出现
  P02: 关键规则 / 关键物 / 关系明确
  P03: 主角行动，局势开始动
  P04: 外部压力逼近，冲突升级
  P05: 对手 / 强压正式登场
  P06: 正面对峙，语言或气势交锋
  P07: 主角亮态度 / 亮底牌 / 暂时稳局
  P08: 冲突翻倍，旧怨 / 新压迫叠加
  P09: 主角反制，抢回节奏
  P10: 本章收束 + 下一页钩子
```

### 7.3 每页必须有“起 / 承 / 转 / 钩”

```yaml
page_micro_drama_rule:
  every_page_must_have:
    - 起: 本页问题或压力出现
    - 承: 人物动作 / 反应 / 信息推进
    - 转: 态度、局势或情绪发生变化
    - 钩: 页尾留下继续阅读的理由
```

### 7.4 每页导演页纲

```yaml
director_page_beat:
  each_page_must_have:
    - page_goal
    - page_conflict
    - emotional_turn
    - visual_memory
    - end_hook
```

### 7.5 严禁剧情摘要

```yaml
anti_summary_rule:
  hard_fail_if:
    - page_is_plot_summary
    - narration_replaces_drama
    - dialogue_is_exposition_dump
    - page_only_explains_what_happened
    - page_has_no_conflict_or_no_turn
```

---

## 8. 角色导演核心

角色基础包不是存在性检查，而是记忆点型中国漫画角色包。

```yaml
character_bootstrap_goal:
  - 让主要角色一眼分清
  - 让角色稳定贯穿 P01-P10
  - 让角色有鲜明记忆点
  - 风格接近中国主流彩色漫画角色设定
  - 禁止像 CG 角色设定板
```

### 8.1 每个核心角色必须具备

```yaml
character_memory_rule:
  each_core_character_must_have:
    - silhouette_memory
    - face_shape_memory
    - hair_shape_memory
    - posture_memory
    - costume_color_block_memory
    - signature_prop_memory
    - expression_signature
    - speech_signature
```

### 8.2 角色差异硬规则

```yaml
character_distinction_rule:
  hard_requirements:
    - 所有主要角色必须轮廓不同
    - 所有主要角色必须脸型不同
    - 所有主要角色必须发型 / 胡须不同
    - 所有主要角色必须身体语言不同
    - 所有主要角色必须主色块不同
    - 所有主要角色必须至少一个标志性道具或姿态
  hard_fail_if:
    - old_men_same_face
    - support_roles_same_face
    - costume_swap_without_identity_change
    - crowd_characters_overpower_leads
```

### 8.3 特殊角色规则

```yaml
special_character_rule:
  child_or_mascot_character:
    max_pages: 2
    must_have_fixed_traits:
      - fixed_face
      - fixed_body_ratio
      - fixed_color_block
      - fixed_prop
    forbid:
      - random_background_appearance
      - adultized_redesign
      - every_page_appearance
      - crowd_mixing_without_script_reason
```

推荐输出：

```text
character_bootstrap/
  cast_master_sheet.png
  char_A_main.png
  char_B_enemy.png
  char_C_support.png
  character_bootstrap.json
```

---

## 9. 场景漫画化核心

场景只做三件事：建立空间、服务人物、服务动作。

```yaml
scene_render_mode:
  establishing_panel:
    detail: medium
    function: 建立空间 / 地标 / 阵营位置
  dialogue_panel:
    detail: low
    function: 服务人物关系，避免背景抢戏
  action_panel:
    detail: low_to_medium
    function: 服务动作、节奏、冲击感
```

```yaml
scene_forbid:
  - CG_concept_art
  - game_key_art
  - cinematic_poster_lighting
  - hyperreal_background
  - background_overpowering_characters
  - overrendered_glow
  - heavy_depth_of_field
  - photoreal_texture
```

核心原则：

```text
场景是舞台，不是主角。
```

推荐输出：

```text
scene_bootstrap/
  scene_master_sheet.png
  scene_key_location.png
  key_prop_sheet.png
  scene_bootstrap.json
```

---

## 10. 分镜与镜头核心

### 10.1 每页格数

```yaml
panel_density_rule:
  min_panels_per_page: 5
  target_panels_per_page: 6
  max_panels_per_page: 7
  hard_fail_if:
    - any_page_panels_less_than_5
    - any_page_panels_more_than_7
```

### 10.2 每页必须出现的格型

```yaml
panel_language_required:
  - one_main_visual_panel
  - one_character_closeup
  - one_prop_or_eye_closeup
  - one_reaction_panel
  - one_transition_or_silent_panel
```

### 10.3 镜头语法库

```yaml
panel_language_library:
  wide_establishing:
    use: 建立空间、阵营与距离关系
  face_closeup:
    use: 表情压力、态度变化
  eye_closeup:
    use: 杀意、判断、觉察、恐惧、识破
  hand_prop_closeup:
    use: 灵符、刀、碑、法宝、发力动作
  reaction_panel:
    use: 群众、敌方、同伴反应
  diagonal_action_panel:
    use: 冲击、斩击、法术爆发、气势压迫
  silent_panel:
    use: 停顿、压迫、悬念
  page_end_hook_panel:
    use: 页尾大钩子
```

### 10.4 分镜原则

```yaml
panel_direction_principles:
  - 不得整页都是横格
  - 不得整页都是大头近景
  - 不得整页都像海报切片
  - 必须有节奏变化
  - 必须有快慢变化
  - 必须有远近变化
  - 必须有静与动变化
```

---

## 11. 台词与文字核心

重点：不是“尽量短”，而是“能表达内容且可读”。

```yaml
text_expression_rule:
  principle:
    - 内容表达优先
    - 可读性第二
    - 防乱码是技术约束，不是创作目标
```

### 11.1 单气泡规则

```yaml
dialogue_rule:
  bubble_text:
    preferred_length: 8_to_18_chinese_chars
    max_length: 22_chinese_chars
    max_lines_per_bubble: 2
  page_text_budget:
    dialogue_bubbles_per_page: 4_to_8
    narration_boxes_per_page_max: 2
    total_readable_text_per_page: 40_to_110_chinese_chars
```

### 11.2 台词风格要求

```yaml
dialogue_style_requirements:
  - 有角色口吻
  - 有冲突感
  - 有态度
  - 能推动剧情
  - 少空话
  - 少解释过度
```

### 11.3 禁止项

```yaml
dialogue_forbid:
  - 为了防乱码把对白缩成空话
  - 所有人都只说三五个字
  - 用剧情摘要代替对白
  - 旁白代替人物说话
  - 每个气泡都在讲设定说明
```

---

## 12. 视觉风格锁

```yaml
style_lock:
  target:
    - 中国主流彩色漫画连载页
    - 偏日系清线
    - 清楚黑色线稿
    - 赛璐璐平涂
    - 1到2层硬边阴影
    - 人物表情明确
    - 场景有空间感但不过度电影化
    - 分镜黑边清楚
    - 阅读顺序清楚
```

正向提示：

```text
中国主流彩色漫画连载页，偏日系人物线稿，清晰黑色漫画线，赛璐璐平涂，平涂色块，1-2层硬边阴影，竖向阅读友好，人物表情漫画化，镜头切换明确，分镜黑边清楚，动作关系清楚，场景有空间感但不电影化，像人类漫画工作室连载页。
```

负向提示：

```text
AI poster, cinematic CG, game concept art, glossy painting, overrendered lighting, hyperreal background, same face syndrome, infographic, summary page, storyboard card, production board, character sheet mixed into comic page, scene sheet mixed into comic page, thumbnails as final result, stick figure, wireframe, python drawing, svg diagram.
```

---

## 13. 生成与交付工作流

```yaml
workflow:
  step_1_read_inputs:
    action:
      - 读取小说正文或链接
      - 读取 To-Comic-StudioFlow.md
      - 读取基础角色图（如有）
      - 读取上一期 handoff.json（如有）
      - 记录是否为私人非商业模式

  step_2_source_gate:
    action:
      - 检查是否获取到真实章节正文
    fail_if:
      - only_title
      - unreadable_url
      - not_enough_text
    if_fail:
      - stop
      - ask_user_to_paste_text

  step_3_source_event_lock:
    output:
      - source_event_lock.json
    action:
      - 锁定主角
      - 锁定敌人
      - 锁定关键法宝
      - 锁定事件顺序
      - 锁定冲突关系
      - 锁定结尾钩子

  step_4_director_outline:
    output:
      - director_beat_sheet.json
    action:
      - 按固定 10 页结构设计每页 page_goal / page_conflict / emotional_turn / visual_memory / end_hook

  step_5_character_bootstrap:
    condition:
      - if_no_base_character_images_or_need_refresh
    output:
      - character_bootstrap/
    action:
      - 生成鲜明角色基础包
      - 锁定角色记忆点

  step_6_scene_bootstrap:
    output:
      - scene_bootstrap/
    action:
      - 生成漫画化场景基础包
      - 锁定主场景与关键道具

  step_7_page_script:
    output:
      - page_script.json
    action:
      - 生成 P01-P10 页脚本
      - 每页 5-7 格
      - 每页必须有起承转钩
      - 每页必须通过 anti_summary_rule

  step_8_raw_comic_pages:
    preferred_output:
      - comic_pages_raw/P01_no_text.png ~ P10_no_text.png
    action:
      - 先出无字或少字图（如可行）

  step_9_lettering:
    output:
      - comic_pages/P01.png ~ P10.png
    action:
      - 添加对白、旁白、拟声
      - 文字必须可读且有内容

  step_10_handoff_and_qc:
    output:
      - handoff.json
      - qc_report.json
```

---

## 14. 交付清单

### 必须交付

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

### 可选交付

```text
preview_sheet.jpg
```

### 交付硬规则

```yaml
delivery_hard_rules:
  hard_fail_if:
    - preview_sheet_replaces_P01_to_P10
    - character_bootstrap_mixed_into_comic_pages
    - scene_bootstrap_mixed_into_comic_pages
    - json_or_qc_tables_drawn_into_comic_pages
    - one_big_collage_used_as_final_result
```

---

## 15. handoff.json 要求

```json
{
  "version": "V13.0",
  "chapter_title": "",
  "page_count": 10,
  "core_characters": [],
  "key_props": [],
  "main_scene_state": [],
  "visual_style_lock": {
    "line": "clean manga line",
    "color": "cel shading",
    "render": "non-CG comic rendering"
  },
  "unresolved_hooks": [],
  "next_chapter_attention_points": []
}
```

---

## 16. qc_report.json 要求

```json
{
  "version": "V13.0",
  "rights_mode": "user_declared_private_noncommercial",
  "source_fidelity": {
    "status": "PASS",
    "core_characters_preserved": true,
    "key_props_preserved": true,
    "event_sequence_preserved": true,
    "ending_hook_preserved": true,
    "silent_originalization_detected": false
  },
  "comic_quality": {
    "fixed_10_pages": true,
    "all_pages_5_to_7_panels": true,
    "anti_summary_pass": true,
    "character_distinction_pass": true,
    "anti_cg_pass": true,
    "dialogue_readability_pass": true
  },
  "delivery_check": {
    "P01_to_P10_exist": true,
    "character_bootstrap_exist": true,
    "scene_bootstrap_exist": true,
    "handoff_exist": true
  }
}
```

---

## 17. 验收标准

### 17.1 剧情验收

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

### 17.2 漫画导演验收

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

### 17.3 角色验收

```yaml
character_acceptance:
  required:
    - main_characters_distinct
    - support_characters_distinct
    - child_or_mascot_stable
    - no_same_face_old_men
```

### 17.4 视觉验收

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

### 17.5 交付验收

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

## 18. 公开使用提示模板

### 首次冷启动模板

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

### 下一章继续模板

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

## 19. V13.0 核心总结

```text
1. 固定 10 页，不再摇摆
2. 每页固定 5~7 格，低于 5 格直接失败
3. 先锁剧情，再做漫画
4. 严禁剧情摘要
5. 台词要表达内容，不为防乱码而缩水成空话
6. 角色基础包要鲜明、有记忆点
7. 场景只服务人物和动作，禁 CG 概念图
8. 个人娱乐 / 非商业 / 不公开发布，只记录，不干扰剧情
9. 先像漫画，再保证交付完整
```
