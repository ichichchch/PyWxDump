# Python API

PyWxDump 可以作为 Python 库导入使用。

## 安装

```bash
pip install pywxdump
```

## 快速开始

```python
from pywxdump import *

# 获取微信信息
result = get_wx_info(WX_OFFS, is_print=True)
```

---

## 核心模块

### wx_core - 核心功能

#### get_wx_info - 获取微信信息

获取当前登录微信的账号信息。

```python
from pywxdump import get_wx_info, WX_OFFS

result = get_wx_info(
    WX_OFFS=WX_OFFS,      # 版本偏移字典
    is_print=True,         # 是否打印结果
    save_path=None         # 保存路径（JSON文件）
)
```

**返回值：**
```python
[{
    "pid": 12345,
    "version": "3.9.8.25",
    "account": "账号",
    "mobile": "手机号",
    "nickname": "昵称",
    "mail": "邮箱",
    "wxid": "wxid_xxx",
    "key": "64位十六进制密钥",
    "wx_dir": "微信文件夹路径"
}, ...]
```

#### get_wx_db - 获取数据库路径

```python
from pywxdump import get_wx_db

result = get_wx_db(
    msg_dir="C:\\Users\\xxx\\Documents\\WeChat Files",
    db_types=["MSG", "MicroMsg"],  # 可选，过滤数据库类型
    wxids=["wxid_xxx"]              # 可选，过滤用户
)
```

**返回值：**
```python
[{
    "wxid": "wxid_xxx",
    "db_type": "MSG",
    "db_path": "完整路径",
    "wxid_dir": "用户目录"
}, ...]
```

#### get_core_db - 获取核心数据库

```python
from pywxdump import get_core_db

success, result = get_core_db(
    wx_path="C:\\...\\WeChat Files\\wxid_xxx",
    db_types=None  # 可选
)
```

---

### decryption - 解密功能

#### decrypt - 解密单个数据库

```python
from pywxdump import decrypt

success, result = decrypt(
    key="64位十六进制密钥",
    db_path="数据库文件路径",
    out_path="输出文件路径"
)
```

#### batch_decrypt - 批量解密

```python
from pywxdump import batch_decrypt

success, result = batch_decrypt(
    key="64位十六进制密钥",
    db_path="数据库路径（文件或目录）",
    out_path="输出目录",
    is_print=True
)
```

**返回值：**
```python
(True, [
    (True, ["输入路径", "输出路径", "密钥"]),
    (False, "错误信息"),
    ...
])
```

---

### merge_db - 数据库合并

#### merge_db - 合并数据库

```python
from pywxdump import merge_db

result = merge_db(
    dbpaths=[
        {"db_path": "MSG0.db"},
        {"db_path": "MSG1.db"}
    ],
    out_path="output/merge.db"
)
```

#### decrypt_merge - 解密并合并

```python
from pywxdump import decrypt_merge

result = decrypt_merge(
    key="密钥",
    db_path="数据库目录",
    out_path="输出目录"
)
```

#### merge_real_time_db - 实时合并

```python
from pywxdump import merge_real_time_db

result = merge_real_time_db(
    key="密钥",
    db_path="数据库目录",
    out_path="输出目录"
)
```

---

### BiasAddr - 获取基址偏移

```python
from pywxdump import BiasAddr

bias = BiasAddr(
    account="微信账号",
    mobile="手机号",
    name="昵称",
    key="密钥（可选）",
    db_path="数据库路径（可选）"
)

result = bias.run(
    is_logging=True,
    WX_OFFS_PATH=None  # 偏移文件路径
)
```

---

## 数据库处理

### DBHandler - 数据库处理器

统一的数据库查询接口。

```python
from pywxdump import DBHandler

db_config = {
    "key": "my_key",
    "type": "sqlite",
    "path": "merge_all.db"
}

handler = DBHandler(db_config, my_wxid="wxid_xxx")

# 获取用户列表
users = handler.get_user(word="搜索关键字")

# 获取聊天记录
msgs, users = handler.get_msgs(
    wxids=["wxid_xxx"],
    start_index=0,
    page_size=500
)

# 获取消息数量
count = handler.get_msgs_count(wxids=["wxid_xxx"])
```

### 专用处理器

| 处理器 | 用途 |
|--------|------|
| `MsgHandler` | 聊天消息 |
| `MicroHandler` | 联系人信息 |
| `MediaHandler` | 媒体文件 |
| `FavoriteHandler` | 收藏内容 |
| `PublicMsgHandler` | 公众号消息 |
| `OpenIMContactHandler` | 企业微信联系人 |
| `SnsHandler` | 朋友圈 |

---

## API 服务

### start_server - 启动服务器

```python
from pywxdump import start_server

start_server(
    port=5000,
    online=False,        # 是否允许局域网访问
    debug=False,
    isopenBrowser=True,  # 是否自动打开浏览器
    merge_path="",       # 合并数据库路径
    wx_path="",          # 微信文件夹路径
    my_wxid=""           # 当前用户 wxid
)
```

### gen_fastapi_app - 生成 FastAPI 应用

```python
from pywxdump import gen_fastapi_app

app = gen_fastapi_app(handler, origins=None)
```

---

## 导出功能

### export_html - 导出 HTML

```python
from pywxdump.api.export import export_html

export_html(...)
```

### export_csv - 导出 CSV

```python
from pywxdump.api.export import export_csv

export_csv(...)
```

### export_json - 导出 JSON

```python
from pywxdump.api.export import export_json

export_json(...)
```

---

## 完整示例

### 示例 1：获取信息并解密

```python
from pywxdump import get_wx_info, batch_decrypt, WX_OFFS

# 获取微信信息
info_list = get_wx_info(WX_OFFS, is_print=True)

if info_list:
    info = info_list[0]
    key = info['key']
    wx_dir = info['wx_dir']
    
    # 解密数据库
    if key and wx_dir:
        db_path = f"{wx_dir}\\MSG"
        success, result = batch_decrypt(
            key=key,
            db_path=db_path,
            out_path="./decrypted",
            is_print=True
        )
```

### 示例 2：查询聊天记录

```python
from pywxdump import DBHandler

db_config = {
    "key": "session_key",
    "type": "sqlite", 
    "path": "./decrypted/merge_all.db"
}

handler = DBHandler(db_config, my_wxid="wxid_xxx")

# 搜索用户
users = handler.get_user(word="张三")

# 获取与某人的聊天记录
msgs, related_users = handler.get_msgs(
    wxids=["wxid_target"],
    start_index=0,
    page_size=100
)

for msg in msgs:
    print(f"{msg['CreateTime']}: {msg['msg']}")
```

---

## 模块导出清单

```python
__all__ = [
    # 核心功能
    "BiasAddr", "get_wx_info", "get_wx_db", "batch_decrypt", "decrypt", "get_core_db",
    # 数据库合并
    "merge_db", "decrypt_merge", "merge_real_time_db", "all_merge_real_time_db",
    # 数据库处理器
    "DBHandler", "MsgHandler", "MicroHandler", "MediaHandler", 
    "OpenIMContactHandler", "FavoriteHandler", "PublicMsgHandler",
    # API
    "start_server",
    # 配置
    "WX_OFFS", "WX_OFFS_PATH", "__version__"
]
```

---

[返回首页](Home.md) | [上一步：命令行工具](命令行工具.md) | [下一步：项目架构](项目架构.md)
