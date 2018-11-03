# ArrayList
## ArrayList扩容
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
