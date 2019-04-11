# ArrayList
## 扩容
###### 初次扩容容量置为10，以后每次扩容容量增大一半
###### 自动扩容公式：capacity=capacity+(capacity>>1)
```java
public class TestCommon {
    public static void main(String[] args) {
        ArrayList list = new ArrayList();
        for (int i = 0; i < 20; i++) {
            list.add(1);

            System.out.println("当前i=" + i + "；当前elementData的length=" + getArrayListCapacity(list));
        }
        System.out.println(15>>1);
    }

    /**
     * 反射获取ArrayList的elementData的length
     *
     * @param arrayList
     * @return
     */
    public static int getArrayListCapacity(ArrayList<?> arrayList) {
        Class<ArrayList> arrayListClass = ArrayList.class;
        try {
            Field field = arrayListClass.getDeclaredField("elementData");
            field.setAccessible(true);
            Object[] objects = (Object[]) field.get(arrayList);
            return objects.length;
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
            return -1;
        } catch (IllegalAccessException e) {
            e.printStackTrace();
            return -1;
        }
    }

}

```
结果为：
```java
当前i=0；当前elementData的length=10
当前i=1；当前elementData的length=10
当前i=2；当前elementData的length=10
当前i=3；当前elementData的length=10
当前i=4；当前elementData的length=10
当前i=5；当前elementData的length=10
当前i=6；当前elementData的length=10
当前i=7；当前elementData的length=10
当前i=8；当前elementData的length=10
当前i=9；当前elementData的length=10
当前i=10；当前elementData的length=15
当前i=11；当前elementData的length=15
当前i=12；当前elementData的length=15
当前i=13；当前elementData的length=15
当前i=14；当前elementData的length=15
当前i=15；当前elementData的length=22
当前i=16；当前elementData的length=22
当前i=17；当前elementData的length=22
当前i=18；当前elementData的length=22
当前i=19；当前elementData的length=22
7
```
## 线程不安全
## 序列化
今天同事问到ArrayList中的
```java
private transient E[] elementData; 
```
声明为transient，为什么还可以序列化成功呢？
我的回答是ArrayList重写了writeObject
```java
private void writeObject(java.io.ObjectOutputStream s)  
        throws java.io.IOException{  
    int expectedModCount = modCount;  
    // Write out element count, and any hidden stuff  
    s.defaultWriteObject();  
  
        // Write out array length  
        s.writeInt(elementData.length);  
  
    // Write out all elements in the proper order.  
    for (int i=0; i<size; i++)  
            s.writeObject(elementData[i]);  
  
    if (modCount != expectedModCount) {  
        throw new ConcurrentModificationException();  
    }  
  } 
```
在使用ObjectOutputStream序列化对象时会调用这个writeObject方法。
第二个问题是为什么要声明为transient呢？
既然是数组，要序列化到文件中，那就单独测试下数组对象的序列化和反序列化吧
```java
String[] stra = new String[4];  
stra[0] = "mmmmmmmmmm";  
stra[2] = "nnnnnnnnnn";  
  
oos = new ObjectOutputStream(new FileOutputStream("D:\\sa.tmp"));  
oos.writeObject(stra);  
  
ois = new ObjectInputStream(new FileInputStream("D:\\sa.tmp"));  
  
String[] str  = (String[])ois.readObject();  
for(String s: str)  
{  
    System.out.println(s);  
}  
```
 输出结果为：
 ```java
 mmmmmmmmmm  
null  
nnnnnnnnnn  
null  
 ```
从输出结果来看，数组序列化时，不管是否有值，都会将整个数组序列化到文件中。

由此可以看出，比较靠谱的原因是：

ArrayList是会开辟多余空间来保存数据的，而系列化和反序列化这些没有存放数据的空间是要消耗更多资源的，所以ArrayList的数组就声明为transient，告诉虚拟机这个你别管，我自己来处理，然后就自己实现write/readObject方法，仅仅系列化已经存放的数据。
