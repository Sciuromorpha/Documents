# What's this

`Meta` 是系统索引的核心数据，其本质是一个 key:value 字典，而各个处理微服务需要做的就是保证该字典能够尽可能正确的描述该 `Meta` 数据指代的对象。

其中，`Meta` 对象的 `Key` 需要统一并方便索引，所以不同的 `Parser` 应该根据自身处理的 `Document` 数据特性进行 `value` 的生成和更新；

## Key List

|        Key | 类型 | 描述               |
| ---------: | :--- | ------------------ |
| origin_url | str  | 数据原始 URL       |
|   filename | str  | 文档 原始名称      |
|       hash | hex  | 文档 hash 值       |
|      phash | hex  | 图片类 hash 值     |
|      whash | hex  | 音频类 hash 值     |
|       EXIF | dict | 照片原始 EXIF 信息 |

