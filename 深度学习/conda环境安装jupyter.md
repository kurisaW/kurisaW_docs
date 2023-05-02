## 将虚拟环境添加到Jupyter Notebook

1.查看conda中的环境

首先查看已有的虚拟环境

```c
conda info --envs
```

2.激活待添加的环境

比如这里我想把torch0.4这个环境添加到jupyter notebook中的kernel环境里

```c
conda activate myenv
```

3.安装ipykernel

```c
conda install ipykernel
```

4.将待加入的环境添加到jupyter notebook

```c
python -m ipykernel install --name myenv
```

---

## Jupyter中删除虚拟环境步骤：

1. 查看安装了哪些虚拟环境kernel（在base或虚拟环境下运行都可以）：

```text
jupyter kernelspec list
```

2. 删除指定的kernel：

```text
jupyter kernelspec uninstall myenv
```
