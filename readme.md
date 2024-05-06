### 一个基本的高可用kafka集群需要多少个节点呢？

- 一个基本的高可用kafka集群至少需要3个节点，因为kafka的高可用是通过副本机制来实现的，而副本机制至少需要两个副本，所以至少需要3个节点。
    - 3个controller+3个broker+3个zookeeper
    - 利用这种 ISR 模型和 f+1 个副本，Kafka 主题可以承受 f 次故障而不会丢失已提交的消息
    - 意思是，如果topic的复制因子replication factor是2（复制因子是包括leader的，见官方文档：The total number of replicas including the leader constitute the replication factor），那么在一个节点失败的情况下，Kafka还是可以正常工作的。这里Kafka采用的算法和ZooKeeper, Elasticsearch集群的算法是不一样的。如果换成ZK和ES，只有两个节点，这时ZK和ES是无法工作的。
    - 那这么看来，Kafka集群其实只需要2个节点就可以了，为什么还是3个节点呢？带着这个疑问，我查了一下stackoverflow，还真有人问过这个问题，见[in-kafka-ha-why-minimum-number-of-brokers-required-are-3-and-not-2](https://stackoverflow.com/questions/58761164/in-kafka-ha-why-minimum-number-of-brokers-required-are-3-and-not-2)
        - 如果复制因子是2，min.insync.replicas也是2，当有一个节点失败时，生产者无法完成写入
        - 如果复制因子是2，min.insync.replicas是1，当leader失败时，follower则可能没有改消息
        - 如果复制因子是3，min.insync.replicas是2，当有一个节点失败时，生产者还是可以正常写入
        - 如果复制因子是3，min.insync.replicas是2，当leader失败时，至少还有一个follower的消息时和leader同步的，所以这时可以完成leader的切换
