## 1.如何解压.zip.001 .zip.002 .zip.003文件

由于某些数据集过大，一般上传者会将一个zip文件进行分割成.zip.001 .zip.002...等多个zip文件，并且不能直接使用unzip命令解压

解决办法：将多个zip文件合并成一个zip文件

```c
cat dataset.zip* > dataset.zip
unzip dataset.zip
```

## 2.如何解压xxx.part1.rar文件

对任何一个分卷执行解压命令即可：

```c
rar e test.part1.rar
```

## 3.解压xxx.tar.gz文件

```c
tar zxvf xxx.tar.gz
```

