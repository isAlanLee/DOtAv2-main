# Codex 进展记录

## 2026-05-29 21:52:43 +08:00
- 开始处理用户请求：根据项目 README 整理虚拟环境配置相关命令。
- 检查到根目录存在 `README.md`，未发现既有 `codex.md`，因此新建本进展文档。
- 已读取 `README.md`、`requirements.txt`、`setup.py` 和 `logreplay/README.md`，确认根 README 将安装步骤指向 OpenCOOD/OPV2V 文档，本仓库提供 `requirements.txt` 与 `setup.py` 可用于安装 Python 依赖和本地包。

## 2026-05-29 21:53:08 +08:00
- 补充核对 README 引用的 OpenCOOD 安装文档，确认安装流程包括：Python 3.7 环境、安装项目依赖、按本机 CUDA 安装 PyTorch、安装 spconv、编译 `opencood/utils/setup.py` CUDA 扩展，FPV-RCNN 依赖为可选扩展。
- 准备向用户整理 Windows PowerShell 下的虚拟环境配置命令，并标明 CUDA/PyTorch 版本需要按本机环境选择。

## 2026-05-29 22:22:26 +08:00
- 用户反馈 `pip install -r requirements.txt` 安装 `timm` 依赖时失败，错误发生在 `safetensors` 的构建后端依赖安装阶段，提示 `puccinialin` 没有适配当前 Python 版本的可用发行包。
- 初步判断为 Python 3.7 与未固定版本的 `timm`/`safetensors` 最新依赖链不兼容，准备给出固定旧版本依赖并继续安装的命令。

## 2026-05-29 23:06:27 +08:00
- 用户反馈环境依赖已安装完成，要求给出正式运行命令，优先生成标签。
- 复核 README 与当前仓库文件，确认训练与推理入口为 `opencood/tools/train.py`、`opencood/tools/inference.py`、`opencood/tools/MBE.py`、`opencood/tools/box_score_for_mbe.py`。
- 发现 README 中 label-free yaml 路径存在差异：实际文件为 `opencood/hypes_yaml/point_pillar_intermediate_fusion_lable_free.yaml`（`lable` 拼写），不是 README 写的 `hypes_yaml/point_pillar_intermediate_fusion_label_free.yaml`。
- 发现 `MBE.py` 和 `box_score_for_mbe.py` 内部仍有多个硬编码数据路径，正式运行过滤/打分前需要先改为本机 OPV2V 数据、初始伪标签输出和最终伪标签输出目录。

## 2026-05-29 23:10:05 +08:00
- 用户说明实际在服务器上运行，且 `point_pillar_intermediate_fusion_lable_free.yaml` 内数据路径已经正确。
- 准备只给出 README 第一阶段命令：使用 label-free 配置训练初始检测器，作为后续伪标签生成的 checkpoint 来源。

## 2026-05-29 23:12:37 +08:00
- 用户要求后续所有正式运行命令都输出日志文件以便追溯，示例为通过 `2>&1 | tee logs/<name>_$(date +%Y%m%d_%H%M%S).log` 保存 stdout/stderr。
- 后续给服务器命令时将默认包含 `mkdir -p logs`、`set -o pipefail` 和带时间戳的 `tee` 日志输出。

## 2026-05-29 23:15:13 +08:00
- 用户运行 label-free 训练第一步时报错：`ValueError: invalid literal for int() with base 10: '.ipynb_checkpoints'`。
- 复核 `opencood/data_utils/datasets/basedataset.py`，确认代码会枚举每个场景目录下的子目录并将第一个 CAV 目录名转为整数；数据集中混入 `.ipynb_checkpoints` 隐藏目录会被误当作 CAV 目录。
- 处理建议：先在服务器数据集目录内查找并删除 `.ipynb_checkpoints` 目录，设置合法的 `OMP_NUM_THREADS`，再带日志重跑训练命令。

## 2026-05-30 11:19:42 +08:00
- 用户反馈 label-free 初始检测器训练已完成，checkpoint 保存到 `/autodl-fs/data/DOtAv2-opv2v/opencood/tools/../logs/point_pillar_intermediate_fusion_2026_05_29_23_26_20`。
- 下一阶段按照 README 执行 initial pseudo-label generation：使用该 checkpoint 运行 `opencood/tools/inference.py --fusion_method intermediate --pseudo_lable_save 0`，并创建 `pseduo_label_val/pre_box_test_full` 与 `pseduo_label_val/pre_score_test_full` 输出目录。

## 2026-05-30 11:35:47 +08:00
- 用户反馈 initial pseudo-label generation 已完成，`inference.py` 处理 6765 个样本，用 epoch 15 checkpoint，AP@0.3/0.5/0.7 均输出为 0.13。
- 下一阶段按照 README 进入 MBE 伪标签过滤：`python opencood/tools/MBE.py`，但需注意该脚本内部存在硬编码数据路径和伪标签输入/输出路径，运行前应先检查并按服务器实际路径修正。

## 2026-05-30 11:37:56 +08:00
- 用户展示 `opencood/tools/MBE.py` 中仍存在 `/mnt/32THHD/...` 等旧硬编码路径，要求直接修改为服务器对应地址，之后由用户推送到服务器。
- 已修改 `opencood/tools/MBE.py`：新增 `DATA_ROOT=/root/autodl-tmp/opv2v/train`、`PROJECT_ROOT=/autodl-fs/data/DOtAv2-opv2v`、`PSEUDO_BOX_DIR=<PROJECT_ROOT>/pseduo_label_val/pre_box_test_full`、`MBE_OUTPUT_DIR=<PROJECT_ROOT>/pseduo_label_val/mbe_filter`。
- 同步修正 MBE 场景循环中的潜在问题：过滤非数字 CAV 目录，函数内显式定义 `scenario_folder` 和 `timestamps`，并将固定 43 个场景改为根据 `len(scenario_folders)` 动态生成。
- 本地 `rg` 检查已确认 `MBE.py` 中不再存在旧的 `/mnt/32THHD`、`pseduo_label_moma_1`、`out_xqm_moma`、`F:\OPV2V` 路径；本地 `python -m py_compile` 因 Windows Python 登录会话问题未能执行，需在服务器同步后执行语法检查。
