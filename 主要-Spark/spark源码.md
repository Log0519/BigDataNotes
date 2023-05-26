# spark源码

### 切片大小

分区数=切片数，切片的计算规则如下

切片大小splitSize=computSplitSize(goalSize,minSize,blovkSize),默认为块大小，如果文件路径后面传了个期望切片参数6，相当于文件大小除6，然后取结果和块大小中取小的一个，然后再和min取大的一个，如果剩下的小于了1.1倍切片，那就分为一个切片
