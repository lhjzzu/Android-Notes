## SD卡

### 一 添加相应的权限

```java
<!-- 在SDCard中创建与删除文件权限 -->  
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>  
<!-- 往SDCard写入数据权限 -->  
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```



### 二 基本用法

#### 1 判断sd卡是否存在

```java
//SD是否存在
    private boolean ExistSDCard() {
        if (Environment.getExternalStorageState().equals(
                Environment.MEDIA_MOUNTED)) {
            return true;
            } else 
                return false;
    }
```

#### 2 sd卡剩余空间

```java
//SD剩余空间
    public long getSDFreeSize(){  
         //取得SD卡文件路径  
         File path = Environment.getExternalStorageDirectory();   
         StatFs sf = new StatFs(path.getPath());   
         //获取单个数据块的大小(Byte)  
         long blockSize = sf.getBlockSize();   
         //空闲的数据块的数量  
         long freeBlocks = sf.getAvailableBlocks();  
         //返回SD卡空闲大小  
         //return freeBlocks * blockSize;  //单位Byte  
         //return (freeBlocks * blockSize)/1024;   //单位KB  
         return (freeBlocks * blockSize)/1024 /1024; //单位MB  
    }

```



#### 3 sd卡总容量

```java
 //SD总容量
    public long getSDAllSize(){
        //取得SD卡文件路径 
        File path = Environment.getExternalStorageDirectory();
        StatFs sf = new StatFs(path.getPath());
        //获取单个数据块的大小(Byte)
        long blockSize = sf.getBlockSize();
        //获取所有数据块数 
        long allBlocks = sf.getBlockCount();
        //返回SD卡大小 
        //return allBlocks * blockSize; //单位Byte
        //return (allBlocks * blockSize)/1024; //单位KB
        return (allBlocks * blockSize)/1024/1024; //单位MB
    }

```





