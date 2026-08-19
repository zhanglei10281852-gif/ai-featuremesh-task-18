# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

算力池完成一致性校准接口返回成功，但再次读取仍是校准中，最后校准时间也未更新，因此后续退役和推理调度被错误阻塞。请修复校准完成状态的持久化。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/ai-featuremesh-task-18
- 仓库地址：https://github.com/zhanglei10281852-gif/ai-featuremesh-task-18.git
- parent SHA：91690e4759ca4cfe78c56ce3f60d3c3b563886d6

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/ai-featuremesh-task-18.git bug-repro
cd bug-repro
git checkout --detach 91690e4759ca4cfe78c56ce3f60d3c3b563886d6
go test ./internal/service -run "^TestComputePoolReconcilingAndRetirementLifecycle$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestComputePoolReconcilingAndRetirementLifecycle$" -count=1
--- FAIL: TestComputePoolReconcilingAndRetirementLifecycle (0.48s)
    service_test.go:325: complete reconciliation = {ID:pool_04d03f671528dd39d4ece56b SerialNumber:BOX-1 State:reconciling CapacityRows:1000 AttestationDueAt:2026-08-20 08:00:00 +0000 UTC LastReconciledAt:2026-08-18 08:00:00 +0000 UTC ReservedRunID: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:2}, error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/ai-featuremesh-base/internal/service	0.480s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestComputePoolReconcilingAndRetirementLifecycle$" -count=1
--- FAIL: TestComputePoolReconcilingAndRetirementLifecycle (1.23s)
    service_test.go:325: complete reconciliation = {ID:pool_f93a0bd58e3a7c275e4ba6bb SerialNumber:BOX-1 State:reconciling CapacityRows:1000 AttestationDueAt:2026-08-20 08:00:00 +0000 UTC LastReconciledAt:2026-08-18 08:00:00 +0000 UTC ReservedRunID: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:2}, error=<nil>
FAIL
FAIL	github.com/zhanglei10281852-gif/ai-featuremesh-base/internal/service	1.411s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令必须由修复前失败变为修复后通过；相关包、go test ./... -count=1、go vet ./... 和 linux/amd64 构建必须通过；回退 gold 关键修改后定向命令重新失败。
