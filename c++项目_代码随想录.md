# 分布式缓存



## 0. 项目基础结构

![image (1).jpg](https://cdn.nlark.com/yuque/0/2025/jpeg/39162301/1752996251494-d6dbfab9-d664-4f32-8913-163f64c54f69.jpeg?x-oss-process=image%2Fformat%2Cwebp%2Finterlace%2C1)





## 1.LRU缓存

![image-20251019111742799](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20251019111742799.png)

#### 1.1基础思想：采用双向链表（std::list）以及哈希表（unordered map）来实现

在LRUCache类中：

```c++
std::unordered_map<std::string, ListElementIter> cache_;
std::list<Entry> list_;   //Entry是封装的缓存中存放数据的基本单位，包括key_:字符串类型的键  value_: ByteView类型的值
```

- `std::list<Entry> list_;`：维护缓存条目的访问顺序，链表头部存放最近访问的元素，链表尾部存放最久未访问的元素；
- `std::unordered_map<std::string,ListElementIter> cache_;`：key 为缓存的键，value 指向 `list_`中对应元素的迭代器；

#### 1.2 同时向上提供get set 和delete实现

可以把 LRUCache 的结构理解为：

> **“用 `std::list` 存储缓存数据，用 `std::unordered_map` 做索引来快速定位 list 里的元素。”**

下面我们一步步展开讲清楚它的设计逻辑👇

------

## 🧠 核心思想：O(1) 时间实现 LRU 缓存

LRU（Least Recently Used，最近最少使用）缓存的目标是：

- **快速查找**（根据 key 找到数据）；
- **快速更新访问顺序**（把最近访问的元素移动到队头）；
- **快速删除最久未使用的元素**（队尾弹出）。

而要在 **O(1)** 时间内完成这些操作，单一的数据结构做不到：

| 操作                     | list       | unordered_map |
| ------------------------ | ---------- | ------------- |
| 按 key 查找              | ❌ 需要遍历 | ✅ O(1)        |
| 维护访问顺序（头尾操作） | ✅ O(1)     | ❌ 不支持顺序  |
| 删除最久未访问元素       | ✅ O(1)     | ❌ 不支持顺序  |

👉 因此 LRUCache 采用 **组合结构：`unordered_map + list`**





## 2.一致性哈希

一致性哈希是分布式系统中使用的一种技术，用于在多个节点之间分发数据，以最大限度的**减少**添加或删除节点时的重组

![image.jpg](https://cdn.nlark.com/yuque/0/2025/jpeg/39162301/1753611575177-f5318037-b146-476b-8b30-1823fca24223.jpeg?x-oss-process=image%2Fformat%2Cwebp)

当节点数量发生变化时，使用 `hash(key) % n`（其中 n 是节点数）的**传统基于哈希的分发会遇到重大的重新分发问题**。一致性哈希通过确保在节点计数更改时只需要重新映射一小部分键来解决此问题。

一致性哈希的大致架构用上图就能说明：由一**个 `[0, 2^32-1]` 的“线”围成了一个环**，对于分布式系统中的每个真实物理节点，都会映射成多个虚拟节点（如上图中使用了 3 个物理节点，这三个物理节点又各自映射了三个虚拟节点），对这些虚拟节点的标识进行哈希运算，就可以得到一个 `[0, 2^32-1]`之间的值。如果把刚刚说的“线”用数组来表示，那么虚拟节点哈希后的值就是这个数组中的一个元素。

当有新节点时或者原有节点 node 发生故障，**需要调整只有 node 附近的 key 所属**

####  总结一句话

> 一致性哈希通过将哈希空间组织成一个 2³²−1 的环形结构，并使用多个虚拟节点来均衡负载，使得节点的增删只影响环上相邻区域的键，避免全量迁移。



#### 2.1ConsistentHashMap类

在Kcache中，一致性哈希在类中实现，它创建了一个哈希环，其中节点和键都被映射，每个键都发配给围绕此环**顺时针方向最近的节点**

```c++
//一致性哈希配置
struct HashConfig{

};
class ConsistenHashMap{
public:
    explicit ConsistentHashMap(HashConfig cfg = kDefaultConfig);
    """  省略其他代码  """
        
private:
    mutable std::shared_mutex mtx_;  // 读写互斥量，类似于 Go 的 sync.RWMutex
    // 配置信息
    HashConfig config_;

    // 哈希环
    std::vector<uint32_t> keys_;
    // 哈希环到节点的映射
    std::unordered_map<uint32_t, std::string> hash_map_;
    // 节点到虚拟节点数量的映射
    std::unordered_map<std::string, int> node_replicas_;
    // 节点负载统计
    // 使用 std::atomic<long long> 保证对 nodeCounts 中每个节点计数的原子操作
    std::unordered_map<std::string, std::atomic<long long>> node_counts_;
    // 总请求数
    std::atomic<long long> total_requests_;

    std::thread balancer_thread_;         // 负载均衡器线程
    std::atomic<bool> is_balancer_stop_;  // 控制负载均衡器线程停止的标志
    
	
}
```

在这其中利用一个vector和两个unordered_map 极大的方便了对于节点在一致性哈希环中的CRUD



#### 2.2 虚拟节点

通常对于一个缓存节点（物理节点），会在一致性哈希中创建多个**虚拟节点**这是为了 

1. 均匀分布数据
2. 减少数据迁移
3. 提升扩展性：虚拟节点允许在不重新哈希所有数据的情况下扩展
4. 平衡负载：真实节点的负载被分散到多个虚拟节点



#### 2.3添加节点操作

当每当一个**新的物理节点来到的时**候，**原本存在的节点都会通过etcd得知消息**，然后**每一个节点都会*更新自己的*一致性哈希环**

⭐是的，每个节点都会有一个一致性哈希环，不过不用担心他们之间的内容会不一致，因为每个节点都会使用 etcd 去监测是否有新的节点到来，同时也监测现在有哪些节点，然后去更新哈希环。





#### 2.4删除节点操作

当我们需要删除一个节点时，也就是要**删除它在哈希环中对应的所有虚拟节点**，要根据虚拟节点的key值去操作`keys_,`和`hash_map_`





#### 2.5 获取节点操作

根据给定的值找到对应的节点

![image (3).jpg](https://cdn.nlark.com/yuque/0/2025/jpeg/39162301/1752996287029-ca9f3c4a-663a-4a41-9b1d-f4ff6acde7ed.jpeg?x-oss-process=image%2Fformat%2Cwebp)



如上图，假设我们有一个键为“**kama**”，那么按照一致性哈希的原理，缓存了这个键对应的值的应该是节点 **Peer2**。在代码实现时原理为：

1. **参数验证与锁机制**：先检查输入键是否为空，使用共享读锁允许多个线程同时读取，提高并发性能
2. **边界检查**：确保哈希环上至少有一个节点
3. **哈希计算**：将输入的键通过配置的哈希函数映射到哈希环上的位置
4. **二分查找**：找到第一个大于等于目标哈希值的位置，这实现了哈希环上的"顺时针查找"
5. **环形处理**：当哈希值超过所有节点位置时，需要"绕回"到环的开始，这体现了一致性哈希的环形特性
6. **节点映射**：通过哈希位置获取对应的节点名称





#### 2.6负载均衡

一致性哈希实现中的负载均衡是一个动态自适应系统，通过**监控节点的访问情况**并**调整虚拟节点数量来实现负载均衡**

在ConsistentHashMap中有两个成员变量：

```c++
class ConsistentHashMap {
    ...
private:
    ...
    // 节点负载统计 - 每个节点的访问次数
    std::unordered_map<std::string, std::atomic<long long>> node_counts_;
    // 总请求数
    std::atomic<long long> total_requests_;
};
```

每次调用ConsistentHashMap::Get时候都会更新统计

```c++
++node_counts_[node];    //节点访问计数
++total_requests_;		//总请求计数
```

**后台监控：当我们创建一个一致性哈希组件的时候，都会在后台启动一个线程来监控**

```c++
void ConsistentHashMap::StartBalancer() {
    is_balancer_stop_ = false;
    balancer_thread_ = std::thread{[this] {
        while (!is_balancer_stop_) {
            std::this_thread::sleep_for(std::chrono::seconds(1));  // 每秒检查一次
            if (!is_balancer_stop_) {                              // 再次检查，防止在 sleep 期间被要求停止
                CheckAndRebalance();
            }
        }
    }};
}
```

#### 2.7负载不均衡检测

- 触发条件检查

  - 当总请求数少于1000时候，认为样本量不足，不进行负载均衡调整
  - 通过读锁安全访问节点信息（node_replicas_  node_counts__)

- 计算负载均衡度

  - 计算平均负载：avg_load = 总请求数 / 节点数
  - 遍历所有节点，计算每个节点的负载与平均负载的差异百分比（diff / avg_load）
  - 记录最大差异百分比max_diff ,作为负载不均衡度的指标

- 触发重平衡

- - 当 `max_diff` 超过配置的阈值（`config_.load_balance_threshold`）时，调用 `RebalanceNodes()` 进行重平衡。

- 

#### 2.8节点再平衡 

- 获取写锁
  - 由于需要修改哈希环结构，使用写锁确保线程安全
- 计算负载比例
- **调整虚拟节点数量（多减去，少增加）**
- 更新哈希环
- 重置计数器
- 重新**排序哈希环**
  - 对`keys_`排序，确保哈希环按哈希值有序排列，便于后续查找（后续需要二分查找）



## 3.SingleFlight机制



#### 缓存三大问题：击穿，雪崩、穿透

- **缓存击穿**  某个热点key在缓存过期瞬间，同时有大量请求访问这个key，导致**所有请求都落在数据库上**，造成数据库瞬时压力过大
- **缓存雪崩：** 大量缓存key同时失效，导致所有请求都落在数据库上，引起数据库压力激增甚至崩溃
- **缓存穿透：**查询不存在的数据，导致每次请求都绕过缓存直接访问数据库。



#### 什么是SingleFlight

是一种用于**合并并发请求**的设计模式。核心思想就是合并重复请求：**当多个并发请求同时访问同一个资源时，只让第一个请求执行实际操作，其他请求等待该操作完成后直接复用结果**

在这个项目中，就是为了防止缓存击穿

**简单来说，SingleFlight 将"多个相同请求"转换为"一次执行+多次共享"。**



这个是调用的结构体

```c++
class SingleFlight {
    using Result = std::optional<ByteView>;
    using Func = std::function<Result()>;

public:
    Result Do(const std::string& key, Func func) {
        std::unique_lock<std::mutex> glock(mtx_);

        // 首先进行请求合并，检查是否有进行中的调用
        if (map_.find(key) != map_.end()) {
            auto existing_call = map_[key];
            glock.unlock();  // 释放组锁，避免阻塞其他键的处理

            // 直接等待 future 的结果
            auto result = existing_call->fut.get();
            return result;
        }

        // 创建新的调用对象
        auto new_call = std::make_shared<Call>();
        map_[key] = new_call;
        glock.unlock();

        // 执行用户函数并设置 promise
        Result val = func();
        new_call->prom.set_value(val);

        // 确保从映射中删除条目
        std::lock_guard<std::mutex> lock(mtx_);
        map_.erase(key);

        return val;
    }

private:
    struct Call {
        std::promise<Result> prom;
        std::shared_future<Result> fut = prom.get_future().share();
    };

    std::mutex mtx_;
    std::unordered_map<std::string, std::shared_ptr<Call>> map_;
};
```

对于Call结构体中：

- std::promise 是c++11的多线程同步机制，用于**承诺**在未来提供一个值

  - 这个值通常由另一个线程设置，供 `std::future` 或 `std::shared_future` 获取
  - 模板参数 `Result` 表示它保存的结果类型（例如 `int`、`std::string` 等）。

- std::shared_future<Result> fut = prom.get_future().share();

  - prom.get_future():从std::promise 中获取一个std::future 对象，用于读取prom设置的值

  - .share():将std::future 转换为std::shared_future.

  - 这一行表示**在结构体创建时**，就立刻从 `promise` 创建一个可共享的 future。

  - 也就是：

  - ```
    prom    // 写入者：设置结果
    fut     // 读取者：多个线程可以读这个结果
    ```

    工作流程类似

    ```
    Thread 1: Get("user:123") → 创建 Call → 执行 func() → 设置结果 → 清理
    Thread 2: Get("user:123") → 发现存在 Call → 等待结果
    Thread 3: Get("user:123") → 发现存在 Call → 等待结果
    Thread 4: Get("user:456") → 创建新 Call → 独立执行
    ```





## 4. 缓存组

`CacheGroup` 是分布式缓存系统中的核心组件，它代表了一个逻辑上的缓存分组，具有**本地缓存、远程节点访问和数据加载**的能力。这个设计允许系统在多个节点之间共享缓存数据，同时保持高效的本地访问性能。

![img](https://cdn.nlark.com/yuque/0/2025/jpeg/39162301/1753615312470-d3bee3d8-b60f-421b-84b8-e82aa3292c97.jpeg)

**每个缓存组都是一个独立的命名空间，gRPC服务器维护了一个groups_映射，每个组都有独立的CacheGroup实例。****同一个缓存组内的节点通过一致性哈希协同工作，而不同缓存组之间完全独立。**



#### 创建缓存组

通常，我们不直接使用`CacheGroup`的构造函数来创建缓存组，而是通过以下两个函数来操作（因为请求缓存节点时是通过gRPC，那gRPC Server就应该接受i请求后去创建/使用缓存组）

```c++
std::unordered_map<std::string, CacheGroup> cache_groups;
std::mutex mtx;

auto MakeCacheGroup(const std::string& name, int64_t bytes, DataGetter getter) -> CacheGroup& {
    if (getter == nullptr) {
        spdlog::critical("no getter function!");
        std::exit(1);
    }
    std::lock_guard lock{mtx};
    cache_groups[name] = std::move(CacheGroup{name, bytes, getter});
    return cache_groups[name];
}

auto GetCacheGroup(const std::string& name) -> CacheGroup* {
    std::lock_guard lock{mtx};
    if (cache_groups.find(name) == cache_groups.end()) {
        return nullptr;
    }
    return &cache_groups[name];
}
```



```c++
//语法解释
using DataGetter = std::function<ByteViewOptional(const std::string& key)>;
//DataGetter 是一种可以保存任何可调用对象（函数、lambda、函数对象、bind结果等）的类型，
//这些可调用对象必须满足以下函数签名
ByteViewOptional f(const std::string& key);

```

**为何要使用：std：：function**

- 灵活性/类型擦除    std::function<R(Args...)>可以保护任意满足签名R（Args...）的可调用对象（普通函数、静态成员函数、lambda、函数对象、std::bind的结果等）
- 统一接口     使用DateGetter（别名）后，代码其他部分只关注调用它并得到ByteViewOptional，不用模板化CacheGroup或在多个重载之间切换，代码更简洁
- 方便存储/赋值     std::function是值语义（可复制/移动）可以直接存入CacheGroup的成员、放进容器、赋值、传递、使用简单直观
- 





为何要分缓存组

```
、直觉解释：缓存组 = 命名空间隔离

你可以先这样理解：

一个 CacheGroup 就像一个独立的“命名空间”或“逻辑数据库”。

每个组（Group）独立维护：

自己的缓存策略（容量、过期时间、加载函数等）；

自己的节点一致性哈希环；

自己的底层数据加载逻辑。

举个比喻：

想象一个网站：

“用户资料缓存”；

“商品信息缓存”；

“推荐结果缓存”。

如果所有缓存都混在一个全局空间里：

key 容易冲突；

容量不好分配；

过期策略、更新逻辑混乱；

一个缓存模块出错可能拖垮全部系统。

所以我们分出多个 CacheGroup，每个独立运行，互不干扰。
```

#### 获取数据

##### 数据加载

当缓存组中本地缓存未命中的时候会调用load来加载数据

```c++
auto CacheGroup::Load(const std::string& key) -> ByteViewOptional {
    auto ret = loader_.Do(key, [&] { return LoadData(key); });//采用singleflight来访问数据
    if (!ret) {
        spdlog::error("Failed to load data for key: {}", key);
        return std::nullopt;
    }
    cache_->Set(key, ret.value());
    // TODO 记录加载时间
    return ret;
}
```

`Load` 会使用之前我们实现的 SingleFlight 机制来防止重复请求，其中的具体加载实现为 `LoadData`：

```c++
auto CacheGroup::LoadData(const std::string& key) -> ByteViewOptional {
    // 先尝试从远程节点获取
    if (peer_picker_ != nullptr) {
        if (auto peer = peer_picker_->PickPeer(key); peer) {
            auto val = LoadFromPeer(peer, key);
            if (val) {
                ++status_.peer_hits;
                return val;
            }
            ++status_.peer_misses;
        } else {
            spdlog::warn("Failed to get from peer");
        }
    }

    // 通过getter从数据源获取
    auto val = getter_(key);
    if (!val) {
        spdlog::error("Failed to get [{}] from data source", key);
        return std::nullopt;
    }
    ++status_.local_hits;
    return val;
}

auto CacheGroup::LoadFromPeer(Peer* peer, const std::string& key) -> ByteViewOptional {
    auto value = peer->Get(name_, key);
    if (!value) {
        return std::nullopt;
    }
    return value;
}
```

`LoadData` 方法实现了**分层加载策略**：

1. **首先尝试从远程节点获取**：如果配置了 `peer_picker_`，会先尝试从其他节点获取数据
2. **回退到数据源**：如果远程节点也没有数据，最后调用 `getter_` 函数从原始数据源加载 

*那为什么要先从远程节点获取数据呢？*

这是因为如果其他节点已经从原始数据源加载过这个数据，那么从远程节点获取比重新从数据源加载要**快得多**。



#### Get操作

首先本地缓存中获取数据，如果获取失败，就调用`Load`去从远程节点或者数据源获取数据





#### 数据同步

`SyncToPeers` 方法的主要作用是在分布式缓存环境中保持数据一致性，当本地节点执行 Set 或 Delete 操作时，将这些操作同步到集群中的其他相关节点。

为什么需要将数据同步到其他远程节点？

系统使用一致性哈希算法来分布数据，每个键都有一个"权威"节点负责存储，但其他节点可能也会缓存这个数据。当权威节点的数据发生变化时，需要通知其他可能缓存了该数据的节点。

如果不进行同步，会出现以下问题：

- 节点A修改了某个键的值
- 节点B仍然缓存着旧值
- 客户端从节点B获取到过期数据

##### 调用时机

`SyncToPeers` 在以下场景被调用:

1. **Set 操作**: 当本节点执行 `Set()` 且 `is_from_peer=false` 时触发同步
2. **Delete 操作**: 当本节点执行 `Delete()` 且 `is_from_peer=false` 时触发同步
3. **Invalidate 操作**: 当本节点主动失效缓存时触发同步

`is_from_peer` 参数用于防止循环同步:如果操作来自其他节点的 RPC 请求,则不再向其他节点同步。

##### 实现机制

`SyncToPeers` 是 `CacheGroup` 类中用于将缓存操作同步到集群中其他节点的方法。该方法根据不同的操作类型(SET/DELETE/INVALIDATE)采用不同的同步策略。

- **set操作的同步策略**
  - 主节点同步：首先通过`peer_picker_->PickPeer(key)`使用一致性哈希确定该key的主节点，然后调用`primary_peer->Set() 将数据同步到主节点`
  - 其他节点失效   ：向其他所有节点发送`Invalidate请求`，删除本地的缓存副本，确保最终一致性
- **DELETE操作的同步策略**
  - 通过 `peer_picker_->GetAllPeers()` 获取所有节点
  - 向每个节点发送 `Delete` RPC 请求
  - 确保所有节点都删除该 key
- **INVALIDATE 操作的同步策略**
  - 向所有节点发送 `Invalidate` RPC 请求
  - 每个节点只删除本地缓存,不重新加载数据

```c++
void CacheGroup::SyncToPeers(const std::string& key, SyncFlag op, ByteView value) {
    if (!peer_picker_) {
        return;
    }

    switch (op) {
        case SyncFlag::SET: {
            // 对于 SET 操作，实现最终一致性：
            // 1. 将数据同步到管理该 key 的节点
            auto primary_peer = peer_picker_->PickPeer(key);
            if (primary_peer) {
                bool ok = primary_peer->Set(name_, key, value);
                if (!ok) {
                    spdlog::warn("Failed to sync SET to primary peer for key: {}", key);
                }
            }

            // 2. 向所有其他节点（除了主节点）发送失效通知，保证最终一致性
            auto all_peers = peer_picker_->GetAllPeers();
            for (auto peer : all_peers) {
                if (peer != primary_peer) {  // 排除已经同步过的主节点
                    bool ok = peer->Invalidate(name_, key);
                    if (!ok) {
                        spdlog::warn("Failed to invalidate key [{}] on peer", key);
                    }
                }
            }
            spdlog::debug("SET operation synced: key [{}] set on primary node, invalidated on {} other nodes", key,
                          all_peers.size() - (primary_peer ? 1 : 0));
            break;
        }
        case SyncFlag::DELETE: {
            // 对于 DELETE 操作，需要广播到所有节点
            auto all_peers = peer_picker_->GetAllPeers();
            for (auto peer : all_peers) {
                bool ok = peer->Delete(name_, key);
                if (!ok) {
                    spdlog::warn("Failed to sync DELETE to peer for key: {}", key);
                }
            }
            spdlog::debug("DELETE operation synced: key [{}] deleted on {} nodes", key, all_peers.size());
            break;
        }
        case SyncFlag::INVALIDATE: {
            // 向所有其他节点发送失效通知
            auto all_peers = peer_picker_->GetAllPeers();
            for (auto peer : all_peers) {
                bool ok = peer->Invalidate(name_, key);
                if (!ok) {
                    spdlog::warn("Failed to invalidate key [{}] on peer", key);
                }
            }
            spdlog::debug("INVALIDATE operation synced: key [{}] invalidated on {} nodes", key, all_peers.size());
            break;
        }
        default:
            spdlog::warn("Unknown sync operation: {}", static_cast<int>(op));
            return;
    }
}
```

`SyncToPeers` 支持两种同步操作类型，通过 `SyncFlag` 枚举定义：

```c++
enum class SyncFlag {
    SET,
    DELETE,
};
```



#### Set操作

调用本地缓存（LRUCache）的`set`接口，设置对应的缓存，并将操作同步到其他远程节点

```c++
bool CacheGroup::Set(const std::string& key, ByteView b, bool is_from_peer = false) {
    if (is_close_) {
        spdlog::error("Cache group [{}] is closed!!!", name_);
        return false;
    }
    if (key.empty()) {
        spdlog::warn("The key [{}] is empty, you can't set it into cache group", key);
        return false;
    }
    cache_->Set(key, b);
    // 必须是本身这个节点设置的值，才会同步到其他节点
    if (!is_from_peer && peer_picker_) {
        SyncToPeers(key, SyncFlag::SET, b);
    }
    return true;
}
```



#### Delete操作

调用本地缓存的 `Delete`接口，删除对应的缓存，并将操作同步到其他远程节点。



```c++
bool CacheGroup::Delete(const std::string& key, bool is_from_peer = false) {
    if (is_close_) {
        spdlog::error("Cache group [{}] is closed!!!", name_);
        return false;
    }
    if (key.empty()) {
        spdlog::warn("The key [{}] is empty, you can't delete it from cache group", key);
        return false;
    }
    cache_->Delete(key);
    // 必须是本身这个节点设置的值，才会同步到其他节点
    if (!is_from_peer && peer_picker_) {
        SyncToPeers(key, SyncFlag::DELETE, ByteView{""});
    }
    return true;
}
```





## 5. gRPC Server

概述：

每个节点在启动时候会通过MakeCacheGroup函数创建需要的缓存组；

在需要获取其他远程节点的相同缓存组中的数组时候，会在gRPC请求中携带缓存组的名字，然后远程节点会通过GetCacheGroup（name）获取其内部对应的缓存组，然后在这个缓存组上进行操作。

![image-20251021224940371](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20251021224940371.png)
