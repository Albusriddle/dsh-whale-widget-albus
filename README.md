# DSH 会话 Token 计数小鲸鱼挂件（dsh-whale-widget-albus）

![DSH 小鲸鱼挂件](assets/DSH2.png)

DeepSeek Harness（DSH）Web 界面右下角的常驻小鲸鱼挂件：小鲸鱼气泡图 + **会话 Token 计数**（本会话 / 全部会话总量，含输入/输出/缓存/推理明细）+ 每轮对话消耗统计 + 随机台词与音效，随界面每次打开自动启用。本项目是标准 DSH 插件包，可通过 `dsh plugin` 安装/卸载。

> 本项目是 [MeteorNOX/DeepSeek-Balance-Whale-Widget](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget)（余额挂件 v0.2.10）的**本地改造分支**：核心功能由「DeepSeek 余额（¥）」改造为「会话 Token 计数」，其余交互、拖拽、音效、随机台词机制沿袭原版。版权与许可见 [LICENSE](./LICENSE)（MIT），致谢原作者。

## 与上游（余额版）的差异

| 维度 | 上游 v0.2.10（DeepSeek-Balance-Whale-Widget） | 本分支 v0.3.0（dsh-whale-widget-albus） |
| --- | --- | --- |
| 显示内容 | 账户余额（¥）+ 今日已用金额 | 会话 Token 总量 + 入/出/缓/推 明细 |
| 数据源 | `/dsh-whale/balance.json`（余额 API，需 `DEEPSEEK_API_KEY`） | `/dsh-whale/session-tokens.json`（宿主侧读取会话存档并实时累加，无需令牌） |
| 范围切换 | 记账 / 实时·令牌 两种「今日已用」模式 | **当前会话 / 全部会话** 两种 Token 范围（记忆上次选择） |
| 每轮消耗气泡 | 本轮消耗金额 ¥ | 本轮消耗 Tokens |
| 随机台词 | 含峰谷提示/今日已用等 | 新增雌小鬼组、暧昧组、Token 梗组、彩蛋组 |
| 点击行为 | 点击一次展示/关闭气泡 | 奇数点击展示 Token 明细，偶数点击展示随机俏皮话 |
| 刷新间隔 | 60s | 5s |
| 会话存档解析 | 无 | `zstd` 多帧扫描 + `session.jsonl.zstd` 历史种子 |

## 特性

- 🐋 **常驻自启**：随 DSH Web 界面每次打开自动出现（标准 DSH bundle 插件）
- 🔢 **Token 计数**：5 秒自动刷新 + 点击鲸鱼手动刷新；显示**当前会话**或**全部会话**的 Token 总量，气泡内给出 入/出/缓/推 明细（<1万 千分位、<100万 显示 K、其余显示 M）
- 💬 **每轮对话消耗统计**：监听本机会话事件流，每轮对话结束后弹出本轮消耗 Tokens（精确 usage，非估算）；菜单可开关并自定义自动关闭秒数
- 🧭 **范围切换**：菜单「范围」可选 当前会话 / 全部会话，选择记忆在本地（`dshw-scope`）
- 🖱️ **拖拽 + 四边四分之一吸附**（左/右/上/下，角落可组合）
- 🔄 左吸附时整体**水平镜像翻转**（文字同步反向、带动画）
- 🧸 **按压 Q 弹**玩偶效果（按压时底部坐标不变）
- 🎚️ **汉堡菜单**：大小滑块（0.6–2.5 倍）、音效切换、音量调节、Token 范围、气泡开关、每轮消耗开关与自动关闭时间
- 🔊 **音效**：按压/松手音效（包内 mp3，缺失时静默降级）
- 💬 **随机台词**：点击气泡切换随机台词段（加权随机，含 Token 梗/雌小鬼吐槽/暧昧/彩蛋/gif），再点一次关闭；气泡总显示 5 秒自动收起
- 📐 随浏览器窗口自动缩放；文字位置/字号与图片联动

## 工作原理（Token 计数）

宿主侧插件（`lib/index.js`）在 `apply(ctx)` 内维护一个会话级 `TOKEN_STORE`：

1. **启动种子**：扫描 `~/.dsh/sessions/*/<sessionId>/session.jsonl.zstd`，用 `node:zlib` 的 `zstdDecompressSync` 按多帧容器格式解压，累加 `usage` chunk 的 input/output/cache 作为历史基数（避免与实时事件重复计数）。
2. **实时累加**：监听会话事件流中的 `usage` 数据，按 `(sessionId)` 累加 input/output/cache/reasoning。
3. **服务端路由**：注册 `/dsh-whale/session-tokens.json?scope=current|global&sessionId=...`，返回当前会话或全部会话的总量。
4. **前端渲染**：每 5s 轮询，token 变化时数字滚动动画。

## 目录结构

```text
dsh-whale-widget-albus/
├── package.json          # DSH bundle 插件元数据
├── README.md             # 本文件
├── LICENSE               # MIT（Copyright (c) 2026 MeteorNOX，本项目为改造分支）
├── cordis.patch.yml      # 插件挂载声明
├── lib/
│   └── index.js          # 宿主侧插件本体（含 Token 计数改造）
├── assets/
│   ├── DSH2.png          # README 顶部展示图
│   ├── DSniang1.png      # 小鲸鱼本体（cut-out，气泡由代码绘制）
│   ├── DSniang02.png     # 备用整图（兼容旧版手动安装路径）
│   ├── rua.gif           # 随机台词 gif（可选）
│   ├── Ya1.mp3 / Ya2.mp3 # 小黄鸭音效（可选）
│   └── D1.mp3 / D2.mp3   # 音效1（可选）
```

## 安装

### 方式 A：直接从 GitHub 安装

```powershell
dsh plugin --profile web add github:Albusriddle/dsh-whale-widget-albus
```

### 方式 B：本地安装（从当前仓库）

在**仓库根目录**（`package.json` 所在目录）执行：

```powershell
dsh plugin --profile web add link:.
```

- 安装完成后重启 `dsh web`，再 F5 刷新浏览器
- 若提示已存在/冲突，先 `dsh plugin --profile web remove dsh-whale-widget-albus` 再重新 add

## 验证

- `dsh --profile web --dump-config` 应能看到 `dsh-whale-widget-albus` 在 bundles 里
- `curl http://127.0.0.1:3080/dsh-whale/session-tokens.json` 应返回 200 JSON（含 `total/input/output/cache/reasoning`）

## 致谢

- [MeteorNOX/DeepSeek-Balance-Whale-Widget](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget)：本项目的上游与 UI/交互基础。
- DeepSeek Harness（DSH）社区。

## 许可

[MIT](./LICENSE)
