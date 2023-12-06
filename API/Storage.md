# Storage API

## Storage Meta Struct

存储服务的元数据结构：

```json
{
  "storage":{
    "filename": "提交到存储服务的文件夹名/文件名.后缀",
    "relative_path": "提交到存储服务的相对路径",
    "folder": {
      // 如果提交的是文件夹，则会生成该内容，否则不存在该字段
      "subfolders": ["子文件夹列表"],
      "files": [{
        "relative_path": "文件在被提交文件夹中的相对路径/文件名.后缀",
        "meta_id": "文件对应的 Meta ID"
      }, ...],
      "hash": {
        // 如果是文件类型，则为文件二进制Hash
        // 如果是文件夹，则为 folder property 的hash结果
        "算法":"HASH结果",
      },
      "commit_time":"该文件提交时间戳，ISO时间格式，UTC",
      "length": "文件尺寸/Folder Meta 尺寸",
      "access": {
        "allow": ["*"],
        "forbid": [],
      },
      "service": ["当前已经完成该文档归档的Storage Service名称", "..."]
    }
  }
}
```

## GenerateTempFolder

根据微服务的请求上下文，Hash后生成对应的临时目录供微服务写入；

  注意：如果提供的 Service Context 一致，则会始终获取到同一临时目录，在同一进程或者同一容器中请求时需要微服务自身注意写入冲突。

参数：
  Service Context:String: 微服务生成的自身状态上下文，例如即将提交文件的URL等。存储服务仅做 Hash 使用，并不解析其内容。

返回：
  Full Path：可供微服务写入的绝对路径。

## CommitFile

将文件提交给存储服务，以便其它服务可以共享。

参数：
  FilePath：Storage 可读写的绝对路径；
  RelativeRoot: （可选）相对根路径，Storage会以此为起点计算 relative_path 
  Meta：与文件相关的Meta data；注意 storage 节会被 Storage 服务覆盖。

注意：相对根路径必须是提交文件路径的父文件夹，否则将会忽略。

返回：
  Meta： 包含 Storage 服务扫描后合并的完整 Meta数据（含ID、Hash等）

## CommitFolder

将文件夹提交给存储服务，保留其结构和内部所有文件；

参数：
  FolderPath： Storage 可读写的绝对路径；
  Meta：与文件相关的Meta data；注意 storage 节会被 Storage 服务覆盖。

返回：
  Meta： 该文件夹存储后的 Meta；

## BorrowFile

请求从 Storage 里借阅对应的文件

注意：同一微服务多次借阅同一文件将会获得幂等的结果，而归还仅需一次，不会记录其引用计数。

参数：
  Meta： 可以仅包含 Meta ID，抑或是 Storage 生成 Meta 中可索引部分（相对路径/文件名/Hash等）

返回：
  Full Path： 可供微服务读取/遍历查询的绝对路径。

## ReleaseFile

归还借阅的文件

参数：
  Meta： 可以仅包含 Meta ID，抑或是 Storage 生成 Meta 中可索引部分（相对路径/文件名/Hash等）

## BorrowFolder

请求从 Storage 服务借阅文件夹

注意：同一微服务多次借阅同一文件将会获得幂等的结果，但是可以每次请求不同的文件被准备。

参数：
  Meta： 可以仅包含 Meta ID，抑或是 Storage 生成 Meta 中可索引部分（相对路径/文件名/Hash等）
  Files: 需要同时准备的文件列表（可仅包含Meta ID或 相对路径）

## ReleaseFolder

归还借阅的文件夹

注意：会递归解除该文件夹及其下所有子文件夹/文件的引用，无论是否由 Storage 准备。

参数：
  Meta： 可以仅包含 Meta ID，抑或是 Storage 生成 Meta 中可索引部分（相对路径/文件名/Hash等）