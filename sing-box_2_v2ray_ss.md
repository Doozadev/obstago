以下是一个将singbox的shadowsocks转为v2ray的shadowsocks的脚本
---

### **Python 脚本**

```python
import json
import argparse

# 定义命令行参数解析器
parser = argparse.ArgumentParser(description="Convert JSON data format with optional streamSettings.")
parser.add_argument("-i", "--input", required=True, help="Path to the input JSON file")
parser.add_argument("-o", "--output", required=True, help="Path to the output JSON file")
parser.add_argument("-m", "--mode", choices=["true", "false"], default="true", help="Whether to include 'streamSettings' (true/false)")
args = parser.parse_args()

# 读取输入的 JSON 文件
try:
    with open(args.input, "r", encoding="utf-8") as f:
        input_data = json.load(f)
except FileNotFoundError:
    print(f"Error: Input file '{args.input}' not found.")
    exit(1)
except json.JSONDecodeError:
    print(f"Error: Invalid JSON format in '{args.input}'.")
    exit(1)

# 检查输入数据是否包含 "target" 列表
if "target" not in input_data or not isinstance(input_data["target"], list):
    print("Error: Input JSON must contain a 'target' list.")
    exit(1)

# 初始化输出数据结构
output_data = {
    "reslut": [],  # 注意：这里拼写为 "reslut"，如果需要更正为 "result"，请告知
    "tags": []     # 新增字段，用于保存所有 tag 的集合
}

# 遍历 target 列表并进行转换
for item in input_data["target"]:
    try:
        # 构建基础转换结果
        converted_item = {
            "tag": item["tag"],  # 对应 tag
            "protocol": item["type"],  # 对应 type
            "settings": {
                "address": item["server"],  # 对应 server
                "method": item["method"],  # 对应 method
                "port": item["server_port"],  # 对应 server_port
                "password": item["password"]  # 对应 password
            }
        }

        # 根据 -m 参数决定是否添加 streamSettings
        if args.mode == "true":
            converted_item["streamSettings"] = {
                "socketSettings": {
                    "mark": 255,
                    "tcpFastOpen": True,
                    "tproxy": "tproxy"
                }
            }

        output_data["reslut"].append(converted_item)

        # 将 tag 添加到 tags 集合（去重）
        if item["tag"] not in output_data["tags"]:
            output_data["tags"].append(item["tag"])

    except KeyError as e:
        print(f"Error: Missing key {e} in one of the target items.")
        exit(1)

# 将结果写入输出的 JSON 文件
try:
    with open(args.output, "w", encoding="utf-8") as f:
        json.dump(output_data, f, indent=2, ensure_ascii=False)
    print(f"转换完成！结果已保存到 {args.output}")
except Exception as e:
    print(f"Error: Failed to write to output file '{args.output}'. Error: {e}")
    exit(1)
```

---

### **运行方式**
1. **保存脚本**：
   - 将上述代码保存为一个 Python 文件，例如 `json_converter.py`。

2. **运行脚本**：
   - 使用命令行运行脚本，并通过 `-i` 和 `-o` 参数指定输入和输出文件路径，同时使用 `-m` 参数控制是否添加 `streamSettings`。例如：
     ```bash
     python json_converter.py -i input.json -o output.json -m true
     ```
   - 如果不需要 `streamSettings`，可以将 `-m` 参数设置为 `false`：
     ```bash
     python json_converter.py -i input.json -o output.json -m false
     ```

3. **示例输入文件**（`input.json`）：
   ```json
   {
       "target": [
           {
               "type": "shadowsocks",
               "tag": "日本-小犬-IEPL",
               "server": "mb.blobsms.com",
               "server_port": 33399,
               "method": "chacha20-ietf-poly1305",
               "password": "passwd"
           },
           {
               "type": "shadowsocks",
               "tag": "美国-大猫-IEPL",
               "server": "us.example.com",
               "server_port": 44499,
               "method": "aes-256-gcm",
               "password": "anotherpasswd"
           }
       ]
   }
   ```

4. **生成的输出文件**（`output.json`）：
   - 当 `-m true` 时：
     ```json
     {
       "reslut": [
         {
           "tag": "日本-小犬-IEPL",
           "protocol": "shadowsocks",
           "settings": {
             "address": "mb.blobsms.com",
             "method": "chacha20-ietf-poly1305",
             "port": 33399,
             "password": "passwd"
           },
           "streamSettings": {
             "socketSettings": {
               "mark": 255,
               "tcpFastOpen": true,
               "tproxy": "tproxy"
             }
           }
         },
         {
           "tag": "美国-大猫-IEPL",
           "protocol": "shadowsocks",
           "settings": {
             "address": "us.example.com",
             "method": "aes-256-gcm",
             "port": 44499,
             "password": "anotherpasswd"
           },
           "streamSettings": {
             "socketSettings": {
               "mark": 255,
               "tcpFastOpen": true,
               "tproxy": "tproxy"
             }
           }
         }
       ],
       "tags": [
         "日本-小犬-IEPL",
         "美国-大猫-IEPL"
       ]
     }
     ```

   - 当 `-m false` 时：
     ```json
     {
       "reslut": [
         {
           "tag": "日本-小犬-IEPL",
           "protocol": "shadowsocks",
           "settings": {
             "address": "mb.blobsms.com",
             "method": "chacha20-ietf-poly1305",
             "port": 33399,
             "password": "passwd"
           }
         },
         {
           "tag": "美国-大猫-IEPL",
           "protocol": "shadowsocks",
           "settings": {
             "address": "us.example.com",
             "method": "aes-256-gcm",
             "port": 44499,
             "password": "anotherpasswd"
           }
         }
       ],
       "tags": [
         "日本-小犬-IEPL",
         "美国-大猫-IEPL"
       ]
     }
     ```

---

### **说明**
1. **新增参数 `-m`**：
   - `-m true`：在每个元素中添加 `streamSettings` 字段。
   - `-m false`：不添加 `streamSettings` 字段。

2. **灵活性**：
   - 用户可以根据需求灵活选择是否添加 `streamSettings`。
   - 默认情况下，`-m` 参数为 `true`，即默认添加 `streamSettings`。

3. **错误处理**：
   - 如果输入文件不存在、格式不正确，或 `target` 列表中的某个元素缺少必要字段，脚本会提示具体的错误信息。

---
