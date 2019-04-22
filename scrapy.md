# scrapy框架
* Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。 用途非常广泛。
* 用户只需要定制编写几个模块就可以轻松的实现一个爬虫，用来抓取网页内容。
* Scrapy 使用了 Twisted异步网络库来处理网络通讯。
## Scrapy框架图
![](https://raw.githubusercontent.com/Yuzzzzzzz/imag/master/scrapy1/1.png)
* `Scrapy Engine(引擎)`：负责Spider、ItemPipeline、Downloader、Scheduler中间的通讯、信号和数据传递等。

* `Scheduler(调度器)`：复制下载Engine发送过来的Request请求，并按照一定方式进行整理排列，入队，当Engine需要时交给它。

* `Downloader(下载器)`：负责下载Engine发送过来的Request请求，并将下载下来的Responses交还给Engine，然后由Engine交给Spider处理。

* `Spider(爬虫)`：负责从Responses中分析提取数据，获取Item字段需要的数据并返还给Engine，也可以从中提取URL给Engine继续交给scheduler。

* `Item Pipeline(管道)`：负责处理Spider中获取到的Item，并进行后期处理。

* `Downloader Middlewares(下载器中间件)`：位于Scrapy引擎和下载器之间的框架，主要是处理Scrapy引擎与下载器之间的请求及响应。

* `Spider Middlewares(爬虫中间件)`：介于Scrapy引擎和爬虫之间的框架，主要工作是处理蜘蛛的响应输入和请求输出。

* `Scheduler Middewares(调度中间件)`：介于Scrapy引擎和调度之间的中间件，从Scrapy引擎发送到调度的请求和响应。

## Scrapy运作流程
* `Spider`将起始的URL交给`Engine`，`Engine`将URL交给`Scheduler`。
* `Scheduler`把从`Engine`得到的URL入队处理，然后取出队头的URL交给`Engine`。
* `Engine`把URL封装成一个请求(Request)传给`Downloader`。
* `Downloader`把资源下载下来，并封装成应答包(Response)通过`Engine`交给`Spider`处理。
* `Spider`解析Response，如果解析出`Item`,则通过`Engine`交给`Item Pipeline`进行进一步的处理,如果解析为URL则重复上述处理。
* `Item Pipeline`得到`Item`后进行后期处理。
## 安装
```
    1、安装wheel
        pip install wheel
    2、安装lxml
        https://pypi.python.org/pypi/lxml/4.1.0
    3、安装pyopenssl
        https://pypi.python.org/pypi/pyOpenSSL/17.5.0
    4、安装Twisted
        https://www.lfd.uci.edu/~gohlke/pythonlibs/
    5、安装pywin32
        https://sourceforge.net/projects/pywin32/files/
    6、安装scrapy
        pip install scrapy
```
![](https://raw.githubusercontent.com/Yuzzzzzzz/imag/master/scrapy1/2.png)
命令行里输入 scrapy 测试是否安装成功
![](https://raw.githubusercontent.com/Yuzzzzzzz/imag/master/scrapy1/3.png)
* scrapy bench ：测试scrapy效率
* scrapy fetch : 获取一个URL
* scrapy genspider : 创建一个爬虫
* scrapy runspider : 运行一个爬虫
* scrapy shell : 获得一个URL的response
* startproject : 创建一个爬虫
## scrapy实例
1.创建工程
```
scrapy startproject douban
```
![](https://raw.githubusercontent.com/Yuzzzzzzz/imag/master/scrapy1/4.png)

2.创建爬虫
```
cd douban
scrapy genspider Douban douban.com
```
![](https://raw.githubusercontent.com/Yuzzzzzzz/imag/master/scrapy1/5.png)

3.查看文件目录
```
tree /f
```
![](https://raw.githubusercontent.com/Yuzzzzzzz/imag/master/scrapy1/6.png)

4.编辑items.py
```python
import scrapy


class DoubanItem(scrapy.Item):
    # define the fields for your item here like:
    #评论人名字
    name = scrapy.Field()
    #评论内容
    content = scrapy.Field()
```

5.编辑Douban.py
```python
import scrapy
from douban.items import DoubanItem


class DoubanSpider(scrapy.Spider):
    name = 'Douban'
#    allowed_domains = ['www.douban.com']
    start_urls = ['https://movie.douban.com/subject/27622447/comments?start='+str(x)+'&limit=20&sort=new_score&status=P' 
                  for x in range(0,201,20)]

    def parse(self, response):
        node_list = response.xpath("//div[@class='comment']")
        
        for node in node_list:
            item = DoubanItem()
            
            item['name'] = node.xpath("./h3/span[@class='comment-info']/a/text()").extract()[0]
            item['content'] = node.xpath("./p/span/text()").extract()[0]

            yield item
```

6.编写pipelines.py
```python
class DoubanPipeline(object):
    def __init__(self):
        self.f = open('comment.txt', 'w')
    
    def process_item(self, item, spider):
        comment = item['name'] + '\n' + item['content'] + '\n\n' 
        self.f.write(comment)
        return item
    
    def close_spider(self,spider):
        self.f.close()
```

7.settings.py添加
```python
FEED_EXPORT_ENCODING = 'utf-8'

USER_AGENT = 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36'

ITEM_PIPELINES = {
    'douban.pipelines.DoubanPipeline': 300,
}
```

8.运行爬虫
创建一个存放数据的data文件夹，进入该文件夹执行爬虫
```
cd /data
scrapy crawl Douban
```
![](https://raw.githubusercontent.com/Yuzzzzzzz/imag/master/scrapy1/7.png)

9.查看结果
![](https://raw.githubusercontent.com/Yuzzzzzzz/imag/master/scrapy1/8.png)
可以看见，爬虫已经把要爬取的内容存在我们指定的文件了。
