prompt:选择适当的编程语言（运行环境：Linux Ubuntu），帮我写一个程序，实现大文件的拆分与还原（用户可以指定拆分个数，拆分后的文件大小相近）。（例：将一个约7.5G的文件拆分成大小相近的2个文件）

**推荐编程语言：Python 3**

**原因**：  
- Ubuntu 默认自带 Python 3（无需额外安装）。  
- 处理大文件（7.5G+）非常高效，使用二进制流 + 缓冲区，不会把整个文件加载到内存。  
- 代码简洁、跨平台、可读性高，用户只需一条命令即可拆分/还原。  
- 拆分后各文件大小**几乎完全相等**（最多相差 1 字节），完全满足你的需求。

### 完整程序（保存为 `file_splitter.py`）

```python
# -*- coding: utf-8 -*-
import argparse
import os
import sys

def split_file(input_file: str, num_parts: int):
    """拆分大文件为 num_parts 个大小相近的文件"""
    if num_parts < 2:
        print("❌ 拆分个数至少为 2")
        return

    try:
        total_size = os.path.getsize(input_file)
    except FileNotFoundError:
        print(f"❌ 文件不存在：{input_file}")
        return

    chunk_size = total_size // num_parts
    remainder = total_size % num_parts

    print(f"📊 文件大小：{total_size:,} 字节")
    print(f"🔪 拆分成 {num_parts} 部分，每部分约 {chunk_size:,} 字节")

    with open(input_file, 'rb') as infile:
        for part_num in range(1, num_parts + 1):
            part_size = chunk_size + (1 if part_num <= remainder else 0)
            part_file = f"{input_file}.{part_num:03d}"   # 示例：bigfile.001

            print(f"正在写入 → {part_file}（{part_size:,} 字节）...")
            with open(part_file, 'wb') as outfile:
                remaining = part_size
                buffer_size = 64 * 1024 * 1024  # 64MB 缓冲区，速度最优
                while remaining > 0:
                    read_size = min(buffer_size, remaining)
                    data = infile.read(read_size)
                    if not data:
                        break
                    outfile.write(data)
                    remaining -= len(data)

    print("✅ 拆分完成！")


def merge_files(output_file: str, part_files: list):
    """按顺序合并多个部分文件"""
    if not part_files:
        print("❌ 需要至少提供一个部分文件")
        return

    print(f"🔄 正在合并到 → {output_file}")

    with open(output_file, 'wb') as outfile:
        for part in part_files:
            if not os.path.exists(part):
                print(f"❌ 部分文件不存在：{part}")
                return
            print(f"读取 ← {part}")
            with open(part, 'rb') as infile:
                buffer_size = 64 * 1024 * 1024
                while True:
                    data = infile.read(buffer_size)
                    if not data:
                        break
                    outfile.write(data)

    print("✅ 合并完成！")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="大文件拆分与还原工具（支持任意大小）")
    subparsers = parser.add_subparsers(dest="command", required=True)

    # 拆分命令
    split_p = subparsers.add_parser("split", help="拆分文件")
    split_p.add_argument("input_file", help="要拆分的原文件路径")
    split_p.add_argument("num_parts", type=int, help="拆分个数（≥2）")

    # 合并命令
    merge_p = subparsers.add_parser("merge", help="合并文件")
    merge_p.add_argument("output_file", help="合并后的输出文件名")
    merge_p.add_argument("part_files", nargs="+", help="部分文件列表（必须按顺序）")

    args = parser.parse_args()

    if args.command == "split":
        split_file(args.input_file, args.num_parts)
    elif args.command == "merge":
        merge_files(args.output_file, args.part_files)
```

### 使用方法（Ubuntu 终端）

1. **保存代码**  
   ```bash
   cat > file_splitter.py << 'EOF'
   # 把上面的完整代码粘贴到这里
   EOF
   ```

2. **给权限**（可选）  
   ```bash
   chmod +x file_splitter.py
   ```

3. **示例：把 7.5G 文件拆成 2 个大小相近的文件**  
   ```bash
   python3 file_splitter.py split /path/to/your-7.5G-file 2
   ```
   生成两个文件：  
   `/path/to/your-7.5G-file.001`  
   `/path/to/your-7.5G-file.002`  
   （每个约 3.75G）

4. **还原（合并）**  
   ```bash
   python3 file_splitter.py merge restored-file /path/to/your-7.5G-file.001 /path/to/your-7.5G-file.002
   ```
   得到完整的 `restored-file`（与原文件完全一致）。

### 其他常用例子
- 拆分成 5 份：`python3 file_splitter.py split bigfile 5`
- 合并任意多个部分：`python3 file_splitter.py merge newfile part1 part2 part3 part4 part5`

**优点**：  
- 纯标准库，无需任何 pip 安装。  
- 支持任意大文件（10G、50G、100G 都可）。  
- 缓冲区 64MB，读写速度最优。  
- 部分文件名带 `.001`、`.002` 自动排序，永不混乱。

把代码保存好，直接运行即可！需要我再帮你加上进度条、MD5 校验或 GUI 版本吗？随时说～
