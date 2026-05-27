# Codex 进展记录

## 2026-05-27 20:20:00 +08:00
- 开始处理缺失 `yaml_utils` 的问题。
- 已检查项目根目录，确认此前不存在 `codex.md`。
- 已定位项目中多处代码引用 `opencood.hypes_yaml.yaml_utils`，目标路径为 `opencood/hypes_yaml/yaml_utils.py`。

## 2026-05-27 20:21:57 +08:00
- 已从 DerrickXuNu/OpenCOOD 仓库的 `main` 分支读取 `opencood/hypes_yaml/yaml_utils.py` 源码。
- 已在本项目新增 `opencood/hypes_yaml/yaml_utils.py`。
- 已验证模块可导入，`load_yaml` 与 `save_yaml_wo_overwriting` 可访问。

## 2026-05-27 20:22:43 +08:00
- 已使用 `load_yaml` 成功加载 `opencood/hypes_yaml/point_pillar_intermediate_fusion.yaml`。
- 验证结果包含配置名 `point_pillar_intermediate_fusion`，并成功计算 `anchor_args` 的 `W=704`、`H=200`。
- 已清理由验证产生的 `__pycache__` 缓存目录。

## 2026-05-27 20:24:43 +08:00
- 开始修改 label-free 对应 YAML。
- 已确认实际存在的配置文件为 `opencood/hypes_yaml/point_pillar_intermediate_fusion_lable_free.yaml`。
- 已确认代码使用字段拼写 `lable_free`，且 label-free 初始训练不读取 `pseudo_lable_path`。

## 2026-05-27 20:25:29 +08:00
- 已将 `point_pillar_intermediate_fusion_lable_free.yaml` 的 `root_dir` 修改为 `/root/autodl-tmp/opv2v/train`。
- 已将同一文件的 `validate_dir` 修改为 `/root/autodl-tmp/opv2v/train`，保持 label-free 初始训练使用 train 数据的原配置语义。
- 已通过 `load_yaml` 验证配置可解析，且 `lable_free=True`、`iterative_training=False` 保持不变。
- 已清理由验证产生的 `__pycache__` 缓存目录。

## 2026-05-27 20:28:06 +08:00
- 已将 `point_pillar_intermediate_fusion_lable_free.yaml` 的 `pseudo_lable_path` 修改为 `/root/autodl-tmp/opv2v/out_1`。
- 已通过 `load_yaml` 验证 `pseudo_lable_path`、`root_dir`、`validate_dir` 均为预期值。
- 已清理由验证产生的 `__pycache__` 缓存目录。
