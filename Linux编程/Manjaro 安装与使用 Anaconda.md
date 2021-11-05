# Manjaro 安装与使用 Anaconda

28 Feb 2019

## 安装

```
yaourt anaconda
source /opt/anaconda/bin/active root
```

### 添加环境变量

在 `~/.bashrc` 中添加

```
export PATH=/opt/anaconda/bin:$PATH
```

### 激活

```
source /opt/anaconda/bin/activate root
```

关于 `zsh`，打开 `~/.zshrc`：

```
vim ~/.zshrc
```

添加下面这条语句：

```
export PATH="/opt/anaconda/bin:$PATH"
```

添加完成并保存退出后，使环境变量生效：

```
source ~/.zshrc
```

此后在 `zsh` 中输入 `conda` 可直接运行。

### 配置

配置镜像：

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
```

## 常用命令

### 创建环境

创建名为 `py34` 的基于 `Python 3.4` 的 `conda` 环境：

```
conda create -n py34 python=3.4   
```

### 激活与退出环境

安装完成后：

```

 To activate this environment, use

     $ conda activate py34
     $ source activate network

 To deactivate an active environment, use

     $ conda deactivate
```

### 删除环境

```
conda remove -n py34 --all
```

### 查看已有环境

```
conda info -e
```

### 常用操作

```
# 查看当前环境下已安装的包
conda list

# 查看某个指定环境的已安装包
conda list -n python34

# 查找package信息
conda search numpy

# 安装package
conda install -n python34 numpy
# 如果不用-n指定环境名称，则被安装在当前活跃环境
# 也可以通过-c指定通过某个channel安装

# 更新package
conda update -n python34 numpy

# 删除package
conda remove -n python34 numpy

# 更新conda，保持conda最新
conda update conda

# 更新anaconda
conda update anaconda

# 更新python
conda update python
# 假设当前环境是python 3.4, conda会将python升级为3.4.x系列的当前最新版本
```

