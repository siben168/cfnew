# cfnew 部署说明

## 项目概述

这是一个基于 Cloudflare Workers 的代理订阅服务，fork 自 [byJoey/cfnew](https://github.com/byJoey/cfnew)。

---

## 文件结构


| 文件               | 说明                           |
| ------------------ | ------------------------------ |
| `明文源吗`         | 源代码（JS，可读）             |
| `少年你相信光吗`   | 混淆后的代码（由 CI 自动生成） |
| `clash-config.ini` | **自定义订阅转换配置**         |
| `dist/`            | 本地产物目录（已 gitignore）   |

---

## 🔧 自定义修改

### 1. 订阅配置优化 (2026-02-08)

**修改文件**：`明文源吗` 第 1349 行

```javascript
// 添加了 config= 参数，使用自定义配置文件
const converterUrl = `${scu}?target=clash&url=${encodedUrl}&insert=false&config=${encodeURIComponent("https://raw.githubusercontent.com/siben168/cfnew/main/clash-config.ini")}&emoji=...`;
```

**配置文件**：`clash-config.ini`

包含以下优化：

- ⚠️ **Cloudflare 回环规则** - CF 资源走 DIRECT，防止回环
- 🎯 **智能优选组** - AI 专用 fallback，排除 HKG
- 🔄 **全能优选组** - 通用 fallback，包含 HKG
- 🤖 **AI 服务规则** - OpenAI、Claude、Google（使用 No_Resolve 版本）
- 🌐 **非港自动组** - url-test，排除香港节点

---

## 🚀 部署流程

### 日常开发

```bash
# 1. 修改源代码
vim 明文源吗

# 2. 提交代码
git add 明文源吗 clash-config.ini
git commit -m "feat: 你的修改说明"
git push
```

推送后 GitHub Actions 自动：

- 运行 `obfuscate.yml` 混淆代码 → 生成 `少年你相信光吗`

### 发布 Release

```bash
# 创建并推送 tag
git tag v0.0.2
git push origin v0.0.2
```

推送 tag 后 GitHub Actions 自动：

- 运行 `test.yml` 创建 Release
- 将 `少年你相信光吗` → `_worker.js` → `Pages.zip`
- 上传到 GitHub Releases

---

## 🔄 合并上游更新

```bash
# 1. 添加上游仓库（只需一次）
git remote add upstream https://github.com/byJoey/cfnew.git

# 2. 拉取上游更新
git fetch upstream
git merge upstream/main

# 3. 解决冲突（注意保留以下自定义修改）
#    - 明文源吗 第 1349 行的 config= 参数
#    - clash-config.ini 文件

# 4. 提交并推送
git push
```

---

## ⚠️ 注意事项

1. **clash-config.ini 必须放在项目根目录**（不是 dist/）
2. **修改 `明文源吗` 后会自动触发混淆**
3. **发布新版本需要创建 tag**
4. **合并上游时注意保留第 1349 行和 1403 行的修改**

---

## 📦 自定义规则配置

### 文件说明


| 文件                | 说明                                |
| ------------------- | ----------------------------------- |
| `custom-rules.yaml` | 自定义 rule-providers 和 rules 配置 |

### 修改规则流程

```bash
# 1. 编辑规则文件
vim custom-rules.yaml

# 2. 提交推送
git add custom-rules.yaml
git commit -m "update: 修改规则"
git push
```

### 清除 CDN 缓存（可选）

git raw库有缓存问题，为了加速并可以更新缓存，做了如下更改：

```javascript
// 原来
https://raw.githubusercontent.com/siben168/cfnew/main/custom-rules.yaml
// 改为 jsDelivr（自动缓存刷新更快）
https://cdn.jsdelivr.net/gh/siben168/cfnew@main/custom-rules.yaml
```

规则通过 jsDelivr CDN 加载，修改后如需立即生效：

```bash
# 访问此 URL 清除缓存
https://purge.jsdelivr.net/gh/siben168/cfnew@main/custom-rules.yaml
```

> 💡 如果不手动清除，CDN 缓存会在约 24 小时后自动刷新。
