# SKIIS V12：CN Novel To Comic StudioFlow

> 单文件版。用户只需要上传 `SKIIS_V12.md` + 小说章节原文/链接 + 基础角色图 + 上一期 `handoff.json`，即可直接生成本章漫画成品 P01-P10 与新的 `handoff.json`。  
> 本文件已经合并原本 `docs/OPERATION_GUIDE.md` 与 `docs/NEXT_STEPS.md` 的核心内容，不再依赖额外说明文档。

---

## 0. 一句话目标

```text
输入一章小说，直接输出中国漫画成品页 P01-P10，并生成 handoff.json，下一章继续用同一个 SKIIS + 基础角色图 + 上一期 handoff + 新章节原文/链接接着生产。
```

---

## 1. 必须输入

```yaml
required_inputs:
  - 小说当前章节原文或小说章节链接
  - SKIIS_V12.md
  - 基础角色图
  - 上一期 handoff.json
```

如果是第一期，没有上一期 `handoff.json`：

```yaml
first_issue_rule:
  - 自动新建空白 handoff
  - 以基础角色图作为最高优先级角色锚点
```

如果是下一章：

```yaml
next_issue_rule:
  - 必须读取上一期 handoff.json
  - 继承角色外观、道具状态、场景状态和未解决钩子
  - 不得随意改变上一期已经锁定的人物、服装、法宝和关系
```

---

## 2. 默认输出

默认不要停在 P01 测试页，也不要停在 P08 测试页。除非用户明确说“先测试”，否则直接输出完整结果包。

```yaml
default_outputs:
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

## 3. 基础配置

```yaml
SKIIS_NAME: CN_Novel_To_Comic_StudioFlow_V12
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

## 4. 生产目标

```yaml
goal:
  - 把一章小说稳定改编成中国漫画
  - 直接生成 P01-P10 漫画页
  - 保持人物、场景、道具和画风连续
  - 生成下一章可继续使用的 handoff.json
  - 降低 AI 味、同脸、CG感和角色漂移
  - 让结果像人类漫画团队生产，而不是AI海报拼图
```

---

## 5. 禁止项

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

## 6. 自动工作流

```yaml
workflow:
  step_1_read_inputs:
    action:
      - 读取小说当前章节原文或链接内容
      - 读取基础角色图
      - 读取上一期 handoff.json
      - 读取本 SKIIS_V12.md

  step_2_chapter_analysis:
    output:
      - chapter_card.json
    action:
      - 提取本章主事件
      - 提取角色意图
      - 提取关键场景、道具、冲突和结尾钩子
      - 删除重复解释和低视觉价值旁白

  step_3_director_beat:
    output:
      - director_beat_sheet.json
    action:
      - 每页设计 reader_question
      - 每页设计 page_answer
      - 每页设计 new_question
      - 每页设计 key_visual 和 page_turn_hook

  step_4_lock_assets:
    output:
      - character_lock.json
      - scene_lock.json
    action:
      - 用基础角色图锁定主角和关键角色
      - 用 handoff 继承上一期状态
      - 锁定本章场景地标、道具状态

  step_5_page_script:
    output:
      - page_script.json
    action:
      - 生成 P01-P10 页面脚本
      - 每页 3-6 个分镜
      - 每页一个剧情变化点
      - 每页有 hook_line

  step_6_generate_comic_pages:
    output:
      - comic_pages/P01.png ... comic_pages/P10.png
    action:
      - 按页面脚本生成漫画页
      - 每页体现大中小分镜变化
      - 保持角色连续、场景连续、风格连续
      - 文字可后期排版，但读者必须能理解剧情

  step_7_output_handoff:
    output:
      - handoff.json
    action:
      - 记录本期角色、场景、道具、未解决钩子
      - 记录下一章不能改变的内容
```

---

## 7. 小说分析规则

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

## 8. P01-P10 结构

默认一章输出 10 页。如果章节非常短，可以压缩为 6-8 页；但默认仍以 P01-P10 输出最稳定。

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

## 9. 页面脚本格式

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

## 10. 每页分镜规则

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

panel_forbid:
  - 全页都是横向矩形
  - 全页都是大场景
  - 全页都是人物近景
  - 没有反应格
  - 没有结尾钩子
```

分镜切块库：

```yaml
panel_cut_library:
  A_wide_establishing:
    use: 建立大场景
    shape: 横向大格
  B_tall_pressure:
    use: 高处压迫/敌方登场
    shape: 窄长竖格
  C_closeup_slice:
    use: 眼神/手/法宝
    shape: 横向窄条
  D_diagonal_action:
    use: 冲击/飞行/斩击
    shape: 斜切格
  E_silent_gap:
    use: 悬念/停顿/压抑
    shape: 大留白或无字小格
  F_reaction_stack:
    use: 群像反应
    shape: 2-3个小格叠放
  G_bleed_impact:
    use: 高潮爆点
    shape: 无边框溢出版
```

---

## 11. 角色锁定规则

基础角色图优先级最高。文字设定只能补充，不能覆盖基础角色图。

```yaml
character_shape_language:
  name:
  tier:
  role:
  silhouette:
  age_read:
  face_shape:
  eye_brow:
  hair_beard:
  body_posture:
  outfit_shape:
  color_block:
  prop_or_mark:
  speech_style:
  appearance_limit:
  forbidden:
```

角色分级：

```yaml
character_tier:
  A_main:
    rule: 主角，必须最稳定，基础角色图优先

  B_key_support:
    rule: 关键配角，出场少但强识别

  C_named_pressure:
    rule: 命名敌方，必须有独立轮廓和脸型差异

  D_crowd:
    rule: 群像不精画脸，靠阵型、服色、武器、旗帜识别
```

幼态/儿童角色：

```yaml
child_character_rule:
  max_appearance_per_issue: 2
  no_random_background_appearance: true
  no_crowd_mixing: true
  must_keep:
    - 小体型
    - 圆脸大眼
    - 固定发型
    - 固定服装色
    - 固定道具
  forbid:
    - 成人化
    - 长发少女化
    - 黑衣化
    - 随机出现在群像中
```

---

## 12. 场景锁定规则

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

## 13. 画风锁定

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

---

## 14. 统一风格提示词

```text
中国漫画，偏日系人物线稿，清晰黑色漫画线，主轮廓略粗，内部线条较细，赛璐璐平涂，平涂色块，1-2层硬边阴影，少量局部法宝光，背景简化，分镜黑边清楚，竖向滚动漫画，人物表情漫画化，强镜头切换，留白节奏，像人类漫画工作室连载页。
```

英文辅助：

```text
Chinese comic style, Japanese-influenced manga character line art, clean black ink outline, variable line weight, cel-shading, flat color blocks, simple hard-edge shadows, low-to-mid detail, simplified background, comic panel readability, hand-drawn comic feeling, rough line texture, vertical scrolling comic layout
```

负面提示：

```text
AI poster, cinematic CG, game concept art, glossy painting, painterly rendering, oil painting, realistic skin, 3D face, excessive glow, volumetric light, depth of field, hyper detailed background, same face syndrome, character sheet, infographic, summary page, thumbnails
```

---

## 15. 输出结果包结构

```text
output/
  chapter_card.json
  director_beat_sheet.json
  character_lock.json
  scene_lock.json
  page_script.json
  comic_pages/
    P01.png
    P02.png
    P03.png
    P04.png
    P05.png
    P06.png
    P07.png
    P08.png
    P09.png
    P10.png
  handoff.json
```

---

## 16. handoff.json 必须记录

```yaml
handoff_must_include:
  - 角色外观锚点
  - 角色当前状态
  - 场景地标
  - 道具归属与状态
  - 本章已解决事件
  - 下一章未解决钩子
  - 下次不能改变的内容
```

下一章启动时，用户只需要上传：

```text
1. 下一章小说地址或原文
2. SKIIS_V12.md
3. 基础角色图
4. 上一期 handoff.json
```

系统必须直接继续生成下一章 P01-P10 与新的 handoff.json。

---

## 17. 验收标准

```yaml
acceptance:
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

---

## 18. 用户启动提示词

用户只需要这样说：

```text
请读取我上传的：
1. 小说当前章节原文或小说地址
2. SKIIS_V12.md
3. 基础角色图
4. 上一期 handoff.json

按 SKIIS V12 工作，直接输出本章完整结果包：
- chapter_card.json
- director_beat_sheet.json
- character_lock.json
- scene_lock.json
- page_script.json
- comic_pages/P01-P10.png
- handoff.json

如果没有上一期 handoff，就新建空白 handoff。
不要先停在 P01 测试页。
不要让我逐页确认。
直接生成可以看到的漫画页。
```
