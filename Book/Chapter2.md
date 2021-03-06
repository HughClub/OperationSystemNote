# Chapter2 进程的描述与控制

## 2.1 前驱图和程序执行

### 2.1.1 前驱图

- 前驱图是有向无环图, 描述程序执行的先后顺序
- 结点间的有向边表名前驱关系, 起始点先于结束点

### 2.1.2 程序的顺序执行

- 应用程序有多个程序段, 每个段一个特定功能
- 它们按照某种先后顺序执行, 存在前驱关系
- 顺序执行的三个特征
  - 顺序性: 严格按照规定顺序执行
  - 封闭性: 封闭环境, 独占全机, 只有本身可改变资源状态
  - 可再现性: 只要初始条件和执行环境相同, 程序重复执行结果都相同

### 2.1.3 程序并发执行

- 不存在前驱关系的代码段可以并发执行, 互不依赖
- 并发执行的三个特征
  - 间断性: 共享资源, 相互合作制约, 执行断断续续
  - 无封闭性: 系统资源共享, 资源状态都可改变, 互相影响
  - 不可再现性: 不封闭, 就不可再现, 数次执行的结果可不同

## 2.2 进程的描述

### 2.2.1 进程的定义和特征

- 进程是(可并发)程序(在一定数据集合)的一次执行过程, 是OS调度和资源分配的基本单位
- 进程实体由程序段、相关数据段、进程控制块组成, 简称进程
- 进程有五个特征
  - 动态性: 进程的实质是进程实体的执行过程, 程序是静态的, 进程是动态的
  - 并发性: 多个进程可同时存在于内存, 在一段时间内同时运行, 程序不能
  - 独立性: 进程是能独立运行, 独立获得资源, 独立接受调度的基本单位
  - 异步性: 进程按异步方式运行, 各自独立又不可预知地运行
  - 结构性: PCB?

### 2.2.2 进程的基本状态及转换

- 进程的三种基本状态
  - 就绪状态: 处于就绪队列, 已获得除CPU的所有必要资源
  - 执行状态: 已获得CPU, 进程正在执行
  - 阻塞状态: 执行中的进程遇到某事件, 暂时无法继续执行, 阻塞队列
- 三种状态的转换
  - 就绪->执行: 进程调度为之分配了CPU
  - 执行->就绪: 时间片用完, 被剥夺CPU
  - 执行->阻塞: 因某事而执行受阻, 无法继续
  - 阻塞->就绪: 阻塞的原因消除, 如IO完成
- 额外的两种状态
  - 创建步骤: 先申请PCB并填入信息, 再分配资源, 再进入就绪队列
  - 若所需资源不足, 如内存不够无法装入, 则创建尚未完成, 处于创建状态
  - 进程达到自然结束点或遇见错误, 会被OS或其他进程终结, 进入终止状态
  - 终止状态的程序无法执行, 但会保留信息供提取, 提取完后再删除

### 2.2.3 挂起操作和进程状态的转换

- 挂起和激活操作
  - 挂起操作让进程处于静止状态, 暂停执行, 不接受调度
  - 挂起操作引入的原因/需要
    - 终端用户的需要
    - 父进程的请求
    - 负荷调节的需要
    - 操作系统的需要
  - 挂起导致的状态转化
    - 执行->静止就绪
    - 活动就绪->静止就绪
    - 活动阻塞->静止阻塞
  - 激活导致的状态转化
    - 静止就绪->活动就绪
    - 静止阻塞->活动阻塞
  - 创建后若性能和内存都允许, 则进入活动就绪, 反之静止就绪

### 2.2.4 进程管理中的数据结构

- 为管理计算机中各类资源, OS将之抽象为数据结构, 与操作资源的命令
- 用户用命令进行操作, OS通过数据结构建立和维护信息和资源
- OS为资源建立资源信息表或进程信息表, 包含标识,描述,状态等
  - 进程表
  - 内存表
  - 设备表
  - 文件表
- 进程控制块PCB记录进程当前状况和关系运行的信息, 作用有
  - 作为独立运行基本单位的标志
  - 能实现间断性运行方式
  - 提供进程管理所需要的信息
  - 提供进程调度所需要的信息
  - 实现与其他进程的同步与通信
- PCB主要包含四个方面的信息
  - 进程标识符: 外部标识符用于用户/进程, 内部标识符用于OS
  - 处理机状态: 亦称上下文, 主要是处理机的各种寄存器内容
  - 进程调度信息: 进程当前状态, 进程优先级, 调度的事件算法等
  - 进程控制信息: 程序和数据地址, 资源清单, 链接指针等
- PCB们的两种组织方式
  - 线性方式: 都存在线性表中, 表首地址存在内存专用区域
  - 链接方式: 相同状态进程的PCB连成链表, 不同队列排序不同, 各有头指针
  - 索引方式: 相同状态的进程建立索引表, 映射到同一PCB表中的地址

## 2.3 进程控制

- 主要包括进程的创建终止, 状态转换等
- 一半由OS内核的原语来实现

### 2.3.1 操作系统内核

- 将OS划分为不同层次, 不同功能设置在不同层次中
- 硬件相关,常用驱动,高频模块, 安排得紧靠硬件, 常驻内存, 称之OS内核
  - 便于对软件进行保护, 防止破坏
  - 提高OS的运行效率
- 为了防止OS本身和关键数据被破坏, 引入两种CPU执行状态
  - 系统态/内核态: 特权高, 可执行一切指令, 访问所有寄存器和存储区
  - 用户态/目态: 仅能执行指定指令, 访问指定寄存器和存储区, 不能OS部分
- 大多数OS内核提供两大方面的功能
  - 支撑功能: 其他模块的基本功能
    - 中断处理: 最基本功能, 系统调用,IO,调度等, 都依赖于中断
    - 时钟管理: 基本功能, 时间片用完产生中断信号, 时间控制也依赖
    - 原语操作: 是由若干条指令组合的完成一定功能的过程, 不可分割和中断
  - 资源管理功能
    - 进程管理: 进程管理模块运行频率高, 且被多模块需要
    - 存储器管理: 频率也高, 如地址变换, 内存分配和调换等
    - 设备管理: 和硬件紧密相关, 如驱动, 缓冲等

### 2.3.2 进程的创建

- 进程的层次结构: 指进程可创建子进程, 形成树状层次结构
  - 子进程可继承父进程所拥有的资源, 包括文件, 缓冲区
  - 撤销子进程会归还资源, 撤销父进程前要撤销完子进程
  - Windows不存在进程层次, 创建者获取句柄来控制, 但句柄可移交
  - 进程图: 描述进程层次结构的树状图
- 创建进程的事件
  - 操作系统初始化
  - 用户登录: OS为用户创建一个进程
  - 作业调度: 多道批处理中, 调入作业时为之创建进程, 放入就绪队列
  - 提供服务: 用户程序提出请求, 系统创建进程提供服务
  - 应用请求: 上述是内核为用户创建, 现在是用户为自己创建
- 进程的创建步骤
  - 申请空白PCB块: 获取唯一标识符, 并从PCB集合中索取
  - 分配所需资源: 包括各种物理和逻辑资源, 内存文件等
  - 初始化PCB块: 初始化标识信息, 处理机状态, 处理机控制信息
  - 插入就绪队列: 若能容纳新进程

### 2.3.3 进程的终止

- 引起终止的事件
  - 正常结束: 任务完成, 会有表明完成的指示
  - 异常结束: 发生异常, 如越界,非法访问,非法指令等
  - 外界干预: 相应外界请求而终止运行, 如操作员/OS,父进程请求,父进程终止
- 进程的终止过程
  - 根据进程标识符, 检索PCB, 从中读出进程状态
  - 若正在执行, 则终止执行, 并表明终止后重新调度
  - 若还有子孙, 则将子孙都终止
  - 将该进程所有资源归还给父进程或系统
  - 将PCB从队列/链表移除, 等待被收集信息

### 2.3.4 进程的阻塞与唤醒

- 引起进程阻塞和唤醒的事件
  - 请求共享资源: 资源不足, 等待释放
  - 等待某种操作: 有操作执行的先后顺序
  - 新数据未到达: 要获取另一进程的数据
  - 等待新任务到达: 如收发数据包
- 进程的阻塞过程
  - 立即停止执行
  - 把PCB中执行状态改为阻塞
  - 把PCB插入阻塞队列
  - 转调度程序重新调度运行
  - 保留阻塞进程的处理机状态

### 2.3.5 进程的挂起与激活

- 进程的挂起过程
  - 检查进程状态, 据此改变状态
  - 复制PCB至指定内存区域, 方便考察
  - 若进程曾在执行, 则重新调度
- 进程的激活过程
  - 把进程从外存调入内存
  - 检查现行状态, 据此改变状态
  - 根据调度程序进行调度

## 2.4 进程同步

### 2.4.1 进程同步的基本概念

- 主要任务: 让多个进程在次序上协调, 按照规则共享合作, 使得可再现性
- 两种形式的制约关系
  - 间接制约: 共享临界资源, 必须互斥访问, 使用前先申请
  - 直接制约: 完成同一项任务而合作, 需要有先后顺序
- 临界资源: 进程间应采取互斥方式进行资源共享
  - 生产者-消费者
  - 缓冲池buffer[n], 输入输出指针in和out, 当前数量count
  - 这些东西是临界资源, 需要互斥访问, 加互斥信号量
- 临界区: 进程中访问临界资源的代码片段
  - 进程互斥进入临界区, 则可实现互斥访问临界资源
  - 进入区: 临界区前检查资源是否正在被访问的代码
  - 退出区: 恢复临界资源未被访问的标志的代码
  - 除了这三个区的代码叫剩余区
- 同步机制遵循的四个规则
  - 空闲让进: 无进程在临界区, 临界资源空闲, 允许请求的进程进入
  - 忙则等待: 有进程在临界区, 其他试图访问的进程必须等待
  - 优先等待: 应当保证有限时间内能进入临界区
  - 让权等待: 当不能进入时, 需要释放处理机

### 2.4.2 硬件同步机制

- 对临界区可看为锁, 锁开进入, 锁关等待
- 关中断: 进入锁测试前关闭中断, 测试完成并上锁后再打开, 访问中不会调度和切换
  - 滥用权力后果严重
  - 时间过长影响效率
  - 多CPU无法防止其他处理器执行临界区
- Test-and-Set: 是原语, 进入前先检测锁状态
  - 若锁开, 则进入并关锁, 出来再开锁
  - 若锁关, 则等待锁开, 再进入关锁, 出来开锁
- Swap: 交换局部key和全局lock的内容
  - key初始为true, 一直交换直到key为false
  - 若锁开, 则自动把锁变为true, 并进入
  - 若锁关, 则不改变锁true的状态
  - 出来后开锁, 变为false
- 以上都不能放权等待

### 2.4.3 信号量机制

- 整型信号量: 用于表示资源数目的整型量
  - 仅能通过初始化和PV改变其值
  - PV操作为原子操作, 前者等待到有资源则-1, 后者+1
  - 没有让权等待
- 记录型/资源信号量: 采用让权等待, 引入阻塞进程队列链表
  - P先-1, 若负则进入等待队列, 等待唤醒; 表示申请资源
  - V先+1, 若非正则唤醒, 出队一个; 表示释放资源
  - 信号量初值代表资源数目, 非负值表示剩余资源数, 负值表示阻塞队列长度
- AND型信号量

### 2.4.4 信号量的应用

### 2.4.5 管程机制

## 2.5 经典进程同步问题

## 2.6 进程通信
