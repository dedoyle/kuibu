# Guide

## Commands

根目录下输入 hexo g 生成静态网页，然后输入 hexo s 可以本地预览效果，最后输入 hexo d 上传到 github 上

### new

```bash
hexo new [layout] <title>
```

layout 不传默认 `_config.yml` 的 `default_layout`。标题包含空格用引号括起来。

| 参数          | 描述                                          |
| :------------ | :-------------------------------------------- |
| `-p`          | 路径                                          |
| -r, --replace | 如果存在同名文章，将其替换                    |
| -s, --slug    | 文章的 Slug，作为新文章的文件名和发布后的 URL |

### generate

```bash
hexo g -d # 生成文件后立即部署网站
```

### deploy

```bash
hexo d -g # 生成部署
```

### server

```bash
hexo s -g # 生成预览
```

| 参数         | 描述                           |
| :----------- | :----------------------------- |
| -p, --port   | 端口                           |
| -s, --static | 只是用静态文件                 |
| -l, --log    | 启动日记记录，使用覆盖记录格式 |

### clean

```bash
hexo clean
```

清除缓存文件 (db.json) 和已生成的静态文件 (public)。
