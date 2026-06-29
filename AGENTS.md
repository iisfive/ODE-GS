# AGENTS.md

本文件用于约束 Codex / AI coding agent 在本仓库中的行为。
请所有 agent 在执行任何任务前先阅读本文件，并严格遵守。

本仓库当前用途：复现并理解 ODE-GS: Latent ODEs for Dynamic Scene Extrapolation with 3D Gaussian Splatting，并为后续基于 ODE-GS 的动态 3D/4D scene extrapolation 方法创新做准备。

---

## 1. 总体原则

* 所有面向用户的解释、总结、报告、错误分析、阶段结论、下一步建议，默认使用中文。
* 代码变量名、命令、路径、配置字段、日志字段可以保持英文。
* 不要生成过多冗余文件、文件夹、临时脚本或重复日志。
* 不要在没有明确指令的情况下大规模重构代码。
* 不要在没有明确指令的情况下修改核心训练逻辑、模型结构、数据读取逻辑或配置默认值。
* 不要自动启动长时间训练、下载大数据集、上传大文件或删除重要文件。
* 每次任务应尽量做到：目标明确、改动最小、结果可追踪、方便 `git diff` 检查。
* 如果任务目标不明确，先用中文说明疑问和建议，不要擅自扩大任务范围。

---

## Conda 环境规则

当前项目使用 conda 环境 `odegs`。

运行任何 Python 命令前，必须确认：

```bash
which python
echo $CONDA_DEFAULT_ENV
echo $CONDA_PREFIX
python -c "import torch; print(torch.__version__, torch.cuda.is_available())"
```
期望结果：
/environment/miniconda3/envs/odegs/bin/python
CONDA_DEFAULT_ENV=odegs
CONDA_PREFIX=/environment/miniconda3/envs/odegs
torch.cuda.is_available() == True

如果 shell 变量显示为 base，但 Python 指向 odegs，需要在文档中标记为环境不一致，不要据此判断真实训练环境不可用。

优先使用：
```bash
source /environment/miniconda3/etc/profile.d/conda.sh
conda activate odegs
```

或直接使用：
```bash
/environment/miniconda3/envs/odegs/bin/python
```


然后提交：

```bash
git add AGENTS.md
git commit -m "Add conda environment rules for agents"
env -u LD_LIBRARY_PATH -u LD_PRELOAD GIT_SSH_COMMAND="/usr/bin/ssh -i /home/featurize/work/.ssh/id_rsa -o IdentitiesOnly=yes" /usr/bin/git push
```

---

## 2. 当前项目阶段

当前项目处于 ODE-GS 复现阶段，而不是方法创新阶段。

阶段目标优先级如下：

1. Stage 0：多数据集仓库扫描、环境检查、入口脚本参数检查。
2. Stage 1：D-NeRF 多场景 smoke test。
3. Stage 2：D-NeRF 多场景正式复现。
4. Stage 3：HyperNeRF smoke test。
5. Stage 4：NeRF-DS smoke test。
6. Stage 5：跨数据集结果整理与复现实验矩阵总结。
7. Stage 6：基于复现结果进行方法级修改和研究创新。

在 Stage 0 中，只允许运行非训练命令，例如：

```bash
python train_interpolation.py --help
python train_extrapolation.py --help
python evaluate_extrapolation.py --help
```

不要启动完整训练。

---

## 3. 多数据集复现策略

本项目不是只复现单个 D-NeRF 场景，而是逐步复现 ODE-GS 在多类动态场景数据上的表现。

优先级如下：

### 3.1 D-NeRF synthetic scenes

* 先做 smoke test。
* 再选择多个代表场景正式训练。
* synthetic 数据运行时通常需要 `--is_blender`。
* 优先作为第一批复现对象，因为数据格式相对规整，适合验证完整 pipeline。

建议优先考虑的场景包括但不限于：

* `hook`
* `standup`
* `jumpingjacks`
* `trex`
* `mutant`

具体以实际下载到的数据为准。

### 3.2 HyperNeRF real-world scenes

* 在 D-NeRF 链路跑通后进行。
* real-world 数据运行时通常不使用 `--is_blender`。
* 重点记录数据格式、相机读取、显存占用、运行错误和真实动态场景上的复现难点。

### 3.3 NeRF-DS real-world scenes

* 作为第二类真实数据补充。
* 重点验证 ODE-GS 在真实动态场景上的泛化复现情况。
* 同样需要重点记录数据路径、相机格式、读入方式和潜在错误。

### 3.4 NVFi synthetic scenes

* 如果数据获取顺利，再作为额外 synthetic 数据补充。
* 不作为第一优先级，但需要在 Stage 0 中检查代码是否有对应 dataloader 或配置入口。

每个数据集都应先完成 smoke test，再决定是否进入正式训练。不要在没有 smoke test 的情况下直接启动长时间训练。

---

## 4. 允许优先创建或修改的文件

优先将复现记录、检查结果和实验计划写入以下目录：

* `docs/`：复现日志、环境检查、阶段报告、错误分析、实验总结。
* `scripts/`：可复用的运行脚本、检查脚本、smoke test 脚本。
* `envs/`：环境记录文件，例如 conda / pip freeze。
* `configs/`：只有在明确需要配置实验参数时才修改或新增配置文件。

除非用户明确要求，不要随意创建新的顶层目录。

---

## 5. 禁止加入 Git 的内容

不要将以下内容加入 git：

* 数据集目录：`data/`、`datasets/`
* 训练输出：`output/`、`outputs/`、`logs/`、`runs/`
* wandb 日志：`wandb/`
* checkpoint 或模型权重：`*.pth`、`*.pt`、`*.ckpt`
* 大型渲染结果：`*.mp4`、`*.avi`、`*.mov`
* 大量图片结果：`*.png`、`*.jpg`、`*.jpeg`
* numpy 中间文件：`*.npy`、`*.npz`
* Python 缓存：`__pycache__/`、`*.pyc`
* notebook 缓存：`.ipynb_checkpoints/`
* 临时文件：`tmp/`、`temp/`

如果发现这些文件出现在 `git status` 中，应提醒用户检查 `.gitignore`，不要直接提交。

---

## 6. 每次任务开始前

在执行任何修改前，先检查：

```bash
pwd
git branch
git status
```

并确认当前分支是否为预期分支，例如：

```bash
repro/baseline-setup
```

如果工作区不干净，先说明当前已有改动，不要直接覆盖。

如果需要修改文件，应先说明：

* 为什么需要修改；
* 预计修改哪些文件；
* 是否涉及核心代码；
* 是否会影响原始 ODE-GS 方法逻辑。

---

## 7. 每次任务结束后

每次任务结束时，需要输出中文总结，包括：

* 本轮完成了什么；
* 修改了哪些文件；
* 运行了哪些命令；
* 是否有报错；
* 当前是否可以进入下一阶段；
* 建议用户执行哪些 git 命令提交结果。

不要自动执行 `git commit` 或 `git push`，除非用户明确要求。

---

## 8. Git 管理规则

当前仓库使用如下远程关系：

* `origin`：用户自己的 fork，用于保存复现过程。
* `upstream`：原作者仓库，用于同步官方更新。

不要向 upstream push。

当前推荐分支：

```bash
repro/baseline-setup
```

如果需要新建阶段分支，应先询问用户。推荐命名：

```bash
repro/stage1-dnerf-smoke
repro/stage2-dnerf-multiscene
repro/stage3-hypernerf-smoke
repro/stage4-nerfds-smoke
research/method-prototype
```

由于当前服务器环境中 conda 可能污染 OpenSSL，普通 `git push` 可能失败。需要提醒用户使用 clean push：

```bash
env -u LD_LIBRARY_PATH -u LD_PRELOAD GIT_SSH_COMMAND="/usr/bin/ssh -i /home/featurize/work/.ssh/id_rsa -o IdentitiesOnly=yes" /usr/bin/git push
```

除非用户明确要求，不要自动执行 push。

---

## 9. 文档命名规范

复现相关文档统一放在 `docs/` 下，推荐命名：

```text
docs/reproduce_log.md
docs/codex_stage0_repo_scan.md
docs/codex_stage1_dnerf_smoke.md
docs/codex_stage2_dnerf_multiscene.md
docs/codex_stage3_hypernerf_smoke.md
docs/codex_stage4_nerfds_smoke.md
docs/experiment_matrix.md
docs/error_notes.md
docs/experiment_plan.md
```

不要随意创建多个类似名称的重复文件，例如：

```text
note.md
notes2.md
test_result_new.md
final_final_report.md
```

如果已有合适文档，应优先更新已有文档，而不是新建重复文档。

---

## 10. 脚本命名规范

可复用脚本统一放在 `scripts/` 下，推荐命名：

```text
scripts/check_env.sh
scripts/run_dnerf_hook_interpolation_smoke.sh
scripts/run_dnerf_hook_extrapolation_smoke.sh
scripts/run_dnerf_hook_eval_smoke.sh
scripts/run_hypernerf_interpolation_smoke.sh
scripts/run_nerfds_interpolation_smoke.sh
```

脚本应尽量简洁，并在开头注明用途，例如：

```bash
#!/usr/bin/env bash
# 用途：检查 ODE-GS 当前环境和 CUDA extension 是否可用。
```

不要生成大量一次性脚本。如果只是临时命令，优先写入文档，而不是创建脚本。

---

## 11. 实验矩阵规范

如果涉及多个数据集或多个场景，需要维护：

```text
docs/experiment_matrix.md
```

建议表格格式：

```markdown
# ODE-GS Experiment Matrix

| Dataset | Scene | Stage | Status | Command / Config | Output Path | Notes |
|---|---|---|---|---|---|---|
| D-NeRF | hook | interpolation smoke | pending | TBD | TBD | synthetic, use --is_blender |
| D-NeRF | standup | interpolation smoke | pending | TBD | TBD | synthetic, use --is_blender |
| HyperNeRF | TBD | interpolation smoke | pending | TBD | TBD | real-world, no --is_blender |
| NeRF-DS | TBD | interpolation smoke | pending | TBD | TBD | real-world, no --is_blender |
| NVFi | TBD | optional | pending | TBD | TBD | data availability TBD |
```

每次实验后，应更新该表格中的状态和备注。

---

## 12. 修改代码原则

如果需要修改 Python 源码，必须遵守：

* 优先定位问题，不要直接大改。
* 优先添加小范围兼容性修复。
* 修改前说明原因。
* 修改后说明影响范围。
* 不要改变原始方法逻辑，除非用户明确要求进入方法创新阶段。
* 对任何核心文件修改，都要说明对应的测试方式。
* 如果只是为了复现兼容环境，优先将修改写成最小 patch。
* 如果只是为了记录命令或实验参数，优先新增 `docs/` 或 `scripts/` 文件，而不是修改核心代码。

核心文件包括但不限于：

```text
train_interpolation.py
train_extrapolation.py
evaluate_extrapolation.py
scene/
gaussian_renderer/
utils/
arguments/
```

---

## 13. 配置文件原则

如果需要新增配置文件，优先放在 `configs/` 下，并使用清晰命名：

```text
configs/repro_dnerf_hook_smoke.yaml
configs/repro_hypernerf_smoke.yaml
configs/repro_nerfds_smoke.yaml
```

不要覆盖官方默认配置，除非用户明确要求。

如果修改了配置文件，必须在总结中说明：

* 修改了哪些字段；
* 修改原因；
* 预计影响；
* 是否只是 smoke test；
* 是否适合正式训练。

---

## 14. 训练和实验运行原则

不要自动启动长时间训练。

如果用户要求运行训练，应先确认：

* 数据集路径是否存在；
* 输出目录是否在 `.gitignore` 中；
* 当前 GPU 是否可用；
* 预计运行时间；
* 预计显存占用；
* 是否只是 smoke test；
* 是否需要保存 checkpoint。

对于 smoke test，优先使用小迭代数、小场景、小输出目录，目标是验证 pipeline，而不是追求指标。

对于正式复现，必须先写清楚：

* 数据集；
* 场景；
* 命令；
* 配置；
* 输出目录；
* checkpoint 路径；
* 评估方式；
* 预期产物；
* 如何记录结果。

---

## 15. Stage 0 任务要求

Stage 0 的目标是完成多数据集仓库扫描和环境检查。

需要阅读：

```text
README.md
train_interpolation.py
train_extrapolation.py
evaluate_extrapolation.py
configs/default_config.yaml
configs/render_hyper.yaml
configs/
arguments/
scene/
gaussian_renderer/
utils/
```

需要总结：

* ODE-GS 官方复现 pipeline；
* 数据集目录结构；
* D-NeRF / NVFi / HyperNeRF / NeRF-DS 的差异；
* interpolation stage 如何运行；
* extrapolation / ODE stage 如何运行；
* evaluation 如何运行；
* checkpoint 和输出目录在哪里；
* synthetic 和 real-world 数据的关键参数差异；
* 当前环境是否能进入 Stage 1。

只允许运行：

```bash
python train_interpolation.py --help
python train_extrapolation.py --help
python evaluate_extrapolation.py --help
```

除非用户明确要求，不要运行训练命令。

Stage 0 输出文档：

```text
docs/codex_stage0_repo_scan.md
```

---

## 16. Stage 1 任务要求

Stage 1 的目标是 D-NeRF 多场景 smoke test。

推荐先选择：

```text
D-NeRF/hook
D-NeRF/standup
D-NeRF/jumpingjacks 或 D-NeRF/trex
```

每个场景先验证：

* 数据路径存在；
* 相机和图像能读取；
* interpolation 训练能启动；
* checkpoint 能保存；
* output 目录没有进入 git；
* 训练不会立即出现 import / CUDA / dataloader 错误。

Stage 1 输出文档：

```text
docs/codex_stage1_dnerf_smoke.md
```

如果需要脚本，优先放在：

```text
scripts/run_dnerf_hook_interpolation_smoke.sh
scripts/run_dnerf_standup_interpolation_smoke.sh
```

---

## 17. 错误处理原则

遇到错误时，不要立刻大改代码。应先记录：

* 运行命令；
* 完整错误信息；
* 错误出现阶段；
* 可能原因；
* 最小修复建议；
* 是否需要用户确认。

错误记录优先写入：

```text
docs/error_notes.md
```

如果错误属于当前阶段，也可以写入当前阶段文档。

---

## 18. 输出格式要求

每次给用户的最终总结应使用中文，并尽量采用以下结构：

```markdown
## 本轮完成

## 修改文件

## 运行命令

## 结果与问题

## 是否可以进入下一步

## 建议用户执行的 git 命令
```

不要输出大段无关解释。不要反复生成重复建议。不要为了显得完整而创建不必要文件。

---

## 19. 安全和数据管理

不要删除用户数据、训练输出或 checkpoint，除非用户明确要求。

如果需要清理文件，应先列出将删除的路径，并等待用户确认。

不要把数据集、模型权重、渲染结果、视频结果加入 git。

如果发现 `.gitignore` 不完整，应先建议补充 `.gitignore`，再继续。

---

## 20. 当前默认目标

当前默认目标是：

```text
完成 ODE-GS 的多数据集复现准备，建立可追踪、可扩展、可复查的复现记录体系。
```

短期目标：

```text
Stage 0 multi-dataset repo scan -> Stage 1 D-NeRF multi-scene smoke test -> Stage 2 D-NeRF formal reproduction -> Stage 3/4 real-world dataset smoke tests
```

长期目标：

```text
在充分理解 ODE-GS 代码、数据、训练和评估 pipeline 后，再进入动态 3D/4D scene extrapolation 的方法创新阶段。
```
