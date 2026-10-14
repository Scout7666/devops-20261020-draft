# E7 B：DRAFT 教学工作流验证

## 现场结果（2026-10-14）

服务镜像 e7-draft:ta1020001，24 项单元测试通过。完整回归目录为 /home/hello/devops-course/20261020/B-draft/work/e7-b-final-msDMyB，整套门禁 passed=true，退出码 0。

| 场景 | 工作流终态 | 工作流退出码 | 测试结果 |
|---|---|---:|---|
| 正常候选回放 | SUCCEEDED | 0 | PASS |
| 预期输出不符 | FAILED | 1 | PASS |
| 候选改变 CMD | 构建前 FAILED | 1 | PASS |
| 修复候选加入 RUN exit 23 | FAILED | 1 | PASS |
| 受控容器启动延迟 | TIMED_OUT | 1 | PASS，运行容器已清理 |

正常回放 trace_id 为 e6-b-07e8c9d4555243dd849ea5fe37a823d6；运行超时 trace_id 为 e6-b-6096540e40db4a089c33c680d5278a70。候选无效在构建前被拒绝，当前 CLI 无 trace_id；这是当前可观测性限制，应如实记录。

## 网络故障与测试修正

初始运行超时样例通过修改 main.c 增加 sleep(10)，导致 COPY 后的依赖安装层缓存失效。首次因 Debian 软件源连接失败，随后下载约 83 MB 依赖时触发 300 秒及 1200 秒构建限时。三轮都未到运行阶段，不计为运行超时测试通过。

最终保持 main.c、Makefile 不变，在独立副本中同步修改初始和候选 Dockerfile 的 CMD，注入 sleep 10; exec ./hello，运行预算仍为 1 秒。变更来源单独写入 fault-injection.json。这验证容器运行预算与清理，不代表程序内部死锁复现，也不是新的模型回复。

修正后的单项目录 work/e7-b-delay-Mu0sgZ 通过，再执行全部五种场景得到上述完整回归结果。必须保留网络失败与最终通过两类证据。

## 交付与边界

提交工作流测试源码、门禁脚本、完整回归 gate-result.json/results.csv、各组日志/候选/输入副本/镜像信息，以及失败归因说明。模型候选是 E5 手工取得的 DeepSeek 回复，本轮为回放与受控故障测试。

本地门禁已验证；远程 CI 流水线、成功/失败运行链接、仓库合并保护尚未验证。三个指定项目、完整论文实验及真实模型生成成功率也不能据本样例声称完成。
