```c
conda activate xxxx #开启xxxx环境
conda deactivate #关闭环境
​
conda env list #显示所有的虚拟环境
conda info --envs #显示所有的虚拟环境
conda info -e #显示所有已经创建的环境
​
conda update -n base conda #update最新版本的conda
​
conda create -n xxxx python=3.6 #创建python3.6的xxxx虚拟环境
​
conda remove --name xxxx  --all #彻底删除旧环境
​
conda remove -n tensorflow --all  #彻底删除旧环境
​
#Conda是没有重命名环境的功能, 要实现这个基本需求, 可以通过克隆-删除的过程。
#切记不要直接mv移动环境的文件夹来重命名, 会导致一系列无法想象的错误的发生!
conda create --name newname --clone oldname //克隆环境
conda remove --name oldname --all //彻底删除旧环境
​
conda list          #查看已经安装的文件包
conda list -n xxx   #指定查看xxx虚拟环境下安装的package
conda update xxx    #更新xxx文件包
conda uninstall xxx #卸载xxx文件包
​
#pip 安装本地包
pip install   ～/Downloads/a.whl
#conda 安装本地包
conda install --use-local  ~/Downloads/a.tar.bz2
​
​
#显示目前conda的数据源有哪些
conda config --show channels
​
#添加数据源：例如, 添加清华anaconda镜像：
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
​
#删除数据源
conda config --remove channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
​
# conda 升级默认包
conda update -n base -c defaults conda
```

