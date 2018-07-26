# 文档搭建介绍

## 1. 启动

文档系统由docsify实现，可以直接启动：

先安装docsify：

```bash
sudo npm i docsify-cli -g
```

国内网速较慢， 可以使用cnpm

获取docs, 并启动:

```bash
git clone git@github.com:eostio/wiki.git
cd wiki/docs
docsify serve .
```

默认运行在 http://localhost:3000 上，直接可以浏览。

## 2. 添加新的文档

创建markdown文档之后，在侧边栏中添加链接即可。

侧边栏是默认加载 docs/_sidebar.md:

```markdown
- [首页](README.md)
- Examples
    - [markdown示例](examples/Example.md)
    - [常用数学公式示例](examples/Example_math.md)
```

可以添加子目录：

```
- Examples
    - [markdown示例](examples/Example.md)
```

## 3. 文档示例

- markdown语法参考： [markdown示例](examples/Example.md)
- docsify参考 : [docsify - 无需构建快速生成文档网站](https://segmentfault.com/a/1190000007656679)

## 4. 图片

文档中需要的图片放在assert中