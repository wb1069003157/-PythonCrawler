# -PythonCrawler
一个Python编写的静态抓取百度词条小爬虫

## 一个静态爬虫需要：
- urlManager  url管理器，负责管理待爬取的url和已爬取过的url
- html_downloader  html下载器，负责将传递过来的url的网页内容进行下载，交由解析器解析
- html_parser  html解析器，负责解析网页中的内容
- html_outputer  html输出器，负责将解析出来的内容进行处理，如写入文件中或直接输出等

## 抓取内容流程

```
    def __init__(self):
        self.urls=url_manager.UrlManager()
        self.downloader=html_downloader.HtmlDownloader()
        self.parser=html_parser.HtmlParser()
        self.outputer=html_outputer.HtmlOutputer()

    def craw(self, root_url):
        count=1
        self.urls.add_new_url(root_url)
        while self.urls.has_new_url():
            try:
                new_url=self.urls.get_new_url()
                print('craw %d : %s' % (count,new_url))
                html_cont=self.downloader.downloader(new_url)
                new_urls,new_data=self.parser.parse(new_url,html_cont)
                self.urls.add_new_urls(new_urls)
                self.outputer.collect_data(new_data)
                if count==10:
                    break
                count=count+1
            except:
                print('craw failed')

        self.outputer.output_html()
```

### url_manager

```
class UrlManager(object):


    def __init__(self) :
        self.new_urls=set()
        self.old_urls=set()

    # 添加新的url
    def add_new_url(self, url):
        if url is None:
            return
        if url not in self.new_urls and url not in self.old_urls:
            self.new_urls.add(url)

    # 批量添加新的url
    def add_new_urls(self, urls):
        if urls is None or len(urls)==0:
            return
        for url in urls:
            self.add_new_url(url)

    # 判断新url中是否还有
    def has_new_url(self):
        return len(self.new_urls)!=0

    # 取出一条新的url
    def get_new_url(self):
        new_url=self.new_urls.pop()
        self.old_urls.add(new_url)
        return new_url
```

### html_downloader

```
    def downloader(self, url):
        if url is None:
            return None
        response=request.urlopen(url)
        if response.getcode() !=200:
            return None
        return response.read()
```

### html_parser

```
  # 解析下载器中传来的内容
    def parse(self, page_url, html_cont):
        if page_url is None or html_cont is None:
            return
        soup=BeautifulSoup(html_cont,'html.parser',from_encoding='utf-8')
        new_urls=self._get_new_urls(page_url,soup)
        new_data=self._get_new_data(page_url,soup)
        return new_urls,new_data


    # 获取新的url列表
    def _get_new_urls(self, page_url, soup):
        # https://baike.baidu.com/item/Python
        new_urls=set()
        links=soup.find_all('a',href=re.compile(r'/item/*'))
        for link in links:
            new_url=link['href']
            new_full_url=urllib.parse.urljoin(page_url,new_url)
            new_urls.add(new_full_url)
        return new_urls

    # 获取内容
    def _get_new_data(self, page_url, soup):
        # <dd class="lemmaWgt-lemmaTitle-title"> <h1>Python</h1>
        res_data={}

        res_data['url']=page_url

        title_node=soup.find('dd',class_='lemmaWgt-lemmaTitle-title').find('h1')
        res_data['title']=title_node.get_text()

        # <div class="lemma-summary" label-module="lemmaSummary">
        summary_node=soup.find('div',class_='lemma-summary')
        res_data['summary']=summary_node.get_text()
        return res_data

```

### html_outputer

```
  def __init__(self):
        self.datas=[]


    def collect_data(self, data):
        if data is None:
            return None
        self.datas.append(data)
    
    # 将爬取的内容以HTML格式保存
    def output_html(self):
        fout=open('output.html','w',encoding='utf-8')
        fout.write('<html><meta http-equiv=\'content-type\' content=\'text/html;charset=utf-8\'>')
        fout.write('<body>')
        fout.write('<table border=\'1\'>')

        for data in self.datas:
            fout.write('<tr>')
            fout.write('<td>%s</td>'% (data['url']))
            fout.write('<td>%s</td>'% (data['title']))
            fout.write('<td>%s</td>'% (data['summary']))
            fout.write('</tr>')

        fout.write('</table>')
        fout.write('</body>')
        fout.write('</html>')
        fout.close()
```
