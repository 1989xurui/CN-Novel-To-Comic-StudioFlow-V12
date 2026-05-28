# SKIIS V12：CN Novel To Comic StudioFlow

## 0. 当前版本原则

本版取消默认的 **P01 测试页**、**P08 旧敌测试页** 等中间确认节点。

只要用户一次性提供以下 4 类输入，就直接输出完整漫画生产结果包：

```text
1. 小说当前章节原文
2. SKIIS_V12.md
3. 基础角色图
4. 上一期 handoff.json
```

如果没有上一期 `handoff.json`，则新建一个空白 handoff 作为第一期起点。

---

## 1. 基础配置

```yaml
SKIIS_NAME: CN_Novel_To_Comic_StudioFlow_V12
MODE: 中国小说转漫画 / 中国漫画 / 日系人物线稿 / 人类漫画工作流
UNIT: 1个小说章节 = 1期漫画
PAGE_COUNT:
  simple_chapter: 6
  normal_chapter: 8
  dense_chapter: 10
  max: 10
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

## 2. 目标

```yaml
goal:
  - 把一章小说稳定改编成中国漫画
  - 保持人物、场景、道具和风格连续
  - 输出可以继续下一章生产的 handoff.json
  - 尽量减少 AI 味、同脸、CG感和角色漂移
  - 让漫画像人类团队生产，而不是AI海报拼图
```

---

## 3. 核心输入

```yaml
required_inputs:
  - current_chapter_text: 小说当前章节原文
  - skiis_file: SKIIS_V12.md
  - base_character_images: 基础角色图
  - previous_handoff: 上一期 handoff.json
```

输入解释：

```yaml
current_chapter_text:
  purpose: 当前章剧情来源

SKIIS_V12.md:
  purpose: 工作流、画风、脚本、分镜、验收规则

base_character_images:
  purpose: 锁定主角和关键配角外观，优先级高于文字描述

previous_handoff.json:
  purpose: 继承上一期角色、道具、场景、未解决钩子和禁改项
```

---

## 4. 默认输出

默认直接输出完整结果包，不停在测试页。

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
      - ...
  - handoff.json
```

可选内部资产：

```yaml
optional_internal_assets:
  - panel_images/
  - lettering_data.json
  - qc_report.json
  - prompt_pack.json
```

---

## 5. 禁止项

```yaml
forbid:
  - 不要默认先输出P01测试页
  - 不要默认先输出P08测试页
  - 不要要求用户逐页确认后才继续
  - 不要生成章节总览图
  - 不要一张图塞完整章
  - 不要角色设定栏画进漫画
  - 不要AI厚涂海报感
  - 不要CG电影感
  - 不要游戏概念图感
  - 不要所有配角同脸
  - 不要让AI直接生成大量中文小字
```

如果用户明确要求“先测试一页”，才进入单页测试模式；否则默认完整输出。

---

## 6. 生产流程

```yaml
workflow:
  step_1_read_inputs:
    action:
      - 读取小说当前章节原文
      - 读取基础角色图
      - 读取上一期 handoff.json
      - 读取 SKIIS_V12.md

  step_2_chapter_analysis:
    output:
      - chapter_card.json
    action:
      - 提取本章主事件
      - 提取角色意图
      - 提取关键场景、道具、冲突和结尾钩子

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
      - 用 handoff 继承上一章状态
      - 锁定本章场景地标、道具状态

  step_5_page_script:
    output:
      - page_script.json
    action:
      - 生成6-10页漫画脚本
      - 每页3-6个分镜
      - 每页一个剧情变化点

  step_6_generate_pages:
    output:
      - comic_pages/P01.png ... Pxx.png
    action:
      - 按单格无字图逻辑生成画面
      - 合成页面
      - 添加中文气泡和拟声字

  step_7_handoff:
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

## 8. 页数判断

```yaml
page_count_selector:
  simple_chapter:
    pages: 6
    condition: 单事件、少角色、无大战

  normal_chapter:
    pages: 8
    condition: 有发现、行动、转折、钩子

  dense_chapter:
    pages: 10
    condition: 有关键道具、多方势力、身份反转、高潮宣言

  hard_limit:
    max_pages: 10
    reason: 减少角色漂移，提高完成度
```

---

## 9. 导演页纲格式

```yaml
director_beat:
  page_no:
  reader_question: 这一页开头，读者想知道什么？
  page_answer: 这一页回答什么？
  new_question: 这一页结尾留下什么？
  emotional_target: 好奇/紧张/压迫/爽点/反转/悬念
  key_visual: 本页最大视觉记忆点
  camera_strategy: 远景/近景/特写/斜切/留白/反应格
  page_turn_hook: 让读者继续下滑的钩子
```

---

## 10. 页面脚本格式

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

## 11. 每页分镜规则

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

---

## 12. 角色锁定规则

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

---

## 13. 场景锁定规则

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

## 14. 画风锁定

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

## 15. 统一风格提示词

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

## 16. 输出结果包结构

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
    ...
  handoff.json
```

---

## 17. handoff.json 必须记录

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

---

## 18. 验收标准

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

## 19. 新项目启动提示词

```text
请读取我上传的：
1. 小说当前章节原文
2. SKIIS_V12.md
3. 基础角色图
4. 上一期 handoff.json

按 SKIIS V12 工作。

不要先停在 P01 测试页。
不要先停在 P08 测试页。
不要让我逐页确认。
除非我明确要求测试，否则直接输出完整结果包。

请直接输出：
1. chapter_card.json
2. director_beat_sheet.json
3. character_lock.json
4. scene_lock.json
5. page_script.json
6. comic_pages/P01-Pxx.png
7. handoff.json

要求：
- 保持人物、场景、道具连续
- 不要CG电影感
- 不要AI厚涂
- 不要一页塞完整章
- 不要生成章节总览图
- 中文排版尽量后期添加
```
