# Jupyter Notebooks 简介


jupyter可以看作是一款强化版的python terminal, 实时运行命令脚本, 实时反映结果, 非常适用于科学计算和数据分析. 除此之外通过安装kernel也可以用其他语言工作

<!--more-->

### 安装(参考[quickstart](https://jupyter.readthedocs.io/en/latest/content-quickstart.html))

 - 装anaconda, 见[官方主页](https://www.continuum.io/downloads)
 - anaconda自带了jupyter, 使用`jupyter notebook`会在`http://localhost:8888`启动
 - 生成配置文件: `jupyter notebook --generate-config`, 配置文件会生成在`~/.jupyter`下

### 快捷键参考
1. 基本操作
 - `Enter`: 进入编辑模式, cell标签会变成绿色
 - `Esc`: 进入选择模式, cell标签会变成蓝色
 - `上下左右`: 在cell之间移动
 - `Shift-­Enter`: 运行cell, 并进入下一个cell

2. 编辑cell的操作
 - `X`: 剪切选择的cell
 - `C`: 复制选择的cell
 - `V`: 在下一行粘贴所在的cell
 - `A/B`: 在光标所在位置上面/下面插入cell
 - `D,D`:连续按两下D, 删除所在cell
 - `Shift-M`: 合并所在cell和下面的cell
 - `Ctrl-S­hif­t-S­ubtract`: 在光标所在位置分裂cell
