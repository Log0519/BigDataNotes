# Java源码阅读之Array.sort()
@[toc]
阅读起点：

    Arrays.sort(nums1);

 `使用ctrl+左键进入sort()方法`

### 1.Arrays.sort()

关于`sort()`的方法一共有`14`个，就目前调用的来看是以下这种最基础的。

     public static void sort(int[] a) {
        DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
    }

### 2.DualPivotQuicksort

`DualPivotQuicksort`即双轴快排，定义了`七种`原始类型的排序方法。DualPivotQuicksort中使用了`private DualPivotQuicksort() {}`，防止实例化，实现了sort方法并且定义了以下调整参数:

    //归并排序的最大运行次数
    private static final int MAX_RUN_COUNT = 67;
    //归并排序的最大运行长度
    private static final int MAX_RUN_LENGTH = 33;
    //如果要排序的数组的长度小于该常数，则优先使用快速排序而不是归并排序
    private static final int QUICKSORT_THRESHOLD = 286;
    //如果要排序的数组的长度小于此常数，则优先使用插入排序而不是快速排序
    private static final int INSERTION_SORT_THRESHOLD = 47;
    //如果要排序的字节数组的长度大于该常数，则优先使用计数排序而不是插入排序
    private static final int COUNTING_SORT_THRESHOLD_FOR_BYTE = 29;
    //如果要排序的 short 或 char 数组的长度大于此常数，则优先使用计数排序而不是快速排序
    private static final int COUNTING_SORT_THRESHOLD_FOR_SHORT_OR_CHAR = 3200;

### 3.DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);

该方法定义：

    static void sort(int[] a, int left, int right,int[] work, int workBase, int workLen) {}

进入DualPivotQuicksort的`sort`方法：

     static void sort(int[] a, int left, int right,
                     int[] work, int workBase, int workLen) {
        // Use Quicksort on small arrays
        if (right - left < QUICKSORT_THRESHOLD) {
            sort(a, left, right, true);
            return;
        }

首先进行了判断，如果要排序的数组`小于`了之前定义的`QUICKSORT_THRESHOLD=286`，则优先使用`快速排序`而不是`归并排序`，即进入if中的排序`sort(a, left, right, true)`;

### 4.DualPivotQuicksort.sort(a, left, right, true)

该方法定义：

    private static void sort(int[] a, int left, int right, boolean leftmost){}

进入if中的`sort(a, left, right, true)`方法，我们只截取他的逻辑部分而非排序实现部分。

    private static void sort(int[] a, int left, int right, boolean leftmost) {
        int length = right - left + 1;
    
        // Use insertion sort on tiny arrays
         if (leftmost) {
                /*
                 * Traditional (without sentinel) insertion sort,
                 * optimized for server VM, is used in case of
                 * the leftmost part.
                 */
                for (int i = left, j = i; i < right; j = ++i) {
                    int ai = a[i + 1];
                    while (ai < a[j]) {
                        a[j + 1] = a[j];
                        if (j-- == left) {
                            break;
                        }
                    }
                    a[j + 1] = ai;
                }
            } else {...........
            ........

该方法中，首先判断了数组长度是否`小于INSERTION_SORT_THRESHOLD=47`，如果小于就使用`插入排序`，而不是`快速排序`。`leftmost`是来选择使用传统的（无标记）插入排序还是成对插入排序，leftmost是表示此部分是否在`范围内的最左侧`，因为我们最先开始调用的就是基础的sort，没有其他参数，所以就是`从头开始排序`，leftmost便默认为`true`，使用传统（无标记）插入排序，如果为false，使用成对插入排序。

### 5.总结

如果使用最基础的`Arrays.sort()`，那么排序中会根据数组的长度进行判断，数组越短，`length<47`,优先选择插入排序，其次`length<286`,选择快排，其次是归并排序。
