# Miyu 软件源

[English](README.en.md)

Miyu 的官方软件源：索引发在 <https://miyu-agent.github.io/packages/>，包文件在各包自己仓库的 Release 里。`miyu pkg update`、`miyu pkg search`、`miyu pkg install <编号>` 用的就是这里。

## 这个仓库里有什么

| 文件 | 做什么 |
|---|---|
| `sources.toml` | 照哪几种来源收。现在只收官方的仓库 |
| `miyu-rev` | 收录和打包模板用核心仓库的哪一版读清单：一行提交号。升级只改这一个文件，两边跟着走 |
| `official.toml` | 官方的仓库，各取最新的 Release 里的 `.miyu` |
| `names.toml` | 编号到来源仓库：先到先得，第一个收录的仓库占住，永远跟着它。新占的由收录写回来 |
| `blocked.toml` | 拉黑的编号或仓库，和理由 |
| `.github/workflows/collect.yml` | 每天收一次，签名，发到 Pages。改了规则、手动触发也跑 |
| `.github/workflows/pack.yml` | 给包作者用的打包模板 |

索引用 Ed25519 签名，公钥编在 `miyu` 里（编号 `f9234109a4740be9`）。签名验不过的索引，`miyu` 整份不收。

## 发一个包

在包自己的仓库里加 `.github/workflows/release.yml`：

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

打 tag（`v` 加上 `package.toml` 里的 `version`）后，模板照 Miyu 装包的那一套查清单、打成 `.miyu`、传进这个 tag 的 Release。

- 包文件夹不在仓库根目录：写 `folder`。
- 打包前要先编程序：写 `build`，一段 bash。
- 清单写了 `targets` 的：一种系统一个包文件，用 `strategy.matrix` 分别写 `runs-on` 和 `target`。
- 打包用的核心版本：不写 `miyu-rev` 的照本仓库根目录的 `miyu-rev`，和收录同一版，不会出现打得出、收不进（或者反过来）。只有要试还没进这里的新写法时才写。

现在只收官方仓库，社区包的自动收录以后再开。
