# To-Comic-StudioFlow V13.1

> 原著改编导演版 SKIIS。  
> **V13.1 = 原著改编硬锁 + 原文专名锁 + source_dialogue_bank + 四级导演脚本链 + 中国漫画页感强化 + 禁止原创替身回退。**  
> 核心目标：输入一章小说，稳定输出 **10页中国漫画独立成品页**。必须使用原小说内容、原主角名、原敌手名、原关键法宝名与原冲突关系；风格接近中国主流彩色漫画 / 国漫连载页，偏日系清线、赛璐璐平涂、连载分镜感，不出现 AI 海报感、CG 概念图感、剧情摘要感。

---

## 0. 一句话目标

```text
输入一章小说，固定输出 10 张独立中国漫画页 P01.png 到 P10.png，并附带 source_event_lock.json、source_dialogue_bank.json、chapter_event_map.json、角色基础包、场景基础包、导演页纲、分镜页纲、脚本文件、handoff.json、qc_report.json；必须使用原小说内容与原主角名，禁止静默原创化、禁止同类型原创替代、禁止总览图代替漫画页、禁止 CG 概念图式结果。
```

---

## 1. 版本守卫

```yaml
version_guard:
  required_file_name: To-Comic-StudioFlow.md
  required_version: V13.1
  deprecated_names:
    - SKIIS_V12.md
    - To-Comic-StudioFlow V12.x
    - To-Comic-StudioFlow V13.0
  if_uploaded_old_file:
    action:
      - warn_user: 请使用 To-Comic-StudioFlow.md V13.1
      - continue_only_if_content_contains: To-Comic-StudioFlow V13.1
```

---

## 2. 最高优先级

```text
1. 原著保真锁
2. 原文专名锁
3. 漫画导演脚本
4. 角色记忆点系统
5. 中国漫画风格锁
6. 分镜密度锁
7. 台词口吻锁
8. 文件交付与 QC
```

---

## 3. 四条总原则

```yaml
core_principles:
  P1_canonical_adaptation_only:
    - 必须是原著改编模式
    - 必须使用原小说内容
    - 必须使用原主角名
    - 必须使用原敌手名
    - 必须使用原关键法宝名
    - 必须保持原冲突关系与原事件顺序

  P2_stop_or_adapt_faithfully:
    - 能做 = 忠实改编
    - 不能做 = 直接停止说明
    - 绝不能 = 静默原创化
    - 绝不能 = 同类型原创替代
    - 绝不能 = 改名后继续输出

  P3_director_first:
    - 先让它像真正的中国连载漫画，再谈交付完整
    - 先保证阅读快感，再做 QC
    - 不允许摘要器思维压住导演思维

  P4_serial_manhua_target:
    - 中国主流彩色漫画连载页感
    - 偏日系清线 + 赛璐璐平涂
    - 角色鲜明、镜头懂戏、页尾有钩
    - 禁止电影 CG、概念图、海报拼贴感
```

---

## 4. 原著改编硬锁：Canonical Adaptation Only Lock

```yaml
canonical_adaptation_only:
  required:
    - use_original_novel_content
    - use_original_protagonist_name
    - use_original_antagonist_name
    - use_original_supporting_names
    - use_original_key_prop_names
    - keep_original_conflict_axis
    - keep_original_event_sequence
    - keep_original_chapter_hook

  hard_fail_if:
    - protagonist_name_changed
    - antagonist_name_changed
    - support_character_name_changed
    - key_prop_name_changed
    - conflict_axis_rewritten
    - story_silently_originalized
    - original_chapter_replaced_by_same_genre_plot
    - named_terms_replaced_by_generic_terms

  if_blocked_by_policy_or_capability:
    action:
      - stop_and_explain
      - do_not_generate_original_substitute
      - do_not_generate_same_genre_fallback
      - do_not_generate_fake_completion
```

核心原则：

```text
不能用原著改编，就停止；绝不能改成原创替身继续跑。
```

---

## 5. 个人娱乐 / 非商业 / 不公开发布模式

该模式只用于减少重复打断，不能改写剧情规则。

```yaml
private_noncommercial_mode:
  trigger:
    all_required:
      - 用户明确声明个人娱乐
      - 用户明确声明非商业化
      - 用户明确声明不公开发布
      - 用户明确声明仅内部流程测试

  action_when_triggered:
    - 不再反复询问版权问题
    - 继续执行主流程
    - 在 qc_report.json 记录 rights_mode
    - 不得因此改写剧情
    - 不得因此改名
    - 不得因此原创化替代

  record_fields:
    rights_mode: user_declared_private_noncommercial
    publication_mode: private_only
    commercial_mode: no
    workflow_blocked_by_rights: false

rights_handling_rule:
  if_private_noncommercial_mode:
    - continue_main_workflow
    - no_repeat_rights_warning
    - still_must_obey_canonical_adaptation_only
  if_cannot_continue_for_any_reason:
    - stop
    - explain
    - no_original_substitute
```

---

## 6. 输入模式

### A. 首次冷启动

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
```

系统必须输出：

```text
source_event_lock.json
source_dialogue_bank.json
chapter_event_map.json
character_bootstrap/
scene_bootstrap/
director_beat_sheet.json
panel_beat_sheet.json
page_script.json
comic_pages/P01.png ~ P10.png
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

系统必须以用户基础角色图为最高优先级角色锚点，并自动补齐所有剧情锁、场景包、脚本链、P01-P10、handoff、qc_report。

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

## 7. 正文读取门槛

```yaml
source_text_gate:
  before_adaptation:
    must_have:
      - chapter_title
      - readable_chapter_main_text
      - enough_plot_events
      - enough_character_actions
  fail_if:
    - only_title
    - only_brief_summary
    - unreadable_url
    - source_page_unreachable
    - chapter_text_too_short
  if_fail:
    status: BLOCKED
    action:
      - stop
      - ask_user_to_paste_chapter_text
      - do_not_generate_comic_pages
      - do_not_generate_fake_bootstrap
```

硬规则：只读到标题、简介、网页壳、评论区、目录页，不得进入漫画生成。

---

## 8. 剧情保真锁

在任何改编之前，必须先生成：

```text
source_event_lock.json
```

### 8.1 必须锁定的内容

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
    - named_terms
    - sect_names
    - technique_names
    - artifact_names
```

### 8.2 允许的改编

```yaml
allowed_adaptation:
  - 删除重复解释
  - 压缩冗长铺陈
  - 把长旁白转成表情、动作、眼神、停顿
  - 把长段心理活动拆成若干小格表演
  - 把长对白压缩成可读短对白
  - 调整镜头顺序增强阅读节奏，但不得改变因果
  - 合并低价值围观反应
  - 将抽象设定转成道具特写或短旁白
```

### 8.3 禁止的改编

```yaml
forbidden_adaptation:
  - 不得更换主角
  - 不得更换敌人
  - 不得更换宗门
  - 不得更换关键法宝
  - 不得更换功法 / 神通 / 宝物专名
  - 不得改变谁主动挑战谁
  - 不得改变谁压制谁
  - 不得改变章节核心事件
  - 不得静默改名
  - 不得静默原创化
  - 不得把原著桥段替换成泛仙侠桥段
  - 不得以安全为名做同类型原创替代
```

### 8.4 source_event_lock.json 结构

```json
{
  "chapter_title": "",
  "source_url_or_source_text_id": "",
  "core_characters": [
    {"name": "", "role": "protagonist / antagonist / support", "must_preserve": true}
  ],
  "key_named_terms": [
    {"name": "", "type": "character / sect / technique / artifact / place", "must_preserve": true}
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

## 9. 原文对白库：source_dialogue_bank.json

在 `source_event_lock.json` 之后，必须生成：

```text
source_dialogue_bank.json
```

作用：先从原文章节中提取可改编对白，保留原著语气，防止对白被 AI 泛化成空话，让漫画页保留原著味。

```yaml
source_dialogue_bank_rule:
  must_extract:
    - high_conflict_lines
    - challenge_lines
    - threat_lines
    - rebuttal_lines
    - decisive_lines
    - hook_lines
    - key_narration_lines
  transform_method:
    - long_lines_can_be_compressed
    - tone_must_be_preserved
    - named_terms_must_be_preserved
```

结构：

```json
{
  "chapter_title": "",
  "dialogue_bank": [
    {
      "speaker": "",
      "source_text": "",
      "compressed_comic_text": "",
      "function": "challenge / rebuttal / threat / hook / narration"
    }
  ],
  "hook_candidates": [],
  "narration_candidates": []
}
```

---

## 10. 漫画导演核心

V13.1 的脚本不是“章节摘要器”，而是“漫画导演器”。

```yaml
director_first_priority:
  1: 剧情保真
  2: 漫画阅读体验
  3: 角色鲜明与稳定
  4: 页面节奏与页尾钩
  5: 场景服务人物和动作
  6: 文件交付完整
```

---

## 11. 固定 10 页结构

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

page_structure_10:
  P01: 开场建立 + 异常出现 + 主角切入
  P02: 局势说明 + 关键关系落位
  P03: 主角表态 / 行动，局势开始发力
  P04: 外部压力逼近，冲突升级
  P05: 对手强势压场 / 第一轮压制
  P06: 规则、力量、法宝进一步展开
  P07: 主角亮底牌 / 抢回节奏
  P08: 冲突翻倍，情绪升压
  P09: 主角反制 / 局势逆转端倪
  P10: 本章收束 + 下一章强钩
```

---

## 12. 每页必须有“起 / 承 / 转 / 钩”

```yaml
page_micro_drama_rule:
  every_page_must_have:
    - 起: 本页问题、压力或悬念出现
    - 承: 人物动作 / 反应 / 信息推进
    - 转: 态度或局势发生变化
    - 钩: 页尾留下必须往下读的理由

hard_fail_if:
  - page_is_plot_summary
  - page_only_explains_what_happened
  - page_has_no_turn
  - page_has_no_end_hook
```

---

## 13. 四级脚本链

V13.1 严禁直接“章节摘要 → page_script”。必须按以下链路工作：

```text
chapter_event_map.json
→ director_beat_sheet.json
→ panel_beat_sheet.json
→ page_script.json
```

### 13.1 chapter_event_map.json

```yaml
chapter_event_map:
  purpose:
    - 列出本章真实事件序列
    - 标出人物行动
    - 标出压力变化
    - 标出章节高潮点
    - 标出结尾钩子
```

结构：

```json
{
  "chapter_title": "",
  "events": [
    {"order": 1, "event": "", "participants": [], "conflict_value": 0, "must_preserve": true}
  ],
  "climax_event": "",
  "ending_hook_event": ""
}
```

### 13.2 director_beat_sheet.json

每页必须定义：

```yaml
director_page_beat:
  each_page_must_have:
    - page_goal
    - page_conflict
    - emotional_turn
    - visual_memory
    - end_hook
```

### 13.3 panel_beat_sheet.json

每页格数锁定：

```yaml
panel_density_rule:
  min_panels_per_page: 5
  target_panels_per_page: 6
  max_panels_per_page: 7
  hard_fail_if:
    - any_page_panels_less_than_5
    - any_page_panels_more_than_7
```

每页必须出现的格型：

```yaml
panel_language_required:
  - one_main_visual_panel
  - one_character_closeup
  - one_prop_or_eye_closeup
  - one_reaction_panel
  - one_transition_or_silent_panel
```

结构：

```json
{
  "chapter_title": "",
  "pages": [
    {
      "page": "P01",
      "panel_count": 6,
      "panels": [
        {
          "panel_id": "P01-1",
          "function": "wide_establishing / face_closeup / prop_closeup / reaction / action / hook",
          "story_beat": "",
          "camera": "",
          "characters": [],
          "must_show": [],
          "text_function": ""
        }
      ]
    }
  ]
}
```

### 13.4 page_script.json

`page_script` 是最终漫画脚本，不再负责重新想剧情。它只能按 `panel_beat_sheet` 落格、按 `source_dialogue_bank` 选对白、按导演页纲组织阅读节奏。

```yaml
page_script_forbid:
  - invent_new_plot
  - replace_named_terms
  - rewrite_conflict
  - convert_page_into_summary
```

---

## 14. 角色导演核心

角色基础包不是“有几张角色图”，而是“角色记忆点工程”。

```yaml
character_bootstrap_goal:
  - 主要角色一眼分清
  - 角色稳定贯穿 P01-P10
  - 角色有鲜明记忆点
  - 风格接近中国主流彩色漫画角色设定
  - 禁止像 CG 角色设定板

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

character_distinction_rule:
  hard_requirements:
    - 所有主要角色必须轮廓不同
    - 所有主要角色必须脸型不同
    - 所有主要角色必须发型 / 胡须不同
    - 所有主要角色必须身体语言不同
    - 所有主要角色必须主色块不同
    - 所有主要角色必须至少一个标志性姿态或道具
  hard_fail_if:
    - old_men_same_face
    - support_roles_same_face
    - costume_swap_without_identity_change
    - crowd_overpowering_leads
```

特殊角色规则：

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

## 15. 场景漫画化核心

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

核心原则：

```text
场景是舞台，不是主角。
```

---

## 16. 中国漫画页感强化：Serial Manhua Render Rule

```yaml
serial_manhua_render_rule:
  required:
    - black_clear_lineart
    - cel_shading
    - readable_panel_borders
    - character_first_readability
    - strong_expression_readability
    - stable_costume_recognition
    - panel_rhythm_variation
    - page_end_hook_visual

  encourage:
    - action_flow_readability
    - reaction_clarity
    - sect_or_camp_distinction
    - fast_mobile_reading
    - manhua_serial_feel

  forbid:
    - poster_first_composition
    - full_page_key_visual_bias
    - painterly_cg_finish
    - heavy_concept_art_background
    - cinematic_lighting_priority
    - illustration_over_story
```

---

## 17. 反电影 CG 规则：Anti Cinematic CG Rule

```yaml
anti_cinematic_cg_rule:
  hard_forbid:
    - cinematic_volumetric_lighting
    - movie_poster_composition
    - concept_art_landscape_priority
    - glossy_painting_texture
    - hyperreal_depth_of_field
    - overrendered_particles
    - dramatic_camera_lens_effects
    - environment_detail_higher_than_character_readability
    - every_panel_as_promo_art

  required_instead:
    - comic_line_priority
    - acting_priority
    - beat_priority
    - panel_storytelling_priority
```

---

## 18. 视觉风格锁

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
    - 像人类漫画工作室周更连载页
```

统一正向提示：

```text
中国主流彩色漫画连载页，偏日系人物线稿，清晰黑色漫画线，赛璐璐平涂，平涂色块，1-2层硬边阴影，人物表情漫画化，角色辨识度强，分镜边框清楚，镜头切换明确，动作关系清楚，场景服务人物而不抢戏，像人类漫画工作室连载页，强调页内阅读节奏与页尾钩。
```

统一负向提示：

```text
AI poster, cinematic CG, game concept art, glossy painting, overrendered lighting, hyperreal background, same face syndrome, infographic, summary page, storyboard board, production board, character sheet mixed into comic page, scene sheet mixed into comic page, big poster page replacing comic page, stick figure, wireframe, python drawing, svg diagram, movie key art, full-page concept art.
```

---

## 19. 台词与文字核心

重点：不是“尽量短”，而是“有内容、能读、像原著人物会说的话”。

```yaml
text_expression_rule:
  principle:
    - 内容表达优先
    - 原著语气优先
    - 可读性第二
    - 防乱码是技术约束，不是创作目标

dialogue_rule:
  bubble_text:
    preferred_length: 8_to_18_chinese_chars
    max_length: 22_chinese_chars
    max_lines_per_bubble: 2
  page_text_budget:
    dialogue_bubbles_per_page: 4_to_8
    narration_boxes_per_page_max: 2
    total_readable_text_per_page: 40_to_120_chinese_chars

dialogue_style_requirements:
  - 有角色口吻
  - 有冲突感
  - 有态度
  - 能推动剧情
  - 少空话
  - 少泛化解释
  - 尽量继承原著句子气质

dialogue_forbid:
  - 为了防乱码把对白缩成空话
  - 所有人都只说三五个字
  - 用剧情摘要代替对白
  - 旁白代替人物冲突
  - 每个气泡都讲设定说明
```

角色口吻差异：

```yaml
speech_identity:
  protagonist:
    style: 克制、强硬、反问、回击、稳中带狠
  antagonist:
    style: 高位、冷压、威胁、命令、蔑视
  support:
    style: 信息推进、局势反应、立场表达
  crowd:
    style: 短促、震惊、议论，不长篇解释
  special_character:
    style: 强记忆点短句，不承担大段设定说明
```

---

## 20. 生成与交付工作流

V13.1 主线顺序固定，不允许跳步。

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

  step_3_canonical_adaptation_lock:
    action:
      - 启动 canonical_adaptation_only
      - 明确本次必须使用原著原名原设定
      - 禁止原创替身回退

  step_4_source_event_lock:
    output:
      - source_event_lock.json
    action:
      - 锁定主角
      - 锁定敌手
      - 锁定关键法宝
      - 锁定事件顺序
      - 锁定冲突关系
      - 锁定结尾钩子

  step_5_source_dialogue_bank:
    output:
      - source_dialogue_bank.json
    action:
      - 提取原文对白 / 旁白 / 钩子句
      - 保留原著语气

  step_6_chapter_event_map:
    output:
      - chapter_event_map.json
    action:
      - 列出本章事件序列
      - 标出高潮点与结尾钩子

  step_7_director_outline:
    output:
      - director_beat_sheet.json
    action:
      - 按固定 10 页结构设计每页目标 / 冲突 / 转折 / 钩子

  step_8_character_bootstrap:
    condition:
      - if_no_base_character_images_or_need_refresh
    output:
      - character_bootstrap/
    action:
      - 生成鲜明角色基础包
      - 锁定角色记忆点

  step_9_scene_bootstrap:
    output:
      - scene_bootstrap/
    action:
      - 生成漫画化场景基础包
      - 锁定主场景与关键道具

  step_10_panel_beat_sheet:
    output:
      - panel_beat_sheet.json
    action:
      - 为每页设计 5-7 格
      - 明确每格功能 / 镜头 / 角色 / 重点道具

  step_11_page_script:
    output:
      - page_script.json
    action:
      - 根据 panel_beat_sheet 与 source_dialogue_bank 生成最终页脚本
      - 禁止变成剧情摘要

  step_12_render_comic_pages:
    output:
      - comic_pages/P01.png ~ P10.png
    action:
      - 生成 10 张独立漫画页
      - 每页必须具备页尾钩或阅读推动力
      - 不得生成一张总览图代替

  step_13_handoff_and_qc:
    output:
      - handoff.json
      - qc_report.json
```

---

## 21. 交付清单

必须交付：

```text
source_event_lock.json
source_dialogue_bank.json
chapter_event_map.json
character_bootstrap/
scene_bootstrap/
director_beat_sheet.json
panel_beat_sheet.json
page_script.json
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

可选交付：

```text
preview_sheet.jpg
```

交付硬规则：

```yaml
delivery_hard_rules:
  hard_fail_if:
    - preview_sheet_replaces_P01_to_P10
    - one_big_collage_used_as_final_result
    - character_bootstrap_mixed_into_comic_pages
    - scene_bootstrap_mixed_into_comic_pages
    - json_or_qc_tables_drawn_into_comic_pages
    - production_board_drawn_into_comic_pages
```

---

## 22. handoff.json 要求

```json
{
  "version": "V13.1",
  "chapter_title": "",
  "page_count": 10,
  "core_characters": [],
  "key_named_terms": [],
  "key_props": [],
  "main_scene_state": [],
  "visual_style_lock": {
    "line": "clean manga line",
    "color": "cel shading",
    "render": "serial manhua, non-CG comic rendering"
  },
  "unresolved_hooks": [],
  "next_chapter_attention_points": []
}
```

---

## 23. qc_report.json 要求

```json
{
  "version": "V13.1",
  "rights_mode": "user_declared_private_noncommercial",
  "canonical_adaptation_check": {
    "status": "PASS",
    "original_protagonist_name_preserved": true,
    "original_antagonist_name_preserved": true,
    "original_named_terms_preserved": true,
    "no_original_substitute_detected": true
  },
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
    "director_chain_pass": true,
    "character_distinction_pass": true,
    "serial_manhua_render_pass": true,
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

## 24. 验收标准

### 24.1 原著改编验收

```yaml
canonical_adaptation_acceptance:
  required:
    - use_original_novel_content
    - original_protagonist_name_kept
    - original_antagonist_name_kept
    - original_named_terms_kept
    - no_original_substitute
  hard_fail_if:
    - protagonist_renamed
    - antagonist_renamed
    - key_terms_genericized
    - same_genre_original_replacement
```

### 24.2 剧情验收

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
    - replaced_key_props
    - changed_conflict_axis
```

### 24.3 漫画导演验收

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

### 24.4 角色验收

```yaml
character_acceptance:
  required:
    - main_characters_distinct
    - support_characters_distinct
    - child_or_special_character_stable
    - no_same_face_old_men
```

### 24.5 视觉验收

```yaml
style_acceptance:
  required:
    - chinese_manhua_serial_feel
    - japanese_influenced_clean_line
    - cel_shading
    - no_cg_concept_art
    - no_poster_like_rendering
    - character_first_readability
    - panel_rhythm_clear
```

### 24.6 交付验收

```yaml
delivery_acceptance:
  required:
    - source_event_lock_exists
    - source_dialogue_bank_exists
    - chapter_event_map_exists
    - character_bootstrap_exists
    - scene_bootstrap_exists
    - director_beat_sheet_exists
    - panel_beat_sheet_exists
    - page_script_exists
    - P01_to_P10_all_exist
    - handoff_exists
    - qc_report_exists
```

---

## 25. 公开使用提示模板

### 25.1 首次冷启动模板

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md

这是个人娱乐用途，非商业化，不公开发布，仅用于内部流程测试。
这是首次冷启动，没有基础角色图，也没有 handoff.json。
请按 To-Comic-StudioFlow V13.1 工作。

要求：
- 必须使用原小说内容与原主角名，不允许改成同类型原创替代
- 固定输出 10 页漫画：P01.png 到 P10.png
- 每页分镜不能少于 5 格，标准 6 格，最多 7 格
- 禁止把漫画做成剧情摘要
- 台词要能表达内容，并尽量保留原著语气
- 必须先生成 source_event_lock.json 与 source_dialogue_bank.json
- 严格保留主角、敌手、关键专名、关键法宝、事件顺序、冲突关系和结尾钩子
- 角色基础包必须鲜明、有记忆点，接近中国主流漫画角色包水平
- 禁止 CG 概念图、游戏宣传图、海报化结果
- 必须输出：
  source_event_lock.json
  source_dialogue_bank.json
  chapter_event_map.json
  character_bootstrap/
  scene_bootstrap/
  director_beat_sheet.json
  panel_beat_sheet.json
  page_script.json
  comic_pages/P01.png 到 P10.png
  handoff.json
  qc_report.json

如果不能忠实改编，请直接停止说明，不要静默改写剧情，不要输出同类型原创替代版本。
```

### 25.2 下一章继续模板

```text
请读取我上传的：
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一期 character_bootstrap
4. 上一期 handoff.json

这是个人娱乐用途，非商业化，不公开发布，仅用于内部流程测试。
请按 To-Comic-StudioFlow V13.1 工作，继续输出本章 P01.png 到 P10.png。

要求继续保持：
- 必须使用原小说内容与原主角名
- 固定10页
- 每页5到7格
- 剧情保真
- 角色稳定
- 禁止剧情摘要
- 禁止 CG 概念图
- 禁止同类型原创替代
```

---

## 26. V13.1 核心总结

```text
1. 必须使用原小说内容与原主角名
2. 不能忠实改编就停止，绝不允许原创替身回退
3. 固定 10 页
4. 每页固定 5~7 格，低于 5 格直接失败
5. 先锁剧情，再锁对白，再做导演页纲，再做分镜页纲，再生成页脚本
6. 严禁剧情摘要
7. 台词要表达内容，并保留原著气质
8. 角色基础包要鲜明、有记忆点
9. 场景服务人物和动作，禁 CG 概念图
10. 目标不是像一张好看插画，而是像一页真正的中国连载漫画
```
