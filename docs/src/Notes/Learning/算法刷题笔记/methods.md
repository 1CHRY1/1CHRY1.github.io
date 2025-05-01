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

##### List

`Arrays.sort` Array排序
`Array.asList(num1, num2, num3)` 生成一个List
`Arrays.copyOfRange(arr, start, end)` Array截取子列

`listName.sort(null)` List排序

`Arrays.stream(nums).max().getAsInt()` 获取数组最大值

##### Map

`map.getOrDefault ` 获取map中key对应的value，如果不存在则返回默认值
`map.size()` 获取map的长度
`map.clear()` 清除map内容

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

##### 队列

`优先队列`：
```java

// 初始化队列
PriorityQueue<int[]> pq = new PriorityQueue<int[]>(new Comparator<int[]>() {
    public int compare(int[] pair1, i[] pair2) {
        return pair1[0] != pair2[0] pair2[0] - pair1[0] : pair2[1] pair1[1];
    }
});

// 添加元素
for (int i = 0; i < k; ++i) {
    pq.offer(new int[]{nums[i], i});
}
```

