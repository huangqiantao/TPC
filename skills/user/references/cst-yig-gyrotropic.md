# CST 中设置 YIG 的 Gyrotropic 磁色散

本说明适用于用户明确提供 YIG 的介电常数、磁性色散和偏置输入，并要求在 CST 建立真实材料的情况。若用户只要求几何而未提供物性，保留 `YIG_PENDING` 占位；不要把下列案例数值当成其他模型的默认值。

## CST 字段与输入检查

本次界面记录的参数为：

| CST 字段 | 本次输入 | 备注 |
|---|---:|---|
| General / Epsilon | 13 | 相对介电常数 |
| Magnetic dispersion model | Gyrotropic | 选择磁性色散模型 |
| Mu infinity | 1.0 | `μ∞` |
| Larmor frequency | `3.52e10 rad/s` | 角频率 |
| Gyrotropic frequency | `8.8e10 rad/s` | 角频率 |
| Damping factor | 0.0 | 按用户输入 |
| Biasing direction (x,y,z) | `(0, 0, 0.2)` | 按界面原值记录 |
| Parameter conversion | SI | 界面所选单位系统 |
| Biasing field | Homogeneous field | 本次界面所选偏置场类型 |

“Parameter conversion frequency”在截图中为 `0 GHz`，这是转换辅助字段，不是 Larmor 或 Gyrotropic 频率；不要把它写进材料的色散系数。

截图字段把偏置输入标为 direction，但 `(0,0,0.2)` 并非单位向量。本例只将三个分量原样保留到 CST，不对其归一化，也不推断其物理单位或等价偏置场强。用户换了偏置约定、单位系统或材料模型时，先确认其含义再转换。

## 可复用的原生 CST 宏模式

先在 CST 参数表中定义有单位/含义注释的模型参数，再将其作为材料系数传入。以下是本次 CST 2025 工程实际接受的原生命令结构；参数值是本案例输入，不能直接套给其他 YIG：

```vba
With Material
 .Reset
 .Name "YIG_Gyrotropic"
 .Folder ""
 .FrqType "all"
 .Type "Normal"
 .SetMaterialUnit "Hz", "mm"
 .Epsilon "yig_epsilon"
 .Mu "1"
 .Kappa "0"
 .DispModelMu "Gyrotropic"
 .UseSISystem "True"
 .MuInfinity "yig_mu_infinity"
 .DispCoeff1Mu "yig_larmor_omega"
 .DispCoeff2Mu "yig_gyrotropic_omega"
 .DispCoeff3Mu "yig_damping"
 .DispCoeff4MuX "yig_bias_x"
 .DispCoeff4MuY "yig_bias_y"
 .DispCoeff4MuZ "yig_bias_z"
 .Colour "0.64", "0.82", "0.88"
 .Transparency "0.25"
 .Create
End With
Solid.ChangeMaterial "Dielectrics:YIG", "YIG_Gyrotropic"
```

本案例的参数表对应值为 `yig_epsilon=13`、`yig_mu_infinity=1`、`yig_larmor_omega=3.52e10`、`yig_gyrotropic_omega=8.8e10`、`yig_damping=0`、`(yig_bias_x,yig_bias_y,yig_bias_z)=(0,0,0.2)`。两个频率输入保持为 CST 界面显示的 rad/s 数值；只有当来源数据用 Hz 表示时才转换成角频率（`ω=2πf`），不要对已经以 rad/s 给出的值再乘 `2π`。

`.SetMaterialUnit "Hz", "mm"`、`.UseSISystem "True"`、`.DispModelMu "Gyrotropic"` 和后续系数项构成本次已接受的 CST 原生设置模式。若 CST 版本、API 或单位系统不同，先检查生成的材料属性和实际读回值，不要仅凭宏无报错就认为参数解释一致。`Homogeneous field` 是本次界面记录的偏置场选择；若脚本接口没有对应字段，需在可用的 CST 材料设置中选定并读回，不能假定默认值正确。

`Solid.ChangeMaterial` 的第一个参数要使用当前模型实际的 `Component:Solid` 全名；上例的 `Dielectrics:YIG` 是本案例对象路径。`.Folder ""` 将材料放在根目录，赋值时使用材料的实际全名，避免材料文件夹导致查找失败。

## 最低限度读回

- 在材料属性中确认介电常数、磁性色散模型、`μ∞`、两个角频率、阻尼、偏置分量和单位系统。
- 检查 YIG 实体的材料名已变为创建的 Gyrotropic 材料名；对象树的材料清单或 CST 原生只读命令均可作为证据。
- 宏已被 CST 接受且材料已赋给实体，只能证明材料对象配置与赋值成功；不代表色散曲线、电磁响应、偏置方向约定或求解收敛已通过物理验证。需要验证这些内容时，应按用户要求另行检查材料响应并执行相应仿真。
