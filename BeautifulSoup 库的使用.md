## BeautifulSoup库的使用记录
### BeautifulSoup 有何用途
如果我们需要通过脚本来抓取网络中的数据时，使用传统的字符解析等方法时是非常低效的，而BeautifulSoup则可以方便的通过接口来获取标签中所想要得到的数据。主要用在解析静态页面的数据，如果设计到动态产生的内容，则还需要结合其他库模块来一起配合使用，如selenium模块等。

### 安装方法
```
pip install beautifulsoup4
```
详情可以见中文文档的地址：https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#id5

### 使用方法
在抓取网页数据时，一般和 requests 库一起使用，如下：
```
import requests
from bs4 import BeautifulSoup

url = 'http://tianqi.com/hangzhou/7'
HEADERS = {'User-Agent':'Mozilla/5.0 (Windows NT 6.2) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.12 Safari/535.11'}

req = requests.get(url, headers=HEADERS, timeout=5)
soup = BeautifulSoup(req.text, "html.parser")
```

假如在request的请求中出现以下连接问题并且你的网路处于代理模式时，则需要在get接口中将代理的IP和端口填写上
```
抛出异常：
Connection Aborted Error(10060 ' A connection attempt failed becvause the connected party did not properly respond after a period of time, or established a connection failed because connected host has failed to respond'

解决方法：
PROXY = {'http': 'http://xx.xxx.xx.xx:xxxx'}  替换为代理IP和端口
req = requests.get(url, headers=HEADERS, proxies=PROXY, timeout=5)
soup = BeautifulSoup(req.text, "html.parser")

timeout 参数是自己可以设定的连接超时时间，单位:秒
```

下文的小程序是用于获取一个城市7天的天气情况

爬取的网页地址是：http://tianqi.com/hangzhou/7

其中的 **hangzhou** 可以替换为其他任何城市

```
def get_soup_from_link(url):
    req = requests.get(url, headers=HEADERS, timeout=5)
    return BeautifulSoup(req.text, "html.parser")

def get_weather_report_list(weather_info):
    weather_report_list = []

    for group in weather_info:
        detail_info = {}
        txt = group.find_all(class_='txt')
        detail_info['date'] = group.find('dl').get_text()
        detail_info['description'] = group.find(class_='temp').get_text()
        detail_info['temperature'] = txt[0].get_text()
        detail_info['wind'] = txt[1].get_text()
        
        weather_report_list.append(detail_info)

    return weather_report_list

def print_day_weather(weather_list, city):
    print('City: ' + city)
    for weather in weather_list:
        print(weather['date'] + '\t' + weather['description'] + '\t' + weather['temperature'] + '\t' + weather['wind'])

def main():
    print(sys.argv)

    city = 'hangzhou'
    if len(sys.argv) == 1:
        city = 'hangzhou'
    elif len(sys.argv) == 2:
        city = sys.argv[1].lower()
    else:
        print('Usage: python Weather.py city_name\n city_name: hangzhou, shanghai, ...')
        return
    
    weather_url = 'http://tianqi.com/' + city + '/7'
    soup = get_soup_from_link(weather_url)
    weather_table_day7 = soup.find_all(class_='table_day7')
    weather_report_list = get_weather_report_list(weather_table_day7)
    print_day_weather(weather_report_list, city)

if __name__ == '__main__':
    main()
```

通过Chrome F12，我们可以看到我们需要解析的标签数据如下，他们都在 class="table_day7 tag" 或者 class="table_day7" 中

```
<dl class="table_day7 tbg">
<dl>01月26日</dl>
<dd class="week">今天</dd>
<dd class="air">
    <b style="background-color:#79b800;" title="空气质量：优">优</b>
</dd>
<dd class="img">
    <img src="http://pic9.tianqijun.com/static/tianqi2018/ico2/b15.png"/>
</dd>
<dd class="temp">雪</dd>
<dd class="txt">
    -1℃ ~ <b>1</b>
    ℃
</dd>
<dd class="txt">东北风 3级</dd>
</dl>
...
...
<dl class="table_day7 ">
<dl>02月01日</dl>
<dd class="week">星期四</dd>
<dd class="air">
    <b style="background-color:#79b800;" title="空气质量：优">优</b>
</dd>
<dd class="img">
    <img src="http://pic9.tianqijun.com/static/tianqi2018/ico2/b1.png"/>
</dd>
<dd class="temp">多云</dd>
<dd class="txt">
    -3℃ ~ <b>6</b>
    ℃
</dd>
<dd class="txt">东北风 1级</dd>
</dl>
```

因此，我们需要先找到这些标签块，通过find_all(class_='table_day7')方法，我们可以很快找到这些内容

```
weather_url = 'http://tianqi.com/' + city + '/7'
soup = get_soup_from_link(weather_url)
weather_table_day7 = soup.find_all(class_='table_day7')

find_all 返回的是一个列表，里面包含的是所有找到的匹配项。
入参使用带下划线的class，是因为class是python中的关键字，所以用class_来进行代替

下面就是我们暂时得到的数据结果：

[<dl class="table_day7 tbg">
<dl>01月26日</dl>
<dd class="week">今天</dd>
<dd class="air"><b style="background-color:#79b800;" title="空气质量：优">优</b></dd>
<dd class="img"><img src="http://pic9.tianqijun.com/static/tianqi2018/ico2/b15.png"/></dd>
<dd class="temp">雪</dd>
<dd class="txt">-1℃ ~ <b>1</b>℃</dd>
<dd class="txt">东北风 3级</dd>
</dl>, <dl class="table_day7 tbg">
<dl>01月27日</dl>
<dd class="week">明天</dd>
<dd class="air"><b style="background-color:#79b800;" title="空气质量：优">优</b></dd>
<dd class="img"><img src="http://pic9.tianqijun.com/static/tianqi2018/ico2/b15.png"/></dd>
<dd class="temp">雪</dd>
<dd class="txt">-2℃ ~ <b>0</b>℃</dd>
<dd class="txt">北风 2级</dd>
</dl>, <dl class="table_day7 tbg">
<dl>01月28日</dl>
<dd class="week">后天</dd>
<dd class="air"><b style="background-color:#79b800;" title="空气质量：优">优</b></dd>
<dd class="img"><img src="http://pic9.tianqijun.com/static/tianqi2018/ico2/b15.png"/></dd>
<dd class="temp">雪</dd>
<dd class="txt">0℃ ~ <b>2</b>℃</dd>
<dd class="txt">北风 3级</dd>
</dl>, <dl class="table_day7 ">
<dl>01月29日</dl>
<dd class="week">星期一</dd>
<dd class="air"><b style="background-color:#79b800;" title="空气质量：优">优</b></dd>
<dd class="img"><img src="http://pic9.tianqijun.com/static/tianqi2018/ico2/b1.png"/></dd>
<dd class="temp">多云</dd>
<dd class="txt">-2℃ ~ <b>3</b>℃</dd>
<dd class="txt">北风 3级</dd>
</dl>, <dl class="table_day7 ">
<dl>01月30日</dl>
<dd class="week">星期二</dd>
<dd class="air"><b style="background-color:#79b800;" title="空气质量：优">优</b></dd>
<dd class="img"><img src="http://pic9.tianqijun.com/static/tianqi2018/ico2/b0.png"/></dd>
<dd class="temp">晴</dd>
<dd class="txt">-2℃ ~ <b>4</b>℃</dd>
<dd class="txt">东北风 2级</dd>
</dl>, <dl class="table_day7 ">
<dl>01月31日</dl>
<dd class="week">星期三</dd>
<dd class="air"><b style="background-color:#79b800;" title="空气质量：优">优</b></dd>
<dd class="img"><img src="http://pic9.tianqijun.com/static/tianqi2018/ico2/b0.png"/></dd>
<dd class="temp">晴</dd>
<dd class="txt">-2℃ ~ <b>5</b>℃</dd>
<dd class="txt">北风 2级</dd>
</dl>, <dl class="table_day7 ">
<dl>02月01日</dl>
<dd class="week">星期四</dd>
<dd class="air"><b style="background-color:#79b800;" title="空气质量：优">优</b></dd>
<dd class="img"><img src="http://pic9.tianqijun.com/static/tianqi2018/ico2/b1.png"/></dd>
<dd class="temp">多云</dd>
<dd class="txt">-3℃ ~ <b>6</b>℃</dd>
<dd class="txt">东北风 1级</dd>
</dl>]
```

所有的数据现在已经全部拿到，现在就需要分解出各个小项中的内容，主要还是使用find方法来查找标签，而通过get_text()方法，我们就可以获取到具体的内容

```
def get_weather_report_list(weather_info):
    weather_report_list = []

    for group in weather_info:
        detail_info = {}
        txt = group.find_all(class_='txt')
        # txt 中包含了温度范围及风向内容
        
        detail_info['date'] = group.find('dl').get_text()
        # dl 标签中对应的是日期
        
        detail_info['description'] = group.find(class_='temp').get_text()
        # temp 对应了天气状态
        
        detail_info['temperature'] = txt[0].get_text()
        detail_info['wind'] = txt[1].get_text()
        
        weather_report_list.append(detail_info)

    return weather_report_list
```

爬取的数据，我们只用了BeautifulSoup中 find()、find_all()及get_text() 这几个方法，就取得了我们想要的文本内容，非常的方便。