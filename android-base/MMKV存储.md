
* 说MMKV前，先了解下mmap内存映射，[mmap是啥](https://www.cnblogs.com/huxiao-tee/p/4660352.html) 
* [MMKV文档](https://github.com/Tencent/MMKV/blob/master/README_CN.md)

1. 常规文件读写操作，因为用户进程无法在内核空间中直接寻址，需要两次拷贝，磁盘 (拷贝)-> 内核空间（内存） (拷贝)-> 用户空间（内存）。  
2. mmap操作文件，直接将磁盘文件映射到用户进程的地址空间，只进行一次拷贝。  
省去了一部拷贝操作，效率得到提升。  
3. mmap和read、write等一样，都属于Linux的系统调用（system call）

* shared preference，sqlite，datastore

