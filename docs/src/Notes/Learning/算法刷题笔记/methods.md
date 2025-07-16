## 算法方法总结

#### <font color=darkblue>1. Java</font>

##### 字符串

`charAt(index)` 获取第index个字符

`toCharArray` 将字符串转为字符数组

`string.subString(start,end)` 截取字符串，左闭右开

char[]和String相互转换：
String s = new String(char);
char[] c = s.toCharArray();

字符串构建器：
`StringBuilder sb = new StringBuilder();`
`sb.append(c);`
`return sb.toString();`

`string.contains("ab")`判断是否为某字符串的子串

##### List

`Arrays.sort` Array排序
`Array.asList(num1, num2, num3)` 生成一个List
`Arrays.copyOfRange(arr, start, end)` Array截取子列

`listName.sort(null)` List排序

`Arrays.stream(nums).max().getAsInt()` 获取数组最大值

`Collection.reverse(list)` List反转

`list1.equals(list2)` 判断List相等

##### Map

`map.getOrDefault ` 获取map中key对应的value，如果不存在则返回默认值
`map.size()` 获取map的长度
`map.clear()` 清除map内容

遍历map的key
`for(String key : map.keySet())`
`for(Map.Entry<Object,Object> entry : map.entrySet())` `entry.getKey()` `entry.getValue()`

##### 类型转换

`type(value)` 记得类型转换

##### Set

`set.add` set中添加元素
`set.remove` set中删除元素
`set.contains` 判断set中是否包含某个元素
`set.size()` 获取set的长度
`set.clear()` 清除set内容
`set1.equals(set2)` 判断两个set是否相等

##### 数值

`Float/Integer.MIN(MAX)_VALUE` 获取数值无限小


##### 栈与队列

`Deque<Object>` 栈
`ArrayDeque<Object>();` 声明双端队列
操作：`stack.push()`,`stack.pop()`,`stack.peek()`,`stack.isEmpty()`

`Queue<Object>` 队列
`LinkedList<>();` 声明队列（链表形式）
操作：`queue.offer()`,`queue.poll()`,`queue.peek()`,`queue.isEmpty()`   

`优先队列(小顶堆的实现)`：
```java

// 写法1 推荐这个吧
PriorityQueue<int[]> queue = new PriorityQueue<>((m, n) -> m[1] - n[1]);

// 写法2
PriorityQueue<int[]> queue = new PriorityQueue<int[]>(
    new Comparator<int[]>() {
        public int compare(int[] m, int[] n) {
            return m[1] - n[1];
        }
    });

```

