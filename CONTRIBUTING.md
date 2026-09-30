# 贡献指南

感谢你对 DevBridge 的关注！欢迎提交 Issue、Pull Request 或参与讨论。

参与本项目即表示你同意遵守 Apache License 2.0 的条款，并同意你的贡献将在该许可证下授权。

---

## 目录

- [项目结构](#项目结构)
- [本地环境配置](#本地环境配置)
- [快速开始](#快速开始)
- [代码规范](#代码规范)
- [测试](#测试)
- [分支策略](#分支策略)
- [提交规范](#提交规范)
- [Pull Request 流程](#pull-request-流程)
- [CLI 开发指南](#cli-开发指南)
- [Go SDK 开发指南](#go-sdk-开发指南)
- [新增文档页面](#新增文档页面)
- [内容准则](#内容准则)
- [Issue 指南](#issue-指南)
- [发布流程](#发布流程)
- [常见问题](#常见问题)
- [联系方式](#联系方式)

---

## 项目结构

DevBridge 是一个 Monorepo，三个组件各自使用独立的首层目录和工具链：

```text
.
├── .github/              # CI 工作流、Dependabot 配置
├── cli/                  # DevBridge CLI（Go，面向终端用户）
│   ├── cmd/              # cobra 命令定义
│   ├── internal/         # 内部业务逻辑（auth、config、i18n、netutil）
│   ├── scripts/          # 发布辅助脚本
│   ├── .golangci.yml     # golangci-lint 配置
│   ├── .goreleaser.yaml  # GoReleaser 生产环境配置
│   ├── .goreleaser.test.yaml  # GoReleaser 测试环境配置
│   ├── Makefile          # 构建、测试、交叉编译入口
│   └── go.mod            # module huawei.com/devbridge
├── go-sdk/               # DevBridge Go SDK（供第三方集成）
│   ├── client.go         # API 客户端
│   ├── tunnel.go         # 隧道管理
│   ├── host.go           # 本地服务托管
│   ├── connect.go        # 远程连接
│   ├── port.go           # 端口转发
│   └── go.mod            # module github.com/huaweicloud/devspace-devbridge/go-sdk
├── docs/                 # VitePress 文档站点
│   ├── guide/            # 按用户任务组织的指南
│   ├── reference/        # CLI、API 和故障排查参考
│   ├── integrations/     # 集成说明
│   ├── .vitepress/       # VitePress 配置与主题
│   └── package.json      # 文档构建与校验命令
├── LICENSE               # Apache 2.0
└── README.md             # 仓库总览
```

> **原则**：后续新增组件应各自使用独立的首层目录，将构建配置和依赖文件保存在对应目录中，避免不同工具链相互影响。

---

## 本地环境配置

### 工具链版本要求

| 组件        | 最低版本   | 用途               |
| ----------- | ---------- | ------------------ |
| **Git**     | 2.20       | 版本控制           |
| **Go**      | 1.26（CLI）/ 1.22（SDK） | Go 编译与测试 |
| **Node.js** | 24         | 文档构建与校验     |
| **npm**     | 随 Node.js 附带 | 依赖管理      |

### 安装 Go（>= 1.26）

CLI 的 `go.mod` 声明 `go 1.26.0`，需安装 Go 1.26 或更高版本。

```bash
# 方式一：官方二进制包（推荐）
# 前往 https://go.dev/dl/ 下载对应平台的 1.26 安装包

# Linux 示例
wget https://go.dev/dl/go1.26.0.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.26.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin

# 方式二：包管理器（版本可能滞后，安装后用 go version 确认）
# macOS:  brew install go
# Ubuntu: sudo apt install golang-go
```

验证安装：

```bash
go version
# 期望输出: go version go1.26.x <platform>
```

> **仅修改 Go SDK 的贡献者**：SDK 要求 Go >= 1.22，安装 1.22 即可。但如果同时需要构建 CLI，请安装 1.26。

### 安装 Node.js（>= 24）

文档站点要求 Node.js 24 或更高版本。

```bash
# 方式一：官方二进制包
# 前往 https://nodejs.org/ 下载 LTS 版本（>= 24）

# 方式二：nvm（推荐，可切换版本）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
nvm install 24
nvm use 24

# 方式三：包管理器
# macOS:  brew install node@24
# Ubuntu: 参见 NodeSource 官方安装脚本
```

验证安装：

```bash
node --version
# 期望输出: v24.x.x
npm --version
```

### 安装 golangci-lint（可选，CLI 贡献者推荐）

```bash
# 参见官方安装指南: https://golangci-lint.run/usage/install/
# macOS: brew install golangci-lint
# Linux:
curl -sSfL https://raw.githubusercontent.com/golangci-lint/golangci-lint/master/install.sh | sh
```

验证安装：

```bash
golangci-lint version
```

### 一次性确认

```bash
git --version      # >= 2.20
go version         # >= 1.26（CLI）/ >= 1.22（SDK）
node --version     # >= 24
```

全部通过后，进入 [快速开始](#快速开始)。

---

## 快速开始

### 1. Fork 并克隆

```bash
# 在 GitHub 上 Fork 本仓库，然后：
git clone https://github.com/<你的用户名>/devspace-devbridge.git
cd devspace-devbridge
```

### 2. 添加上游仓库

```bash
git remote add upstream https://github.com/huaweicloud/devspace-devbridge.git
```

### 3. 安装依赖

```bash
# CLI（Go 依赖自动拉取，无需额外步骤）
cd cli && go mod tidy

# Go SDK
cd go-sdk && go mod tidy

# 文档
cd docs && npm ci
```

### 4. 选择你要修改的组件，本地构建验证

```bash
# CLI
cd cli && make build-dev
./bin/devbridge version

# Go SDK
cd go-sdk && go build ./...

# 文档
cd docs && npm run docs:dev
# 文档本地预览：http://127.0.0.1:5173/devspace-devbridge/
```

---

## 代码规范

### Go 代码（cli/ 和 go-sdk/）

| 规则               | 工具 / 说明                                                       |
| ------------------ | ----------------------------------------------------------------- |
| 格式化             | `gofmt -s -w .` 或 `go fmt ./...`，提交前必须执行                 |
| 静态检查           | `go vet ./...`                                                    |
| Lint               | `golangci-lint run`（CLI 使用 `cli/.golangci.yml` 配置）          |
| 命名               | 导出标识符用 PascalCase，未导出用 camelCase，包名全小写单单词     |
| 导入顺序           | 标准库 → 空行 → 第三方库 → 空行 → 本项目内部包，用 `goimports` 排序 |
| 错误处理           | 不忽略 error 返回值，不使用 `_` 丢弃 error                        |
| 注释               | 导出函数/类型须有 `// Name ...` 格式的文档注释                    |

> **golangci-lint 配置**：当前 `cli/.golangci.yml` 禁用了 `errcheck`、`gosec`、`lll` 等严格 linter。新增代码应尽量遵循这些规则的精神，即使工具不强制。

### Markdown 文档（docs/）

| 规则           | 工具 / 说明                                                              |
| -------------- | ------------------------------------------------------------------------ |
| Markdown lint  | `markdownlint-cli2`，配置见 `docs/.markdownlint-cli2.mjs`               |
| 格式化         | `prettier --check .`，配置见 `docs/package.json`（proseWrap: preserve） |
| 构建验证       | `vitepress build .`，确保无死链、无 frontmatter 错误                     |

当前 markdownlint 规则：

- MD013（行长度限制）关闭——不强制换行宽度。
- MD024（重复标题）仅检查同级重复——允许不同层级使用相同标题。
- MD025（首个标题级别）关闭。
- MD033（内联 HTML）关闭——VitePress 组件需要内联 HTML。
- MD036（强调即标题）关闭。

一键检查（提交前必跑）：

```bash
cd docs && npm run check
# 等价于: markdownlint-cli2 + prettier --check + vitepress build
```

如需自动修复格式问题：

```bash
cd docs && npm run format   # prettier --write .
```

### 通用约定

- **行尾换行**：文件末尾保留一个空行（POSIX 惯例）。
- **缩进**：Go 用 tab，Markdown/YAML/JS 用 2 空格。
- **编码**：UTF-8，不使用 BOM。

---

## 测试

### CLI 测试

```bash
cd cli
make test         # 等同于 go test ./...
```

> **现状**：CLI 目前尚无单元测试文件。新增功能时鼓励附带测试，测试文件与被测文件同目录，命名为 `*_test.go`。

### Go SDK 测试

```bash
cd go-sdk
go test ./...     # 运行全部测试
go test -v ./...  # 详细输出
go test -race ./...  # 竞态检测
go vet ./...      # 静态检查
```

当前已有测试文件：

| 文件                | 覆盖范围              |
| ------------------- | --------------------- |
| `tunnel_test.go`    | 隧道创建与管理        |
| `websocket_test.go` | WebSocket 连接与重连  |
| `example_test.go`   | 公开 API 用法示例     |

新增公开 API 时须附带对应的 `_test.go` 文件。

### 文档测试

```bash
cd docs
npm run check     # markdownlint + prettier --check + vitepress build
npm run audit     # 依赖安全审计
```

三者全部通过后方可提交。`npm run check` 会捕获死链、frontmatter 语法错误、Markdown 格式问题。

---

## 分支策略

| 分支      | 用途         | 能否直接推送      |
| --------- | ------------ | ----------------- |
| `master`  | 稳定发布分支 | ❌ 仅通过合并     |
| `develop` | 开发集成分支 | ❌ 仅通过 PR 合并 |
| `feat/*`  | 新功能分支   | ✅（个人 Fork）   |
| `fix/*`   | Bug 修复分支 | ✅（个人 Fork）   |
| `docs/*`  | 文档变更分支 | ✅（个人 Fork）   |

**PR 目标分支**：默认为 `develop`。仅在发布窗口期由维护者合并至 `master`。

---

## 提交规范

使用 [Conventional Commits](https://www.conventionalcommits.org/) 格式：

```text
<type>(<scope>): <subject>
```

### type 取值

| type       | 说明                   |
| ---------- | ---------------------- |
| `feat`     | 新功能                 |
| `fix`      | Bug 修复               |
| `docs`     | 文档变更               |
| `style`    | 代码格式（不影响逻辑） |
| `refactor` | 重构                   |
| `test`     | 测试相关               |
| `chore`    | 构建、工具、依赖变更   |

### scope 取值

| scope  | 对应组件   |
| ------ | ---------- |
| `cli`  | CLI 工具   |
| `sdk`  | Go SDK     |
| `docs` | 文档站点   |
| `ci`   | CI/CD 配置 |

### 示例

```text
feat(cli): 支持隧道重连自动恢复
fix(sdk): 修复 WebSocket 断线后 goroutine 泄漏
docs: 更新快速入门中的安装步骤
chore(ci): 升级 actions/checkout 至 v7
```

---

## Pull Request 流程

### 1. 创建工作分支

```bash
git checkout develop
git pull upstream develop
git checkout -b feat/your-feature
```

### 2. 编写代码并本地验证

根据修改的组件，运行对应的检查：

```bash
# 修改了 CLI
cd cli && golangci-lint run && go vet ./... && make test && make build-dev

# 修改了 Go SDK
cd go-sdk && go vet ./... && go test ./... && go build ./...

# 修改了文档
cd docs && npm run check
```

### 3. 提交并推送

```bash
git commit -m "feat(cli): your feature description"
git push origin feat/your-feature
```

### 4. 在 GitHub 上创建 PR

- **目标分支**：`develop`
- **PR 标题**：遵循提交规范格式
- **PR 描述**需包含：
  - **变更说明**：做了什么
  - **关联 Issue**：如 `Closes #123`
  - **测试方式**：如何验证
  - **截图**：CLI 输出变更或文档页面变更请附截图

### 5. PR 审查清单

- [ ] CI 全部通过
- [ ] 新功能附带测试
- [ ] 无引入新的 lint 警告
- [ ] API 或 CLI 行为变更已同步更新文档
- [ ] 依赖变更通过 `npm run audit`（文档）或 `go mod tidy`（Go）

### CI 自动检查

PR 创建后将自动触发以下 CI：

| 工作流          | 触发条件                  | 检查内容                                             |
| --------------- | ------------------------- | ---------------------------------------------------- |
| `build-cli.yml` | PR 到 develop/main/master | GoReleaser snapshot 编译验证                         |
| `build-sdk.yml` | PR 修改 `go-sdk/**`       | `go build` + `go vet` + `go test`                    |
| `pages.yml`     | push 到 develop/master    | markdownlint + prettier + VitePress 构建 + npm audit |

---

## CLI 开发指南

CLI 位于 `cli/` 目录，使用 Go + cobra 构建。

### 本地构建

```bash
cd cli
make build-dev    # 开发环境构建（输出到 bin/devbridge）
make build-test   # 测试环境构建
make build-prod   # 生产环境构建
```

### 运行测试

```bash
cd cli
make test         # 等同于 go test ./...
```

### 代码检查

```bash
cd cli
golangci-lint run    # 使用 .golangci.yml 配置
go vet ./...         # 标准静态检查
```

### 交叉编译

```bash
cd cli
make build-all    # 构建 6 个平台产物 + SHA256 校验和
```

### 代码组织约定

- 命令定义放在 `cmd/`，每个文件对应一个子命令
- 业务逻辑放在 `internal/`，按领域分包（`auth`、`config`、`i18n`、`netutil`）
- 不要在 `cmd/` 中写复杂业务逻辑，保持命令层薄
- 新增依赖需在 PR 中说明理由，并确保 `go mod tidy` 后 `go.sum` 一致

### 版本注入

构建时通过 `-ldflags` 注入版本号、服务器地址等参数（见 `Makefile` 中的 `LDFLAGS`）。修改这些注入变量时需同步更新 `Makefile` 和 `.goreleaser.yaml`。

---

## Go SDK 开发指南

Go SDK 位于 `go-sdk/` 目录，供第三方应用集成 DevBridge 能力。

### 本地构建与测试

```bash
cd go-sdk
go build ./...
go test ./...
go vet ./...
```

### 代码组织约定

- SDK 作为独立 Go module（`github.com/huaweicloud/devspace-devbridge/go-sdk`）
- CLI 通过 `replace` 指令引用本地 SDK（见 `cli/go.mod`）
- 公开 API 放在包根目录（`client.go`、`tunnel.go` 等），内部实现放在 `internal/`
- 每个公开类型/函数应有对应的 `_test.go` 文件
- **向后兼容**：已发布的 API 不应引入破坏性变更；如必须变更，在 PR 中标注 `BREAKING CHANGE`

### 发布标签

SDK 发布使用 `go-sdk/v*` 前缀的 tag（如 `go-sdk/v0.2.1`）。tag 一旦推送即不可变（Go module proxy 会永久缓存）。

---

## 新增文档页面

1. 在 `docs/guide/` 或 `docs/reference/` 中新增 Markdown 文件
2. 在 `docs/.vitepress/config.mjs` 的 `sidebar` 中加入入口
3. 使用相对链接连接相关主题
4. 在 `docs/` 中运行 `npm run check` 确保通过

---

## 内容准则

- CLI 命令描述以 DevBridge CLI 的实际行为为准——修改 CLI 行为后，应在同一 PR 中更新对应文档
- 隧道、端口、Host 和 Connect 的说明以 DevBridge 已发布能力为准
- 不要直接编辑 `docs/.vitepress/dist`，它是构建产物且不会提交

---

## Issue 指南

### Bug 报告

请包含以下信息：

- **环境**：操作系统、Go 版本（CLI/SDK）或 Node.js 版本（文档）
- **DevBridge 版本**：`devbridge version` 的输出
- **复现步骤**：逐步可操作
- **期望行为**：应该发生什么
- **实际行为**：实际发生了什么
- **日志/截图**：如有请附上

### 功能请求

- **使用场景**：你在什么情况下需要这个功能
- **期望方案**：你设想的实现方式（可选）
- **替代方案**：你考虑过的其他方式（可选）

### 好的 Issue

- 一个 Issue 只描述一个问题
- 先搜索已有 Issue，避免重复
- 使用清晰的标题概括问题

---

## 发布流程

发布由项目维护者操作，贡献者无需关注，但了解流程有助于理解分支节奏：

### CLI 发布

1. 维护者通过 GitHub Actions `workflow_dispatch` 手动触发 `build-cli.yml`
2. 版本号包含 `release` 时，自动发布到 GitHub Release 和 GitCode Release
3. 产物包含 6 个平台的 tar.gz 包 + checksums.txt + install 脚本

### Go SDK 发布

1. 维护者推送 `go-sdk/v*` 格式的 tag（如 `go-sdk/v0.2.1`）
2. Go module proxy（proxy.golang.org / goproxy.cn）自动索引
3. `build-sdk.yml` 验证构建并可选上传源码包到 GitCode

### 文档发布

1. PR 合并到 `develop` 或 `master` 后，`pages.yml` 自动触发
2. 执行 `npm run check` + `npm run audit`，通过后构建 VitePress 并部署到 GitHub Pages

---

## 常见问题

### go build 报错 "go: go.mod requires go >= 1.26"

本地 Go 版本低于 CLI 要求。升级 Go 至 1.26 或更高版本，参见 [本地环境配置](#本地环境配置)。

如果仅修改 Go SDK，可在 `go-sdk/` 目录下单独构建，该模块要求 Go >= 1.22。

### npm run check 报错 "markdownlint-cli2: command not found"

未安装文档依赖。在 `docs/` 目录下执行 `npm ci` 安装。

### npm run check 报错 prettier 格式不一致

运行 `npm run format` 自动修复格式问题，然后重新提交。

### vitepress build 报错 frontmatter 解析失败

检查 Markdown 文件顶部的 YAML frontmatter，确保冒号后有空格、缩进一致。常见错误是半角冒号紧贴值（如 `title:xxx` 应为 `title: xxx`）。

### make build-dev 报错找不到 go-sdk

CLI 通过 `replace` 指令引用本地 `../go-sdk`。确保在 Monorepo 根目录下克隆了完整仓库，而非仅克隆 `cli/` 子目录。

### PR CI 失败但本地通过

确认本地工具链版本与 CI 一致：Go 1.26（CLI）/ 1.22（SDK）、Node.js 24。版本差异可能导致本地通过但 CI 失败。

---

## 联系方式

- **Issue**：[GitHub Issues](https://github.com/huaweicloud/devspace-devbridge/issues)
- **在线文档**：<https://huaweicloud.github.io/devspace-devbridge/>

---

再次感谢你的贡献！每一个 PR 和 Issue 都让 DevBridge 更好。
