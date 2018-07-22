

序
--

**爬取目标**：微信公众号“纵梦广科”中“表白墙”（可选“吐槽墙”）的文章
**爬取字段**：表白对象、表白内容
**爬取缘由**：分析“表白墙”上的同学什么说得多的词
**爬取工具**：matplotlib、wordcloud、jieba
**爬取结果**: “表白墙”文章160篇
**爬取收获**：简单爬取公众号文章、简单生成词云
**爬取注意**：

 - 因为爬取内容可以简洁地直接保存txt文本进行绘画**词云**分析，所有并没有存入数据库
 - 本项目代码**不具有可复用性**，无论是登录的**cookie**还是文章的**页数**都需要重新手动获取输入
 - 代码中cookie的值太长了，都在一行不方便阅读，于是做了分行，可以根据个人喜好选择
 - 本代码通用于爬取公众号文章的**标题**和**url**，如需要爬取文章内容则需要手动更改爬取规则
 - **token**的值是爬取的公众号的标识符，如果更换公众号就需要更改该值
 - 本项目代码因为“表白墙”与“吐槽墙”网页结构相同，因此可以自行选择输入“表白墙”或“吐槽墙”进行爬取
 - 词云图在本文档后面
 - 获取cookie等操作步骤在本文最后


**ps:**
在参考文章中的例子是直接搜索公众号全部内容文章的，我测试过这样爬取全部的话只能爬几页就被提示”操作太频繁“而无法爬取，但换成关键字"query"搜索的话没有出现问题，目前本代码爬取”表白墙“32页并没有本禁止。本来还尝试如何避免封装爬取全部文章但没有成功，但如果关键字是空白符或者其他标点符号的话也能获取大部分文章

代码
--

```
# -*- coding: utf-8 -*-
import requests
from PIL import Image
from lxml import etree
import time
import random
import matplotlib.pyplot as plt
import numpy as np
from wordcloud import WordCloud
import jieba


# 使用Cookie，跳过登陆操作
headers = {
    "Cookie": "noticeLoginFlag=1; remember_acct=820605644%40qq.com; "
              "ua_id=F89e6CvMPIib8tkPAAAAAE8A9_O5KrS5oMM390XQRHI=; mm_lang=zh_CN; pgv_pvi=1996118016; "
              "noticeLoginFlag=1; remember_acct=820605644%40qq.com; pgv_si=s2063726592; ticket_id=gh_86437b3d3630; "
              "cert=3RRm40LWsECquCbg_jx5lQTMXRR4M0tN; rewardsn=; wxtokenkey=777; "
              "uuid=652947b257247d453cd64dc13a5daf0b; ticket=d19dbee738a3be7f0806c8a5f726b8d8cac125f6; "
              "data_bizuin=3555601673; bizuin=3551846274; "
              "data_ticket=eeN9lRUD61DWiiLZEJyFKGoi70SoJ2dB1BoNi4PnSvNaf6R3jA83ZYyEI1y3LaOU; "
              "slave_sid"
              "=elBZTHhvYlc0VmNnYTM0SnZ6Wl9DaGZTNWh0M0VZVHlxUDBfWHNUW"
              "jFVbEpOcFpmWEpuNUFXTEdGRWI5a3p6OGhrUWYweExnNjN2d0xMUWEwTVlLVWxIWk9mXzhzbkYxWndCQUVYTm"
              "l1UnVxYlNWbmR3Q09VT2pMbEFMZDNhOFhXTnRnMlpDbDhvYzZWN2hQ;"
              " slave_user=gh_86437b3d3630; xid=a5467f49610c64af7a7022c6a4596f40; "
              "openid2ticket_oCS3u05exHidsZqiS_3Q8Yn-YtYI=JjxfUwXvqw0VBHJhW5TvmrOn8W5QMp/ReaanapVptWI=",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/65.0.3325.162 Safari/537.36",
}


# 词云
def get_word(f):
    """
    创建词云图片，默认样式
    :param f:
    :return:
    """
    text_from_file_with_apath = open(f, 'rb').read().decode('utf-8')
    wordlist_after_jieba = jieba.cut(text_from_file_with_apath, cut_all=True)
    print(wordlist_after_jieba)
    wl_space_split = " ".join(wordlist_after_jieba)
    alice_mask = np.array(Image.open("girl.jpg"))  # 以数组的形式加载图画
    my_wordcloud = WordCloud(font_path="simhei.ttf",  # 设置字体
                             background_color="white",  # 背景颜色
                             max_words=2000,  # 词云显示的最大词数
                             mask=alice_mask,  # 设置背景图片
                             max_font_size=100,  # 字体最大值
                             random_state=42,
                             margin=2,  # 设置图片默认的大小,但是如果使用背景图片的话,那么保存的图片大小将会按照其大小保存,margin为词语边缘距离
                             ).generate(wl_space_split)
    plt.imshow(my_wordcloud)
    plt.axis("off")
    plt.show()


def get_info(url):
    """
    获取文章中的吐槽对象和吐槽内容
    :param url:
    :return:
    """
    res = requests.get(url, headers=headers)
    selector = etree.HTML(res.text)
    names = selector.xpath('// *[ @ id = "js_content"] // section / section[2] / section / span / span / text()')
    contents = selector.xpath('// *[ @ id = "js_content"] // section / section[2] / section / text()')
    with open('name.txt', 'ab+') as f:
        for s in names:
            f.write(s.strip().encode('utf-8'))

    with open('content.txt', 'ab+') as f:
        for s in contents:
            f.write(s.strip().encode('utf-8'))
    return 'content.txt', 'name.txt'


def get_list(url, input_name, post_num):
    """
    获取每页搜索结果的json中文章的标题和url
    :param url:
    :return:
    """
    for num in range(post_num):
        data = {
            "token": 1111467131,
            "lang": "zh_CN",
            "f": "json",
            "ajax": "1",
            "action": "list_ex",
            "begin": num * 5,
            "random": 0.040206335386987035,
            "count": "5",
            "query": input_name,
            "fakeid": "MzAwMzExNTQyNQ==",
            "type": "9",
        }
        # 使用get方法进行提交
        content_json = requests.get(url, headers=headers, params=data).json()
        # 返回了一个json，里面是每一页的数据
        for item in content_json["app_msg_list"]:
            # 提取每页文章的标题及对应的url
            print(item["title"], "url:", item["link"])
            f1, f2 = get_info(item["link"])
        time.sleep(random.randint(0, 30))
    return f1, f2


# 目标url
if __name__ == "__main__":
    input_name = "表白墙"  # 表白墙或吐槽墙任选其一
    url = "https://mp.weixin.qq.com/cgi-bin/appmsg"
    f1, f2 = get_list(url, input_name, post_num=32)
    get_word(f1)  # 创建词云
    get_word(f2)

```


图片
--

公众号截图：
 ![](https://img-blog.csdn.net/2018062516322643?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzIzNzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 ![](https://img-blog.csdn.net/20180625163237435?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzIzNzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
表白内容词云：
 ![](https://img-blog.csdn.net/20180625163247521?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzIzNzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
表白对象词云：
![](https://img-blog.csdn.net/20180625163256520?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzIzNzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 

操作步骤
----
1、拥有一个微信个人订阅号，附上登陆和注册链接。[微信公众平台](https://mp.weixin.qq.com/)

2、好在之前无聊注册过一次，所以就可以直接登陆操作。没有注册的童鞋可以用自己的微信号注册一下，过程十分简单，在此就不赘述了

3、登陆之后，点击左侧菜单栏“管理”-“素材管理”。再点击右边的“新建图文素材”

![](https://img-blog.csdn.net/20180626123134321?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzIzNzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

​弹出一个新的标签页，在上面的工具栏找到“超链接”并点击 
![](https://img-blog.csdn.net/20180626123141956?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzIzNzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

弹出了一个小窗口，选择“查找文章”，输入需要查找的公众号
![](https://img-blog.csdn.net/20180626123257615?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzIzNzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
点击之后，可以弹出该公众号的所有历史文章
![](https://img-blog.csdn.net/20180626123346447?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzIzNzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
关键字搜索、页数
![](https://img-blog.csdn.net/20180626123421257?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3MzIzNzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



本文部分参考该博友的文章：https://blog.csdn.net/wnma3mz/article/details/78570580
