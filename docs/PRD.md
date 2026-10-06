# 今天吃什么（today-eat）产品需求文档

版本：v0.3　日期：2026-10-06　状态：草稿

变更记录：
- v0.1 初稿：一周清单生成、餐型模板、健康规则、菜谱与食材管理、购物清单
- v0.2 新增内容归属与官方内容、菜谱详细结构、媒体管理、版本管理、Web 管理后台、分享功能；更新数据模型与接口
- v0.3 菜谱结构改为"准备（备菜 + 备调料）+ 烹饪步骤"，详见 `docs/RECIPE.md`；明确用户在 App 录入；新增 AI 能力（第 6.7、6.8 节）

相关文档：
- `docs/CORE.md` 核心功能：库存驱动的菜谱与每日清单、管理员知识库、AI 生成新菜、导入来源（主功能，优先阅读）
- `docs/RECIPE.md` 菜谱内容规范：结构、写作规范、详情页展示、录入校验
- `docs/SEED_IMPORT.md` 种子数据导入方案：HowToCook → today-eat
- `docs/UI.md` App UI 设计方案

## 1. 背景与目标

每天决定"吃什么"耗费精力，临时决定又容易偏向高油高糖的外卖或重口味菜。本产品帮助用户按健康原则自动生成一周的早餐、晚餐清单，并管理家里的食材与菜谱，减少决策负担和食材浪费。

核心目标：
- 一键生成一周早晚饭，结果符合健康饮食原则
- 每餐的丰富程度可选（一菜一主食、两菜一主食、家庭聚会等）
- 提供详细、可信的官方菜谱库（用料、调料用量、步骤、图片、视频），同时允许用户维护自己的菜谱
- 管理食材库存，自动生成购物清单
- 预留分享能力，后续可在朋友之间传播菜谱

非目标（近期不做）：
- 午餐规划
- 精确的营养计算与热量统计（油、盐、糖用量统计除外，见 6.4）
- 社区评论、关注等社交功能

## 2. 用户与角色

| 阶段 | 用户 | 说明 |
|---|---|---|
| 阶段一 | 作者本人（家庭使用） | 单账号，验证核心流程 |
| 阶段二 | 同事、朋友 | 多账号，数据隔离，共享官方菜谱库 |
| 阶段三 | 更多用户 | 用户投稿、分享链接 |

系统角色：

| 角色 | 使用端 | 权限 |
|---|---|---|
| user | App | 管理自己的菜谱、食材、库存、清单；查看官方内容；复制官方菜谱为自己的版本 |
| admin | Web 管理后台（也可登录 App） | 管理所有官方内容、用户、审核 |
| editor（以后） | Web 管理后台 | 只能编辑官方内容 |
| reviewer（以后） | Web 管理后台 | 只能审核投稿和举报 |

设计约束：v1 虽然只有一个用户，但所有业务数据从一开始就挂在 `owner_id` 下，接口带鉴权，用户表带 `role`，避免后续重构。

## 3. 技术形态

| 部分 | 技术 | 说明 |
|---|---|---|
| App | Flutter（iOS / Android） | 普通用户使用 |
| 管理后台 | Web（建议 React + Ant Design Pro 或 Vue + Element Plus） | 管理员使用，以表格和表单为主 |
| 分享页（以后） | H5 单页 | 未安装 App 的人查看分享的菜谱 |
| 后端 | Rust，REST API | App、后台、H5 共用同一套 API，按角色鉴权（作者自行实现） |
| 数据库 | PostgreSQL | 阶段一用 SQLite 也可 |
| 对象存储 | OSS / COS / R2 / MinIO | 图片、视频；数据库只存 key |
| 视频处理（以后） | 云点播服务 | 转码 HLS、生成封面 |
| AI 模型 | 支持图片输入和结构化输出的大模型 | 仅后端调用，API key 不进 App |

## 4. 功能清单

优先级：P0 = MVP 必须，P1 = 第二版，P2 = 以后再说。

| 模块 | 功能 | 优先级 |
|---|---|---|
| 菜谱 | 菜谱增删改查：基础信息、备菜、备调料、烹饪步骤（结构见 RECIPE.md） | P0 |
| 菜谱 | 步骤的火力、动作、加入、时长、判断标准、结果图 | P0 |
| 菜谱 | 动作预设（选动作自动带出火力、时长、判断标准） | P0 |
| 菜谱 | 口感分支（脆 / 软等，切换后时长和判断标准同步变化） | P0 |
| 菜谱 | 角色、餐次、健康属性标注 | P0 |
| 菜谱 | 用料自动汇总；油、盐、糖按用量自动计算等级 | P0 |
| 菜谱 | 外部视频链接（B 站 / 抖音等） | P0 |
| 菜谱 | 复制官方菜谱为我的版本 | P0 |
| 菜谱 | 烹饪模式（全屏分步、等待项倒计时、计时结束确认判断标准） | P0 |
| 菜谱 | 术语表（点击术语查看解释） | P1 |
| 菜谱 | 语音朗读步骤 | P1 |
| 菜谱 | 评分、收藏、做过次数 | P1 |
| 菜谱 | 时间线视图 | P1 |
| 菜谱 | 上传视频、步骤关联视频片段 | P2 |
| AI | 菜谱导入：文字 / 链接 / 截图 / 只写菜名 → 结构化草稿 | P0 |
| AI | 补全缺失字段、估算油盐糖、匹配食材字典 | P0 |
| AI | 后台批量生成官方菜谱草稿 | P0 |
| AI | 健康改良（高油高糖菜生成减油减糖版本） | P1 |
| AI | 自然语言排菜（"这周清淡点，周三 4 个朋友来"） | P1 |
| AI | 拍小票 / 冰箱照片录入库存 | P2 |
| 食材 | 食材字典（官方 + 用户自建） | P0 |
| 食材 | 库存管理（数量、过期日期、存放位置） | P0 |
| 食材 | 食材别名（西红柿 = 番茄） | P1 |
| 食材 | 临期提醒 | P1 |
| 清单 | 按餐型模板生成一周早晚饭 | P0 |
| 清单 | 锁定、单项 / 单餐 / 整周重新生成、手动替换 | P0 |
| 清单 | 健康规则校验与提示 | P0 |
| 清单 | 自定义餐型模板 | P1 |
| 清单 | 临期食材优先、库存命中优先、模拟库存（见 CORE.md 第 5 节） | P0 |
| 购物 | 根据清单和库存生成购物清单，可勾选 | P0 |
| 购物 | 勾选后自动入库 | P1 |
| 记录 | 用餐打卡，自动扣减库存（见 CORE.md 5.6） | P0 |
| 记录 | 周报：荤素比例、高油高糖次数、油盐糖人均用量 | P1 |
| 账号 | 单用户登录 | P0 |
| 账号 | 多用户注册、忌口与过敏 | P1 |
| 后台 | 官方菜谱编辑与发布 | P0 |
| 后台 | 官方食材字典、标签、菜系、模板管理 | P0 |
| 后台 | 媒体库 | P1 |
| 后台 | 菜谱版本历史与回滚 | P1 |
| 后台 | 食材合并（用户自建 → 官方） | P1 |
| 后台 | 用户管理、数据统计 | P1 |
| 后台 | 审核队列 | P2 |
| 分享 | 分享为图片卡片 | P1 |
| 分享 | 分享链接 + H5 详情页 + 一键加入我的菜谱 | P2 |
| 分享 | 用户投稿到公共菜谱库 | P2 |

## 5. 核心概念

### 5.1 菜品角色

每道菜有且只有一个角色，餐型模板通过角色组织一餐的结构。

| 角色 | key | 示例 |
|---|---|---|
| 主食 | staple | 米饭、杂粮饭、面条、馒头、红薯 |
| 荤菜 | meat | 清蒸鲈鱼、白切鸡、虾仁炒蛋 |
| 素菜 | veg | 蒜蓉西兰花、清炒菠菜 |
| 荤素搭配 | mixed | 番茄炒蛋、青椒肉丝、西芹炒虾仁 |
| 汤 | soup | 冬瓜排骨汤、紫菜蛋花汤 |
| 凉菜 | cold | 凉拌黄瓜、拍木耳 |
| 一碗饭 | one_bowl | 盖饭、炒饭、拌面、汤面（主食和菜在一起） |
| 早餐主食 | bf_staple | 燕麦粥、全麦面包、包子、蒸玉米 |
| 早餐蛋白 | bf_protein | 水煮蛋、鸡蛋饼、豆腐脑 |
| 早餐饮品 | bf_drink | 牛奶、豆浆、无糖酸奶 |
| 水果 | fruit | 苹果、香蕉、圣女果 |

"荤素搭配"可以填补"荤菜"或"素菜"槽位，用于简餐时保证营养完整。

### 5.2 餐型模板（丰富程度）

模板 = 一组槽位，每个槽位是"角色 × 数量"。

| 模板 | 适用 | 槽位 |
|---|---|---|
| 一碗饭 | 晚饭 | 一碗饭 ×1 + 可选素菜或汤 ×1 |
| 简餐 | 晚饭 | 荤素搭配 ×1 + 主食 ×1 |
| 标准 | 晚饭 | 荤菜 ×1 + 素菜 ×1 + 主食 ×1 |
| 丰盛 | 晚饭 | 荤菜 ×1 + 素菜 ×1 + 荤素搭配 ×1 + 汤 ×1 + 主食 ×1 |
| 家庭聚会 | 晚饭 | 菜品总数 = 人数 + 1，荤素各半，+ 凉菜 ×1 + 汤 ×1 + 主食 ×1 |
| 标准早餐 | 早饭 | 早餐主食 ×1 + 早餐蛋白 ×1 + 早餐饮品 ×1 + 水果 ×1 |
| 简单早餐 | 早饭 | 早餐主食 ×1 + 早餐饮品 ×1 |

使用规则：
- 用户设置每周默认值，例如：工作日晚饭"标准"，周末晚饭"丰盛"，每天早餐"标准早餐"
- 生成前可单独调整某一餐的模板；选择"家庭聚会"时需填写人数（2～20）
- 聚会模板中荤菜不超过菜品数的一半，素菜和荤素搭配至少一半
- 内置模板由管理员在后台维护（官方内容）；P1 支持用户自定义模板

### 5.3 健康属性

| 属性 | 取值 | 来源 |
|---|---|---|
| 烹饪方式 | 蒸、煮、炖、凉拌、快炒、烤、红烧、煎、炸 | 手动标注 |
| 油量等级 | 少油、正常、多油 | 优先按用料自动计算，见 6.4；无法计算时手动标注 |
| 糖量等级 | 无糖、少糖、多糖 | 同上 |
| 盐量等级 | 少盐、正常、偏咸 | 同上 |
| 蛋白质来源 | 鱼虾、禽肉、红肉、蛋、豆制品、无 | 手动标注 |
| 是否全谷物/薯类 | 是 / 否（仅主食） | 手动标注 |
| 蔬菜占比 | 无、少量、主要 | 手动标注 |

健康分（用于排序，-5 ～ +5，由后端计算后存储，属性或用料变更时重算）：

| 项 | 分值 |
|---|---|
| 蒸、煮、凉拌 | +2 |
| 炖、快炒、烤 | 0 |
| 红烧 | -1 |
| 煎 | -2 |
| 炸 | -3 |
| 少油 / 多油 | +1 / -2 |
| 多糖 | -2 |
| 偏咸 | -1 |
| 蔬菜为主 | +1 |
| 全谷物或薯类主食 | +1 |

"高油高糖菜"定义：烹饪方式为煎、炸，或油量为多油，或糖量为多糖。

## 6. 菜谱内容

### 6.1 内容归属

官方内容与用户内容使用同一套表，通过字段区分。

| 字段 | 取值 | 说明 |
|---|---|---|
| owner_type | system / user | 管理员发布的内容为 system |
| owner_id | 用户 id | system 时为空 |
| visibility | private / link / public | 仅自己 / 持链接可见 / 所有人可见 |
| status | draft / pending / published / offline | 草稿 / 待审核 / 已发布 / 已下线 |
| source_recipe_id | 菜谱 id | 复制来源，可为空 |

规则：
- 管理员发布的官方菜谱为 `system + public + published`，所有用户可见，可用于生成清单
- 用户在 App 新建的菜谱默认为 `user + private + published`，只有自己可见
- 用户不能修改官方菜谱，只能"复制为我的版本"，新菜谱的 `source_recipe_id` 指向原菜。生成清单时，如果用户已有某道官方菜的个人版本，则只使用个人版本，不重复出现
- 用户可以"隐藏"某道官方菜谱，被隐藏的菜不参与生成
- 删除一律为软删除（`offline`），已引用的清单和用餐记录仍能正常显示
- 食材字典、餐型模板、标签沿用同样的归属模型

### 6.2 菜谱结构

完整规范见 `docs/RECIPE.md`，这里只列要点。

```
概览 → 用料（自动汇总）→ 准备（备菜 + 备调料）→ 烹饪步骤
```

- 概览：名称、简介、封面、角色、餐次、份数、难度、厨具、菜系、标签、健康属性、口感选项
- 准备阶段：所有不开火的工作。切配、泡发、腌制、调料汁、水淀粉都在这一步完成，每份成果放进带编号的"备料碗"（A 肉丝、B 青椒丝、D 料汁）
- 烹饪步骤：一步一个动作，每步包含火力、动作、加入（备料碗或食材 + 用量 + 方式）、时长区间、判断标准（看 / 听 / 闻 / 戳 / 尝）、结果图，以及默认折叠的贴士、注意和补救
- 口感分支：同一步骤可以按口感（脆 / 软、成块 / 软烂）设置不同的时长和判断标准，用户在详情页选择一次即可
- 用料只在第一次出现的地方填用量，"用料"页、购物清单、油盐糖计算都基于后端汇总
- 准备时长、烹饪时长、完整度由后端根据结构化数据计算
- 视频：每道菜一个主视频，步骤通过起止秒数关联片段

### 6.3 调料管理

- 油、盐、酱油、醋、糖、料酒等在食材字典中分类为"调料"，标记 `is_staple = true`
- 购物清单只判断常备调料"有 / 没有"，不累计用量
- 油、盐、糖不允许写"适量"（官方内容为错误，用户内容为警告），用户可以让 AI 估算

### 6.4 油盐糖自动计算

当菜谱中油、盐、糖（含酱油、蚝油等含盐调料，按食材字典中的折算系数）均有数值时，后端按每人份计算并得出等级：

| 指标 | 每人份（一道菜） | 等级 |
|---|---|---|
| 油 | ≤ 5g / 5～10g / > 10g | 少油 / 正常 / 多油 |
| 盐 | ≤ 1.5g / 1.5～2.5g / > 2.5g | 少盐 / 正常 / 偏咸 |
| 添加糖 | 0g / ≤ 5g / > 5g | 无糖 / 少糖 / 多糖 |

参考：《中国居民膳食指南（2022）》推荐成人每天烹调油 25～30g、盐 < 5g、添加糖 < 25g。阈值做成后台可配置参数。

食材字典为含盐调料配置 `salt_ratio`（如生抽约 0.16，即每 1g 生抽约含 0.16g 盐），含糖调料配置 `sugar_ratio`，数值由管理员维护。

### 6.5 媒体管理

所有图片、视频统一存入 Media 表，菜谱、步骤通过 id 引用。

- 上传流程：客户端向后端申请预签名 URL → 直传对象存储 → 通知后端上传完成。后端不转发文件流
- 图片：后端或对象存储生成缩略图（列表 400px、详情 1080px）；单张不超过 10MB
- 视频分阶段支持：

| 阶段 | 方式 | 对象 |
|---|---|---|
| P0 | 填写外部链接（B 站、抖音、YouTube），App 内跳转或内嵌播放 | 所有人 |
| P2 | 上传视频，经云点播转码为 HLS 并生成封面 | 仅管理员 |
| 以后 | 用户上传视频 | 需审核，视成本决定 |

### 6.6 版本管理

- 官方菜谱每次发布时，将完整内容（基础信息、用料、步骤）存为快照 `RecipeRevision`
- 后台可查看历史版本、对比差异、回滚到任一版本（回滚即以旧快照发布一个新版本）
- 清单引用 `recipe_id`，始终显示最新已发布内容

### 6.7 录入方式

用户在 App 内录入，不单独提供用户 Web 后台。理由：用户录入频率低、场景随手（做完菜、刷到视频），手机上的主要障碍是打字量，交给 AI 解决。管理员的批量、精细录入在 Web 管理后台完成。

渐进式录入：
- 必填只有菜名和角色，其余字段可以之后再补
- 菜谱卡片显示完整度，不完整的菜照样可以参与生成清单，健康分标注为"估算"
- 以后如有需求，用 Flutter Web 把同一套编辑表单做成响应式网页，不另外开发

点"＋"新建菜谱时有三种方式：

| 方式 | 流程 |
|---|---|
| 只写名字 | 输入菜名 → AI 生成完整草稿 |
| 导入 | 粘贴文字、网页或视频链接，或上传截图 → AI 解析为结构化草稿 |
| 手动录入 | 分阶段编辑器：基础 → 备菜 → 备调料 → 烹饪 → 检查 |

前两种方式最后都进入确认页。确认页就是编辑器，只是内容已经填好，AI 填写的字段带 ✨ 标记，用户检查修改后保存。

### 6.8 AI 能力

| 场景 | 说明 | 版本 |
|---|---|---|
| 菜谱解析 | 文字 / 链接 / 截图 / 菜名 → 符合 RECIPE.md 结构的 JSON：拆出备料碗，拆分步骤，补全火力、时长、判断标准 | P0 |
| 字段补全 | 对已有菜谱补全缺失项，返回差异供用户确认 | P0 |
| 健康属性 | 判断角色、烹饪方式、蛋白质来源、蔬菜占比 | P0 |
| 油盐糖估算 | 原文写"适量"时按常见做法估算数值，标记为估算 | P0 |
| 食材匹配 | 把原文食材对应到食材字典（含别名），没有的建议新建 | P0 |
| 后台批量生成 | 管理员用 AI 批量生成种子菜谱草稿，人工审核后发布 | P0 |
| 健康改良 | 为高油高糖菜生成减油减糖版本，可一键保存为我的版本 | P1 |
| 自然语言排菜 | 把用户描述转为 meal_overrides 和规则参数，再交给规则算法生成 | P1 |
| 库存识别 | 拍小票或冰箱照片识别食材入库 | P2 |

原则：
- AI 只产出草稿，不直接落库；官方内容还需人工审核
- 一周清单的生成仍由规则 + 打分算法完成，保证可控、可复现、低成本；AI 只负责意图翻译和内容结构化
- 字段级记录 AI 来源（`ai_fields`），用户修改后去除标记
- 用 JSON Schema 约束模型输出，后端再按 RECIPE.md 9.3 的校验规则检查一遍，不通过的字段标记为待确认
- 模型只在后端调用；解析链接、识别图片走异步任务
- 配额：每个用户每天导入次数有上限（默认 20 次），同一链接的解析结果缓存复用
- 链接导入只提取菜谱的结构化信息，不保存原文和原图；在界面上展示来源链接

## 7. 健康规则

规则参考《中国居民膳食指南（2022）》，每条可单独开关，参数可调。默认值由管理员在后台维护，用户可在 App 中覆盖。

硬规则（生成时必须满足，否则不选）：

| 编号 | 规则 | 默认 |
|---|---|---|
| H1 | 每顿晚饭至少一道蔬菜（素菜、荤素搭配、蔬菜为主的汤或凉菜） | 开 |
| H2 | 一周高油高糖菜总数上限 | 2 道 |
| H3 | 同一道菜一周内不重复（主食、饮品、水果除外） | 开 |
| H4 | 排除用户忌口和过敏食材 | 开 |
| H5 | 每顿早餐至少包含一种蛋白质来源（标准早餐模板） | 开 |

软规则（影响打分，尽量满足，不满足时在清单上提示）：

| 编号 | 规则 | 默认 |
|---|---|---|
| S1 | 一周鱼虾至少 2 次 | 开 |
| S2 | 红肉不连续两天出现 | 开 |
| S3 | 一周全谷物或薯类主食至少 3 次 | 开 |
| S4 | 蛋白质来源尽量轮换，同一来源一周不超过 3 次 | 开 |
| S5 | 最近 14 天吃过的菜降低权重 | 开 |
| S6 | 工作日晚饭单道菜烹饪时长不超过 30 分钟 | 开 |
| S7 | 每餐人均油、盐用量不超过每日推荐量的 40%（仅统计有数值的菜） | 开 |

聚会放宽：家庭聚会这一餐允许额外 1 道高油高糖菜，不计入 H2 的周上限，S7 不检查。

## 8. 一周清单生成

### 8.1 输入

- 周起始日期（默认本周一或下周一）
- 每天每餐的模板（来自默认设置，可单独调整）
- 已锁定的菜品
- 用户的健康规则配置、忌口
- 候选菜谱：用户可见的已发布菜谱（自己的 + 官方的），排除已隐藏的，个人版本优先于其来源官方菜
- 最近 14 天的用餐历史、当前库存（P1 用于加分）

### 8.2 流程

```
按日期顺序，对每一天的每一餐：
  1. 根据模板展开槽位列表
  2. 跳过已锁定的槽位，把锁定菜计入本周统计
  3. 对每个槽位：
     a. 候选 = 角色匹配 + 餐次匹配的菜谱
     b. 过滤：违反硬规则的剔除（忌口、本周已选、高油高糖额度已满…）
     c. 打分：健康分 × 权重 + 评分 + 软规则加减分 + 库存命中加分 − 最近吃过扣分
     d. 加权随机选取（同样输入多次生成结果不同）
     e. 更新本周统计
  4. 槽位无候选时，标记为"空缺"并提示用户补充该角色的菜谱
全部完成后检查软规则，未满足的在清单顶部列出提示
```

### 8.3 交互

- 生成后进入预览，可以：锁定某道菜、重新生成某道菜、重新生成某一餐、重新生成整周（保留锁定）、从候选列表中手动替换
- 手动替换时，违反硬规则的菜仍可选，但显示警告
- 确认后保存为当周清单；每周只有一份生效清单，再次生成会覆盖未锁定部分

## 9. 购物清单

- 汇总清单中所有菜谱的食材用量，同一食材合并，按菜谱份数和用餐人数缩放
- 减去当前库存，得到需购买数量；库存充足的不显示（可展开查看）
- 常备调料只按"有 / 没有"判断
- 用料为"适量"的食材只列名称，不计数量
- 按食材分类分组（蔬菜、肉禽、水产、蛋奶、豆制品、主食、调料）
- 支持勾选已购买；P1：勾选后自动加入库存，过期日期按食材默认保质期计算
- 单位：v1 每种食材固定一个基础单位，菜谱中使用的其他单位（个、根）需在食材字典中配置换算到基础单位的系数；无系数时不合并，分行显示

## 10. App 页面结构

详细设计见 `docs/UI.md`。底部导航四个 tab：本周、菜谱、食材、购物；设置从"本周"右上角进入。

## 11. Web 管理后台

只有 `role = admin` 的账号可以登录。

| 模块 | 功能 |
|---|---|
| 仪表盘 | 用户数、菜谱数、本周生成次数、最常被排入清单的菜、空缺最多的角色 |
| 官方菜谱 | 列表（筛选：角色、状态、健康分、完整度、缺结果图/视频）；编辑器（基础信息、备料碗、烹饪步骤拖拽排序、口感分支、视频时间点、实时校验与健康分）；AI 批量生成草稿；保存草稿、发布、下线；版本历史与回滚 |
| 烹饪预设 | 动作预设（默认火力、时长、判断标准、吸油系数）、切法图示、术语表 |
| 食材字典 | 官方食材增删改查；分类、单位与换算系数、保质期、盐糖折算系数；别名管理；把用户自建食材合并到官方食材 |
| 分类与标签 | 菜系、标签、食材分类 |
| 餐型模板 | 内置模板的槽位配置 |
| 健康规则 | 规则默认开关与参数、油盐糖等级阈值 |
| 媒体库 | 已上传的图片和视频，查看引用位置，清理未被引用的文件 |
| 审核（P2） | 用户投稿、举报内容，通过 / 驳回并填写原因 |
| 用户 | 用户列表、禁用账号、调整角色 |

后台操作需记录审计日志（谁在何时修改了什么）。

## 12. 分享

| 阶段 | 形式 | 实现 |
|---|---|---|
| P1 | 分享为图片卡片（菜谱或一周清单） | 客户端生成图片，调用系统分享，后端无改动 |
| P2 | 分享链接 | 菜谱设为 `visibility = link`，生成 `share_token`；H5 页面展示菜谱详情；已安装 App 的用户通过链接打开 App，可"加入我的菜谱"（复制，`source_recipe_id` 指向原菜） |
| P2 | 投稿到公共库 | 用户提交后 `status = pending` 进入审核队列；通过后转为 `public`，显示作者；官方可选择"收录为官方菜谱"（复制为 system 内容） |

约束：
- 分享出的菜谱是快照式复制，原作者后续修改不影响已复制的菜谱
- 原作者可随时关闭分享链接（重置 `share_token` 或改回 `private`）
- 公开内容上线前需接入图片内容安全检测；若在国内面向公众上线，需完成 ICP 备案

## 13. 数据模型

通用字段：所有业务表含 `id`、`created_at`、`updated_at`。可归属的内容表含 `owner_type`、`owner_id`。

```
User              id, email, password_hash, nickname, avatar_media_id, role(user/admin), status

Ingredient        id, owner_type, owner_id, name, category, base_unit, shelf_life_days,
                  is_staple, salt_ratio, sugar_ratio, merged_into_id
IngredientAlias   ingredient_id, alias
IngredientUnit    ingredient_id, unit, to_base_ratio          -- 1 个鸡蛋 = 50 g
Inventory         id, owner_id, ingredient_id, quantity, location, purchased_at, expires_at

Recipe            id, owner_type, owner_id, visibility, status, source_recipe_id, share_token,
                  name, summary, cover_media_id, role, meal_types[], servings,
                  difficulty, cuisine, tags[], equipment[], texture_dims(json),
                  prep_minutes(自动), cook_minutes(自动), completeness(自动),
                  cook_method, oil_level, salt_level, sugar_level, levels_estimated(bool),
                  protein_source, is_whole_grain, veg_level, health_score,
                  video_url, video_media_id, tips, substitutions, source_url,
                  ai_fields[], current_version

-- 准备阶段、烹饪步骤的完整字段见 RECIPE.md 第 10 节
PrepItem          备料碗：备菜（切配、泡发）或备调料（腌制、料汁、水淀粉）
PrepItemInput     备料碗的用料
CookStep          烹饪步骤：火力、动作、时长、判断标准、口感分支、贴士、注意、补救
StepAddition      步骤中加入的备料碗或食材
RecipeIngredientSummary   用料汇总（后端生成，只读）
ActionPreset      动作预设（后台维护）
CutStyle          切法（后台维护）
Glossary          术语表（后台维护）
UserRecipePref    用户对某菜谱的口感选择与份数

RecipeRevision    id, recipe_id, version, snapshot_json, editor_id, note
RecipeHidden      user_id, recipe_id
RecipeFavorite    user_id, recipe_id                                     (P1)

Media             id, owner_id, type(image/video), storage_key, url, thumb_url,
                  width, height, duration, size, status(uploading/processing/ready/failed)

MealTemplate      id, owner_type, owner_id, name, meal_type, slots[{role, count}],
                  is_party, health_relaxed
WeekDefault       user_id, weekday(1-7), meal_type, template_id

HealthRuleConfig  owner_type, owner_id, rule_key, enabled, params(json)   -- system 为默认值
Avoidance         user_id, ingredient_id, reason(忌口/过敏)

MealPlan          id, user_id, week_start, status(draft/active)
PlanMeal          id, plan_id, date, meal_type, template_id, guests
PlanItem          id, plan_meal_id, slot_role, recipe_id, locked, warnings[]
ShoppingItem      id, plan_id, ingredient_id, amount, unit, checked
MealLog           id, user_id, date, meal_type, recipe_id                (P1)

ImportJob         id, user_id, source_type(text/url/image/name), source, status,
                  result_json, error
AiUsage           user_id, date, feature, count, tokens

Submission        id, recipe_id, submitter_id, status, reviewer_id, reject_reason  (P2)
Report            id, target_type, target_id, reporter_id, reason, status          (P2)
AuditLog          id, actor_id, action, target_type, target_id, diff_json
```

## 14. 接口概要

REST，JSON。App 接口前缀 `/api/v1`，后台接口前缀 `/api/admin/v1`（要求 admin 角色），公开分享接口前缀 `/api/public/v1`（无需登录）。

App 接口：

```
POST   /auth/login
POST   /auth/register                          (P1)

GET    /recipes?scope=all|mine|official&role=&meal_type=&q=&sort=health
POST   /recipes
GET    /recipes/{id}                           含用料、步骤、媒体
PUT    /recipes/{id}                           仅限自己的菜谱
DELETE /recipes/{id}
POST   /recipes/{id}/fork                      复制为我的版本
POST   /recipes/{id}/hide | DELETE /recipes/{id}/hide
POST   /recipes/{id}/share                     生成分享链接 (P2)
DELETE /recipes/{id}/share                     关闭分享 (P2)
POST   /recipes/{id}/submit                    投稿 (P2)
POST   /recipes/{id}/validate                  按 RECIPE.md 9.3 返回错误与警告
GET    /recipes/{id}/pref       PUT 同路径     口感选择、份数

POST   /recipes/import                         body: source_type, source → job_id
GET    /recipes/import/{job_id}                → status, draft（结构同菜谱详情）
POST   /recipes/{id}/ai-fill                   补全缺失字段，返回草稿 diff
POST   /recipes/{id}/ai-healthier              生成减油减糖版本草稿 (P1)
POST   /plans/parse-intent                     自然语言 → meal_overrides + 规则参数 (P1)

GET    /presets/actions | /presets/cut-styles | /glossary

GET    /ingredients?category=&q=               含官方 + 自己的，支持别名搜索
POST   /ingredients
GET    /inventory?location=&expiring_days=
POST   /inventory
PUT    /inventory/{id}
DELETE /inventory/{id}

POST   /media/upload-url                       body: type, content_type, size → 预签名 URL + media_id
POST   /media/{id}/complete

GET    /templates
GET    /settings/week-defaults      PUT 同路径
GET    /settings/health-rules       PUT 同路径
GET    /settings/avoidances         PUT 同路径

POST   /plans/generate                         body: week_start, meal_overrides[{date, meal_type, template_id, guests}], seed?
GET    /plans?week_start=
GET    /plans/{id}
POST   /plans/{id}/regenerate                  body: scope(week/meal/item), target_id
GET    /plans/{id}/candidates?item_id=         手动替换的候选列表，含是否违反规则
PUT    /plans/{id}/items/{item_id}             替换菜品或修改 locked
POST   /plans/{id}/activate

GET    /plans/{id}/shopping-list
PUT    /plans/{id}/shopping-list/{item_id}     勾选
```

后台接口：

```
GET/POST        /recipes                       官方菜谱
GET/PUT/DELETE  /recipes/{id}
POST            /recipes/{id}/publish
POST            /recipes/{id}/offline
GET             /recipes/{id}/revisions
POST            /recipes/{id}/revisions/{version}/restore

GET/POST        /ingredients
PUT/DELETE      /ingredients/{id}
POST            /ingredients/{id}/merge        body: source_ids[]
GET/POST/DELETE /ingredients/{id}/aliases

GET/POST/PUT    /templates, /tags, /cuisines
GET/POST/PUT    /action-presets, /cut-styles, /glossary
POST            /ai/batch-generate             body: names[] → 批量生成草稿
GET             /ai/usage
GET/PUT         /health-rules
GET             /media?unused=true
DELETE          /media/{id}
GET/PUT         /users, /users/{id}
GET             /stats/overview
GET             /submissions?status=           (P2)
POST            /submissions/{id}/approve | reject   (P2)
GET             /audit-logs
```

公开接口（P2）：

```
GET    /shared/recipes/{share_token}
```

生成接口返回示例：

```json
{
  "id": 12,
  "week_start": "2026-10-12",
  "rule_checks": [
    { "rule": "S1", "passed": false, "message": "本周鱼虾仅 1 次，建议至少 2 次" }
  ],
  "meals": [
    {
      "date": "2026-10-12",
      "meal_type": "dinner",
      "template": { "id": 2, "name": "标准" },
      "guests": null,
      "items": [
        {
          "id": 301,
          "slot_role": "meat",
          "locked": false,
          "warnings": [],
          "recipe": { "id": 8, "name": "清蒸鲈鱼", "cover_url": "https://…", "health_score": 4, "cook_minutes": 20, "owner_type": "system" }
        }
      ]
    }
  ]
}
```

## 15. 种子数据

由管理员通过后台或导入脚本维护的官方内容：
- 菜谱 80 道左右：晚饭各角色约 60 道，早餐各角色约 20 道；健康属性全部标注，油盐糖尽量填写数值；高油高糖菜控制在 10% 以内
- 食材 150 种左右，含分类、单位与换算系数、默认保质期、调料的盐糖折算系数
- 内置餐型模板（见 5.2）与健康规则默认配置（见第 7 节）

## 16. 非功能需求

- 生成一周清单的接口响应时间 < 1 秒（候选菜谱 1000 道以内）
- 生成算法支持传入随机种子，便于测试复现
- App 离线时可查看本周清单、购物清单和已打开过的菜谱（本地缓存），编辑需联网
- 密码哈希存储（argon2），access token 短期有效 + refresh token
- 权限：所有内容接口校验归属，用户只能修改自己的内容；后台接口校验 admin 角色
- 媒体上传限制类型与大小，预签名 URL 有效期 ≤ 15 分钟
- 后台操作写审计日志
- 多用户上线前补充：注册限流、数据导出、账号注销

## 17. 版本规划

| 版本 | 内容 |
|---|---|
| v0.1 MVP | 见 CORE.md 第 10 节的 P0 项：知识库、HowToCook 种子导入、可做度与库存优先生成、分享菜单导入、打卡扣库存、AI 用库存生成新菜；另含：单用户登录；菜谱（备料碗、详细烹饪步骤、口感分支、视频链接、油盐糖自动计算）；烹饪模式；AI 导入与补全；复制官方菜谱；食材与库存；生成一周清单与重新生成；购物清单；管理后台的官方菜谱、动作预设、食材、模板管理，AI 批量生成种子数据 |
| v0.2 | 自定义模板；忌口；临期提醒与库存优先；购物入库；用餐打卡；周报；术语表；语音朗读；健康改良；自然语言排菜；分享图片卡片；后台媒体库、版本回滚、食材合并、用户管理 |
| v0.3 | 多用户注册，开放给同事朋友使用；收藏与评分 |
| v0.4 | 分享链接与 H5 页；投稿与审核；管理员上传视频 |
| 以后 | 营养数据与热量统计；LLM 导入菜谱、新菜推荐；午餐；家庭成员共享清单 |

## 18. 待确定问题

1. 周起始日：周一还是周日
2. 早餐是否也区分工作日和周末模板
3. 种子菜谱偏向哪些菜系
4. 家庭共享是否需要多人同时编辑同一份清单
5. 对象存储与视频点播服务选哪家（影响上传和播放实现）
6. 管理后台的前端技术栈
7. AI 模型服务选哪家（需支持图片输入与结构化输出；国内公开上线需考虑合规）
