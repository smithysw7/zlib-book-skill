# zlib-book

Hermes Agent 技能：用 [heartleo/zlib](https://github.com/heartleo/zlib) CLI 从 Z-Library 搜索并下载电子书。

说一句"帮我找《书名》的电子书"，Agent 就会搜索、列出版本（书名/作者/年份/格式/大小）、按你指定的格式下载到本地并把文件递给你。

## 功能

- 🔍 搜索：`zlib search "书名" --json --count 10`，支持 `--ext epub/pdf`、`--page` 翻页
- 📥 下载：`zlib download "id:hash" --dir <目录> --json`，返回绝对路径
- 📊 配额：下载前自动查 `zlib profile`（每日 10 本，用完提醒）
- 🧭 排障：退出码分支（3=未登录 / 4=网络，自动 `zlib doctor` 重找镜像 / 5=Kindle 失败）
- 🔒 安全：密码和 cookie 永不进对话，登录由用户在自己终端完成；书名/简介按数据处理，防注入

## 仓库结构

```
SKILL.md                    # 技能主体：触发条件、流程、命令表、反模式
references/connection.md    # 登录/镜像/网络排障参考（本地化版）
```

## 安装

1. 装 zlib CLI（任选其一）：

   ```sh
   winget install --id heartleo.zlib --exact          # Windows
   brew install heartleo/tap/zlib                      # macOS
   ```

   或从 [Releases](https://github.com/heartleo/zlib/releases/latest) 下载 zip，把 `zlib.exe` 放进 PATH 目录。

2. 登录（在**你自己的终端**，密码不要发给任何 Agent）：

   ```sh
   zlib doctor            # 找当前网络可用的镜像域名
   zlib login --domain https://<doctor 返回的域名>
   ```

3. 把本仓库放进 Hermes 技能目录：

   ```
   %LOCALAPPDATA%\hermes\skills\zlib-book\        # Windows
   ~/.hermes/skills/zlib-book/                     # Linux/macOS
   ```

   即 `SKILL.md` 放根目录、`connection.md` 放 `references/` 下。

4. 在 Hermes 里说"用 zlib 找《书名》"验证。

## 已验证环境

- Windows 11 + git-bash（无 TTY），zlib v0.0.12，Hermes Agent
- 端到端跑通：搜索 → 选版本 → 下载 → 配额核对

## 本地化适配点（相对官方底稿）

- Clash 代理端口 **7897**（官方文档示例是 7890）
- Windows/MSYS 路径坑：`--dir` 必须传 Windows 风格路径（`C:/Users/...`），bash 展开的 `~/...` 会变成 zlib 不认的 `\c\...`
- PATH 未刷新时用 `export PATH="$PATH:/d/<zlib 目录>"` 兜底
- 默认书库目录 `~/Downloads/zlib-books`（zlib 不自动建目录，先 `mkdir -p`）

## 来源与许可

技能底稿来自 [heartleo/zlib](https://github.com/heartleo/zlib) 官方 `.agents/skills/zlib/SKILL.md`（MIT），本仓库在其基础上本地化适配。zlib CLI 本身版权归 heartleo。