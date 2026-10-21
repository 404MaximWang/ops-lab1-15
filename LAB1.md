# 实验 1 记录：第 15 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 15 组 |
| 仓库地址 | <https://github.com/404MaximWang/ops-lab1-15> |
| 代码载体 | 统一示例（`enroll-conflict`，Go） |
| runner 与语言版本 | `ubuntu-24.04`，Go 1.26.9 |
| 合并方式 | squash |

## 2. 任务 0：本地基线

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | `name=enroll-conflict`<br>`version=1.0.0`<br>`build_no=1`<br>`run_id=local`<br>`commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`go=go1.26.9` |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | **BUILD_INFO 不变**：`diff ~/BUILD_INFO.first dist/enroll-conflict_1.0.0_linux_amd64/BUILD_INFO` 无任何输出，六行内容逐字相同。<br>**二进制 sha256 变了**：<br>第一次（工作区干净）<br>`f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e`<br>第二次（改了 `conflict.go` 一行、未提交）<br>`a30ec5fc1559e56f0ee9481063580ac92817c16ab6c4f72380d5387bea537ea4`<br>tar.gz 同步变化：<br>`4721adf0554fadec7ea5f4413f028f78bd656720e405f9a04c719a922ac3c919`<br>→ `237406eafbc5322aea84b5a0b0fc7e1286e868b53c039be8df1dacc0d04a3d9f` |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | 能回答该包是什么；无法回答在哪台机器上且无法回答何时；“用哪份代码构建”会出错，因为在脏工作区进行编译，未提交的代码被编译但BUILD_INFO记录的依然是HEAD_SHA。 |

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | 1. **Restrict deletions**（禁止删除分支）<br>2. **Block force pushes**（禁止强制推送）<br>3. **Require a pull request before merging**（合并前必须通过 PR，审批人数 Required approvals: 1，允许合并方式: Squash）<br>4. **Require status checks to pass**（合并前必须通过状态检查，必需检查名称: `ci`） |
| 组长直接 `git push origin main` 被拒的终端输出 | ```text<br>remote: error: GH013: Repository rule violations found for refs/heads/main.<br>remote: Review all repository rules at https://github.com/404MaximWang/ops-lab1-15/rules?ref=refs%2Fheads%2Fmain<br>remote:<br>remote: - Required status check "ci" is expected.<br>remote:<br>remote: - Changes must be made through a pull request.<br>remote:<br>To github.com:404MaximWang/ops-lab1-15.git<br> ! [remote rejected] main -> main (push declined due to repository rule violations)<br>error: failed to push some refs to 'github.com:404MaximWang/ops-lab1-15.git'<br>``` |
| 第一轮：失败的 PR 运行 | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38722867913 |
| 第一轮：修复后通过的 PR 运行 | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38723192831 |
| 第一轮：合入 main 后的运行（含制品） | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38841285101 |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | `conflict.go` 中 `Section.Valid()` 将 `Weekday > 7` 误改为 `>= 7`，导致合法的周日（`Weekday=7`）被判定为非法。原测试用例仅覆盖了周三（3）以及超出范围的 0 和 8，未覆盖周日边界，存在测试盲区，因而该缺陷未被原有测试发现。 |
| 第二轮：仅含缺陷时仍然通过的运行 | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38863528366 |
| 第二轮：补测试后失败的运行 | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38864414631 |
| 第二轮：撤销缺陷后通过的运行 | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38865626252 |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38893574689 |
| Release v1.0.0 | https://github.com/404MaximWang/ops-lab1-15/releases/tag/v1.0.0 |
| Release v1.0.1 | https://github.com/404MaximWang/ops-lab1-15/releases/tag/v1.0.1 |

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | 1.0.0 | 1.0.1 |
| 构建号 | 1 | 3 |
| 提交号（SHA） | `8af81cee015f0f8b474e198551a95541f71811e5` | `ef00dfdca285878d4d4dffae68e534528fc5de1a` |
| 引入该提交的 PR | #3 (https://github.com/404MaximWang/ops-lab1-15/pull/3) | #5 (https://github.com/404MaximWang/ops-lab1-15/pull/5) |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38893369365 | https://github.com/404MaximWang/ops-lab1-15/actions/runs/39103346495 |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | `7f5ed6034035aaa00a50c674ef94550f18a347b776131c710d7b9f62132943dd` | `02a53fce046f69f4851aafbf18c3696c02724f2f02d430cd24b68b2cf1d69303` |

仅凭构建号能否唯一确定是哪次运行？v1.0.0 的构建号在 `ci` 和 `release` 的运行列表中各对应哪次运行？
**不能唯一确定**。因为 `github.run_number` 是在每个工作流文件内部各自从 1 独立递增计数的：
- 在 `ci` 工作流中，构建号 1 对应首次推送工作流文件的运行（Run ID: 38710994783）；
- 在 `release` 工作流中，构建号 1 对应推送 tag `v1.0.0` 的发布运行（Run ID: 38893369365）。
仅凭构建号无法跨工作流区分运行，必须依靠全局唯一的 `run_id`。

发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256：`7f5ed6034035aaa00a50c674ef94550f18a347b776131c710d7b9f62132943dd`。与上表中发布当时的值是否一致？**一致**。

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=1
run_id=38893369365
commit=8af81cee015f0f8b474e198551a95541f71811e5
go=go1.26.9
== 3. 提交 8af81cee015f0f8b474e198551a95541f71811e5
作者: 蒋昊延 <3279131985@qq.com>
时间: 2026-10-16T19:20:25+08:00
标题: ci: add tag-triggered release workflow (#3)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 1
提交页面（含关联的 PR）: https://github.com/404MaximWang/ops-lab1-15/commit/8af81cee015f0f8b474e198551a95541f71811e5
构建它的运行: https://github.com/404MaximWang/ops-lab1-15/actions/runs/38893369365
```

`trace.sh` 输出（v1.0.1）：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=3
run_id=39103346495
commit=ef00dfdca285878d4d4dffae68e534528fc5de1a
go=go1.26.9
== 3. 提交 ef00dfdca285878d4d4dffae68e534528fc5de1a
作者: tywu26 <tywu26@gmail.com>
时间: 2026-10-18T04:08:19+08:00
标题: docs: add running release version check (#5)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.1
版本号: 1.0.1    构建号: 3
提交页面（含关联的 PR）: https://github.com/404MaximWang/ops-lab1-15/commit/ef00dfdca285878d4d4dffae68e534528fc5de1a
构建它的运行: https://github.com/404MaximWang/ops-lab1-15/actions/runs/39103346495
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
<运行 v1.0.1 时>
{"version":"1.0.1","commit":"ef00dfdca285878d4d4dffae68e534528fc5de1a","build_no":"3","go":"go1.26.9"}

<换回 v1.0.0 后>
{"version":"1.0.0","commit":"8af81cee015f0f8b474e198551a95541f71811e5","build_no":"1","go":"go1.26.9"}
```

## 5. 数据

时间均从 GitHub 页面读取，并注明时区。

第一轮 gate 演练的时间线：

| 事件 | 时间（UTC+8 北京时间） | 链接 |
|---|---|---|
| 发起 PR | 2026-10-15 18:48:25 | https://github.com/404MaximWang/ops-lab1-15/pull/1 |
| 第一次 `ci` 失败（运行结束） | 2026-10-15 18:48:44 | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38722867913 |
| 推送修复提交 | 2026-10-15 18:51:03 | https://github.com/404MaximWang/ops-lab1-15/commit/fe910cd0aa7358133f198692e0a24cc9a924901d |
| `ci` 通过（运行结束） | 2026-10-15 18:51:32 | https://github.com/404MaximWang/ops-lab1-15/actions/runs/38723192831 |
| 合入 main | 2026-10-16 10:36:21 | https://github.com/404MaximWang/ops-lab1-15/commit/0626af8eab6ea4ecb5e47bf2e56032424d1a385e |

提交到发布的间隔，至少记录三个合入的 PR（对应首个 Release `v1.0.0` 发布时间：2026-10-16 19:22:42 UTC+8）：

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| #1 | 2026-10-15 18:47:07 | 2026-10-16 10:36:21 | 2026-10-16 19:22:42 | 24 小时 35 分 35 秒 |
| #2 | 2026-10-16 14:46:58 | 2026-10-16 19:17:02 | 2026-10-16 19:22:42 | 4 小时 35 分 44 秒 |
| #3 | 2026-10-16 14:49:52 | 2026-10-16 19:20:26 | 2026-10-16 19:22:42 | 4 小时 32 分 50 秒 |

`ci` 工作流的运行时长（从 Actions 页面读取），至少 5 次，全部列出，不要只给出平均值：

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| #2 | PR (PR #1) | 15s | 故意引入冲突缺陷，测试失败拦截 |
| #3 | PR (PR #1) | 16s | 推送修复提交，测试通过放行 |
| #4 | push (main) | 16s | PR #1 Squash 合入 main，打包主干制品 |
| #5 | PR (PR #2) | 21s | 引入未覆盖的周日缺陷，测试盲区全绿放行 |
| #6 | PR (PR #3) | 18s | 提交 release 工作流，三步测试通过 |
| #9 | PR (PR #2) | 21s | 补充周日测试并撤销缺陷，测试通过 |
| #10 | push (main) | 27s | PR #2 Squash 合入 main，打包主干制品 |
| #11 | push (main) | 22s | PR #3 Squash 合入 main，打包主干制品 |

## 6. 分析问题

每题先列出所用数据（可引用上面的表），再用一段话（4～8 句）写出结论，并说明结论的成立条件。

**问题 1：gate 的收益与成本。** 两轮演练中，gate 拦住了什么，没有拦住什么？用第 5 节的时间线计算：第一轮因失败多花了多少时间，其中等待 `ci` 的时间和人工修改的时间各占多少？一次 `ci` 大约需要多久，在“提交到发布的间隔”中占多大比例？第二轮补充测试之后，还有哪类问题是测试本身无法发现的？

能拦住违反现有测试的修改；无法拦住现有测试尚未覆盖的代码段或逻辑的问题。

第一轮因失败多花 2 分 48 秒（168s）。其中人工修改耗时 2 分 19 秒（139s，占 82.7%），等待 `ci` 重新运行耗时 29 秒（占 17.3%）。单次 `ci` 约 20 秒，在提交到发布的间隔（约 4.5 小时）中仅占约 0.12%。

测试无法发现：环境差异导致的不兼容；测试本身与意图不符；非功能性缺陷如并发问题。

**问题 2：PR 中的构建与 main 上的构建。** 在 PR 触发的运行日志中，`github.sha` 与 `pull_request.head.sha` 是否相同？它们能否在 main 的历史中找到？答案与合并方式有关，请写明本组使用的方式。合并之后，main 上这次改动的提交号是哪个？据此说明，为什么待发布的制品只能从 main 上的提交或 tag 构建。

不相同：github.sha是Action模拟合并的临时merge commit的sha而后者是该分支HEAD commit的sha。前者无论如何都不会出现；后者无法被找到（本组使用squash合并）。

以PR #1为例，main上的提交号是0626af8eab6ea4ecb5e47bf2e56032424d1a385e。
若在分支上构建，由于分支上的commit不会在main上出现，制品和源码的关系被破坏；且分支上的代码没有被审查或准入，可能存在缺陷或质量问题。

**问题 3：可重复构建。** 在本地同一个提交上按下表构建，记录哈希值。④、⑤ 需临时去掉 Makefile 中 tar 的 `--mtime=@0`，完成后用 `git checkout -- Makefile` 恢复，不要提交。

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，`VERSION`、`BUILD_NO`、`RUN_ID`、`COMMIT` 均指定固定值 | `f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e` | `4721adf0554fadec7ea5f4413f028f78bd656720e405f9a04c719a922ac3c919` |
| ② | 与 ① 完全相同，再构建一次 | `f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e`（**同 ①**） | `4721adf0554fadec7ea5f4413f028f78bd656720e405f9a04c719a922ac3c919`（**同 ①**） |
| ③ | 仅将 `BUILD_NO` 换成另一个值 | `daa3d0fda228836f7489a8d2efd62fef9ddedf7e22b8783f4fc15544f717475b` | `2d9a0016bc92f984cae4e0860c04c388c31d6f174dc6eb7528bc23bab0a1a03d` |
| ④ | 去掉 `--mtime=@0`，其余同 ① | `3ccd06b882b8162624f9dc050bc7ecb6493e95a3524f6648edbef10177604737` | `b1fdafecb0c2c8193e4031676cc3e9f6ab1315680d4e953b370d2b425fe5073e` |
| ⑤ | 同 ④，间隔几秒后再构建一次 | `3ccd06b882b8162624f9dc050bc7ecb6493e95a3524f6648edbef10177604737`（**同 ④**） | `0d965427a0021587bc86c33b86f321af10586be756954f00292b6be3c7ab8ca0` |
| Actions | 对 main 上的一次 push 运行执行 Re-run all jobs，比较两次尝试中 package 步骤打印的值 | 第一次：`b3197c846f0c38337b83e5b35974b4d961abe6ba87e7f908aef3565c1a9c4c24`<br>第二次：`b3197c846f0c38337b83e5b35974b4d961abe6ba87e7f908aef3565c1a9c4c24`（**同第一次**） | 第一次：`3b950e44e51734b5c3ead3cf0d341dd6e7ad67e41cc88200234ac547bf97a742`<br>第二次：`3b950e44e51734b5c3ead3cf0d341dd6e7ad67e41cc88200234ac547bf97a742`（**同第一次**） |

构建环境：提交 `7b63b41`，Go 1.26.9，`linux/amd64`。①②③④⑤ 的固定参数为 `VERSION=1.0.0 RUN_ID=local COMMIT=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`，③ 仅把 `BUILD_NO` 换成 2，④⑤ 的 `BUILD_NO` 回到 1。① 与第 2 节第一次打包是同一条命令，所以 tar.gz 的值与第 2 节一致。④ 与 ⑤ 的 `go version -m` 均显示 `vcs.modified=true`（因为改了 Makefile），①②③ 均为 `false`。

哪些相同、哪些不同，原因分别是什么？④ 的二进制文件与 ① 是否相同？如果不同，请结合任务 0 中看到的 vcs 信息分析。据此说明制品中应当包含什么、不应包含什么。另外，`release` 在同一个提交上再次编译，而没有直接发布 `ci` 产出的主干制品：为什么这样做？这与 L-05 第 11 页的“一次构建、多处部署”是否冲突，代价是什么？

- 1和2二进制和压缩包完全相同：输入参数一致且工作区干净。
- 1和3二进制和压缩包不同：build_no被修改后进入二进制和BUILD_INFO文件，因此压缩包一并被修改。
- 1和4二进制和压缩包不同：修改文件后工作区变脏，vcs.modified=true被嵌入二进制元数据；压缩包中含有变化的二进制，一定变化。
- 4和5压缩包不同：tar文件头中的mtime记录了时间。

制品应该包含版本号，提交号；不应当包含时间戳，也不应该包含本实验要求的github run-id。构建需要可复现。

不行。只有通过release workflow，二进制才能被注入相应的版本号等信息。

冲突。两次构建浪费了计算资源，且使得ci和release之间可能存在偏差。

**问题 4：发布与回退。** 假设 v1.0.1 发布后发现缺陷，应当如何处理？为什么不应删除 v1.0.1 的 tag、再用同一版本号重新发布？任务 6 中将服务从 v1.0.1 切换回 v1.0.0 时，使用的是哪个文件，是否需要重新构建？请用两次 `/version` 的输出说明。如果改为从源码重新构建 v1.0.0，得到的文件与原附件是否相同（参考问题 3）？最后，如果有人删除 v1.0.0 的附件，换上一个新文件和对应的新 `.sha256`，`trace.sh` 能否发现？本组在第 4 节记录的值能否发现？可以查阅 GitHub 的 Immutable releases 设置的用途。

若v1.0.1有缺陷，应当切换运行v1.0.0；并在主干修复缺陷。发布v1.0.2。

使用此软件的客户端会因版本号未变动而无法感知更新。对于Go软件，若一个版本被拉取过，proxy.golang.org会记录其hash至append-only缓存。如果同一tag的发布品hash和go缓存不符，这一版本会永远不再可用。此外这还会破坏审计和追溯链。应当严守制品不可变原则。

使用的是dist/release/conflict_1.0.0_linux_amd64。不需要重新构建。两次version输出分别是：
- v1.0.1: `{"version":"1.0.1","commit":"ef00dfdca285878d4d4dffae68e534528fc5de1a","build_no":"3","go":"go1.26.9"}`
- v1.0.0: `{"version":"1.0.0","commit":"8af81cee015f0f8b474e198551a95541f71811e5","build_no":"1","go":"go1.26.9"}`

重新出发CI或者使用默认配置进行本地构建，build_no和run_id不同，因此得到的文件不同；但若显式传入一样的build_no和run_id,文件相同。

若有人同时替换文件和sha256，trace.sh无法发现；第四节记录了原始sha256,可以发现。Immutable release是为了禁止对已发布制品进行删改。

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | 镜像包链接：https://github.com/404MaximWang/ops-lab1-15/pkgs/container/ops-lab1-15 <br>digest：`sha256:b1fd3c0f7928f0a7a970df940882cfda5cb9483f001498de5e814ac02c752aee`<br>运行主机架构：`amd64` |
| O2 缓存对反馈时间的影响 | https://github.com/404MaximWang/ops-lab1-15/pull/6 （保持未合并），详见以下说明 |
| O3 迁移到小组项目 | https://github.com/404MaximWang/sjtu-canvas-cli |

### O2 缓存对反馈时间的影响说明

- 预期：本项目仅依赖 Go 标准库，无外部模块依赖，编译与测试不涉及网络下载，预期开启与关闭缓存耗时差异接近 0 秒。
- 数据（均取自 GitHub Actions `ci` job 运行记录）：
  - 有缓存（第 5 节 PR 运行 #2, #3, #5, #6, #9）：15s、16s、21s、18s、21s（中位数 18s）。
  - 无缓存（PR #6 运行 39479684173 尝试 1～5）：25s、24s、29s、52s、27s（中位数 27s）。
- **结论与波动分析**：无缓存耗时整体稍长且出现单次 52s 高耗时，但步骤拆解显示差异由单步执行波动导致（如第 4 次仅 `go vet` 就耗时 41s），波动主要源于 GitHub Actions 共享 runner 虚拟化宿主机的排队与负载抖动。
- 结论局限：本结论仅适用于无外部依赖的极简项目；在依赖庞大外部模块的项目中，缓存将显著节省网络 I/O 耗时。


## 8. 成员贡献

每位成员一行，只填写 GitHub 用户名，无需填写姓名；行数应与小组人数一致。

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| 404MaximWang | #7, #8 | #2, #3, #5 |
| BearZWasTaken | #1 | #8 |
| Fruitydrops | #2 | #1 |
| HAOYANERIC | #3, #4 | #7 |
| tywu26 | #5 | #4 |
