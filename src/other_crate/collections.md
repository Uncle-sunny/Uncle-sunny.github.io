# 常用collections

## hashbrown
Rust标准库 **HashMap** 和 **HashSet** 替代品,可以在没有 **std** 的环境中工作

```sh
cargo add hashbrown
```

## crossbeam
高效的无锁并发队列，支持多生产者 - 多消费者（MPMC）模型


## indexmap
保留了插入顺序的HashMap
```sh
cargo add indexmap
```

## dashmap

采用分段式锁设计的DashMap,用于并发场景的哈希表

```sh
cargo add DashMap
```
