## 去除重复-set
1. python之使用set对列表去重，并保持列表原来顺序
    * 原始方法，但是会打乱顺序
        ```python
        mylist = [1,2,2,2,2,3,3,3,4,4,4,4]
        myset = set(mylist) #myset是另外一个列表，里面的内容是mylist里面的无重复 项
        ```
     

    * 收件人去重，并保持原来的收件人顺序
         ```pythohn
        mailto = ['cc', 'bbbb', 'afa', 'sss', 'bbbb', 'cc', 'shafa']
        addr_to = list(set(mailto))
        addr_to.sort(key = mailto.index)
        ```
   * 有两个文件，每个都有很多行ip地址，求出两个文件中相同的ip地址：(bisect用法)
        ```python
        # coding:utf-8
        import bisect

        with open('test1.txt', 'r') as f1:
            list1 = f1.readlines()
        for i in range(0, len(list1)):
            list1[i] = list1[i].strip('\n')
        with open('test2.txt', 'r') as f2:
            list2 = f2.readlines()
        for i in range(0, len(list2)):
            list2[i] = list2[i].strip('\n')

        list2.sort()
        length_2 = len(list2)
        same_data = []
        for i in list1:
            pos = bisect.bisect_left(list2, i)
            if pos < len(list2) and list2[pos] == i:
                same_data.append(i)
        same_data = list(set(same_data))
        print(same_data)
        ```

## 正则表达式
原文链接:[Python 正则表达式入门（初级篇）](http://www.cnblogs.com/chuxiuhong/p/5885073.html#4030813),[Python 正则表达式入门（中级篇）](http://www.cnblogs.com/chuxiuhong/p/5907484.html)
## 重命名(rename{s})
1. 看到别人分析`shutil.move()`的源代码,其主要思路是先复制src到dst,然后删除src,并不类似于Ctrl+X-Ctrl+V.看不大懂源码,但依据我的移动公示图的经验,应该不是这样的.当时公示图应该有1G多,如果是复制再删除,根本不会那么快完成.所以**我觉得这个方法还是可以用来移动文件的**.
2. 说重命名为什么说了移动文件呢?因为OS模块中除了rename方法外,还有一个renames方法,该方法不仅能重命名文件,还能重命名文件的上级\上上级...目录,这既可以用来重命名,还可以参考用来移动文件.
    - `os.rename()`:重命名文件名(file)
    - `os.renames()`:Super rename,在重命名文件的同时,还可以重命名目录(file and directory).example:
        ```python
        src=r"d:\桌面\新建文件夹\西门4组\1.txt"
        dst=r"d:\桌面\新建文件夹\西门5组\123.txt"
        os.renames(src,dst)
        ```
        - 上例中,不止重命名了file,还重命名了directory(西门5组).在新建文件夹下,可以没有西门5组这个文件夹(当然也可以本身就有),程序会自动创建.
        - 值得注意的是,**如果西门4组中只有1.txt这一个文件,那么上述程序执行后,西门4组这个文件夹没有了**.
        - 鉴于此,该命令在一定程度上实现了**剪切文件**的功能.
## python2.x与python3.x的区别:
1. 对汉字字符串切片:2.x中1个汉字占2个位置,3.x中,1个汉字占用1个位置\(参考十林公示图中导出图片.py,无法转义??\)
2. 注意map有**映射**的意思
3. **一个奇怪的现象**,dict:
    ```python
    s=['r','s','t']
    t=[1,2,3]
    z=zip(s,t)
    m=list(zip(s,t))
    dict(m)
    ```
   - 对于python3,上述第3行代码,z,返回值为zip迭代对象,python2返回的是元素为元组的列表`[('r', 1), ('s', 2), ('t', 3)]`
   - 第4行代码,python3:`[('r', 1), ('s', 2), ('t', 3)]`,python2:`[('r', 1), ('s', 2), ('t', 3)]`
   - 第5行代码,0python3:`{} \#空字典`,python2:`{'r': 1, 's': 2, 't': 3}`
   - 上步中,python3要想返回非空字典,需使用`dict([('r', 1), ('s', 2), ('t', 3)])`或者`dict(zip(s,t))`或者`dict(list(zip(s,t)))`
    综上,此处最好不要分开写,直接用`dict(zip(s,t))`,可以在python2和python3中都得到预期效果.
    **dict函数创建字典,类似于list函数创建列表,只要参数是可迭代对象即可,所以不难理解,`dict(list(zip(s,t)))`可以,`dict(zip(s,t))`也可以**
4. 如果键是字符串,可以这样创建字典:`d=dict(侯集镇='411324104')`,此时得到`{'侯集镇': '411324104'}`,创建时,侯集镇不能加引号,索引时必须加,`d['侯集镇']`,如果键是中文,此法不适用于python2


a bytes-like object is required, not 'str'
https://www.cnblogs.com/dpf-learn/p/8028121.html
## 查看网页编码方式的通用方法
在python爬虫等各种情景模式下，往往需要查看网页的编码方式。下面是通用，简单的方法。

在各种浏览器打开的任意页面上使用F12功能键，即可使用开发者工具，在窗口console标签下，键入 "document.charset" 即可查看网页的编码方式。

如，可以查看CSDN博客网站编码方式为 "UTF-8"
## 命令行下更新所有第三方库
1. 使用pip查看过期的库:`pip list --outdated`
2. 使用pip-review批量更新:`pip-review --local --interactive`(使用前需pip install pip-review,不必导入,是在命令行下执行,不是在python交互模式下执行)