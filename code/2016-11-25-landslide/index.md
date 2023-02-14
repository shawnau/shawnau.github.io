# Landslide - Markdown to Slide

[Landslide](https://github.com/adamzap/landslide) 是个可以把markdown文件变成幻灯片直接在浏览器里播放的工具. 

<!--more-->

页面之间直接使用`---`间隔即可

#### 环境配置

```bash
pip install -U markdown
pip install -U docutils
pip install Jinja2
pip install Pygments

pip install -U landslide
```

#### Mathjax的使用
使用`-m`指令支持mathjax渲染, 可以直接使用`$$`和`$`作为line和inline渲染, 和本blog无缝衔接. (注: 似乎和markdown语法仍有冲突, 其中`\\`需要改成`\\\`, `_`需要改成`\_`, 未测试不改的效果)

```bash
landslide -m README.md
```

#### 使用
左右键控制播放即可

