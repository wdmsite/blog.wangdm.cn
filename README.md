# blog.wangdm.cn
博客

```shell
hugo new site blog.wangdm.cn
cd blog.wangdm.cn
git init
git submodule add https://github.com/wdmsite/LoveIt.git themes/LoveIt
echo "theme = 'LoveIt'" >> hugo.toml
hugo server -D
```