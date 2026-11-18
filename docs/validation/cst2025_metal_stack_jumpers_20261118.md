# 已建案例与 GitHub 覆盖比较

## 比较基线

- 日期：2026-11-18。
- 仓库：<https://github.com/huangqiantao/TPC>，main 当次读取提交 `3b741ea9db32ec95ab642d0c883b0afe0035e3e7`。
- 核对入口：`.github/skills/{tpc-user,topo-quickstart,cst-solver-dev}/SKILL.md`、`.dsh/skills/topo-quickstart/SKILL.md` 及其用户手册。
- 核对实现及文档：`skills/user/{tpc-usage,topo-quickstart,topo-modeler,hex-grid}.md`、`cst_solver/{parameters.py,material/materials.py,modeling/primitives.py,simulation/ports.py}`、`topo_modeler/builders/{feed,substrate}.py`、`docs/{SUPPORT_MATRIX.md,next_plan/README.md,packages/cst_solver.md,references/cst-official-api-reference.md,references/vba-official-reference.md,research/cst_vba_gap_analysis.md}`。

| 内容 | 基线覆盖情况 | 本技能处理 |
|---|---|---|
| 参数表达式、布尔消耗、阵列、历史刷新 | 已有通用说明和实现 | 引用，不再写通用教程 |
| 材料库 .mtd 导入、按范围建端口、xmin/xmax | 已有接口 | 继续复用，不复制实现 |
| 硅基三角孔 AB/BA 模板与 C6 环相关案例 | 已有模板或案例；不能等同本器件 | 不另写同类模板技能 |
| 两组矩形金属原胞、5/6 列交替、探针避让、连续 YIG、上下金属与外侧石英组合 | 所核对入口及实现中未见这一组合配方 | 保存组合关系及当前工程入口 |
| 短圆弧线体、落点接触与表面裁剪 | 所核对技能中未见此配方 | 保存已用的解析几何构造 |
| Torus 两半径含义 | `primitives.py` 186–218 行与 `vba-official-reference.md` §4.5 按 R、r 描述并直传；与此例实际尺寸不一致 | 记录 CST 2025 本例需要 R+r、R-r，提醒后续版本避免重复换算 |
| 端口范围和额外背景空间叠加 | 所核对技能中未见此案例提醒 | 保存实际计算域读回与重复扩大的经验 |

“未见”限定于以上核对范围和提交；不表示仓库每个历史 notebook 都没有相似结构。新版仓库如已收录这些内容，应收缩本技能并转向仓库正文，避免维护两份配方。

## 当前工程及证据入口

本次几何是在独立本机工程中完成的，原始 CST 工程和脚本未随本次文档提交上传。以下文件名是本机交付物线索，不是仓库内可点击的文件；下文保留参数与实际读回摘要，供跨机器使用。

| 本机交付物 | 用途 |
|---|---|
| `double_layer_topology.cst` | 当前器件工程；继续该器件时优先修改已有工程 |
| `build_double_layer.py` 与 `history_commands/` | 实际构造顺序与原生 VBA；跳线关键命令已摘入[配方](../../skills/user/references/cst-metal-stack-jumpers.md) |
| `double-layer-parameterized-geometry.json` | 参数及顶点表达式来源 |
| `simulation_setup.py` | 端口、背景、边界及实际设置读回 |
| `shape_inventory.txt`、`simulation_setup_actual.txt` | 原生对象清单及设置读回；关键值见下文 |

这些交付物没有上传到 GitHub，仓库文档不能代替本机工程文件。通用方法和可读的案例参数以仓库为查阅入口；接续具体项目仍须取得其当前工程和最新项目记录。

本机实际建模环境为 CST Studio Suite 2025，原生 Python 库来自安装目录 `AMD64/python_cst_libraries`，运行时 Python 3.12.14。它与仓库支持矩阵的其他验证组合分开记录，不据此承诺所有 CST/Python 版本兼容。构造脚本使用旧 `setup` 的任务级子类，没有修改原后端。

## 案例关键值（仅供接续本器件）

- 全部长度 mm。a=.664，l=a/√3；A 宽 .237、高 .1352、基准中心 `(0,-a√3/6-.032)`、相位 30°；B 宽 .04848、高 a/6、基准中心 `(0,a/3+a/12)`、相位 0°。各组绕原点 60° 复制共 6 个并集。
- 15 列 x 步长 1.5l，奇数列 5 个、偶数列 6 个，y 步长 a，各列居中，82 个/层。
- 左探针中心线 y=-1.328，宽 .1；剪除宽 .6。剪除至第三列右侧 A 矩形近侧边界，探针伸入；右侧仅 x 镜像。
- YIG 厚 .05，xy 边界约 x=±4.32492651084651、y=±1.992。金属位于 z=0 与 -.05，XY 完全正对。
- 左右石英各 .5×.5×.05，间隙 .01，微带宽 .233，底板复用石英边界。两根顶部跳线 r=.0085、中心线拱高 .02、落点距两侧边界均 .03。
- 用户选择所有金属面零厚度，材料 `Copper (pure)`；石英 `Quartz (Fused) (lossy)`。跳线为铜实体。YIG 使用 `YIG_PENDING` 占位，不是真实 YIG 参数。
- 3 个介质实体、2 个拓扑金属面、4 个石英金属面、2 个跳线实体，共 11 个对象。上下拓扑面体积均为 0，原生返回面积同为 32.1818007787591。
- 当前端口 1/2 在 x=±4.83492651084651，y∈[-1.5945,-1.0615]，z∈[-.05,.15]。实际计算域 x 同端口、y∈[-1.992,1.992]、z∈[-.05,.15]。
- 用户指定空气背景、六面 electric、120–130 GHz。上述是建模设置的证据，不是物理适用性或收敛性的结论。

## 使用证据的边界

此前只建模和读回设置，未由建模脚本启动求解。用户后来报告迭代求解及网格自适应警告，尚未完成该问题的模型诊断；不要把这里的几何修正当成收敛修复方案。YIG 电磁参数、磁偏置、端口模式及网格需在后续仿真任务中按要求处理。
