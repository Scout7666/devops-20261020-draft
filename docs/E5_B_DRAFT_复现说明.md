# E5 B轨道 DRAFT 人工转交模型复现说明

实验日期：2026-10-13。证据来自用户在本任务中回传的终端记录与模型回答，尚未由助手独立重跑。单个课程人工项目已完成失败构建、提示、候选选择、成功构建与运行验证。

## 输入与版本

- 工作仓库：/home/hello/devops-course/20261020/B-draft。
- 初始模板提交：a4088bc。
- 实验目录：work/e5-b-XsZXMk。
- 被测项目：课程人工样例Tiny Greeting，构建make，运行./hello，期望标准输出hello E3。
- E4服务镜像：e4-draft:TA1020001，ID为sha256:6c00d18770393e3a9447d79fee5aed2d49bff782f0d51e46e02111577b45fd6d。
- 服务容器Docker客户端28.5.1，宿主机Docker引擎29.1.3，服务容器Python3.13.14。
- 本实验被测Dockerfile固定基础镜像：python:3.13-slim@sha256:70729b46c69b4f1e97c4822c1af3df53a1476cf5ddc6c087c0c10bc3a5678c2f。
- 模型来源：用户选择Deepseek，人工在对话界面转交提示并回传3份候选；具体模型版本与采样参数未提供。

被测项目基础镜像摘要与E4服务镜像基础摘要不同，分别记录，不混为同一个环境版本。实验副本未生成独立源码commit，需保留project目录源文件和内容摘要。

## 环境与异常记录

make doctor通过；服务镜像构建成功；指定镜像单元测试3 passed且退出码0；make scan未发现问题。

镜像下载曾出现EOF。一次make smoke的JSON显示passed=true，但外层Make返回Error 1，不能将该外层命令登记为正常通过。随后直接运行程序得到网络失败时，程序正确报告passed=false并返回1。外层Compose返回问题尚未定位，后续演练使用明确镜像、非交互docker run与显式退出码记录。

第一次独立实验构建报/lab/project not found，未进入项目构建。调整服务容器用户ID与Ubuntu执行者一致后，容器可读取挂载文件。这类目录访问失败和网络失败不作为DRAFT目标项目缺少工具链的预期失败。

## 第一轮真实失败

iter-01/Dockerfile保留固定FROM、WORKDIR /work、COPY main.c Makefile、RUN make和CMD ./hello，没有安装构建工具。

有效失败证据：iter-01/build-retry.log与result-retry.json。

- Docker构建退出码：1。
- 已执行到RUN make。
- 错误：/bin/sh: 1: make: not found。
- 配方内部退出码：127。

iter-01/build.log和result.json是此前目录访问失败，保留作排障记录，不混入有效构建失败。

## 提示与候选

prepare_e5_prompts.py读取真实失败日志、Dockerfile和项目README，生成error-localization.json、prompt-error-driven.txt和prompt-context-driven.txt。两类提示为课程改写，不是论文补充材料模板的逐字复制。脚本没有模型调用。

本轮使用error-driven提示获取3个候选；context-driven提示已准备，但尚未调用。3份候选均保留固定基础镜像、源码、构建命令和运行命令，新增build-essential安装与apt列表清理。

| 候选 | 指令顺序 | 排版 |
|---|---|---|
| 1 | COPY后安装工具，再make | apt安装命令分行续写 |
| 2 | 安装工具后COPY，再make | apt安装命令分行续写 |
| 3 | COPY后安装工具，再make | apt安装命令单行 |

候选1和3原始文本不同；明确采用合并反斜杠续行、清除行首尾排版空白并保留指令顺序的教学比较规则后，两份逻辑指令一致。按这一规则的一致候选优先策略选择候选1，平局取该组编号最小者，依据保存在decision.json。不能宣称原始文本逐字相同，也未核验原论文工具是否使用同一归一化实现。

没有使用参考Dockerfile代替模型候选，也没有将人工回答获取时间当作纯API模型调用耗时。

## 第二轮构建与功能验证

候选1复制为iter-02/Dockerfile，使用E4服务容器内Docker客户端、宿主机Docker引擎执行构建，保存iter-02/build.log与result.json。

- 构建退出码0。
- 日志显示build-essential安装完成。
- RUN make执行cc main.c -o hello并成功完成。
- 生成镜像：e5-draft-fixed:ta1020001。
- 构建日志中的镜像清单标识：sha256:c4c5de32259b7c66a4437227242f61e4113a033dbd516aeadc64b64395161ed5。

在禁用网络的运行容器中执行镜像默认命令，保存verify.log、verify-stderr.log和verify-result.json：

- 标准输出hello E3。
- 验证退出码0。
- 错误日志为空。

因此本例同时满足构建成功与课程功能验证。镜像inspect完整记录仍需按交付步骤保存，不能仅依赖构建日志中的tag。

## 当前结论与范围

E5完成了一个预先给定错误Dockerfile的修复案例：真实失败→日志定位与提示→人工获取3个Deepseek候选→选择→真实构建→禁网程序验证。

这不是完整的无人值守DRAFT实现：初始Dockerfile由课程提供，模型经人工转交；尚无自动模型适配、连续候选编排、完整终止策略和通用日志定位。E6将核心过程工具化；E7继续验证异常行为、测试与CI门禁，并按课程要求扩展指定项目。本例不能用于声称复现论文成功率或效率数值。

方法依据：ICSE DRAFT论文PDF第3—5页；课程原设计§3；E5课件B轨道步骤。候选归一化与人工转交为已披露的课程实施差异。
