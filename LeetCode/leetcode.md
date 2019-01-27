# 十大经典排序算法
排序算法可以分为**内部排序**和**外部排序**  
内部排序：数据记录在内存中进行排序  
外部排序：因排序的数据很大，一次不能容纳全部的排序记录，在排序过程中需要访问外存  
常见内部排序算法：插入排序、希尔排序、选择排序、冒泡排序、归并排序、快速排序、堆排序、基数排序。  
![](https://mmbiz.qpic.cn/mmbiz_png/D67peceibeISwc3aGibUlvZ0XqVnbWtBRiaKhGcwh6KibXbSiadtHqwgjmmzBYCa2DNuj5Vhw3lHc96z1wge3ZbDAeg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  
注：logn是以2为底。  
* 关于时间复杂度  
1.平方阶(O(n2))排序：直接插入、直接选择和冒泡排序；2.线性对数阶(O(nlog2 n))排序：快速排序、堆排序和归并排序；O(n1+§)排序，k是介于0和1之间的常数：希尔排序；线性阶(O(n))排序：基数排序，桶排序、箱排序。
* 关于稳定性  
1.稳定排序算法：冒泡排序、插入排序、归并排序和基数排序；2.不稳定排序算法： 选择排序、快速排序、希尔排序、堆排序
## 冒泡排序
### 步骤
* 比较相邻的元素。如果第一个比第二个大，就交换他们两个
* 对每对相邻元素做同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
* 针对所有的元素重复以上的步骤，除了最后一个。
* 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。
### 参考代码
```java
  public class BubbleSort implements IArraySort {
    @Override
    public int[] sort(int[] data) throws Exception {
      int[] arr = Arrays.copyOf(data,data.length);//拷贝一份数组，不改变参数内容
      int len = arr.length;
      //5 4 2 3 8   比较次数
      //4 2 3 5     4次
      //2 3 4       3次
      //2 3         2次
      //2           1次
      for(int i=1;i<len;i++){
        //设定一个标记，若为true，表示此次循环没有进行交换，也就是待排序列已经有序，排序已经完成
        boolean flag =truw;
        for(int j=0;j<len-i;j++){
          if(arr[j]>arr[j+1]){
            int tmp=arr[j];
            arr[j]=arr[j+1];
            arr[j+1]=tmp;
            flag=false;
          }
        }
        if(flag){
          break;
        }
      }
      return arr;
    }
  }
```
## 选择排序
### 步骤
* 首先在未排序序列中找到最小(大)元素，存放到排序序列的起始位置
* 再从剩余未排序元素中继续寻找最小(大)元素，然后放到已排序序列的末尾.
* 重复第二步，直到所有元素均排序完毕。
### 参考代码
```java
  public class SelectionSort implements IArraySort{
    @Override
    public int[] sort(int[] data) throws Exception{
      int[] arr Arrays.copyof(data,data.length);
      //总共要经历n-1轮比较
      int len=arr.length;
      //4 2 3 6 5
      for(int i=0;i<len-1;i++){
        int min=i;
        //每轮需要比较的次数n-i
        for(int j=i+1;j<len;j++){
          if(arr[j]<arr[min]){
            //记录目前能找到最小值元素的下标
            min=j;
          }
        }
        //将找到最小值和i位置所在的值进行交换
        if(i!=min){
          int tmp=arr[i];
          arr[i]=arr[min];
          arr[min]=tmp;
        }
      }
      return arr;
    }
  }
```
## 插入排序
#### 步骤
* 将第一待排序序列第一个元素看做一个有序序列，把第二个元素到最后一个元素当成是未排序序列
* 从头到尾依次扫描未排序序列，将扫描到的每个元素插入有序序列的适当位置。(如果待插入的元素与有序序列中的某个元素相等，则将待插入元素插入到相等元素的后面)
### 参考代码
```java
  public class InserSort implements IArraySort{
    @Override
    public int[] sort(int[] data) throws Exception{
      int[] arr = Arrays.copyof(data,data.length);
      int len=arr.length;
      //5 3 4 7 2
      //从下标为1的元素开始选择合适的位置插入，因为下标为0的只有一个元素，默认有序的
      for(int i=1;i<len;i++){
        //记录要插入的数据
        int tmp= arr[i];
        //从已经排序的序列最右边的开始比较，找到比其小的数
        int j=i;
        while(j>0&&tmp<arr[j-1]){
          arr[j]=arr[j-1];
          j--;
        }
        //存在比其小的数，插入
        if(j!=i){
          arr[j]=tmp;
        }
      }
      return arr;
    }
  }
```
## 希尔排序
### 步骤
* 选择一个增量序列t1,t2,...,tk,其中ti>tj,tk=1;
* 按增量序列个数k，对序列进行k趟排序
* 每趟排序，根据对应的增量ti，将待排序列分割成若干个长度为m的子序列，分别对各子表进行直接插入排序。仅增量因子为1时，整个序列作为一个表来处理，表长度即为整个序列的长度。
### 参考代码
```java
  public class ShellSort implements IArraySort{
    @Override
    public int[] sort(int[] data) throws Exception{
      int[] arr=Arrays.copyof(data,data.length);
      int gap=1;
      int len=arr.length;
      //8 9 1 7 2 3 5 4 6 0
      //2 3 1 4 8 9 5 7 6 0
      //2 3 1 4 6 9 5 7 8 0
      //0 3 1 4 6 2 5 7 8 9
      while(gap<len){
        gap=gap*3+1
      }
      while(gap>0){
        for(int i= gap;i<len;i++){
          int tmp=arr[i];
          int j=i-gap;// 4 5 6 7 8 9
          while(j>=0&&arr[j]>tmp){
            arr[j+gap]=arr[j];
            j-=gap;
          }
          arr[j+gap]=tmp;
        }
        gap=(int)Math.floor(gap/3);
      }
      return arr;
    }
  }
```
## 归并排序
### 步骤
* 申请空间，使其大小为两个已经排序序列之和，该空间用来存放合并后的序列
* 设定两个指针，最初位置分别为两个已经排序序列的起始位置
* 比较两个指针所指向的元素，选择相对小的元素放入到合并空间，并移动指针到下一位置
* 重复步骤3直到某一指针达到序列尾
* 将另一个序列剩下的所有元素直接复制到合并序列尾
### 参考代码
```java
  public class MergeSort implements IArraySort{
    @Override
    public int[] sort(int[] data)throws Exception{
      int[] arr=Arrays.copyof(data,data.length);
      int len=arr.length;
      if(len<2){
        return arr;
      }
      //6 4 3 7 5 1 2
      int middle=(int)Math.floor(len/2);
      int[] left=Arrays.copyOfRange(arr,0,middle);
      int[] right=Arrays.copyOfRange(arr,middle,arr.length);
      return merge(sort(left),sort(right));
    }
    
    protected int[] merge(int[] left,int[] right){
      int[] result=new int[left.length+right.length];
      int i=0;
      while(left.length>0&&right.length>0){
        if(left[0]<=right[0]){
          result[i++]=left[0];
          left=Arrays.copyOfRange(left,1,left.length);
        }else {
          result[i++]=right[0];
          right=Arrays.copyOfRange(right,1,right.length);
        }
      }
      while(left.length>0){
        result[i++]=left[0];
        left=Arrays.copyOfRange(left,1,left.length);
      }
      while(right.length>0){
        result[i++]=right[0];
        right=Arrays.copyOfRange(right,1,right.length);
      }
      return result;
    }
  }
```
## 快速排序
### 步骤
* 从数列中挑出一个元素，称为"基准"(pivot);
* 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面(相同的数可以到任一边)。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区(partition)操作
* 递归地把小于基准值元素的子数列和大于基准值元素的子数列排序
### 参考代码
```java
  //3 5 8 1 2 9 4 7 6
  public class QuickSort implements IArraySort{
    @Override
    public int[] sort(int[] data) throws Exception{
      int[] arr=Arrays.copyof(data,data.length);
      return quickSort(arr,0,arr.length-1);
    }
    
    private int[] quickSort(int[] arr,int left,int right){
      if(left<right){
        int partitionIndex=partition(arr,left,right);
        quickSort(arr,left,partitionIndex-1);
        quickSort(arr,partitionIndex+1,right);
      }
      return arr;
    }
    private int partition(int[] arr,int left,int right){
      //设定基准值
      int pivot=left;//0
      int index=pivot+1;//1
      for(int i=index;i<=right;i++){//8
        if(arr[i]<arr[pivot]){
          swap(arr,i,index);
          index++;
        }
      }
      swap(arr,pivot,index-1);
      return index-1;
    }
    
    private void swap(int[] arr,int i,int j){
      int temp=arr[i];
      arr[i]=arr[j];
      arr[j]=temp;
    } 
  }
```
## 堆排序
### 步骤
* 创建一个堆 H[0...n-1];
* 把堆首(最大值)和堆尾互换
* 把堆的尺寸缩小1，并调用shift_down(0),目的是把新的数组顶端数据调整到相应位置
* 重复步骤2，直到堆的尺寸为1
### 参考代码
```java
  public class HeapSort implements IArraySort{
    @Override
    public int[] sort(int[] data) throws Exception{
      int[] arr=Arrays.copyof(data,data.length);
      int len = arr.length;
      //5 2 7 3 6 1 4
      buildMaxHeap(arr,len);
      for(int i=len-1;i>0;i--){
        swap(arr,0,i);
        len--;
        heapify(arr,0,len);
      }
      return arr;
    }
    
    private void buildMaxHeap(int[] arr,int len){
      for(int i=(int)Math.floor(len/2);i>=0;i--){
        heapify(arr,i,len);//3 7
      }
    }
    
    private void heapify(int[] arr,int i,int len){
      int left=2*i+1;
      int right=2*i+2;
      int largest=i;//2
      if(left<len&&arr[left]>arr[largest]){
        largst=left;
      }
      if(right<len&&arr[right]>arr[largest]){
        largest=right;
      }
      if(largest!=i){
        swap(arr,i,largest);
        heapify(arr,largest,len);
      }
    }
    
    private void swap(int[] arr,int i,int j){
      int temp=arr[i];
      arr[i]=arr[j];
      arr[j]=temp;
    }
  }
```
## 计数排序
### 步骤
* 花O(n)时间扫描一下整个序列A，获取最小值min和最大值max
* 开辟一块新的空间创建新的数组B，长度为(max-min+1)
* 数组B中index的元素记录的值是A中某元素出现的次数
* 最后输出目标整数序列，具体的逻辑是遍历数组B，输出相应元素以及对应的个数
### 参考代码
```java
  public class CountingSort implements IArraySort{
    @Override
    public int[] sort(int[] data) throws Exception{
      int[] arr=Arrays.sort(data,data.length);
      int maxValue=getMaxValue(arr);
      //5 3 4 7 2 4 3 4 7
      return countingSort(arr,maxValue);
    }
    
    private int getMaxValue(int[] arr){
      int maxValue=arr[0];
      for(int value: arr){
        if(maxValue<value){
          maxValue=value;
        }
      }
      return maxValue;
    }
    
    private int[] countingSort(int[] arr,int maxValue){
      int bucketLen=maxValue+1;
      int[] bucket=new int[bucketLen];
      for(int value:arr){//0..7
        bucket[value]++;
        //0 1 2 3 4 5 6 7
        //0 0 1 2 3 1 0 2
      }
      int sortedIndex=0;
      for(int j=0;j<bucketLen;j++){//2,3
        while(bucket[j]>0){//1,2
          arr[sortedIndex++]=j;//0,1,2
          bucket[j]--;
        }
      }
      return arr;
    }
    
  }
```
## 桶排序
### 步骤
* 设置固定数量的空桶
* 把数据放到对应的桶中
* 对每个不为空的桶中数据进行排序
* 拼接不为空的桶中数据，得到结果
### 参考代码
```java
  //7 12 56 23 19 33 35 42 42 2 8 22 39 26 17
  //范围(56-2+1)/5=11
  public class BucketSort implements IArraySort{
    private static final InsertSort insertSort=new InsertSort();
    @Override
    public int[] sort(int[] data) throws Exception{
      int[] arr=Arrays.copyof(data,data.length);
      return bucketSort(arr,5);
    }
    
    private int[] bucketSort(int[] arrs,int bucketSize) throws Exception{
      if(arr.length==0){
        return arr;
      }
      int minValue=arr[0];
      int maxValue=arr[0];
      for(int value:arr){
        if(value<minValue){
          minValue=value;
        }else if(value>maxValue){
          maxValue=value;
        }
      }
      int bucketCount=(int)Math.floor((maxValue-minValue)/bucketSize)+1;//11
      int[][] buckets=new int[bucketCount][0];
      //利用映射函数将数据分配到各个桶中
      for(int i=0;i<arr.length;i++){//15
        int index=(int)Math.floor((arr[i]-minValue)/bucketSize);
        buckets[index]=arrAppend(buckets[index],arr[i]);
      }
      int arrIndex=0;
      for(int[] bucket: buckets){
        if(bucket.length<=0){
          continue;
        }
        //对每个桶进行排序，这里使用了插入排序
        bucket=inserSort.sort(bucket);
        for(int value: bucket){
          arr[arrIndex++]=value;
        }
      }
      return arr;
    }
    /**
    *自动扩容，并保存数据
    */
    private int[] arrAppend(int[] arr,int value){
      arr=Arrays.copyof(arr,arr.length+1);
      arr[arr.length-1]=value;
      return arr;
    }
  }
  
```
## 基数排序
### 步骤
* 将所有待比较数值(正整数)统一为同样的数位长度，数位较短的数前面补零
* 从最低位开始，依次进行一次排序
* 从最低位排序一直到最高位排序完成以后，数列就变成一个有序序列
### 参考代码
```java
  //321 1 10 60 577 743 127
  public class RadixSort implements IArraySort{
    @Override
    public int[] sort(int[] data) throws Exception{
      int[] arr=Arrays.copyof(data,data.length);
      int maxDigit=getMaxDigit(arr);
      return radixSort(arr,maxDigit);
    }
    //获取最高位数
    private int getMaxDigit(int[] arr){
      int maxValue=getMaxValue(arr);
      return getNumLength(maxValue);
    }
    
    private int getMaxValue(int[] arr){
      int maxValue=arr[0];
      for(int value:arr){
        if(maxValue<value){
          maxValue=value;
        }
      }
      return maxValue;
    }
    
    private int getNumLength(long num){
      if(num==0){
        return 1;
      }
      int length=0;
      for(long temp=num;temp!=0;temp/=10){
        length++;
      }
      return length;
    }
    
    private int[] radixSort(int[] arr,int maxDigit){
      int mod=10;
      int dev=1;
      for(int i=0;i<maxDigit;i++,dev*=10,mod*=10){
        //考虑负数的情况，这里扩展一倍队列数，其中[0-9]对应负数，[10-19]对应正数(bucket+10)
        int[][] counter=new int[mod*2][0];
        for(int j=0;j<arr.length;j++){
          int bucket=((arr[j]%mod)/dev)+mod;
          counter[bucket]=arrayAppend(counter[bucket],arr[j]);
        }
        int pos=0;
        for(int[] bucket:counter){
          for(int value:bucket){
            arr[pos++]=value;
          }
        }
      }
      return arr;
    }
    private int[] arrayAppend(int[] arr,int value){
      arr=Arrays.copyof(arr,arr.length+1);
      arr[arr.length-1]=value;
      return arr;
    }
  }
```
# Tow Sum
```
  给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。示例：
  给定nums=[2,7,11,15],target=9,因为nums[0]+nums[1]=2+7=g,所以返回[0,1]
```
## 思路
使用查找表来解决该问题
> 设置一个map容器record用来记录元素的值与索引
> 遍历数据nums
>> * 每次遍历时使用临时变量complement,用来保存目标值与当前值的差值
>> * 在此次遍历中查找record，查看是否有与complement一致的值，如果查找成功则返回查找值的索引值与当前变量的值i
>> * 如果未找到，则在record保存该元素与索引值i
### 参考代码
```
  //时间复杂度：O(n)
  //空间复杂度：O(n)
  class Solution{
    public int[] towSum(int[] nums,int target){
      int len=nums.length;
      Map<Integer,Integer> map=new HashMap(len);
      for(int i=0;i<len;i++){
        int curValue=nums[i];
        int complement=target-curValue;
        if(map.containsKey(complement)){
          return new int[]{i,map.get(complement)}
        }else{
          map.put(curValue,i);
        }
      }
      throw new IllegalArgumentException("No two sum solution");
    }
  }
```
# 两个数字相加
> 给定两个非空链表来表示两个非负整数。位数按照逆序方式存储，它们的每个节点只存储单个数字。将两数相加返回一个新的链表。
> 你可以假设除了数字0之外，这两个数字都不会以零开头。
> 示例：输入:(2->4->3)+(5->6->4),输出：7->0->8,结果：342+465=807
## 思路
设立一个表示进位的变量carried，建立一个新链表，把输入的两个链表从头往后同时处理，每两个相加，将结果加上carried后的值作为一个新节点到新链表后面。
## 参考代码
```java
  public class Solution{
    public ListNode addTowNumbers(ListNode node1,ListNode node2){
      ListNode dummy=new ListNode(-1);
      ListNode cur=dummy;
      int carry=0;
      while(node1!=null||node2!=null){
        int num1=node1==null?0:node1.val;
        int num2=node2==null?0:node2.val;
        int sum=num1+num2+carry;
        carry=sum>=10?1:0;
        cur.next=new ListNode(sum%10);
        cur=cur.next;
        if(node1!=null){
          node1=node1.next;
        }
        if(node2!=null){
          node2=node2.next;
        }
      }
      if(carry==1) cur.next=new ListNode(1);
      return dummy.next;
    }
  }
```
## 无重复字符的最长字串
> 给定一个字符串，找出不含有重复字符的最长子串的长度。
> 示例 1：
> 输入："abcabcbb"
> 输出：3
> 解析：无重复字符的最长子串是"abc"，其长度为3
> 输入："bbbbb"
> 输出：1
> 解析：无重复字符的最长子串是"b",其长度为1
> 输入："pwwkew"
> 输出：3
> 解析：无重复字符的最长子串是"wke",其长度为3
> 请注意，答案必须是一个子串，"pwke"是一个子序列，而不是子串
### 思路解析
建立一个256位大小的整型数组freg，用来建立字符和其出现位置之间的映射。维护一个滑动窗口，窗口内的都是没有重复的字符，去尽可能的扩大窗口的大小，窗口不停的向右滑动。
* 1.如果当前遍历到的字符从未出现过，那么直接扩大右边界
* 2.如果当前遍历到的字符出现过，则缩小窗口(左边索引向右移动)，然后继续观察当前遍历到的字符
* 3.重复1 2步骤，直到左边索引无法再移动
* 4.维护一个结果res，每次用出现过的窗口大小来更新结果res，最后返回res获取结果。
### 参考代码
```java
  //滑动窗口
  //时间复杂度：O(len(s))
  //空间复杂度：O(len(charset))
  public class Solution{
    publc int lengthOfLongestSubstring(String s){
      int len = s.length(),ans=0;
      int[] index=new int[128];//当前字符下标
      //扩容
      for(int j=0,i=0;j<n;j++){
        i=Math.max(index[s.charAt(j)],i);
        ans=Math.max(ans,j-i+1);
        index[s.chatAt(j)]=j+1;
      }
      return ans;
    }
  }
```
## 删除链表的倒数第N个节点
> 给定一个链表，删除链表的倒数第n个节点，并返回链表的头结点。
> 示例：
> 给定一个链表：1->2->3->4->5,和n=2。当删除了倒数第二个节点后，链表变为1->2->3->5
> 说明：给定的n保证是有效的。
> 进阶：你能尝试使用一趟扫描实现吗？
### 解题思路
我们可以设想假设设定了双指针p和q的话，当q指向末尾的Null，p与q之间相隔的元素个数为n时，那么删除掉p的下一个指针就完成要求了。
* 设置虚拟节点dummyHead指向head
* 设定双指针p和q，初始都指向虚拟节点dummyHead
* 移动q，直到p与q之间相隔的元素个数为n
* 同时移动p与q，直到q指向的为Null
* 将p的下一个节点指向下下个节点
### 参考代码
```JAVA
  public ListNode removeNthFromEnd(ListNode head,int n){
    ListNode dummy=new ListNode(0);
    dummy.next=head;
    ListNode first=dummy;
    ListNode second=dummy;
    for(int i=1;i<=n+1;i++){
      first=first.next;
    }
    while(first!=null){
      first=first.next;
      second=second.next;
    }
    second.next=second.next.next;
    return dummy.next;
  }
```
## 有效的括号
> 给定一个只包括'(',')','{','}','[',']'的字符串，判断字符串是否有效。有效字符串需满足：
> 左括号必须用相同类型的右括号闭合
> 左括号必须以正确的顺序闭合
> 注意空字符串可被认为是有效字符串
> 示例1： 输入："()", 输出：true
> 示例2： 输入："(){}[]",输出：true
> 示例3： 输入："(]",输出：false
> 示例4： 输入："([)]",输出：false
> 示例5： 输入："{[]}",输出：true
### 解题思路
* 遍历输入字符串
* 如果当前字符为左半边括号时，则将其压入栈中
* 如果遇到右半边括号时，分类讨论：
* 1.如栈不为空且为对应的左半边括号，则取出栈顶元素，继续循环
* 2.若此时栈为空，则直接返回false
* 3.若不为对应的左半边括号，反之返回false
### 参考代码
```java
  public class Solutiong{
    private  HashMap<Character,Character> mappings;
    public Solution(){
      this.mappings=new HashMap<>();
      this.mappings.put(")","(");
      this.mappings.put("}","{");
      this.mappings.put("]","[");
    }
    public boolean isValid(String s){
      Stack<Character> stack=new Stack<>();
      for(int i=0;i<s.length();i++){
        char c=s.charAt(i);
        if(this.mappings.containsKey(c)){
          char topElement=stack.empty()?"#":stack.pop();
          if(topElement!=this.mappings.get(c)){
            return false;
          }
        }else{
          stack.push(c);
        }
      }
      return stack.isEmpty();
    }
  }
```






























# 感谢
[五分钟学算法](https://mp.weixin.qq.com/s/vn3KiV-ez79FmbZ36SX9lg)



















