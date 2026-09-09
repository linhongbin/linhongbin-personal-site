# RUNBOOK —— hongbinlin.com 运维手册

写给半年后的自己。只记**症状 → 敲什么命令 → 多久好**，不记别的。
发布前的敏感内容检查保存在本地私有位置，不进入本仓；发布时必须先执行完整的本地检查。

---

## 1. 这个站是什么、怎么部署的

| 项 | 值 |
|---|---|
| 站点类型 | Jekyll 静态站（`minima` 主题 + `github-pages` gem） |
| 仓库 | `linhongbin/linhongbin-personal-site`（`git@github.com:linhongbin/linhongbin-personal-site.git`） |
| 分支 | `master` —— push 上去就自动重建 |
| 构建方式 | GitHub Pages 自带的 Jekyll 构建器（仓里**没有** `.github/workflows/`，源＝分支根目录） |
| 域名 | `hongbinlin.com`，由仓根 `CNAME` 文件绑定（文件内容就一行 `hongbinlin.com`） |
| 相关配置 | `_config.yml` 里 `url: https://hongbinlin.com`、`baseurl: ""` |

**push 到线上可见：通常 30–60 秒**（构建 + CDN 刷新）。

超过 3–5 分钟没变化，去 GitHub 仓库页看那次 Pages 构建是成功还是失败，别干等。

验证线上状态（浏览器缓存会骗人，用命令别用刷新）：

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://hongbinlin.com    # 期望 200
```

---

## 2. 🔴 最重要的一条排障记忆：仓库改名 → 全站 404

**已经踩过一次，会再踩，先看这条。**

> **原因**：仓库**改名**（把 user site 变成 project site）会**撤掉 GitHub Pages 的部署，但保留域名绑定**。
>
> **症状**：所有配置看着都对——`CNAME` 在、Pages 设置页面里一切正常、DNS 也没动——但站点就是 **404**。
>
> **解法**：**push 一个空提交触发重建**即可。约 **45 秒**恢复。

```bash
git commit --allow-empty -m "chore: trigger pages rebuild"
git push
sleep 45
curl -s -o /dev/null -w "%{http_code}\n" https://hongbinlin.com    # 期望 200
```

已发生过一次，修复提交是 **`7261297`**（`chore: trigger pages rebuild after repo rename`）。

**判断是不是这个病**：`curl` 能连上、只是回 404（而不是连接失败 / 证书错 / DNS 解析不了）
⇒ 域名解析是好的，问题在部署侧 ⇒ 空提交重建先试。

---

## 3. 裸地址已失效

`linhongbin.github.io`（**裸根**）随改名**永久失效**，现在直接 404。**唯一入口是 `hongbinlin.com`**。

任何文件里出现指向裸根的链接都是要修的 bug：

```bash
git grep -n "linhongbin.github.io"
```

⚠️ 看结果时要分清两种，**别一把全改**：

- 指向**裸根**（`https://linhongbin.github.io/` 或 `https://linhongbin.github.io`）→ **死链，必须改成 `https://hongbinlin.com`**
- 带**子路径**的（如 `https://linhongbin.github.io/gas/`）→ 那是**别的仓库的项目页**，**仍然活着（200），留着别动**

---

## 4. 本地开发

已验证可用的版本组合：**Ruby 3.4.1 / bundler 2.3.26 / jekyll 3.10.0**

```bash
ruby -v                          # ruby 3.4.1
bundle -v                        # Bundler version 2.3.26
bundle exec jekyll --version     # jekyll 3.10.0
```

首次（或换机器后）装依赖：

```bash
bundle install
```

起本地服务：

```bash
bundle exec jekyll serve         # http://localhost:4000
bundle exec jekyll serve --port 4001   # 4000 被占时
```

只想构建、**不想弄脏仓库**（产物写到别处）：

```bash
bundle exec jekyll build --destination "$TMPDIR/site-check"
```

注意事项：

- `_site/` 是产物目录，已 gitignore（`git check-ignore -v _site` 会打印命中的规则），**永远不要提交**
- **改了 `_config.yml` 不会热重载**，必须重启 `jekyll serve`
- 本地构建会扫描工作区里**未被 git 跟踪**的文件，可能冒出与线上无关的 Liquid 警告；线上只构建 git 里有的文件，这类警告可以忽略

---

## 5. 提交与推送窗口

| | 时段 |
|---|---|
| ✅ 可以 `git commit` **和** `git push` | 周一至周五 **20:00 → 次日 10:00** · 周六周日**全天** |
| ⛔ 两者都不要做 | 周一至周五 **10:00 – 20:00** |

**commit 和 push 都按这个窗口**，不是只管 push。

仓库装了 **pre-commit + pre-push 双守卫**，禁窗内会被直接拦下（非零退出，命令不执行）。

确有必要在禁窗内操作：

```bash
WINDOW_OK=1 git commit -m "..."
WINDOW_OK=1 git push
```

### ⚠️ 守卫不进 git —— 换机器 / 重新 clone 之后会**静默**失效

守卫是本地 hook，位于 `.git/hooks/pre-commit` 和 `.git/hooks/pre-push`。`.git/` 不进版本库
⇒ **新 clone 出来的仓库里没有它们，而且不会有任何提示**。

检查装没装：

```bash
ls -l .git/hooks/pre-commit .git/hooks/pre-push    # 两个都要在，且可执行（-rwxr-xr-x）
```

重装方式（二选一）：

1. 跑守卫安装脚本 `install-push-guard.sh`（**不在本仓版本库里**，在本地另外保存）
2. 从另一台已装好的机器把这两个文件拷过来，然后 `chmod +x`
   —— 两个文件**内容完全相同**（同一份脚本按调用名分支），可以互相复制

---

## 6. 新机器 checklist

```bash
# 1. clone
git clone git@github.com:linhongbin/linhongbin-personal-site.git
cd linhongbin-personal-site

# 2. 装 Ruby 3.4.1 + bundler，确认版本
ruby -v && bundle -v

# 3. 装依赖
bundle install

# 4. ⚠️ 重装提交/推送守卫（clone 不会带来，见 §5），然后确认
ls -l .git/hooks/pre-commit .git/hooks/pre-push

# 5. 确认 context/ 一个文件都没被跟踪：必须**没有任何输出**
git ls-files context/

# 6. 构建跑通
bundle exec jekyll build --destination "$TMPDIR/site-check"
```

第 4 步和第 5 步最容易忘，忘了不会报错，所以每次都照着敲一遍。

---

## 7. 发布前

⚠️ **push 即发布** —— push 到 `master` 就会重建 hongbinlin.com，站上每个字都是公开的。

推之前逐条执行本地私有的发布检查清单。那份清单是发布前检查的唯一依据，本文件不重复其敏感规则。
