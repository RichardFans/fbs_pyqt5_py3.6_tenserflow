# 用FBS 在UOS上 对pyqt5/tensorflow应用进行打包
### 创建开发环境

官网安装最新的miniconda，python可以选最新的3.9下载

```shell
# c
conda create -n py3.6 python=3.6
conda activate py3.6
conda install pip
```

```bash
pip install PyYaml PyQt5 qtawesome fbs rx tensorflow tensorflow_addons matplotlib opencv-python-headless qt_material
```

**注意**：运行若不是训练没有必要安装tensorflow-gpu！

python main/python/main.py报错找不到xcb库解决方案：

```
cd /usr/lib/x86_64-linux-gnu 
sudo ln -s libxcb-util.so.0  libxcb-util.so.1 
```



### 打包预处理：

```
fbs freeze
```



fbs freeze报错找不到 libpython3.6m.so.1.0解决方案

```shell
export LD_LIBRARY_PATH=/opt/sdk/python/miniconda3/envs/py3.6/lib/
```

或者vim /etc/ld.so.conf.d/miniconda3-py36.conf，写入：

```shell
/opt/sdk/python/miniconda3/envs/py3.6/lib
```

执行 

```bash
ldconfig
```



```bash
#/opt/sdk/python/miniconda3/envs/py3.6/lib/python3.6/site-packages/PyInstaller/hooks
cd /opt/sdk/python/miniconda3/envs/py3.6/lib/python3.6/site-packages/fbs/freeze/hooks
vim hook-tensorflow.py:
```

```python
from PyInstaller.utils.hooks import collect_all

def hook(hook_api):
    packages = [
        'tensorflow'
    ]
    for package in packages:
        datas, binaries, hiddenimports = collect_all(package)
        hook_api.add_datas(datas)
        hook_api.add_binaries(binaries)
        hook_api.add_imports(*hiddenimports)
```



### 执行下列命令进行打包

```
fbs installer
```

解决打包时报错：Your Linux distribution is not supported ...

```shell
vim /opt/sdk/python/miniconda3/envs/py3.6/lib/python3.6/site-packages//fbs_runtime/platform.py
```

添加uos支持，修改：

```python
def is_ubuntu():
    try:
        return linux_distribution() in ('Ubuntu', 'Linux Mint', 'uos') # 这里
    except FileNotFoundError:
        return False
```

解决打包时报错：FileNotFoundError: fbs could not find executable 'fpm'.

```bash
apt-get install ruby-dev build-essential && sudo gem i fpm -f
```



### 打包时裁剪掉重复或不必要的库

执行fbs installer之前，先删掉下面这些文件可以让最终打包出来的大小减少一半

```bash
cd /opt/project/python/hwkb/target/hwkb 
rm -r _pywrap_tensorflow_internal.so libtensorflow_framework.so.2 matplotlib scipy PIL tensorflow/python/tpu tensorflow/python/training tensorflow/python/_pywrap_tf_item.so tensorflow/include 

```

