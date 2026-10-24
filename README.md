# xhs-helper-config

红薯猪手（`com.skyhand.redbookhelper`）在小红书 **9.51.0** 上的适配配置分发仓库。

## 目录结构

```
redbook/
├── adapter26/
│   └── a9.51.0.json      # 9.51.0 单版本适配表（106 键）
└── config-26.json        # 全局配置（已去 TG/打赏、关强更）
```

## 文件格式

两个文件都是同一套加密管线：

```
明文 → 每字符 +0x0C → AES/ECB/PKCS5 → base64 → base64
```

密钥：

| 文件 | 密钥 |
|---|---|
| `adapter26/*.json` | `6666666666666666` |
| `config-26.json` | `config0026666666` |

模块运行时按 `{镜像base}redbook/adapter26/a{版本}.json` 拉取，
所以只要模块的镜像列表指向本仓库，就会自动取到 `a9.51.0.json`。

## 9.51.0 适配要点

* **保留 106 键**：`*Cls` / `m*` / `v*` 逐项对照 9.51.0 dex 索引核对。
* **剔除 17 键**：这些键指向的类在 9.51.0 已被删除或改名，保留只会让
  `findClass` 拿到 null 并静默失效。剔除清单：

  ```
  MatrixNoteDetailRIdCls  MatrixCommentRIdCls  MatrixProfileRIdCls
  MatrixRIdCls            HeyRIdCls            AliothRIdCls
  XhsRIdCls               RedViewRIdCls        MatrixDetailFeedRIdCls
  AlphaRIdCls             NoteRIdCls           HomePageRIdCls
  MatrixFeedbackRIdCls    ExploreViewCls       PoiFeedActivityCls
  VideoFeedItemPresenterCls  HeyDetailItemFragmentCls
  ```

  其中 12 个是 `R$id` 资源类 —— 9.51.0 把它们全部删掉了（8.56.0 有 164 个，
  9.51.0 只剩 1 个空类），控件 id 改由本地常量表补回。

## 混淆名陷阱

9.51.0 里 `up6.f`、`zbb.c`、`zza.a`、`by8.h` 这些名字**依然存在，但已是无关的类**。
直接沿用旧名会让 hook 挂错目标：不崩溃、不报错、功能静默失效。
本仓库的映射均按「字段 ∪ 方法名」指纹（Jaccard）重新推导并逐个验证。

## 校验

```python
from xpcodec import decode_raw, deshift
t = deshift(decode_raw('adapter26/a9.51.0.json', b'6666666666666666'))
```
