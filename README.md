# craft-update

给 macOS 上的 [PdfCraft](https://github.com/storytold/pdfcraft)、[PhotoCraft](https://github.com/storytold/photocraft) 和 [WordCraft](https://github.com/storytold/wordcraft) 用的自动更新器。

要加 storytold 的其他 app，在脚本开头的 `APPS` 里加一行 `名字|仓库|资源前缀|bundle id` 即可（前提是同样发布 `<前缀>-X.Y.Z-macos-universal.dmg`、由同一个 Team ID 签名）。

上游只提供 Help ▸ Check for updates（打开下载页），明确不做自动安装（见 [storytold/pdfcraft#28](https://github.com/storytold/pdfcraft/issues/28)）。这个脚本在 app 外面补上这一步：

- 登录时和之后每小时检查一次 GitHub Release，只认 `<名字>-X.Y.Z-macos-universal.dmg`
- 下载后核对 SHA-256、Developer ID 签名（锁定 Team ID `DJ6XS33FX8`）和苹果公证，任何一项不对就放弃
- app 开着时只暂存，退出后下一次运行再安装
- 保留上一个版本，可以回滚；回滚的版本会被跳过，等更新的版本出来再升级

只依赖系统自带的 `bash`、`curl`、`jq`、`codesign`、`spctl`、`diskutil`。

## 文件

| 文件 | 装到哪里 |
| --- | --- |
| `craft-update` | `~/.local/bin/craft-update`（本机是指向本仓库的符号链接） |
| `com.local.craft-update.plist` | `~/Library/LaunchAgents/` |

运行时会生成：

- 日志：`~/Library/Logs/craft-update.log`
- 暂存和备份：`~/Library/Application Support/craft-update/`

## 安装 / 恢复

plist 里写的是 `/Users/gauss` 的路径，换了用户名的话要先改。

```bash
git clone https://github.com/wanzhenj1118/craft-update.git ~/craft-update
```

```bash
ln -sf ~/craft-update/craft-update ~/.local/bin/craft-update
```

```bash
cp ~/craft-update/com.local.craft-update.plist ~/Library/LaunchAgents/
```

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.local.craft-update.plist
```

## 用法

```bash
craft-update status            # 已装 / 暂存 / 备份的版本和最近日志
craft-update check             # 只查有没有新版本，不下载
craft-update run               # 立即检查并更新（后台任务跑的就是这个）
craft-update rollback PdfCraft # 退回上一个版本
```

## 停用

```bash
launchctl bootout gui/$(id -u)/com.local.craft-update
```

要彻底删掉的话，再删除上面列出的 plist、符号链接和两个运行时目录。
