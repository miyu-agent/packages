<p align="center">
  <a href="README.md">English</a> | 简体中文
</p>

# Miyu 软件源

> <sub>本文档由 Anthropic 的 Claude Fable 5.2 生成。</sub>

这是 Miyu 的官方软件源：索引发布在 <https://miyu-agent.github.io/packages/>，软件包文件放在各个软件包自己仓库的发行版里。`miyu pkg update`、`miyu pkg search`、`miyu pkg install <编号>` 使用的就是这个软件源。

## 这个仓库里有什么

下表列出这个仓库里的文件和它们的作用：

| 文件 | 作用 |
|---|---|
| `sources.toml` | 按哪几种来源收录：官方的仓库，以及加了 `miyu-package` 话题的社区仓库 |
| `miyu-rev` | 收录和打包模板使用核心仓库的哪一个版本读取清单：内容是一行提交号。升级时只需要修改这一个文件，收录和打包模板都会跟着更新 |
| `official.toml` | 官方的仓库，每个仓库收录最新的发行版里的 `.miyu` 文件 |
| `names.toml` | 编号到来源仓库的对应关系：先到先得，编号由第一个被收录的仓库占用，以后永远属于这个仓库。新占用的编号由收录过程写回这个文件 |
| `blocked.toml` | 被拉黑的编号或者仓库，以及拉黑的理由 |
| `.github/workflows/collect.yml` | 每天收录一次，给索引签名，然后发布到 GitHub Pages。修改了规则或者手动触发时也会运行。它还会统计每个软件包所有发行版的下载次数，把最近 8 天的数据记在 GitHub Pages 上的 `stats.json` 里（不签名），并在索引里给每个软件包写上上线时间、下载次数和最近 7 天的下载次数 |
| `.github/workflows/pack.yml` | 给软件包作者使用的打包模板 |
| `.github/workflows/pack-selftest.yml` | 打包模板的自测：修改了模板或者 `miyu-rev` 时，在三种系统上把 `tests/hello` 打包一遍，不上传到发行版 |

索引用 Ed25519 签名，公钥编译在 `miyu` 程序里（编号 `f9234109a4740be9`）。如果索引的签名验证失败，`miyu` 会拒绝整份索引。

## 发布一个软件包

在软件包自己的仓库里添加 `.github/workflows/release.yml`：

```yaml
name: release
on:
  push:
    tags: ["v*"]
jobs:
  pack:
    uses: miyu-agent/packages/.github/workflows/pack.yml@main
    permissions:
      contents: write
    with:
      package: <包编号>
```

打标签（`v` 加上 `package.toml` 里的 `version`）以后，打包模板会按 Miyu 安装软件包时的规则检查清单，打包成 `.miyu` 文件，并上传到这个标签的发行版。

- 如果软件包文件夹不在仓库根目录：填写 `folder`。
- 如果打包之前需要先编译程序：填写 `build`，内容是一段 bash 脚本。
- 如果清单里写了 `targets`：每种系统打一个软件包文件，用 `strategy.matrix` 分别填写 `runs-on` 和 `target`。
- 打包使用的核心版本：没有填写 `miyu-rev` 时，使用本仓库根目录的 `miyu-rev`，和收录使用同一个版本，所以不会出现能打包却无法收录（或者反过来）的情况。只有在需要试用这个版本还不支持的新写法时，才填写 `miyu-rev`。

社区软件包会被自动收录：给仓库加上 `miyu-package` 话题，并在最新的发行版里放上 `.miyu` 文件，每天的收录运行时就会把它收进来，标为社区软件包。编号先到先得，作恶的编号或仓库会被写进 `blocked.toml`。进入官方列表（`official.toml`）需要项目主人同意。
