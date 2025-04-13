以下是一个将singbox的shadowsocks转为v2ray的shadowsocks的脚本，添加了tproxy的路由标签
---

### **Python 脚本**

```python
import json
import argparse

# 定义命令行参数解析器
parser = argparse.ArgumentParser(description="Convert JSON data format.")
parser.add_argument("-i", "--input", required=True, help="Path to the input JSON file")
parser.add_argument("-o", "--output", required=True, help="Path to the output JSON file")
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
    "reslut": []  # 注意：这里拼写为 "reslut"，如果需要更正为 "result"，请告知
}

# 遍历 target 列表并进行转换
for item in input_data["target"]:
    try:
        converted_item = {
            "tag": item["tag"],  # 对应 tag
            "protocol": item["type"],  # 对应 type
            "settings": {
                "address": item["server"],  # 对应 server
                "method": item["method"],  # 对应 method
                "port": item["server_port"],  # 对应 server_port
                "password": item["password"]  # 对应 password
            },
            "streamSettings": {
                "socketSettings": {
                    "mark": 255,
                    "tcpFastOpen": True,
                    "tproxy": "tproxy"
                }
            }
        }
        output_data["reslut"].append(converted_item)
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
   - 使用命令行运行脚本，并通过 `-i` 和 `-o` 参数指定输入和输出文件路径。例如：
     ```bash
     python json_converter.py -i input.json -o output.json
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
     ]
   }
   ```

---

### **说明**
1. **命令行参数**：
   - `-i` 或 `--input`：指定输入文件的路径。
   - `-o` 或 `--output`：指定输出文件的路径。

2. **错误处理**：
   - 如果输入文件不存在或格式不正确，脚本会提示错误信息并退出。
   - 如果 `target` 列表中的某个元素缺少必要的字段（如 `tag`、`type` 等），脚本也会提示具体的错误信息。

3. **灵活性**：
   - 用户可以根据需要自由指定输入和输出文件的路径。
   - 支持批量处理 `target` 列表中的多个元素。

---

### **总结**
通过上述脚本，你可以轻松实现从命令行指定输入和输出文件路径的功能，并完成 JSON 数据的格式转换。如果还有其他需求或需要进一步调整，请随时告诉我！
