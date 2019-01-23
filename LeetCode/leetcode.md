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

# 感谢
[五分钟学算法](https://mp.weixin.qq.com/s/vn3KiV-ez79FmbZ36SX9lg)



















