<!--
    需要填充的占位符：
    
    README.md
    
        计算机中文电子书：Java：文档中文名
        {nameEn}：文档英文名
        {urlEn}：文档原始链接
        iteb-java：域名前缀
        飞龙：负责人名称
        wizardforcel：负责人 Github 用户名
        562826179：负责人 QQ
        it-ebooks-java-zh：ApacheCN 的 Github 仓库名称
        it-ebooks-java-zh：DockerHub 仓库名称
        it-ebooks-java-zh：PYPI 包名称
        it-ebooks-java-zh：NPM 包名称
    
    CNAME
    
        iteb-java：域名前缀

    index.html
    
        计算机中文电子书：Java：文档中文名
        #e51837：显示颜色
        it-ebooks-java-zh：ApacheCN 的 Github 仓库名称

    asset/docsify-flygon-footer.js
    
        it-ebooks-java-zh：ApacheCN 的 Github 仓库名称
-->

# 计算机中文电子书：Java

> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)
> 
> 真相一旦入眼，你就再也无法视而不见。——《黑客帝国》

* [在线阅读](https://iteb-java.flygon.net)

## 下载

### Docker

```
docker pull apachecn0/it-ebooks-java-zh
docker run -tid -p <port>:80 apachecn0/it-ebooks-java-zh
# 访问 http://localhost:{port} 查看文档
```

### NPM

```
npm install -g it-ebooks-java-zh
it-ebooks-java-zh <port>
# 访问 http://localhost:{port} 查看文档
```
