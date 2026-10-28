# 登录与连接排障（本地化版）

登录/会话/域名/网络出错时看这里，末节含下载故障与中断清理。用退出码和 CLI 报错判断；密码和 cookie 不进工具输出。

## 会话缺失或过期（退出码 3）

```sh
zlib doctor --json    # 不需登录；JSON 里 status:"healthy" = 可用；[] = 此网络无可用域名
```

把可用域名给用户，让他在**自己的终端**完成交互式登录：

```sh
zlib login --domain https://<doctor 返回的域名>
```

登录必须用户自己做，我不代跑、不收密码。CLI 会存 session 和域名到
`~/.config/zlib/`（Windows: `C:/Users/Administrator/.config/zlib/`）。
用户登录完成后用 `zlib profile --json`（退出码 0）确认，恢复主流程。

### `Authorization failed`（密码被拒但会话可能有效）

部分账号 EAPI 登录会报这个，但浏览器已有会话仍可用。让用户：
1. 浏览器登录 Z-Library，导出 Netscape 格式 cookie 文件（如 `cookies.txt`）
2. 自己终端跑（域名必须与 cookie 匹配）：
   ```sh
   zlib login --cookies ./cookies.txt --domain https://<域名>
   ```
cookie 文件留在用户机器上，内容不进对话。真报 `Incorrect email or password`
才让用户在交互提示里重输；不要挨个域名重试登录。

## 域名或网络故障（退出码 4）

```sh
zlib doctor --json
zlib doctor --proxy http://127.0.0.1:7897 --json   # 本机 Clash 端口 7897（官方文档示例是 7890，别照抄）
```

代理测试通过 → 让用户在全局 `C:/Users/Administrator/.config/zlib/.env` 加一行持久生效：

```dotenv
ZLIB_PROXY=http://127.0.0.1:7897
```

（只读/编辑该文件指定的一行，不 dump 整个文件，不动其他设置。）

不因"另一个域名可用"就换掉当前配置域名；不代跑 `zlib mirrors add` 或 `--yes`
（信任新镜像 = 把登录 cookie 交给对方，必须用户自己确认）。自建镜像要先
`zlib mirrors add <域名>` 信任后才能用；`zlib mirrors list --json` 查已信任列表。

配置优先级：进程环境变量 → 工作目录 `.env` → `~/.config/zlib/.env`。
登录后仍报错 = 高优先级里有过期的 `ZLIB_DOMAIN`，按此顺序查并让用户改。

## 命令或 --json 标志不存在

`zlib version` + `zlib <command> --help` 看版本；旧版要 `zlib update`
（或让用户 `winget upgrade --id heartleo.zlib --exact`）。不回落到 HTML 抓取、
裸发认证请求或交互式命令。

## 下载故障

- 用当前搜索/历史结果里完整的 `id:hash`（旧短 ID 无效）
- `zlib profile` 看剩余配额；目标目录要存在且可写；**配额按请求计数，失败的下载也扣**——先把 `--dir` 路径（Windows 风格 `C:/...`，非 `~/...`）和 id 核对再执行
- 502/503/504 和断连会自动重试 `ZLIB_DOWNLOAD_RETRIES` 次（默认 3），持续失败先查域名/网络

### 取消或中断的下载清理

超时/取消后 `.part` 可能被还活着的 zlib 进程占着（`rm` 报 Device or resource busy），按序处理：

```sh
ps -W | grep -i zlib              # 第 4 列 WINPID 是 Windows 进程号
taskkill /F /PID <WINPID>         # 必须裸斜杠：bash kill 认不到 Windows pid 空间，//F 也不会被转换（本机 MSYS 路径转换关闭）
rm -f "C:/Users/Administrator/Downloads/zlib-books/.zlib-"*.part
```

被杀的未完成下载按经验不额外扣配额，但重下前重查 `profile` 确认。
