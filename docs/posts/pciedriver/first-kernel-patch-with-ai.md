# 用 AI 提交你的第一个 Linux 内核补丁：没有想象中那么难

> 内核补丁听起来很吓人？其实在 AI 的帮助下，门槛比你想象的低得多。

## 前言

说到给 Linux 内核贡献代码，很多人第一反应是："这不是大佬才能干的事吗？" 我之前也是这么想的。但最近我借助 Claude Code（Anthropic 的 CLI AI 工具）成功提交了我的第一个内核补丁，整个过程比预想中顺畅很多。这篇文章就来分享一下我的经验，希望能帮到同样想尝试的同学。

---

## 一、背景知识：提交补丁前你需要知道的事

### 1. 先读教程，别上来就莽

内核社区有一套成熟的贡献流程，官方文档写得很清楚。提交之前**务必**阅读以下文档：

- [`Documentation/process/submitting-patches.rst`](https://www.kernel.org/doc/html/latest/process/submitting-patches.html) — 提交补丁的完整流程
- [`Documentation/process/coding-style.rst`](https://www.kernel.org/doc/html/latest/process/coding-style.html) — 内核编码风格（和你平时写的 C 不太一样）
- [`Documentation/process/howto.rst`](https://www.kernel.org/doc/html/latest/process/howto.html) — 内核开发入门概览

不读这些就提交，大概率会被 maintainer 直接忽略，或者收到一封措辞"直率"的回复。

### 2. 内核社区用邮件，不用 GitHub PR

这可能是最让新人困惑的地方：**Linux 内核不接受 Pull Request，所有补丁通过邮件列表提交。**

流程大致是：
1. 用 `git format-patch` 生成补丁文件
2. 用 `git send-email` 发送到对应的邮件列表和 maintainer
3. 在邮件列表上进行 review，收到反馈后修改再重新提交

所以你需要提前配置好 `git send-email`。我用的是 Gmail + msmtp，网上有很多配置教程，这里就不展开了。

### 3. 从 `drivers/staging/` 开始

内核源码树里有一个 `drivers/staging/` 目录，里面放的是质量还不够高、尚未合入主线的驱动代码。这些代码往往有大量的编码风格问题，非常适合新手练手。社区也鼓励新人从这里开始贡献。

### 4. `checkpatch.pl` 是你的好朋友

内核源码里自带一个脚本 `scripts/checkpatch.pl`，用来检查代码风格是否符合规范。用法：

```bash
# 检查整个文件
perl scripts/checkpatch.pl --file drivers/staging/xxx/xxx.c

# 检查一个补丁
perl scripts/checkpatch.pl 0001-your-patch.patch
```

它会报告 ERROR、WARNING 和 CHECK 三个级别的问题。修复这些问题就是最简单的补丁类型。

### 5. Signed-off-by 不能少

每个补丁的 commit message 末尾都需要加上：

```
Signed-off-by: Your Name <your@email.com>
```

这相当于你声明这个补丁是你写的、你有权提交它（参见 Developer Certificate of Origin）。用 `git commit -s` 可以自动加上。

### 6. `get_maintainer.pl` 告诉你该发给谁

```bash
perl scripts/get_maintainer.pl your-patch.patch
```

会列出对应文件的 maintainer 和邮件列表，照着填 `--to` 和 `--cc` 就行。

---

## 二、实战：用 Claude Code 生成补丁

下面是我的实际操作过程。

### Step 1：克隆最新内核源码

```bash
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
```

源码很大（约 4GB），耐心等待。也可以用 `--depth=1` 浅克隆加速，但后续操作可能会受限。

### Step 2：启动 Claude Code

直接在内核源码目录下启动 Claude Code：

```bash
claude
```

### Step 3：让 AI 帮你找问题

我的提示词大致是这样的：

> 在 `drivers/staging/` 下找一个被 checkpatch.pl 报告的真实问题，不要 false positive，帮我生成一个可以被接受的补丁。

Claude 会：
1. 在 staging 目录下扫描多个文件
2. 运行 `checkpatch.pl` 找到真实的 warning/error
3. 验证这个问题是否是 false positive（比如它发现了一个 struct 内部的 blank line 警告，确认是误报后会跳过）
4. 找到真正的问题并修复

在我的例子中，Claude 找到了 `drivers/staging/media/atomisp/` 下的一个 ERROR 级别问题：

```c
// 修复前
return(&sh_css_sp_group.pipe_io_status);

// 修复后
return &sh_css_sp_group.pipe_io_status;
```

`return` 不是函数，不需要括号。很简单对吧？但这就是一个合格的内核补丁。

### Step 4：生成补丁

Claude 会帮你：
- 修改代码
- 写符合规范的 commit message
- 加上 `Signed-off-by`
- 运行 `checkpatch.pl` 验证补丁
- 用 `git format-patch` 生成 `.patch` 文件

### Step 5：发送补丁

Claude 还能帮你用 `get_maintainer.pl` 找到该发给谁，然后构造 `git send-email` 命令：

```bash
git send-email \
  --to maintainer1@kernel.org \
  --to maintainer2@kernel.org \
  --cc reviewer@intel.com \
  --cc linux-staging@lists.linux.dev \
  0001-your-patch.patch
```

一条命令，补丁就发出去了。

---

## 三、AI 到底帮了什么

回顾整个过程，AI 主要帮了这几件事：

| 环节 | 没有 AI | 有 AI |
|------|--------|-------|
| 找问题 | 自己翻代码 + 跑 checkpatch | AI 批量扫描，自动排除误报 |
| 判断是否 false positive | 需要自己理解 checkpatch 规则 | AI 分析上下文帮你判断 |
| 写 commit message | 查文档、模仿格式 | AI 按规范自动生成 |
| 找 maintainer | 手动跑脚本、整理邮件地址 | AI 一条龙搞定 |
| 构造 send-email 命令 | 容易搞混 --to 和 --cc | AI 按角色自动分配 |

**但 AI 不能帮你的：**

- 理解你在做什么。你需要知道基本的内核开发流程。
- 保证补丁被接受。maintainer 可能有其他意见。
- 替代你学习。第一个补丁是入门，后面的路还得自己走。

---

## 四、一些踩坑经验

1. **不要盲目相信 checkpatch 的每一条输出。** 有些警告是 false positive，比如 struct 内部的 "Missing a blank line after declarations"——这条规则是针对函数体里变量声明后要空一行，不适用于 struct 字段定义。如果你提交了一个修"假问题"的补丁，只会暴露你没理解规则。

2. **也不要盲目相信 AI。** 我第一次让 AI 生成补丁时，它就犯了上面那个错误——给 struct 里加了个空行。AI 很强，但你需要有判断力。

3. **Commit message 很重要。** 内核社区对 commit message 的格式要求很严格：
   - 第一行是 `subsystem: component: 简短描述`
   - 正文解释 *为什么* 要改，而不是 *改了什么*（改了什么看 diff 就知道了）
   - 结尾必须有 `Signed-off-by`

4. **配置 `git send-email` 提前搞定。** 别等补丁都生成好了才发现邮件发不出去。测试方法：先给自己发一封。

5. **用最新的内核源码。** 如果你的补丁基于旧版本，可能改的地方已经被别人改过了。提交前 `git fetch && git rebase`。

---

## 五、接下来做什么

提交了第一个风格修复补丁后，你可以挑战更有意义的贡献：

- **修复真正的 bug**：比如逻辑错误、内存泄漏、竞态条件
- **改进代码质量**：用内核提供的 helper 函数替换 open-coded 实现
- **写文档**：内核文档永远缺人维护
- **参与 review**：在邮件列表上看别人的补丁，学习 maintainer 是怎么 review 的

内核社区有时候确实很"直"，但只要你态度认真、补丁质量过关，大家还是很 welcoming 的。

---

## 参考资料

- [内核新手网站 (kernelnewbies.org)](https://kernelnewbies.org/)
- [提交你的第一个内核补丁](https://kernelnewbies.org/FirstKernelPatch)
- [内核官方文档：提交补丁](https://www.kernel.org/doc/html/latest/process/submitting-patches.html)
- [Greg KH 关于 staging 补丁的说明](https://www.kernel.org/doc/html/latest/process/coding-style.html)
- [Claude Code](https://claude.com/claude-code)

---

*如果这篇文章对你有帮助，欢迎转发。也欢迎在评论区分享你的第一个内核补丁经历！*
