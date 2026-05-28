# To-Comic-StudioFlow V12.1

> 单文件版中国小说转漫画 SKIIS。  
> 用户只需要上传本文件 + 小说章节原文/链接，即可冷启动生成角色基础图、P01-P10 漫画页与 `handoff.json`。  
> 如果用户已经有基础角色图或上一期 `handoff.json`，系统必须优先继承它们，继续生成下一章。

---

## 0. 一句话目标

```text
输入一章小说，直接输出中国漫画 P01-P10、角色基础图包和 handoff.json；下一章继续上传本文件 + 基础角色图/角色基础包 + 上一期 handoff + 新章节原文/链接，即可接着生产。
```

---

## 1. 三种启动模式

### A. 首次冷启动：没有基础角色图，也没有 handoff

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
```

系统必须先自动生成角色基础图包，再生成漫画。

输出：

```text
character_bootstrap/
  cast_master_sheet.png
  char_A_main_01.png
  char_B_support_01.png
  char_C_enemy_01.png
  character_bootstrap.json

chapter_card.json
director_beat_sheet.json
character_lock.json
scene_lock.json
page_script.json
comic_pages/P01-P10.png
handoff.json
```

### B. 首次标准启动：有基础角色图，没有 handoff

用户输入：

```text
1. 小说当前章节原文或小说章节链接
2. To-Comic-StudioFlow.md
3. 基础角色图
```

系统自动新建空白 handoff，并以基础角色图作为最高优先级视觉锚点。

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

## 2. 冷启动角色基础图生成规则

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
```

冷启动必须先输出：

```yaml
cold_start_required_outputs:
  - character_bootstrap/character_bootstrap.json
  - character_bootstrap/cast_master_sheet.png
  - character_bootstrap/char_A_main_01.png
  - character_bootstrap/char_B_support_01.png
  - character_bootstrap/char_C_enemy_01.png
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

角色图必须在 `character_bootstrap.json` 里标记来源：

```yaml
source_type:
  - text_explicit
  - text_inferred
  - genre_inferred
  - user_base_image
```

---

## 3. 角色基础图必须包含什么

### cast_master_sheet.png

作用：让用户一眼看到本章主要角色关系和视觉差异。

必须显示：

```yaml
cast_master_sheet:
  must_show:
    - 主角
    - 关键配角
    - 主要敌方
    - 群像阵营色块
    - 身高/体型对比
    - 发型差异
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

## 4. 角色建档优先级

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

## 5. 生成漫画页前置门槛

```yaml
pre_page_gate:
  before_generating_comic_pages:
    must_have_one_of:
      - base_character_images
      - previous_handoff_with_character_state
      - generated_character_bootstrap

  if_none:
    action: create_character_bootstrap_first
```

如果页面脚本需要角色出现，则每页都必须有明确角色：

```yaml
page_qc:
  each_page:
    must_contain:
      - at_least_one_visible_story_character_if_script_requires
      - clear story action
      - clear relationship or conflict
      - visual continuity with character_lock

  fail_conditions:
    - blank_page_like_layout
    - placeholder_boxes_only
    - no_character_when_character_should_appear
    - storyboard_wireframe_output
    - character_not_matching_bootstrap
```

---

## 6. 默认输出结果包

默认不要停在 P01 测试页，也不要停在 P08 测试页。除非用户明确说“先测试”，否则直接输出完整结果包。

```yaml
default_outputs:
  - character_bootstrap/            # 仅冷启动时必须输出
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
```

最终用户必须能直接看到漫画：

```yaml
comic_result_required:
  - P01-P10 必须是漫画成品页或可阅读测试页
  - 不允许只输出 JSON 而没有漫画页
  - 不允许只输出分镜说明而没有图
  - handoff.json 必须和漫画页一起输出
```

---

## 7. 基础配置

```yaml
SKIIS_NAME: To-Comic-StudioFlow
VERSION: 12.1
MODE: 中国小说转漫画 / 中国漫画 / 人类漫画工作流
UNIT: 1个小说章节 = 1期漫画
DEFAULT_PAGE_COUNT: 10
PAGE_RANGE: P01-P10
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

## 8. 禁止项

```yaml
forbid:
  - 不要默认停在 P01 测试页
  - 不要默认停在 P08 测试页
  - 不要要求用户逐页确认后才继续
  - 不要只输出制作文件，不输出漫画页
  - 不要生成章节总览图
  - 不要一张图塞完整章
  - 不要角色设定栏画进漫画
  - 不要AI厚涂海报感
  - 不要CG电影感
  - 不要游戏概念图感
  - 不要所有配角同脸
  - 不要让AI直接生成大量中文小字
```

---

## 9. 自动工作流

```yaml
workflow:
  step_1_read_inputs:
    action:
      - 读取小说当前章节原文或链接内容
      - 读取基础角色图，如果有
      - 读取上一期 handoff.json，如果有
      - 读取本 To-Comic-StudioFlow.md

  step_2_start_mode_detection:
    action:
      - 判断是否有基础角色图
      - 判断是否有上一期 handoff.json
      - 如二者都没有，进入 cold_start_character_bootstrap

  step_3_cold_start_character_bootstrap:
    condition: no_base_character_images and no_previous_handoff
    output:
      - character_bootstrap/cast_master_sheet.png
      - character_bootstrap/char_A_main_01.png
      - character_bootstrap/char_B_support_01.png
      - character_bootstrap/char_C_enemy_01.png
      - character_bootstrap/character_bootstrap.json

  step_4_chapter_analysis:
    output:
      - chapter_card.json

  step_5_director_beat:
    output:
      - director_beat_sheet.json

  step_6_lock_assets:
    output:
      - character_lock.json
      - scene_lock.json

  step_7_page_script:
    output:
      - page_script.json
    action:
      - 生成 P01-P10 页面脚本
      - 每页 3-6 个分镜
      - 每页一个剧情变化点
      - 每页有 hook_line

  step_8_generate_comic_pages:
    output:
      - comic_pages/P01.png ... comic_pages/P10.png

  step_9_output_handoff:
    output:
      - handoff.json
```

---

## 10. 小说分析规则

```yaml
chapter_analysis:
  story_spine:
    start_state:
    turning_point:
    end_state:
    chapter_hook:

  character_intent:
    protagonist_goal:
    antagonist_goal:
    support_character_value:
    pressure_source:

  visual_must_have:
    key_scene:
    key_prop:
    key_action:
    key_symbol:

  emotion_curve:
    curiosity:
    tension:
    pressure:
    reversal:
    hook:

  cut_list:
    remove:
      - 重复解释
      - 长篇内心独白
      - 低视觉价值旁白
    convert:
      - 内心判断 -> 眼神/手部停顿
      - 世界观说明 -> 道具特写+短旁白
      - 群体压迫 -> 阵型/剪影/色块
```

---

## 11. P01-P10 结构

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

## 12. 页面脚本格式

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

## 13. 每页分镜规则

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

## 14. 场景锁定规则

```yaml
scene_anchor:
  name:
  landmarks:
  color_palette:
  repeated_angle:
  simplified_background_version:
  used_pages:
  do_not_change:

scene_style:
  establishing:
    detail: medium
    use: 交代地标

  dialogue:
    detail: low
    use: 人物关系优先，背景简化

  action:
    detail: very_low
    use: 速度线、气浪、碎石、色块背景
```

---

## 15. 画风锁定

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
    - 背景简化
    - 只保留地标
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
中国漫画，偏日系人物线稿，清晰黑色漫画线，主轮廓略粗，内部线条较细，赛璐璐平涂，平涂色块，1-2层硬边阴影，少量局部法宝光，背景简化，分镜黑边清楚，竖向滚动漫画，人物表情漫画化，强镜头切换，留白节奏，像人类漫画工作室连载页。
```

负面提示：

```text
AI poster, cinematic CG, game concept art, glossy painting, painterly rendering, oil painting, realistic skin, 3D face, excessive glow, volumetric light, depth of field, hyper detailed background, same face syndrome, character sheet, infographic, summary page, thumbnails
```

---

## 16. handoff.json 必须记录

```yaml
handoff_must_include:
  - 角色外观锚点
  - 角色当前状态
  - 角色图来源
  - 场景地标
  - 道具归属与状态
  - 本章已解决事件
  - 下一章未解决钩子
  - 下次不能改变的内容
```

冷启动时 handoff 还必须记录：

```yaml
handoff_character_anchor_source:
  mode: auto_bootstrap
  source_chapter:
  generated_character_pack:
    - character_bootstrap/cast_master_sheet.png
    - character_bootstrap/char_A_main_01.png
    - character_bootstrap/char_B_support_01.png
    - character_bootstrap/char_C_enemy_01.png

  note:
    - 本期角色图由系统根据小说冷启动生成
    - 下一章可继续使用
    - 如果用户上传新基础角色图，新图优先级高于自动生成图
```

下一章启动时，用户上传：

```text
1. 下一章小说地址或原文
2. To-Comic-StudioFlow.md
3. 基础角色图或上一章 character_bootstrap
4. 上一期 handoff.json
```

系统必须直接继续生成下一章 P01-P10 与新的 handoff.json。

---

## 17. 用户启动提示词

### 首次无角色图

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

### 首次有角色图

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图

这是第一期，没有上一期 handoff.json。
请自动新建 handoff，然后直接输出本章完整结果包。
```

### 下一章继续

```text
请读取我上传的：
1. 下一章小说原文或小说地址
2. To-Comic-StudioFlow.md
3. 基础角色图或上一章 character_bootstrap
4. 上一期 handoff.json

请继承上一期 handoff，直接输出下一章 P01-P10 与新的 handoff.json。
```

---

## 18. 验收标准

```yaml
acceptance:
  cold_start:
    if_no_base_character_images_and_no_handoff:
      character_bootstrap: required
      cast_master_sheet: required
      character_bootstrap_json: required
      no_placeholder_pages: required

  complete_result:
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
