#HashMap系列     
	public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable			
	public abstract class AbstractMap<K,V> implements Map<K,V>
	  
内部存储数据使用Node数组，默认大小是16，数组中每个元素都是一个Node，实际上是链表，所有hash值相同的key会被存储到同一个链表里。     
有自动扩容机制，如果元素大小达到数组大小\*loadFactor后会扩大数组的大小，例如数组大小是16，loadFactor为0.75，当HashMap中的元素超过16\*0.75=12时，数组的大小会扩展为2\*16=32，并且重新计算每个元素在新数组中的元素		
	
	transient Node<K,V>[] table;
	static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
	}
##HashMap和TreeMap的区别		

1. HashMap是基于散列表实现的，时间复杂度平均能达到)(1)           
   TreeMap是基于红黑树(一种平衡二叉查找树)实现的，时间复杂度平均能达到O(logn)     
2. 两者都继承自AbstractMap(实现了Map接口的一个抽象类)，TreeMap实现了NavigableMap接口，NavigableMap继承自SortedMap接口，所以TreeMap是有序的。    
		
		public class TreeMap<K,V> extends AbstractMap<K,V> implements NavigableMap\<K,V\>, Cloneable, java.io.Serializable      
		public interface NavigableMap<K,V> extends SortedMap<K,V>            
3. HashMap适用于在Map中插入、删除、定位元素，TreeMap适用于遍历key，在需要排序的时候可以使用TreeMap。         
   
##HashMap和Hashtable的区别	    
	 
1. Hashtable继承自Dictionary       
		
		public class Hashtable<K,V> extends Dictionary<K,V> implements Map<K,V>, Cloneable, java.io.Serializable      
2. Hashtable的key和value都不能为null，HashMap可以      
3. HashMap是非线程安全的，Hashtable是线程安全的(方法都是synchronized)，如果不需要同步，只需要单线程，那么使用HashMap的性能要好过Hashtable。HashMap可以通过下面的语句进行同步：    
    
		Map map = Collections.synchronizeMap(hashMap);         
4. HashMap的迭代器是fail-fast的，HashTable的迭代器不是fail-fast的，在遍历map的同时对map增删元素，会抛出ConcurrentModificationException       
	
	
##HashMap和HashSet的区别	     
	
1. HashSet继承自AbstractSet       
	
		public class HashSet<E> extends AbstractSet<E> implements Set<E>, Cloneable, java.io.Serializable	  
	 	public abstract class AbstractSet<E> extends AbstractCollection<E> implements Set<E>    
		public abstract class AbstractCollection<E> implements Collection<E>         
		public interface Collection<E> extends Iterable<E> 
		     
2. HashSet是基于HashMap实现的，HashSet内部有一个HashMap类型的成员变量，HashSet只存储对象，不存储键值对，HashSet存储时，将传入的对象作为key，new Object()作为value存入map中，使用传入的对象来计算hashcode值，HashMap用key来计算hashcode值   
             
##ConcurrentHashMap	    

	public class ConcurrentHashMap<K,V> extends AbstractMap<K,V> implements ConcurrentMap<K,V>, Serializable       
	public interface ConcurrentMap<K, V> extends Map<K, V>   
	
ConcurrentHashMap的key和value不能为null，是线程安全的     
ConcurrentHashMap在迭代的时候被修改，不会抛出ConcurrentModificationException。        
ConcurrentHashMap锁的粒度比Hashtable更小，Hashtable锁住了整个hash表，Hashtable包含Segment数组，Segment是一个继承自ReenTrantLock的类，将数据分段存储，每个Segment是一个传统意义上的hashtable，每个分段一个锁。
##LinkedHashMap	    

	public class LinkedHashMap<K,V>extends HashMap<K,V>implements Map<K,V>
    		
LinkedHashMap是HashMap的一个子类，重新定义了HashMap中保存元素的Node       
	
	static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
        LinkedHashMapEntry<K,V> before, after;
        LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
使用了双向链表来保存数据，遍历数据只和实际数据有关，和容量无关，HashMap遍历速度与容量有关。       


##SparseArray
	public class SparseArray<E> implements Cloneable {
		private int[] mKeys;
    	private Object[] mValues;
	}
存储key为int的元素，存储元素按照key的值从小到大排列，put和get方法使用二分法查找，在获取数据的时候非常快。但在数据量大的情况下性能并不明显，建议在数据量不大(千以内)使用。