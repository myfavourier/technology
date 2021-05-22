# 算法

## 八大排序算法

插入、冒泡太傻，所以不做讨论

### 快速排序

#### 原理

1.选一个基准数，一般是选第一个数

2.从前往后选比基准数大的，从后往前选比基准数小的，交换两个数，直到遍历完一次数组

3.交换第一个数跟基准数，随后分前半部分和后半部分递归

#### 复杂度分析

数组基本有序时为最坏情况，退化为冒泡排序，时间复杂度为O(n2)

每次排序都分为n/2和n/2-1,则为最好情况，时间复杂度为O(nlog2n)

平均时间复杂度为O(nlog2n)，同时是不稳定的排序

同数量级排序算法中平均性能最好

空间复杂度，由于是递归调用，递归的本质是栈，所以是O(nlog2n)

#### 改进

较为简单的是数组长度的选择，一般将长度大于8的子递归调用快速排序，然后对整个基本有序序列用插入算法排序，可有效简单时间复杂度

```
static void kuaisu(int[] arr,int left,int right){
        if(left >= right ) return;
        int i = left, j = right;
        while(i < j){
            while(i < j && arr[j] >= arr[left]) j--;
            while(i < j && arr[i] <= arr[left]) i++;
            swap(arr,i,j);
        }
        swap(arr,i,left);
        kuaisu(arr,left,i -1);
        kuaisu(arr,i +1,right);
    }
```

### 直接插入排序

#### 原理

从前往后，逐渐往排好序的序列里添加数组元素

#### 复杂度分析

时间复杂度O(n2),空间复杂度O(1)

#### 改进

二分插入

```
static void zhijiesort(int[] arr){
        int insertNum;
        for(int i = 1;i<arr.length;i++){
            insertNum = arr[i];
            int j = i -1;
            while(j >= 0 && arr[j] > insertNum){
                arr[j+1] = arr[j];
                j--;
            }
            arr[j+1] = insertNum;
        }
    }
```

### 希尔排序

#### 原理

1. 选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
2. 按增量序列个数k，对序列进行k 趟排序；
3. 每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

#### 复杂度分析

时间复杂度O(n^(1.3-2))，空间复杂度O(n),因此对中等大小规模表现良好，但对规模非常大的数据排序不是最优选择，总之比一般 **O(n^2 )** 复杂度的算法快得多。

属于不稳定排序

#### 改进

目前还没有人给出选取最好的增量因子序列的方法



```
static void xier(int[] arr){
        int d = arr.length/2;
        while(d >= 1){
            xiersort(arr,d);
            d /= 2;
        }
    }
    static void xiersort(int[] arr,int d){
        for(int i = d;i<arr.length;i++){
            int insertNum = arr[i];
            int j;
            for( j = i -d;j >= 0 && arr[j] > insertNum;j -= d){
                arr[j+d] = arr[j];

            }
            arr[j + d] = insertNum;
        }
    }
```

### 桶排序

求指数，随后从低位到高位分桶排序，无太多讨论空间，可死记算法

```
private static void radixSort(int[] arr) {
        //待排序列最大值
        int max = arr[0];
        int exp;//指数

        //计算最大值
        for (int anArr : arr) {
            if (anArr > max) {
                max = anArr;
            }
        }

        //从个位开始，对数组进行排序
        for (exp = 1; max / exp > 0; exp *= 10) {
            //存储待排元素的临时数组
            int[] temp = new int[arr.length];
            //分桶个数
            int[] buckets = new int[10];

            //将数据出现的次数存储在buckets中
            for (int value : arr) {
                //(value / exp) % 10 :value的最底位(个位)
                buckets[(value / exp) % 10]++;
            }

            //更改buckets[i]，
            for (int i = 1; i < 10; i++) {
                buckets[i] += buckets[i - 1];
            }

            //将数据存储到临时数组temp中
            for (int i = arr.length - 1; i >= 0; i--) {
                temp[buckets[(arr[i] / exp) % 10] - 1] = arr[i];
                buckets[(arr[i] / exp) % 10]--;
            }

            //将有序元素temp赋给arr
            System.arraycopy(temp, 0, arr, 0, arr.length);
        }

    }
```

### 堆排序

#### 原理

利用堆结构建堆，然后是堆顶跟堆的最后一个元素交换位置

#### 复杂度分析

时间复杂度最好、最坏、平均时间复杂度都是O(nlog2n)，空间复杂度是O(1)

属于不稳定排序

#### 改进

无



```
 public static void sort(int[] a){
        for(int i = a.length/2 -1 ;i >= 0;i--){
            heapsort(a,i,a.length);
        }
        for(int i = a.length -1;i > 0;i--){
            int tmp = a[0];
            a[0] = a[i];
            a[i] = tmp;
            heapsort(a, 0,i);
        }

    }
    public static void heapsort(int[] arr,int i,int n){
        int maxIndex = i;
        int left = 2*i+1;
        int right = 2*i+2;
        if(left < n && arr[left] > arr[maxIndex]){
            maxIndex = left;
        }
        if(right < n && arr[right] > arr[maxIndex]){
            maxIndex = right;
        }
        if(maxIndex != i){
            int tmp = arr[maxIndex];
            arr[maxIndex] = arr[i];
            arr[i] = tmp;
            heapsort(arr,maxIndex,n);
        }


    }
```

### 归并排序

#### 原理

归并（Merge）排序法是将两个（或两个以上）有序表合并成一个新的有序表，即把待排序序列分为若干个子序列，每个子序列是有序的。然后再把有序子序列合并为整体有序序列。

#### 复杂度分析

时间复杂度O(nlog2n)，空间复杂度O(n),属于稳定排序

#### 改进

无

```
 public static void mergeSort(int[] arr,int left,int right){
        int mid = left + (right - left)/2;
        if(left < right){
            mergeSort(arr,left,mid);
            mergeSort(arr,mid + 1, right);
            merge(arr,left,mid,right);
        }
    }
    public static void merge(int[] arr,int left,int mid,int right){
        int[] tmp = new int[right - left  +1];
        int l = left,j = mid +1,r = right;
        int k = 0;
        while( l <= mid && j<=r){
            if(arr[l] < arr[j]){
                tmp[k++] = arr[l++];
            }
            else{
                tmp[k++] = arr[j++];

            }
        }
        while(l <= mid){
            tmp[k++] = arr[l++];
        }
        while(j <= r){
            tmp[k++] = arr[j++];
        }
        for(int i = 0;i<tmp.length;i++){
            arr[left + i] = tmp[i];
        }
    }
```

