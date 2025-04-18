请参照[这里](https://github.com/LIJIAWEI040301/Rosetta-learning-Chinese-/blob/main/%E5%85%A8%E5%B1%80%E5%AF%B9%E6%8E%A5.md#2-%E7%BB%93%E6%9E%84%E5%87%86%E5%A4%87relax)

---

## 或使用如下脚本



### code

```bash
#!/bin/bash
# 并行运行 Rosetta relax 协议脚本

# Rosetta relax 可执行文件路径
ROSETTA_BIN="/root/rosetta/rosetta.binary.ubuntu.release-371/main/source/bin/relax.linuxgccrelease"
ROSETTA_DB="/root/rosetta/rosetta.binary.ubuntu.release-371/main/database/"

# 原始 PDB 文件列表（每行一个 PDB 文件路径）
PDB_LIST="pdb_files.list"

# 并行任务数（根据你的机器 CPU 核心数进行设置）
N_JOBS=70

# 每个任务处理的结构数
TOTAL_STRUCTS=$(wc -l < "$PDB_LIST")
STRUCTS_PER_JOB=$(( (TOTAL_STRUCTS + N_JOBS - 1) / N_JOBS ))

# 输出目录
OUTDIR="relax_output"
mkdir -p "$OUTDIR"

# 拆分 PDB 文件列表为多个子列表
split -l $STRUCTS_PER_JOB "$PDB_LIST" "$OUTDIR/pdb_sublist_"

# 启动并行 relax 任务
i=0
for sublist in $OUTDIR/pdb_sublist_*; do
    SCOREFILE="$OUTDIR/score_relax_$i.sc"
    LOGFILE="$OUTDIR/log_relax_$i.txt"
    SUFFIX="_relax_$i"

    echo "启动任务 $i，处理结构列表：$sublist"

    $ROSETTA_BIN \
        -database "$ROSETTA_DB" \
        -ignore_unrecognized_res \
        -relax:constrain_relax_to_start_coords \
        -ex1 -ex2 -use_input_sc \
        -l "$sublist" \
        -out:file:scorefile "$SCOREFILE" \
        -out:suffix "$SUFFIX" \
        -out:path:all "$OUTDIR" \
        > "$LOGFILE" 2>&1 &

    ((i++))
done

# 等待所有后台任务完成
wait
echo "完成！！！。结果保存在 $OUTDIR/"
```

### run
执行脚本：
   ```bash
   bash run_parallel_relax.sh
   ```

---

## 📫 联系方式

如有错误，麻烦指出，谢谢！
如有问题，欢迎联系我。
youxiang：lijiawei_jn at yeah.net

---
