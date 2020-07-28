# 2020 Web Mining 期末项目
|项目名称|中国特色联合国发展：联合国发展目标与习近平新时代中国特色社会主义思想之主题模型可视化|
| ---------- | --- |
|小组成员| 周昱瑾 卢佳燕 孙思盼|
|任务分配-周昱瑾|①习近平新时代中国特色社会主义思想三十讲②生态文明建设
# 一、项目简介
## 1.1 背景
- 联合国发展目标：

联合国所有会员国于2015年通过的《 2030年可持续发展议程》为当今和未来的人类与地球的和平与繁荣提供了共同的蓝图。它的核心是17个可持续发展目标（SDG），这是所有国家（无论是发达国家还是发展中国家）在全球伙伴关系中迫切需要采取的行动。他们认识到，消除贫困和其他贫困现象必须与改善健康和教育，减少不平等并刺激经济增长的战略紧密结合，同时应对气候变化并努力保护我们的海洋和森林。

- 习近平新时代中国特色社会主义思想：

习近平新时代中国特色社会主义思想，从理论和实践结合上系统回答了新时代坚持和发展什么样的中国特色社会主义、怎样坚持和发展中国特色社会主义这一重大时代课题，是马克思主义中国化最新成果，是当代中国马克思主义、21世纪马克思主义，是党和国家必须长期坚持的指导思想。深入学习贯彻习近平新时代中国特色社会主义思想，必须深刻认识领会这一思想的时代意义、理论意义、实践意义、世界意义。习近平新时代中国特色社会主义思想顺应和平、发展、合作、共赢的时代潮流，推动构建新型国际关系和人类命运共同体，为世界和平与发展作出重大贡献。

## 1.2 相关来源
- [习近平新时代中国特色社会主义思想三十讲](http://www.qstheory.cn/xjpsxkj/index.html)
- [生态文明建设](http://theory.people.com.cn/GB/68294/417224/index.html)


# 二、代码步骤分步详析
## 《习近平新时代中国特色社会主义思想三十讲》爬取代码
    
    import requests

    from bs4 import BeautifulSoup



#### 自定义函数获得html文本

    ### 请求url
    def getHtmlText(url):
    r = requests.get(url)
    #统一编码
    r.encoding = r.apparent_encoding
    return r.text

    rooturl = "http://gzw.fujian.gov.cn/ztzl/xxxcgcddsjdjs/xxzl_12064/"
    html = getHtmlText(rooturl)
    soup = BeautifulSoup( html , "html.parser")
    print(soup)



#### 根url
     rooturl = "http://gzw.fujian.gov.cn/ztzl/xxxcgcddsjdjs/xxzl_12064/"
#### 获取html文本
     html = getHtmlText(rooturl)
#### 解析文本
     soup = BeautifulSoup( html , "html.parser")
#### 获得包含二级url的标签
     uls = soup.find_all(name= "div", attrs= {"class" : "newslist" })
     # print((type(uls[0])))
     # print(uls)
#### 存储新的url
     urls = [] 
    titles = [] # 标题
    for child in uls:
        hrefs = child.find_all(name= 'a')
    # print(hrefs) # 观察超链接
    # print(len(hrefs)) # 寻找判定条件
#### 防止存在空标签
    if len(hrefs) > 1:
        for a in hrefs:
            # print(type(a))
            # print(a.attrs)
            #apppend新url
            seq = (rooturl, a.attrs["href"])
            urls.append('/'.join(seq))
            titles.append(a.attrs["title"])

            print(urls)
            print(titles)



#### 新一页
    tempurl = 'http://gzw.fujian.gov.cn/was5/web/search?channelid=277442&sortfield=-DOCORDERPRI%2C-DOCRELTIME&classsql=chnlid%3D12064&prepage=20&page=2'
#### 请求url
    r = requests.get(tempurl)
    r.encoding = r.apparent_encoding
    #输出json格式
    html = r.json()
    # print(html)
    docs = html['docs']
    for i in range(0, 16):
        urls.append(docs[i]['url'])
        titles.append(docs[i]["title"])
    print(urls)
    print(titles)



#### 预设文章，时间，来源变量
    articles = []
    times = []
    sources = []
#### 遍历二级url
    for url in urls:
        html = getHtmlText(url)
        # print(html)
        #解析
        soup = BeautifulSoup( html , "html.parser")
        time = soup.find(name='meta', attrs= {"name" : "PubDate"})
        # print(time['content'])
        times.append(time['content'])
        source = soup.find(name='meta', attrs= {"name" : "ContentSource"})
        sources.append(source['content'])
        # contents = soup.find_all(name= "p",attrs= {"align" : "justify" })
        contents = soup.find_all(name= "p")
        # print(type(contents))
        str = "文章："
#### 有些标签不包含文章
        for content in contents:
            if content.string is not None :
                str = str + content.string
        articles.append(str)


#### print(len(articles))
#### print(articles[2])
#### 输出到excel
    output = open('data.xls','w',encoding='gb18030')
    output.write('链接')
    output.write('\t')
    output.write('时间')
    output.write('\t')
    output.write('来源')
    output.write('\t')
    output.write('标题')
    output.write('\t')
    output.write('文章')
    output.write('\n')
    for i in range(len(urls)):
        # print(i)
        output.write(urls[i])
        output.write('\t')
        output.write(times[i])
        output.write('\t')
        output.write(sources[i])
        output.write('\t')
        # gbk_str = titles[i].encode('gbk').decode('gbk')
        output.write(titles[i])
        output.write('\t')
        # gbk_str = articles[i].encode('utf-8').decode('utf-8')
        output.write(articles[i])
        output.write('\n')
    output.close()
    print("ok")


#### 输出到csv
    import  pandas as pd
    name = ['链接', '时间', '来源', '标题', '文章']
    # data = [urls, times, sources, titles, articles]
    data = []
    for i in range(0,len(urls)):
        temp = []
        temp.append(urls)
        temp.append(times)
        temp.append(sources)
        temp.append(titles)
        temp.append(articles)
        data.append(temp)

    csv_data = pd.DataFrame(columns=name, data=data)
    csv_data.to_csv('csv_data.csv')
    print("csv ok")
    
## 生态文明建设代码

    import  requests
    from bs4 import BeautifulSoup


#### 自定义函数
    def getHtmlText(url):
        r = requests.get(url)
        r.encoding = r.apparent_encoding
        return r.text

    rooturl = "http://theory.people.com.cn/GB/68294/417224/index.html?tdsourcetag=s_pctim_aiomsg"
    html = getHtmlText(rooturl)
    soup = BeautifulSoup( html , "html.parser")
    # print(soup)


#### 获得urls
    urls = []
    ps = soup.find_all(name= 'p', attrs= {'class' : 'tr'})
    print(type(ps))
    for item in ps:
        a = item.find('a')
        urls.append(a.attrs['href'])
    print(urls)


#### 获得标题
    introductions = []
    fs = soup.find_all(name= 'font', attrs= {'size' : 4})
    # print(fs)
    for item in fs:
        # print(item.string)
        introductions.append(item.string)
#### 去除最后一个的换行
    introductions[6] = introductions[6][0:2] + introductions[6][3:]
    print(introductions[6])


#### 获得来源
    sources = []
    divs = soup.find_all(name= 'div', attrs= {'style' : 'text-align: right; font-size: 12px;'})
    # print(divs)
    for item in divs:
        sources.append(item.string[3:])
    print(sources[6])


#### 获得介绍2
    introductions2 = []
    divs = soup.find_all(name= 'div', attrs= {'style' : 'font-size: 12px;' })
    # print(divs)
    for item in divs:
        introductions2.append(item.string[2:-2])
    print(introductions2)


#### 输出到excel
    output = open('total.xls','w',encoding='gb18030')
    output.write('链接')
    output.write('\t')
    output.write('来源')
    output.write('\t')
    output.write('标题')
    output.write('\t')
    output.write('简介')
    output.write('\n')
    for i in range(len(urls)):
        # print(i)
        output.write(urls[i])
        output.write('\t')
        output.write(sources[i])
        output.write('\t')
        output.write(introductions[i])
        output.write('\t')
        output.write(introductions2[i])
        output.write('\t')
        output.write('\n')
    output.close()
    print("ok")

#### 第二层
    for i in range(len(urls)):
        html = getHtmlText(urls[i])
        soup = BeautifulSoup( html , "html.parser")
        # print(soup)
        ps = soup.find_all(name= 'p', attrs= {'style' : 'text-indent: 2em;'})
        # print(ps[56])
#### 预设一些变量
        abstractString = ''
        Second_source_list = []
        Second_href_list = []
        Second_content = []
        no_href = '无引用链接'
        is_content_time = True # 哨兵
#### 遍历
    for item in ps:
        # print(item)
        abstract = item.find(name= 'span', attrs= {'style' : 'font-family: 楷体; text-indent: 2em; display: block;'})
        Second_source =  item.find(name= 'span', attrs= {'style' : 'color: rgb(255, 0, 0); text-indent: 2em; display: block;'})
        hrefs = item.find(name= 'a')
        # print(item.string)
        if abstract is not None:
            # print(abstract.string)
            abstractString = abstractString + abstract.string
        elif Second_source is not None: #来源，每组content都有
            # print(Second_source.string)
            if is_content_time is False:
                is_content_time = True
                Second_source_list.append(Second_source.string[2:])
        elif hrefs is not None:
            # print(hrefs.attrs['href'])
            if is_content_time is True:
                is_content_time = False
                Second_href_list.append(hrefs.attrs['href'])
                Second_content.append(hrefs.string)
            else:
                Second_source_list.append("同下")
                Second_href_list.append(hrefs.attrs['href'])
                Second_content.append(hrefs.string)
        else:
            # print(item.string)
            if is_content_time is True:
                is_content_time =False
                Second_href_list.append(no_href)
                Second_content.append(item.string)
            else:
                Second_source_list.append("同下")
                Second_href_list.append(no_href)
                Second_content.append(item.string)
#### 输出到excel
    filename = introductions[i] + '.xls'
    output = open( filename,'w',encoding='gb18030')
    output.write(abstractString)
    output.write('\n')
    output.write('段落内容')
    output.write('\t')
    output.write('来源')
    output.write('\t')
    output.write('链接')
    output.write('\n')
    for i in range(len(Second_content)):
        # print(i)
        t = str(Second_content[i])
        t = ''.join(t.split())
        output.write(t)
        output.write('\t')
        output.write(Second_source_list[i])
        output.write('\t')
        output.write(Second_href_list[i])
        output.write('\t')
        output.write('\n')
    output.close()
    print(filename + "ok")
