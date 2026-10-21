# 安装、更新与迁移

## 1. 选择一种来源

推荐通过本仓库的插件市场安装。宿主将一个包中的三个技能一起发现，不需要再把它们复制到全局技能目录。仓库名是 `embedded-mcu-dev`，插件名是 `embedded-mcu-development`，实际入口带插件命名空间。

本文命令按 Codex CLI 0.154.0 的本机帮助核对。先运行 `codex --version` 和 `codex plugin --help`；没有对应命令时，应使用当前宿主支持的插件安装入口，不要直接改写插件缓存。打包结构参考 [OpenAI 插件打包文档](https://developers.openai.com/plugins/build/plugins)，桌面连接说明见[官方连接与测试文档](https://developers.openai.com/plugins/deploy/connect-chatgpt)。

## 2. 全新安装

```sh
codex plugin marketplace add https://github.com/tlcrgbe/embedded-mcu-dev.git
codex plugin add embedded-mcu-development@embedded-mcu-dev
codex plugin list --marketplace embedded-mcu-dev --json
```

也可先克隆再安装，便于阅读和修改。以下命令应在准备存放仓库的位置执行：

```sh
git clone https://github.com/tlcrgbe/embedded-mcu-dev.git
codex plugin marketplace add ./embedded-mcu-dev
codex plugin add embedded-mcu-development@embedded-mcu-dev
```

远程和本地方式二选一，不要重复注册同名市场。插件安装后新开聊天，确认三个入口已出现，再显式指定一个入口试用。磁盘存在、插件已安装、聊天已发现、实际读取并应用是不同检查项。

## 3. 已有安装如何迁移

先运行 `codex plugin list --json`，核对已启用来源，并检查自己维护的独立全局技能目录。

| 当前情况 | 处理方式 |
| --- | --- |
| 已安装 `embedded-mcu-development@embedded-mcu-local` | 可以继续使用。要切换本仓库来源时，先备份原配置与自定义内容，再移除旧包、安装新包 |
| 全局目录中已有独立 `embedded-mcu-code` 或 `embedded-mcu-design` | 备份后将旧目录移到技能发现范围之外，再安装包；不要只改文件夹名后留在扫描根目录 |
| 已有其他来源安装的同名包 | 核对内容和定制差异，选定一种来源；不要覆盖未知自定义内容 |

旧本机包切换命令如下，仅在已确认该来源确实是要替换的旧包时执行：

```sh
codex plugin remove embedded-mcu-development@embedded-mcu-local
codex plugin marketplace add https://github.com/tlcrgbe/embedded-mcu-dev.git
codex plugin add embedded-mcu-development@embedded-mcu-dev
```

若 AGENTS.md 中仍写有旧独立技能的绝对路径，将其改为包内完整名称，并从宿主发现结果获取实际安装路径。例如编码规则引用 `embedded-mcu-development:embedded-mcu-code`。不要把本机缓存版本号写死在项目约定里。

新开聊天后应只有三种有效入口；若发现同一技能多次出现，先处理来源重复，再判断技能是否生效。此次仓库发布不自动把已有本机安装切换到新市场。

## 4. 更新与卸载

更新前查看 [CHANGELOG](../CHANGELOG.md)，保留本地定制与当前可用版本。远程市场的更新命令先用 `codex plugin marketplace --help` 核对；本地克隆可用 `git pull --ff-only` 更新源文件。市场源更新不代表已安装缓存自动变化，应使用宿主支持的更新方式；没有更新入口时，备份后移除再重新添加插件，并重新检查发现结果。

卸载本仓库来源：

```sh
codex plugin remove embedded-mcu-development@embedded-mcu-dev
```

恢复旧独立技能时，先卸载或停用包，再恢复备份目录及对应引用，避免重新引入重复来源。
