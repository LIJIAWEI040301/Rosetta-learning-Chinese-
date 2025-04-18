---

# AbinitioRelax 从头折叠

适用于没有已知结构的蛋白质序列。

---

## 步骤 1：准备蛋白质序列（FASTA）

创建一个 FASTA 格式的文件，命名为：

```
t000_.fasta
```

如下：

```
>example_sequence
MKTIIALSYIFCLVFADYKDDDDK
```

---

## 步骤 2：准备 3-mer 和 9-mer 片段库

### 2.1 提交片段预测任务

1. 访问 [Robetta Fragment Server](https://robetta.bakerlab.org/fragmentsubmit.jsp)
2. 注册账号
3. Fragment libraries-submit
4. 提交任务：
   - 输入蛋白质序列
   - 填写任务名称
   - 点击 **Submit**

### 2.2 下载片段库文件

1. 在 **Fragment Libraries - Queue** 页面找到你的任务
2. 点击任意一个蓝色链接进入下载页面
3. 下载以下两个文件：
   - `aat000_03_05.200_v1_3`（3-mer 片段）
   - `aat000_09_05.200_v1_3`（9-mer 片段）

> 💡 注意：在 Edge 浏览器中，可能无法直接下载。可尝试点击后右键“另存为”，若保存为 `.txt`，就手动删除 `.txt` 后缀。

---

## 步骤 3：创建拓扑代理文件（Topology Broker）

创建文件 `topology_broker.tpb`，内容如下：

```
CLAIMER SequenceClaimer
FILE ./t000_.fasta
END_CLAIMER
```

---

## 步骤 4：创建 Options 文件

创建一个名为 `abinitio.options` 的文本文件，内容如下：

```bash
# 确保所有路径为绝对路径或相对路径，避免使用 $ 或 ~ 符号

-in
    -file
        # -native your_native.pdb                  # 可选：参考结构（若有）
        -fasta t000_.fasta                          # 蛋白质序列文件
        -frag3 aat000_03_05.200_v1_3                # 3-mer 片段库
        -frag9 aat000_09_05.200_v1_3                # 9-mer 片段库

-abinitio
    -increase_cycles 10                             # 每阶段循环次数增加10倍
    -rg_reweight 0.5                                # 半径项权重
    -rsd_wt_helix 0.5                                # 螺旋区域打分项权重
    -rsd_wt_loop 0.5                                 # loop 区域打分项权重
    -relax                                          # 折叠后执行一次 relax 操作

-relax
    -fast                                           # 使用 FastRelax 进行结构优化

-broker
    -setup topology_broker.tpb                      # 使用拓扑代理配置文件

-run
    -protocol broker                                # 使用 broker 协议
    -reinitialize_mover_for_each_job                # 每个 job 使用新的 mover 实例

-score
    -find_neighbors_3dgrid                          # 使用 3D 网格加速邻居查找

#-evaluation
#    -rmsd NATIVE _core your_core.txt               # 若有 native 结构，可取消注释以评估 RMSD

-out
    -nstruct 100                                    # 生成结构数量（建议至少 1000）
    -file
        -silent t000_broker.out                     # 输出 silent 文件
        -silent_struct_type binary                  # 使用二进制格式
        -scorefile t000_broker.sc                   # 输出打分文件
-overwrite                                          # 允许覆盖已有输出文件
```

---

## 步骤 5：运行 AbinitioRelax

使用以下命令运行：

```bash
/root/rosetta/rosetta.binary.ubuntu.release-371/main/source/bin/AbinitioRelax.default.linuxgccrelease @abinitio.options
```

---

## 步骤 6：提取预测结构（PDB）

运行以下命令从 silent 文件中提取结构：

```bash
/root/rosetta/rosetta.binary.ubuntu.release-371/main/source/bin/extract_pdbs.default.linuxgccrelease \
  -in:file:silent t000_broker.out \
  -in:file:silent_struct_type binary \
  -database /root/rosetta/rosetta.binary.ubuntu.release-371/main/database
```

提取出的结构将以 `S_*.pdb` 的形式保存在当前目录中。

---

## 步骤 7：结果解读（打分文件）

查看输出的打分文件：

```
t000_broker.sc
```

该文件包含每个模型的能量打分信息。一般而言：

- **score**：总能量分数（越低越好，绝对值越大越好）
- 可根据 score 排序，筛选出能量最低的结构用于后续分析

---

## 📫 联系方式

如有错误，麻烦指出，谢谢！
如有问题，欢迎联系我。
youxiang：lijiawei_jn at yeah.net

---
