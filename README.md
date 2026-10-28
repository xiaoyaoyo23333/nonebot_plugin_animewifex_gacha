# nonebot_plugin_aniwifex_gacha

二次元老婆插件的 **Gacha 分支**，基于
[`nonebot_plugin_animewifex_lite`](https://github.com/xiaoyaoyo23333/nonebot_plugin_animewifex)
修改而来：在 Lite 版的「抽老婆 / 查老婆 / 换老婆」之上，新增一套
**「十连抽老婆」卡池玩法**（Pick Up 提升概率 + 120 抽保底）。

适合只需要「每天抽个老婆」并想要一点抽卡仪式感的群，功能比完整版少，不容易打架。

**作者**：xiaoyaoyo23333（移植）、MiniMax Code（AI 辅助开发与维护）、monbed（astrbot插件原作者）
**仓库**：https://github.com/xiaoyaoyo23333/nonebot_plugin_animewifex_gacha

## 与 Lite 版 / 完整版的关系

| | 完整版 | Lite 版 | Gacha 版（本插件） |
| --- | --- | --- | --- |
| 包名 | `nonebot_plugin_animewifex` | `nonebot_plugin_animewifex_lite` | `nonebot_plugin_aniwifex_gacha` |
| 配置前缀 | `ANIMEWIFEX_` | `ANIMEWIFEX_LITE_` | `ANIMEWIFEX_GACHA_` |
| 默认数据目录 | `data/animewifex` | `data/animewifex_lite` | `data/animewifex_gacha` |
| 指令数 | 13 | 4 | **8** |
| 十连抽卡池 | ❌ | ❌ | ✅ |

配置前缀与数据目录完全隔离，三个版本可以同时装在一个 NoneBot2 实例里而不互相污染数据。
但它们的**基础指令名完全相同**（`抽老婆` / `查老婆` / `换老婆`），默认会各答一遍——
同一实例里请**只启用其中一个**，或给其中一个设 `ANIMEWIFEX_GACHA_BLOCK_OTHER_PLUGINS=true` 让它独占。

> 从 Lite 版升级到本插件时，旧数据目录是 `data/animewifex_lite`，
> 本插件默认读 `data/animewifex_gacha`。想沿用旧数据，把旧目录整个拷过去即可；
> 或在 `.env` 里显式指定 `ANIMEWIFEX_GACHA_DATA_DIR=<旧目录>`。
> 另外 `.env` 里的 `ANIMEWIFEX_LITE_*` 仍会被兼容读取（新前缀优先），无需改名。

Lite 版相对完整版移除了：

- **命令**：`牛老婆`、`重置牛`、`切换ntr开关状态`、`重置换`、
  `交换老婆`、`同意交换`、`拒绝交换`、`查看交换请求`
- **配置**：`ntr_max`、`ntr_possibility`、`swap_max_per_day`、
  `reset_max_uses_per_day`、`reset_success_rate`、`reset_mute_duration`、`admins`
- **数据**：`records.json` 中的 `ntr` / `reset` / `swap` 分区、`ntr_status.json`、
  `swap_requests.json`

> 因此本插件**不会调用禁言 API**，也不需要配置管理员 QQ。

> 与**其它老婆插件**（如抽群友的 `today_waifu`）则可以共存，见下方
> 「与其它老婆插件共存」。

## 安装

```bash
nb-cli plugin install ./nonebot_plugin_aniwifex_gacha
```

或直接把整个目录复制到 `src/plugins/aniwifex_gacha/` 后启动 NoneBot2。

依赖：

- `aiohttp` —— NoneBot2 本身已依赖，通常无需处理。
- `tzdata` —— **Windows 上建议安装**（见下方「Windows 注意事项」）。不装也能运行，
  插件会自动回退到固定 UTC+8。安装方式：`pip install tzdata`。

## 已验证的运行环境

以下均为**实际跑通**的组合（集成测试 124 项 + 真实 OneBot V11 端到端 33 项）：

| Python | NoneBot2 | OneBot 适配器 | pydantic | 集成测试 | 端到端 |
| --- | --- | --- | --- | --- | --- |
| **3.11.9** | **2.4.2** | nonebot-adapter-onebot 2.4.3 | 2.14.1 | ✅ 124 | ✅ 33 |
| **3.11.9** | 2.4.2 | nonebot-adapter-onebot 2.4.3 | 1.10.26 | ✅ 124 | ✅ 33 |
| 3.11.9 | 2.5.0 | nonebot-adapter-onebot 2.4.6 | 2.14.1 | ✅ 124 | ✅ 33 |
| 3.12.10 | 2.4.2 | nonebot-adapter-onebot 2.4.3 | 2.14.1 | ✅ 124 | ✅ 33 |
| 3.12.10 | 2.4.2 | nonebot-adapter-onebot 2.4.3 | 1.10.26 | ✅ 124 | ✅ 33 |
| 3.12.10 | 2.5.0 | nonebot-adapter-onebot 2.4.6 | 2.14.1 | ✅ 124 | ✅ 33 |

> 端到端测试用**真实的 `nonebot.adapters.onebot.v11` 事件 / 消息段 / Bot** 驱动插件，
> 校验 `pick up → 十连 → 选图 → 发图` 全流程产出的确实是带文本段 + 图片段的 OneBot V11 消息。

- 源码语法经 `ast.parse(feature_version=...)` 校验，**Python 3.9 – 3.12 均可解析**
  （三个源文件都带 `from __future__ import annotations`）。
- 2.4.2 与 2.5.0 的 API 差异（`Event.get_session_type()` / `Event.get_user_name()` /
  `Event.get_self_id()` 在 2.5 被移除）全部走「能力探测 + 逐级回退」，不绑版本。
- `pydantic` v1 / v2 双兼容（`model_fields` / `__fields__`、`TypeAdapter` / `parse_obj_as`、
  `model_copy` / `copy`）。
- 2.4.2 与 2.5.0 的 `MessageSegment` 基类都是抽象类、都没有 `text()` / `image()` 工厂，
  本插件统一**从事件取适配器的消息段子类**再调用（OneBot V11/V12 均已提供），
  基类缺工厂时另有按标准字段直接构造的兜底分支。

## Windows 注意事项

1. **时区数据（tzdata）**：Windows 的 Python 默认不带 IANA 时区库，NoneBot2 也不依赖
   `tzdata`。未安装时插件不会崩溃，会退化为固定 UTC+8——默认时区本就是东八区，
   「今天」的判断依然正确；但若把 `ANIMEWIFEX_GACHA_TIMEZONE` 改成带夏令时的时区，
   请务必 `pip install tzdata`。也可用环境变量 `TZ=Asia/Shanghai` 指定。

2. **数据目录是相对路径**：默认的 `data/animewifex_gacha` 相对于 **NoneBot2 的启动目录（CWD）**。
   用计划任务 / NSSM / 快捷方式启动且 CWD 不是项目根目录时，数据会落到别处，
   建议显式指定：`ANIMEWIFEX_GACHA_DATA_DIR=D:/nonebot-bot/data/animewifex_gacha`。

3. **本地图库文件名**：支持中文与 `!`、`空格` 等字符（`作品名!角色名.jpg`）。

## 配置

在 `.env` 或系统环境变量中配置，前缀为 `ANIMEWIFEX_GACHA_`：

```dotenv
ANIMEWIFEX_GACHA_NEED_PREFIX=false          # 是否需要命令前缀触发
ANIMEWIFEX_GACHA_CHANGE_MAX_PER_DAY=3       # 每日换老婆次数
ANIMEWIFEX_GACHA_REPLY_DELAY_MIN=2          # 回复前的随机延迟下限（秒）
ANIMEWIFEX_GACHA_REPLY_DELAY_MAX=8          # 回复前的随机延迟上限（秒）；设 0 即关闭延迟
ANIMEWIFEX_GACHA_PICKUP_RATE=0.02           # 单抽命中 Pick Up 老婆的概率（2%）
ANIMEWIFEX_GACHA_PITY_LIMIT=120             # 累计多少次单抽必出 Pick Up
ANIMEWIFEX_GACHA_TEN_DRAW_COUNT=10          # 一次十连抽几张
# ANIMEWIFEX_GACHA_IMAGE_BASE_URL=          # 已停用：远程图床，插件只读本地图库
# ANIMEWIFEX_GACHA_IMAGE_LIST_URL=          # 已停用：远程图片列表
ANIMEWIFEX_GACHA_TIMEZONE=Asia/Shanghai     # 判定「今天」用的时区；留空依次回退 NoneBot2 timezone、TZ、Asia/Shanghai
ANIMEWIFEX_GACHA_COMMAND_PRIORITY=0         # 匹配器优先级，越小越先执行（见下方「与其它老婆插件共存」）
ANIMEWIFEX_GACHA_BLOCK_OTHER_PLUGINS=false  # true=独占本插件命令；false=其它老婆插件仍可各自回应
ANIMEWIFEX_GACHA_DATA_DIR=                  # 数据目录，留空则用 <data_dir>/animewifex_gacha
```

> 配置优先级：默认值 < NoneBot2 插件配置机制 < `.env` 的 `ANIMEWIFEX_GACHA_*`
> < 环境变量的 `ANIMEWIFEX_GACHA_*`。显式实现前缀读取是因为 NoneBot2 的
> `get_plugin_config` 按字段名直接读环境变量（如 `DATA_DIR`），既没有插件前缀，
> 也会被全局同名配置项干扰。
>
> 旧 Lite 版的 `ANIMEWIFEX_LITE_*` 仍会作为兜底被读取（新前缀优先），升级不用改 `.env`。

| 想要的效果 | 配置 |
| --- | --- |
| 让人多换几次 | `ANIMEWIFEX_GACHA_CHANGE_MAX_PER_DAY=10` |
| 几乎不让换 | `ANIMEWIFEX_GACHA_CHANGE_MAX_PER_DAY=0`（抽完就锁死） |
| 不想等待、恢复秒回 | `ANIMEWIFEX_GACHA_REPLY_DELAY_MAX=0` |
| 等待短一点 | `ANIMEWIFEX_GACHA_REPLY_DELAY_MIN=1` `ANIMEWIFEX_GACHA_REPLY_DELAY_MAX=3` |
| UP 更常见 | `ANIMEWIFEX_GACHA_PICKUP_RATE=0.05`（5%） |
| 保底更快 | `ANIMEWIFEX_GACHA_PITY_LIMIT=60` |
| 改成二十连 | `ANIMEWIFEX_GACHA_TEN_DRAW_COUNT=20` |
| 抽不到图时手动补图 | 往 `<数据目录>/img/wife/` 放 `作品名!角色名.jpg`，本地图库优先级最高 |

## 图片来源：本地图库

插件为**纯本地模式**，不请求任何远程图床：

- 图片只从 `<数据目录>/img/wife/` 读取
- `ANIMEWIFEX_GACHA_IMAGE_BASE_URL` / `ANIMEWIFEX_GACHA_IMAGE_LIST_URL` 已**停用**（代码中已注释），
  写在 `.env` 里也不会再生效
- 图库为空时会记录一条 warning 并回复「获取失败」，不会去联网兜底

**图库来源（或自备）：**
 本插件配套图库：<https://github.com/xiaoyaoyo23333/animewifex_repo_x-ver>

下载后放进 `<数据目录>/img/wife/` 即可。

放图方式（**统一按 `作品名!角色名.jpg` 命名**，`!` 前会被当作作品来源显示）：

```text
data/animewifex_gacha/img/wife/
├── 轻小说!明日香.jpg
├── 英雄联盟!阿卡丽.jpg
└── 约会大作战!五河琴里.png
```

> 命名规范是硬约定：Pick Up 的重名判定、十连结果的《作品名》展示都依赖它。
> 万一有文件漏了 `!`，插件**不会删你的图**，但会在扫描图库时打一条 warning 列出这些文件，
> 且这类图片的文案不显示作品来源——建议直接改名。

> 往图库里增删图片后无需重启插件：图库索引以目录 mtime 为缓存键，自动失效重建。

> **前缀触发**：把 `ANIMEWIFEX_GACHA_NEED_PREFIX` 设为 `true` 后，命令必须以
> `COMMAND_START` 中的前缀开头（如 `/抽老婆`）。若 `.env` 里的 `COMMAND_START`
> 含空字符串，前缀模式不会生效，请先去掉空字符串。

## 功能与用法

群聊每天重置的玩法，8 条命令，**仅在群聊生效**，私聊不触发。

### 指令匹配规则（严格匹配）

命令必须是**整条消息**（去掉 @ 等非文本段后）：

| 消息 | 是否响应 |
| --- | --- |
| `换老婆` / `抽老婆` / `老婆帮助` | ✅ |
| `十连抽老婆` | ✅ |
| `查老婆 明日香` / `查老婆 @某人` | ✅ |
| `pick up 五河琴里` / `pick up五河琴里` | ✅ |
| `更换pick up 五河琴里` / `更换pick up五河琴里` | ✅ |
| `帮我换老婆` / `换老婆啊` / `xxxx换老婆xxxx` / `抽老婆抽老婆` | ❌ 不响应 |
| `换老婆。` / `换老婆~` | ❌ 不响应 |

即：**只接受「命令」本身，或「命令 + 空格 + 参数」**。聊天里随口提到「换老婆」
不会再误触发，也不会和别的插件抢消息。

例外三处：

- `pick up` / `更换pick up` 的参数前**可以不带空格**（`pick up五河琴里`）；
- **纯数字**消息只有在「你当前有待选列表」时才会被接管（见下方十连玩法）；
- **「是」/「否」**只有在「你当前有待答问题」时才会被接管（见下方三套抽卡）。

> 开了 `ANIMEWIFEX_GACHA_NEED_PREFIX` 时，前缀会被先剥掉再按同样的规则匹配。

### 指令速查

| 指令 | 作用 | 限制 |
| --- | --- | --- |
| `老婆帮助` | 列出全部命令 | — |
| `抽老婆` | 抽今天的老婆 | 每天一次，当天不再变；与十连二选一 |
| `查老婆 [@用户]` / `查老婆 昵称` | 查自己或别人的老婆 | — |
| `换老婆` | 丢弃当前老婆重抽 | 每天 3 次，需已有老婆 |
| `pick up+角色名` | 指定 Pick Up 老婆 | 图库里要存在该老婆 |
| `更换pick up+角色名` | 更换 Pick Up! 老婆 | 同上 |
| `十连抽老婆` | 十连抽卡 | 每天一次；与单抽二选一 |
| 数字（1 / 2 / 3…） | 从候选列表 / 十连结果中选一个 | 需有待选列表 |
| 是 / 否 | 回答「是否把 Pick Up! 运用至单抽」 | 需有待答问题 |

### 回复形式：带用户名 vs 引用回复

| 回复内容 | 形式 |
| --- | --- |
| 带用户名的（今天的老婆、锁定提示、每天一次提示） | 直接发消息，如「小明，你今天的老婆是来自《约会大作战》的五河琴里（通过十连抽自选）」 |
| 不带用户名的（引导类、结果类） | **引用用户的原消息**发送，如 pick up 提示、十连结果列表、候选列表、帮助 |

> 引用回复走适配器自带的 `reply` 段：OneBot V11 的数据键是 `id`、V12 是 `message_id`，
> 插件统一调适配器工厂构造，不会写死字段名。
> 事件里取不到 `message_id`（部分适配器不提供）时自动退化成普通消息，不会报错。

### 拟人化延迟（不秒回）

机器人不会收到指令就秒回：**每一条回复**在发出前都会随机等 2～8 秒
（`ANIMEWIFEX_GACHA_REPLY_DELAY_MIN` / `ANIMEWIFEX_GACHA_REPLY_DELAY_MAX`），
模拟真人「看到消息 → 打字 → 发送」的过程。

- 区间内每次都是随机取值，节奏不会固定得像个机器人。
- 一条指令要回多条消息时（例如设置 Pick Up 后先回执再追问），**每条各自等一次**，
  总等待会久一点。
- 嫌慢就把上限设成 `0`（`ANIMEWIFEX_GACHA_REPLY_DELAY_MAX=0`）恢复秒回；
  只想短一点就把两个值一起调小。
- 延迟只影响「什么时候发」，不影响消息内容与抽卡结果。

### 抽老婆 / 查老婆

```
小明：抽老婆
Bot ：小明，你今天的老婆是来自《轻小说》的明日香，请好好珍惜哦~ [图片]
```

之后一整天再发 `抽老婆`，返回的是同一个人（幂等）。

```
小明：查老婆
Bot ：小明的老婆是来自《轻小说》的明日香，羡慕吗？ [图片]

小明：查老婆 @小红
Bot ：小红的老婆是来自《单作者》的凉宫，羡慕吗？ [图片]

小明：查老婆 凉宫          ← 不 @ 也可以，昵称匹配
Bot ：小明的老婆是来自《单作者》的凉宫，羡慕吗？ [图片]
```

> **图片名决定文案**：文件名按 `作品名!角色名.jpg` 命名，`!` 前当作作品来源；
> 没有 `!` 就只报角色名。

### 换老婆

```
小明：换老婆
Bot ：小明，你今天的老婆是来自《相合之物》的雪平一果，请好好珍惜哦~ [图片]
```

- 当天还没抽过老婆会被拦：`小明，你今天还没有老婆，先去抽一个再来换吧~`
- 用完次数：`小明，你今天已经换了3次老婆啦，明天再来吧~`
- 今天的老婆是十连抽到并锁定的：`小明，你不是抽到老婆了嘛~杂鱼杂鱼~`
- 抽取失败时老婆和次数都保持不动，不会白扣次数。
- `换老婆` 始终走**默认抽法**（全图库随机），不套 Pick Up、不计保底。

## 三套抽卡

插件里一共保留三套抽卡玩法，互不干扰：

| # | 玩法 | 触发 | 概率与保底 | 抽数记录 |
| --- | --- | --- | --- | --- |
| 1 | **默认抽老婆** | `抽老婆`（没设 Pick Up，或没同意运用到单抽） | 全图库等概率随机，无保底 | 不记 |
| 2 | **开启 Pick Up 的单抽** | `抽老婆`（已同意把 Pick Up 运用到单抽） | Pick Up 命中 2%，满 120 抽必出 | `single_count`（单抽专用） |
| 3 | **十连抽老婆** | `十连抽老婆` | 每抽 Pick Up 命中 2%，满 120 抽必出 | `count`（十连专用） |

**第 2、3 套的保底分开计数**：单抽的累计抽数不会推高十连的累计抽数，反之亦然。
两套的上限都跟 `ANIMEWIFEX_GACHA_PITY_LIMIT`（默认 120）走。

### 怎么开启第 2 套

`pick up+角色名` 设好 Pick Up 之后，插件会追问一句：

```
小明：pick up 明日香
Bot ：已将pick up!老婆设置成《轻小说》明日香
Bot ：可随时发送“更换pick up+老婆名”更换pick up! 老婆
Bot ：是否将pick up!的角色一并运用至单抽？回复「是」或「否」

小明：是
Bot ：好的，之后的「抽老婆」也会按 Pick Up! 概率出图啦（单抽的保底与十连分开计数）
```

- 回「是」→ 单抽改走 Pick Up 机制；回「否」→ 单抽维持默认抽法。
- 答案**绑定当前的 Pick Up 目标**：换了 Pick Up 会重新追问，旧的答案自动作废。
- 每次设置 / 更换 Pick Up 都会追问一次；没回答就按默认抽法走。

### 第 2 套的回复长什么样

单抽命中 Pick Up 时才会多一句提示。平时**不显示任何保底进度**（不像十连那样报「剩余 n 次必出 pick up」），
保底在后台默默计数，直到真的出了 Pick Up：

```
小明：抽老婆
Bot ：小明，你今天的老婆是来自《轻小说》的明日香（🌟pick up!角色），请好好珍惜哦~
Bot ：今天的老婆已锁定不可更换哦~ [图片]
```

和十连一样，命中 Pick Up 的单抽会**锁定**今天的老婆，之后 `换老婆` / `抽老婆` 都会被拒。

> 标记文案两套不同：单抽命中写「（🌟pick up!角色）」，十连结果列表里那一行写「🌟pick up!」。

> 单抽每天只生效一次（当天再发 `抽老婆` 返回同一个人），所以单抽的保底计数每天最多 +1；
> 120 抽的上限意味着最多攒 120 天。想缩短就调小 `ANIMEWIFEX_GACHA_PITY_LIMIT`。

### 一天只能二选一

同一天里 `抽老婆`（单抽）和 `十连抽老婆` 只能选一种，不能先单抽再十连、也不能先十连再单抽：

```
小明：抽老婆
Bot ：小明，你今天的老婆是来自《约会大作战》的五河琴里，请好好珍惜哦~ [图片]

小明：十连抽老婆
Bot ：小明，你今天已经抽过老婆了，十连明天再来吧~
```

```
小明：十连抽老婆
Bot ：[引用] 【十连抽老婆】结果：
Bot ：1.《约会大作战》五河琴里
      …

小明：抽老婆
Bot ：小明，你今天已经使用十连抽老婆了，今天不可使用单抽老婆哦~
```

- 单抽之后发十连 → 十连被拒，单抽抽到的老婆**不会被覆盖**，十连抽数也不增加。
- 十连之后发单抽 → 单抽被拒；十连已经选定老婆时，回复的是 `小明，你不是抽到老婆了嘛~杂鱼杂鱼~`。
- 被拒时不会消耗任何抽数；跨天后两种玩法重新可用。
- `换老婆` 不受这条限制，仍按「每天 3 次」的独立规则走。

## 十连抽老婆（卡池玩法）

### 规则速览

1. **先 pick up**：使用十连前必须先指定一个 Pick Up 老婆，它在十连的**每一抽**中概率提升为 **2%**，
   其余 98% 在图库其它老婆之间均分。
2. **每天一次十连**：一次十连 = 10 次独立单抽。抽取前可以随时更换 Pick Up
   （更换**不影响**累计抽数）。
3. **满 120 抽必出**：累计 119 抽都没出 Pick Up 时，第 120 抽必定是 Pick Up。
   抽到 Pick Up 的那一刻**累计抽数清零**，本次十连剩余抽数继续抽完并从 0 重新累计。
4. **出了 Pick Up**：今天的老婆**直接锁定**为该 Pick Up 卡，不可更换。
5. **没出 Pick Up**：10 张结果全部列出，用户回复**数字**从其中自选一张作为今天的老婆。
6. **今天的老婆锁定**：通过十连得到的老婆不可更换（`换老婆` / `抽老婆` 都会被拒，
   回复 `小明，你不是抽到老婆了嘛~杂鱼杂鱼~`）。
7. **单抽与十连每天二选一**：当天抽过老婆就不能再抽十连，当天用过十连也不能再单抽，
   两种玩法不会互相覆盖今天的老婆（详见上方「一天只能二选一」）。

### pick up 老婆

```
小明：pick up 五河琴里
Bot ：已将pick up!老婆设置成《约会大作战》五河琴里
      可随时发送“更换pick up+老婆名”更换pick up! 老婆
```

还没 pick up 就发十连：

```
小明：十连抽老婆
Bot ：你还未pick up!老婆，请使用“pick up+角色名”进行pick up!
```

图库里没有这个老婆：

```
小明：pick up 不存在的老婆
Bot ：图库暂无该老婆，请pick up!其他老婆
      可提交issue申请扩充：https://github.com/xiaoyaoyo23333/nonebot_plugin_animewifex_gacha/issues
```

**重名**时会列出候选让你选数字——同名不同作品、同名不同扩展名（`.jpg` / `.png`）都会触发：

```
小明：pick up 五河琴里
Bot ：你pick up! 的是?
      1.《约会大作战》五河琴里.jpg
      2.《约会大作战》五河琴里.png
      回复数字进行pick up!

小明：2
Bot ：已将pick up!老婆设置成《约会大作战》五河琴里
      可随时发送“更换pick up+老婆名”更换pick up! 老婆
```

> 角色名可以写成 `五河琴里`、`《约会大作战》五河琴里`、`约会大作战!五河琴里` 三种形式；
> 精确匹配不到时还会用「包含匹配」兜底（如输入 `明日` 命中 `明日香`）。

### 十连抽老婆

```
小明：十连抽老婆
Bot ：【十连抽老婆】结果：
      1.《约会大作战》五河琴里
      2.《轻小说》明日香
      3.《约会大作战》五河琴里
      4.《轻小说》明日香
      5.《轻小说》明日香
      6.《约会大作战》五河琴里
      7.《轻小说》明日香
      8.《轻小说》明日香
      9.《约会大作战》五河琴里
      10.《轻小说》明日香

      已完成10次抽，剩余110次必出pick up!
      已完成十连抽老婆，回复数字以选取今天的老婆

小明：3
Bot ：小明，你今天的老婆是来自《约会大作战》的五河琴里（通过十连抽自选） [图片]
```

**出了 Pick Up** 时（同一轮里命中即锁定，不需要自选）：

```
小明：十连抽老婆
Bot ：【十连抽老婆】结果：
      1.《约会大作战》五河琴里
      2.《轻小说》明日香
      3.《约会大作战》五河琴里
      4.《轻小说》明日香
      5.《轻小说》明日香
      6.《约会大作战》五河琴里 🌟pick up!
      7.《轻小说》明日香
      8.《轻小说》明日香
      9.《轻小说》明日香
      10.《轻小说》明日香

      已完成9次抽，剩余111次必出pick up!
      本次十连已出pick up!老婆，今日不可从结果中自选
      可发送“更换pick up+老婆名”更换pick up! 老婆
Bot ：小明，你今天的老婆是来自《约会大作战》的五河琴里（通过十连抽必出Pick Up!） [图片]
```

其它提示：

- 每天只有一次十连，再发会得到 `小明，你今天已经抽过一次十连了，明天再来吧~`；
- 待选列表跨天自动失效，第二天要重新抽；
- 换了 Pick Up 不会影响累计抽数，所以攒到 119 抽时可以放心换目标；
- 锁定文案会**标明来源**：从十连结果里自选的带「（通过十连抽自选）」，
  同一轮直接命中 Pick Up 的带「（通过十连抽必出Pick Up!）」。

## 与其它老婆插件共存

完整版、或 `nonebot_plugin_today_waifu` 之类的「今日老婆」插件，往往也注册了
`抽老婆` / `换老婆`，而 `on_message` 默认 `block=True`——**谁先匹配到，事件就不再往下传**，
排在后面的插件会「静默无响应」，日志里只能看到
`Event will be handled by Matcher(... module=别的插件 ...)`。

本插件的做法是**只认自己的命令，答完放行**：

| 配置项 | 默认 | 作用 |
| --- | --- | --- |
| `ANIMEWIFEX_GACHA_COMMAND_PRIORITY` | `0` | 排在通用 `on_message` 插件之前，避免被对方 `block` 吞掉 |
| `ANIMEWIFEX_GACHA_BLOCK_OTHER_PLUGINS` | `false` | 答完后**放行**，对方插件仍可对同一条指令各自回应 |
| 匹配器 `rule` | — | 仅消息以本插件命令开头时触发，不响应普通聊天 |

默认状态可与其它老婆插件共存，各答各的；想让二次元老婆独占，把
`ANIMEWIFEX_GACHA_BLOCK_OTHER_PLUGINS` 设为 `true` 即可。

> ⚠️ 别把优先级调大：对方若是 `block=True` 且优先级更低，你就收不到事件了。
> 配置在插件加载时读取，**修改后需重启 NoneBot2**。

## 常见排查

| 现象 | 处理 |
| --- | --- |
| 插件加载报错 / 时区相关异常 | `pip install tzdata`（Windows 上必需） |
| 数据好像清零了 | 检查 NoneBot2 的启动 CWD，建议 `ANIMEWIFEX_GACHA_DATA_DIR` 写绝对路径；从 Lite 版升级的话记得把旧 `data/animewifex_lite` 拷到 `data/animewifex_gacha` |
| 抽不出图 | 看日志有无拉取失败；先往 `img/wife/` 放两张本地图验证链路 |
| 指令没反应 | 必须在群里发（私聊不触发）；若开了 `NEED_PREFIX`，得用 `/抽老婆` |
| 发了 `pick up 五河琴里` 提示图库暂无 | 图库里没有这张图；文件名按 `作品名!角色名.jpg` 放好即可 |
| 数字回复没反应 | 只有在有待选列表（Pick Up 候选 / 十连结果）时才会被接管；列表跨天失效 |
| 某条命令单独没反应、日志显示 `module=` 别的插件 | 优先级被对方 `block` 抢走，保持 `ANIMEWIFEX_GACHA_COMMAND_PRIORITY=0`（默认）即可 |
| 同一个指令被回了两次 | 想独占就设 `ANIMEWIFEX_GACHA_BLOCK_OTHER_PLUGINS=true`；想共存保持默认 |
| 完整版 / Lite 版和本插件同时启用 | **基础指令名相同，必须只启用一个**，否则会各答一遍 |

## 数据存储

数据目录默认为 `<NoneBot2 data_dir>/animewifex_gacha`（全局 `data_dir` 未配置时为
`<项目目录>/data/animewifex_gacha`），可用 `ANIMEWIFEX_GACHA_DATA_DIR` 覆盖：

```text
animewifex_gacha/
├── config/
│   ├── <群号>.json          # 该群成员的老婆记录
│   ├── gacha_<群号>.json     # 该群的十连抽状态（Pick Up / 累计抽数 / 待选列表）
│   └── records.json         # 每日换老婆次数记录（仅 change 一个分区）
└── img/wife/                # 本地图库（可手动放图）
```

`gacha_<群号>.json` 单条状态的结构：

```json
{
  "u12345": {
    "pickup": "约会大作战!五河琴里.jpg",
    "count": 37,
    "ten_date": "2026-10-22",
    "pending": {
      "kind": "gacha",
      "date": "2026-10-22",
      "options": ["轻小说!明日香.jpg", "..."]
    }
  }
}
```

- `pickup`：当前 Pick Up 老婆（图库文件名）
- `count`：十连抽距上次出 Pick Up 的累计抽数（出 UP 即清零）
- `ten_date`：最近一次十连的日期，用于「每天一次」限制，也是「单抽与十连二选一」的判定依据
- `single_count`：单抽自己的累计抽数（与十连分开计，出 UP 即清零）
- `single_pickup`：是否把 Pick Up 运用到单抽（回答过「是」为 true）
- `single_target`：`single_pickup` 这份答案绑定的 Pick Up 目标；与当前 `pickup` 不一致时答案作废
- `ask`：待回答的「是否运用至单抽」问题（跨天失效）
- `pending`：待用户回复数字选择的列表（`gacha`=十连结果 / `pickup`=Pick Up 候选），跨天失效

JSON 均为原子写入（临时文件 + 替换），并对文件被外部改坏的情况做了降级处理；
每天零点后自动清理过期的次数记录（`records.json` 里的换老婆次数）。

## 兼容性

插件通过「能力探测 + 逐级回退」的方式编写，不绑定任何特定 NoneBot2 / 适配器版本：

| 能力 | 处理方式 |
| --- | --- |
| bot / event 获取 | 一律用依赖注入（`bot: Bot, event: Event`），不碰 `matcher.bot` / `matcher.event`——后者在 2.4+ 已不存在 |
| 机器人自身账号 | `bot.self_id`，不依赖 `Event.get_self_id()`（2.4+ 已移除） |
| 群聊判定 | `get_session_type()` → 事件 `message_type` 字段 → `group_id` 属性 |
| 发送者昵称 | `get_user_name()` → `sender.card` / `sender.nickname` → `member.card` / `user.display_name` |
| 消息段构造 | 从事件取适配器的消息段类再调其 `text()` / `at()` / `image()`，缺失则按标准字段直接构造 |
| @ 目标解析 | 按 `segment.type == "at"` + `data["qq"]` 解析，不使用适配器特有的 `is_at()` |
| pydantic | 同时兼容 v1（`__fields__` / `parse_obj_as` / `.copy()`）与 v2（`model_fields` / `TypeAdapter` / `.model_copy()`） |

## 与 AstrBot 版的差异

- 事件监听由 `@filter.event_message_type(GROUP_MESSAGE)` 改为带 `rule` 的 `on_message`：
  只在消息以本插件命令开头时触发，答完默认放行（`block=False`），可与其它老婆插件共存，
  见上方「与其它老婆插件共存」。
- 「触发前缀」由 NoneBot2 的 `COMMAND_START` 实现。
- 时区改读 `ANIMEWIFEX_GACHA_TIMEZONE`（回退到 NoneBot2 的 `timezone`、`TZ` 环境变量，
  最后回退 `Asia/Shanghai`）。
- 牛老婆、交换、重置相关的一切（命令、配置、数据、昵称匹配、禁言调用）已从 Lite 版移除。
- 新增「十连抽老婆」卡池玩法（Pick Up 提升概率、120 抽保底、每日一次、结果自选）。
- 单抽与十连每天二选一：当天用过一种就不能再用另一种，两种玩法不会互相覆盖今天的老婆。
- 新增拟人化回复延迟：每条回复发出前随机等 2-8 秒，不再秒回（`ANIMEWIFEX_GACHA_REPLY_DELAY_MIN` / `_MAX`，上限设 0 可关闭）。
