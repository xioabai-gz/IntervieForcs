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







## 2.一致性哈希

一致性哈希是分布式系统中使用的一种技术，用于在多个节点之间分发数据，以最大限度的**减少**添加或删除节点时的重组

![image.jpg](https://cdn.nlark.com/yuque/0/2025/jpeg/39162301/1753611575177-f5318037-b146-476b-8b30-1823fca24223.jpeg?x-oss-process=image%2Fformat%2Cwebp)

当节点数量发生变化时，使用 `hash(key) % n`（其中 n 是节点数）的传统基于哈希的分发会遇到重大的重新分发问题。一致性哈希通过确保在节点计数更改时只需要重新映射一小部分键来解决此问题。

一致性哈希的大致架构用上图就能说明：由一个 `[0, 2^32-1]` 的“线”围成了一个环，对于分布式系统中的每个真实物理节点，都会映射成多个虚拟节点（如上图中使用了 3 个物理节点，这三个物理节点又各自映射了三个虚拟节点），对这些虚拟节点的标识进行哈希运算，就可以得到一个 `[0, 2^32-1]`之间的值。如果把刚刚说的“线”用数组来表示，那么虚拟节点哈希后的值就是这个数组中的一个元素。

当有新节点时或者原有节点 node 发生故障，需要调整只有 node 附近的 key 所属





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

当每当一个新的物理节点来到的时候，原本存在的节点都会通过etcd得知消息，然后**每一个节点都会*更新自己的*一致性哈希环**

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
- 调整虚拟节点数量
- 更新哈希环
- 重置计数器
- 重新排序哈希环
  - 对`keys_`排序，确保哈希环按哈希值有序排列，便于后续查找