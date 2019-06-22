自动装箱拆箱：
Integer i = new Integer(10);  //javaSE5之前必须这样声明
Integer i = 10;   //javaSE5之后可以这样  提供了自动装箱的特性
int i2 = i; //自动拆箱
装箱时调用valueOf（int） 拆箱过程调用 intValue

Short a = 100;
Short b = 100;
Short c = 200;
Short d = 200;

(a == b) 返回true   
(c == d) 返回false

详解：看short源码 
public static Short valueOf(short s) {
        final int offset = 128;
        int sAsInt = s;
        if (sAsInt >= -128 && sAsInt <= 127) { // must cache
            return ShortCache.cache[sAsInt + offset];
        }
        return new Short(s);
    }  

开辟了256个地址空间  创建short对象值+128 如果大于256就返回新对象 小于则不会
 private static class ShortCache {
        private ShortCache(){}

        static final Short cache[] = new Short[-(-128) + 127 + 1];

        static { 
            for(int i = 0; i < cache.length; i++)
                cache[i] = new Short((short)(i - 128));
        }
    }
装箱时调用valueOf()方法  判断传入参数是否在-128-127 范围内 如果在，就返回该对象 如果不在返回新的对象 
"=="是比较地址，所以（a == b） 判断数值在取值区间 故不会返回新对象 地址不变 返回true （c == d）返回新对象地址不同返回false

整数缓冲区：
预先创建了256个常用的整数包装类型对象。
在实际应用中，对已创建的对象进行复用。

String类的装箱与拆箱：

String s1 = “abc”; //直接赋值时，入池 1个对象
String s2 = "abc";   //复用，不会再创建新的对象  0个对象
String s3 = new String("abc"); 1个对象   //
(s1 == s2)返回true  
(s1 == s3)返回false

1.池中分配空间存入abc 2.堆中开辟空间存入abc 3.s3指向堆中对象
String s3 = new String("abc");  //2个对象
String s1 = "abc"; //0个对象 池中已经有对象了
String s2 = "abc"; //0个对象 



intern（）方法 返回字符串对象的规范化表示形式
一个初始化为空的字符串池，它由类String私有地维护
当调用intern方法时，如果池已经包含一个等于此String对象的字符串（用equal(Object)方法确定），
则返回池中的字符串。否则，将此String对象添加到池中，并返回此String对象的引用。
它遵循以下规则：对任意两个字符串s和t，当且仅当s.equals(t) 为true 时，s.intern() == t.intern()才为true。
所有字面值字符串和字符串赋值常量表达式都使用intern()方法进行操作。

字符串不变; 它们的值在创建后不能被更改。 字符串缓冲区支持可变字符串。   
String类中源码 定义了一个final的char类型数组  private final char value[];
因为String对象是不可变的，它们可以被共享。 例如：
String str = "abc";
相当于
  char data[] = {'a', 'b', 'c'};
  String str = new String(data);

StringBuffer：运行效率慢  、线程安全  since jdk1.0
线程安全的可变字符序列，一个类似于String的字符串缓冲区，但不能修改。
虽然在任意时间点上它都包含某种特定序列，但可以通过某些方法调用来更改序列的长度和内容。
StringBuffer()构造方法 创建一个其中不带字符的字符串缓冲区，初始容量为16


StringBuffer sBuffer = new StringBuffer();
sBuffer.append("abc");
sBuffer.append("def");
不会重新开辟空间，在原有基础上修改

append源码：
@Override
    public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str);
        return this;
    }

public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }

sBuffer.toString()；

    @Override
    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }

StringBuilder： 运行效率快、线程不安全  since jdk1.5
一个可变的字符序列，提供一个与StringBuffer兼容的API，但不保证同步

拓展：
String str = "Hello";
str += "World";    //只开辟一个空间
System.out.println(str);
jdk 1.5之后
编译时创建了一个StringBulider对象并初始化
调用StringBuilder.append()然后把world拼进去
接着调用StringBuilder.toString(); 转换成字符串  


BigDecimal
java.Math包下
不可变的、任意精度的带符号的十进制数字。



集合
概念：对象的容器，存储对象的对象，可代替数组
特点：容器的工具类，定义了对多个对象进行操作的常用方法
位于java.utils包下

继承树
interface： List，set ,SortedSet
实现了List接口的集合类:ArrayList,LinkedList,Vector
实现了Set接口的集合:HashSet,SortedSet接口
实现了SortedSet接口的集合:TreeSet

List接口：有序，有下标，元素可以重复  
Set接口：无序，无下标，元素不能重复

Collection父接口:
特点：代表一组任意类型的对象，无序，无下标。
方法：
boolean add(Object obj)  //添加一个对象
boolean addAll(Collection c);.//将一个集合中所有对象添加到此集合中
void clear(); //清空此集合中所有对象
boolean contains(Object obj) //检查此集合是否与指定对象相等
boolean equals(Object obj) //比较此集合是否与指定对象相等
boolean isEmpty() //判断此集合是否为空
boolean remove(Object obj) // 在此集合中移除obj对象
int size()  //返回此集合中的元素个数
Object[] toArray() //将此集合转换成数组


List接口：extends Collection
特点:有序、有下标元素可以重复
此接口的用户可以对列表中每个元素的插入位置进行精确地控制，用户可以根据元素的整数索引访问元素
并搜索列表中的元素。
与set区别是，List允许重复的元素。更确切的讲，List通常允许满足e1.equals(e2)的元素对e1和e2
并且如果List本身允许null元素的话，通常它们允许多个null元素。


void add(int index, Object obj) // 在指定index位置插入对象obj
boolean addAll(int index, Collection c);// 在指定位置插入集合c
Object get(int index)// 返回集合中指定位置的元素
List subList(int fromIndex, int toIndex)//返回fromIndex到toIndex之间的集合元素


List实现类： 

*ArrayList 
特点：有序、有下标、可重复、线程安全
List接口的大小可变数组的实现。实现了所有可选列表操作，并允许包括null在内的所有元素。
此类还提供一些方法来操作内部用来存储列表的数组的大小，（此类大致上等同于Vector，除了此类不同步）
jdk1.7之前创建时默认大小是10.  之后优化为默认为空
当首次调用add()方法时才会给定空间
调用remove时删除占用空间

















 










