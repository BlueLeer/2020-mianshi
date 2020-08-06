## 区别:
* LinkedHashMap是继承HashMap，是基于HashMap和双向链表来实现的。LinkedHashMap是有序的，可分为插入顺序和访问顺序两种，默认构造是按照插入顺序来的，可以在构造的时候指定是否按照访问顺序来，如果是访问顺序，那get和put操作已经存在的元素时，都会把元素移动到双向链表的末尾（其实是先删除再插入）。
* LinkedHashMap存取数据，还是和HashMap一样，使用的Entry[]的方式，双向链表只是为了保证顺序。
* LinkedHashMap是现成不安全的

## LinkedHashMap的应用场景
当我们希望数据的顺序和插入的数据的顺序一致的时候，可以考虑使用LinkedHashMap，HashMap里面的数据是无序的。
LinkedHashMap默认构造出来的是按照插入顺序的，我们也可以指定为访问顺序。当指定访问顺序的时候，访问一个元素，该元素就会移动到Entry[]的末尾去

## TreeMap
TreeMap主要用于排序，放入其中的元素都是按照key进行排好顺序的，默认是升序排序的。如果想要改变其顺序，可以自己写一个Comparator（比较器）。

