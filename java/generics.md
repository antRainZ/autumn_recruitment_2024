# 泛型

# 集合
Java 集合可分为 Set、List 和 Map 三种体系
+ Set：无序、不可重复的集合
+ List：有序，可重复的集合
+ Map：具有映射关系的集合

Collection 接口的接口 对象的集合（单列集合）
+ List 接口：元素按进入先后有序保存，可重复
  + LinkedList 接口实现类， 链表， 插入删除， 没有同步， 线程不安全
  + CopyOnWriteArrayList 接口实现类
  + ArrayList 接口实现类， 数组， 随机访问， 没有同步， 线程不安全
  + Vector 接口实现类 数组， 同步， 线程安全
│   + Stack
+ Set 接口： 仅接收一次，不可重复，并做内部排序
  + HashSet 使用hash表（数组）存储元素
  + LinkedHashSet, 也继承 `HashSet` 链表维护元素的插入次序
  + TreeSet 底层实现为二叉树，元素排好序
+ Queue 接口
  + PriorityQueue 实现
  + BlockingQueue接口
    + ArrayBlockingQueue
    + DelayQueue
    + LinkedBlockingQueue
    + TransferQueue 接口: 供了一个transfer的方法，生产者可以调用这个transfer方法，从而等待消费者调用take或者poll方法从Queue中拿取数据
  + Deque 接口: 头部或者尾部插入和删除元素
    + ArrayDeque
    + LinkedList
    + BlockingDeque 接口
  
Map 接口 键值对的集合 （双列集合）
+ Hashtable 接口实现类， 同步， 线程安全
+ HashMap 接口实现类 ，没有同步， 线程不安全-
  + LinkedHashMap 双向链表和哈希表实现
  + WeakHashMap
+ TreeMap 红黑树对所有的key进行排序
+ IdentifyHashMap

Iterator 接口主要用于遍历 Collection 集合中的元素，Iterator 对象也被称为迭代器

## 接口

```java
public interface Iterable<T> {
    Iterator<T> iterator();
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
}

public interface Collection<E> extends Iterable<E>{
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    Iterator<E> iterator();
    Object[] toArray();
    <T> T[] toArray(T[] a);
    boolean add(E e);
    boolean remove(Object o);
    boolean addAll(Collection<? extends E> c);
    void clear();
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
}

public interface Queue<E> extends Collection<E> {
    // 添加失败的时候会返回false
    boolean offer(E e);
    E remove();
    E poll(); // 会返回null
    E element();
    E peek(); // 不删除元素,peek返回null
}
```

