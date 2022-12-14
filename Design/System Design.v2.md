# 系统设计

```mermaid
graph LR
  User--Upload-->Document
  User--Archive-->Meta
  User--Query-->Meta
  User--Read-->Document
  subgraph Core
    Meta-. Ref .->Document
    Task
  end
  subgraph Sub Service
    Grabber-->Document
    Document-->Parser-->Meta
    Meta-->Filter-->Task-->Scheduler
    Scheduler-.->Grabber
    Scheduler-.->Parser
    Scheduler-.->Filter
  end
```

1. 系统由 `Core` 服务和可扩充的 `Sub Service`(下文简称 `Service`)共同组成；
1. 系统基本存储/检索单位为 `元数据` (Meta) ，它是一个将 `原始资料` （Document)（HTML/DOC/PDF/IMAGE/VIDEO/...) 经过 `Service` 处理、解析、摘要之后生成的一份 Key/Value 字典；
2. `Meta` 数据可以包含其它 `Meta`/`Document` 的关联引用；
3. `Document` 是系统存储的从各个来源（含用户上传）获取的原始档案；
4. 分析工具对 `Document` 进行解析并生成/更新对应的 `Meta` 数据；
6. `Core` 负责基本的 `Meta` `Document` 和 `Task` 相关服务；
7. `Service` 负责根据服务自身设计范围完成子项任务；
8. `Meta` 、 `Document` 和 `Task` 在创建、 修订、删除等操作前后，会向消息管线发送异步消息，各个 `Service` 可以订阅；
9. `Task` 集中设计原因：服务之间可以通过该方式进行异步互相调用；同时服务自身可以在升降版本的时候有效管理断点；

## Data Flow

```mermaid
graph TD
  subgraph Core
    subgraph meta
        Create--Meta-->Meta-->MetaMessage[Meta Message]
        Update--Meta-->Meta
        Merge--Meta-->Meta
        Meta-->Query
    end
    subgraph storage
        Write-->Document-->DocumentMessage[Document Message]
        Document-->Access
    end
    subgraph register
        RegisterService-->ServiceList-->Configure
    end
    subgraph schedule
        CreateTask-->Task-->TaskMessage[Task Message]
        RescheduleTask-->Task
    end
  end
  subgraph service
    selfRegister-->RegisterService
    Subscribe--Message-->Scheduler--Meta-->Filter--Task-->CreateTask
    Scheduler--Task-->Grabber
    Scheduler--Task-->Parser
    Scheduler--Task-->Worker
    Parser--Meta-->Create
    Parser--Meta-->Update
    Parser--Meta-->Merge
    Worker--Task-->CreateTask
    Worker--Task-->RescheduleTask
    Grabber--Document-->Write
  end
```

## Meta & Document

1. Meta&Document 是系统管理的原子数据；
1. Meta 是一个经过各个 Worker 处理后，可以检索的 Key/Value 字典；
1. Document 是原始获取的内容（文本/媒体/等）；
2. 每个 Document 必然有至少一个对应的 Meta 数据来描述其内容摘要；
3. Meta 可以互相关联，来表述其引用关系；

## Task & Schedule

1. Task 是 Worker 处理的最小事务；
2. Task 主要参数： Worker(Group)/Meta ID(Nullable)/Task Name(Route)/Task Params（使用二进制方式存储，并按照原始内容传递）；
2. Task 可以从 Meta Insert/Update/Select 等操作中生成；其中 Meta Insert/Update 会通过消息管线实时广播，而 Meta Select 可以通过 API 查询后批量处理；
3. Schedule 可以通过排期生成对应 Task；

## Worker & Service

1. 一个 Service 是一组工具集合，可以包含下列部分或全部功能的 Worker；
1. 一个 Worker 代表一个实际进行事务的 Exectuable (服务/进程/线程/函数/etc)；
3. Register 注册服务端点及报告服务状态；
2. Filter 负责订阅消息过滤和 Task 生成；
4. Grabber 一般负责原始文档的获取；
5. Parser 一般负责解析原始文档，生成 Meta；

# 核心组件（Core）

## Index

Index 负责 Meta 相关事务：

1. 存储/更新 Meta 数据；
2. 分发 Meta 消息；
3. 提供 Meta 检索接口；

## Schedule 

Schedule 负责 Task 相关事务：

1. 创建 Task；
2. (根据时间表/繁忙程度/配置额度等)调度 Task 进入执行状态；
3. 接收 Task 执行结果（成功/失败/重新调度）；
4. （定期）清理 Task 列表；
5. Task 调度/查询 接口；

## Storage

Storage 负责 Document 相关事务：

1. 存储 Document, 给出读入路径；
2. 批量 Document 事务操作；
3. 归档 Document ，同时不改变读入路径；
4. 去重、合并、更新 Document；

## Register

Register 负责 Service 相关事务：

1. Service Register；
3. Service Name Search / Poll；
2. Service Route；
4. Service keepalive test；

# 服务组件（Service）

* 服务的核心在于调度器 `Scheduler` ，在服务器初始化完成后，通过调用 `Core` 的 `Schedule API` 来获取 Task 状态并分配给各个 `Worker` 执行；

* 每个服务可以是一组有近似功能的 Worker 集合，或者针对某类文档/某个站点/某种协议的工具集合；

* `Grabber/Parser/Filter` 等都是 `Worker` 的衍生，并不是必须的；


举例：

* `Hash Service` 包装各种 `Hash` 算法的工具集，可以给 `Document` 生成对应的 `Hash` 合并入 `Meta` 里；
* `Image Hash` 则是针对图像内容进行摘要的服务，可以对图像类别的 `Document` 生成摘要合并到 `Meta` 中，同时提供近似摘要文档查询功能；
* `Image Recognition` 包装图片识别AI，可以对图像类别的 `Document` 进行分析并生成 `Tag` 合并到 `Meta` 中；
* `Twitter` 则是包装完整 `Twitter API` 的工具集，针对包含 `twitter.com` 的 `Meta` 进行数据抓取和分析；
