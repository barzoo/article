# Jupyter 入门

[Jupyter](http://jupyter.org/) 是一个非常流行的交互式笔记本。最近用这个笔记本来进行数据分析的工作，感觉对于Python比较熟悉的用户，的确是非常方便、强大的工具。由于 Jupyter 像是一个 web 服务器一样工作，所以需要进行一些额外的设置和服务器端软件包的安装操作。本文主要讨论这一部分的内容。


## 安装

### 准备工作

更新系统和Python相关的软件包，并安装`venv`- Python虚拟环境。这一部分如果比较熟悉，或者准备全局安装，请跳过。如果采用venv方式安装，全程需要都在虚拟环境下操作。

```
apt-get update
apt-get upgrade
apt-get install python3-venv
```

进入工作目录，比如我进入 `cd ~dev/python` 然后，在这里通过下面的命令建立虚拟环境系统。并开始升级这里的python的相关软件包：

```
python3 -m venv jupyter
```
执行后，会显示` (jupyter) xing@localhsot:`这样的提示符，表示在 Python 虚拟环境下操作。

建议升级 pip 安装工具，否则有时候会出现下载 python 模块正常，安装报错的情况。

```
python3 -m pip install --upgrade pip
```

### 安装主要程序

完成相关的操作后，就可以正式安装相关的软件包了：

 - jupyter
 - [jupyter_contrib_nbextensions](https://github.com/ipython-contrib/jupyter_contrib_nbextensions) 扩展模块，不是功能性的工具，不过可以让 Jupyter Notebook 使用起来更加的方便。
 - matplotlib 绘图工具包，这个就不用再说了
 - numpy  数字和效率的不二选择
 - [pandas](https://pandas.pydata.org/) 数据导入和处理工具，它依赖 numpy 来工作


```
python3 -m pip install jupyter
python3 -m pip install jupyter_contrib_nbextensions
python3 -m pip install matplotlib
python3 -m pip install numpy
python3 -m pip install pandas

# jupyter_contrib_nbextensions 相关设置安装
jupyter contrib nbextension install --user
```

其他可能要安装的包：

- sqlite3 和 sqlalchemy 进行数据库访问，我有一些数据是放在数据库里面的，所以需要这个包帮助Pandas读写数据库。
- [autopep8](https://github.com/hhatto/autopep8)  自动按照 [PEP8](https://www.python.org/dev/peps/pep-0008/) 要求格式化 Python 代码
- nbconvert  和 Pandoc 用来做文档格式的转换。请注意 Pandoc 是一个独立的工具，需要`sudo apt-get install pandoc ` 方式安装，而不是只是装对应的 Python 包。
- Basemap 是用来做地图绘制相关的操作。

### Basemap 安装

[Matplotlib Basemap](https://matplotlib.org/basemap/users/installing.html#installation) 需要手动安装，它所需的 [PROJ4](https://trac.osgeo.org/proj/) ，GEOS  需要在系统中安装。整个安装郭晨个相对来说还是有点复杂的，而且很多安装包在 Linux 上很方便安装，在 Windows 上没有进过测试。如果没有地图绘制的需要可以跳过这一部分。

```
# 如果没有编译环境的话，这里也是推荐安装一下的
sudo apt-get install build-essential
sudo apt-get install python3-dev

# PROJ4 
sudo apt-get install proj-bin  
python3 -m pip install pyproj

# GEOS
sudo apt-get install libgeos++    
```

下载basemap包，然后在目录中直接用`python3 setup.py install`。



## 配置启动运行

如果在本机运行，直接用下面命令启动，浏览器会自动打开访问相关的页面：

```
jupyter notebook
```

如果按照服务器来运行，最好用配置文件的方式来控制参数。可以通过下面的命令在用户目录下 产生一个默认的配置文件：`.jupyter/jupyter_notebook_config.py `，然后在其上进行修改。

```
jupyter notebook --generate-config
```

如果在服务器上运行，可以修改下面3项，以便于其他电脑的浏览器访问：

```
c.NotebookApp.ip = '*'
c.NotebookApp.open_browser = False
c.NotebookApp.notebook_dir = '<服务器上目录notebook' # 设置服务器工作目录
```

服务密码可以用下面的额方式生成：

```
jupyter notebook password
```

然后继续使用`jupyter notebook` 命令开始启动服务器，就可以打开网页进行访问了。



## 测试案例

这里举一个比较有复杂的案例，其他的案例可以再 Jupyter 网站上找到很多类似的例子。

第一步，初始化所有的相关包。如果安装了 `snippets`菜单，直接选择初始化下面的包，就会出自动贴现所示的代码：

```
from __future__ import print_function, division
import numpy as np
import pandas as pd
import matplotlib as mpl
import matplotlib.pyplot as plt
from mpl_toolkits.basemap import Basemap
%matplotlib inline
```

第二，我们假设一个全球的销售数据：

```
mapdata = np.array([
    {'city': 'Beijing', 'lon': 116.3882857, 'lat': 39.92889223, 'sales': 300.},
    {'city': 'Nurmberg', 'lon': 11.0799849, 'lat': 49.44999066, 'sales': 120.},
    {'city': 'Atlanta', 'lon': -84.39994938, 'lat': 33.83001385, 'sales': 220.},
    {'city': 'Sao Paulo', 'lon': -46.62502, 'lat': -23.55867959, 'sales': 170.},
    {'city': 'Sydney', 'lon': 151.1851798, 'lat': -33.92001097, 'sales': 50.},
])
mapdata = pd.DataFrame([pd.Series(x) for x in mapdata])
```

数据中有几个城市，他们的经纬度，销售数据。目标是在全球地图上绘制一个气泡图，以其大小显示各个城市销售数据。

第三部，绘图。

```
# 准备地图
fig, ax = plt.subplots(figsize=(10, 8))
map = Basemap(projection='cyl', ax=ax)

# 绘制地图
map.drawmapboundary(fill_color='aqua')
map.fillcontinents(color='coral', lake_color='aqua')
map.drawcoastlines()

# 绘制气泡
xs, ys = map(mapdata.lon.values, mapdata.lat.values)
map.scatter(xs, ys, mapdata['sales'] * 10, alpha=0.7, ax=ax, zorder=100)
```

效果就是在各个国家的首都城市显示一个蓝色的原型，其尺寸和对应的销售额成正比。

在操作中，可以感受到 Jupyter 最好的一点就是图文和代码的完全融合，很像是 Matlab，但输出和语言支持更加灵活。也和一般程序的编写完全不同，不用离开开发环境去查看运行结果。程序中的注释和解释性信息在输出后不会显示出来。像是本文，就是完全在 Jupyter 上写出来的。这样流畅的体验是其他工具很难带来的。