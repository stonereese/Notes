# arcpy
Python 字符串不能以反斜线结尾，即使该字符串以 r 开头也是如此。您必须使用双反斜线。将动态文件名称附加到文件夹路径时，这一点很重要。
```python
import arcpy
mxd = arcpy.mapping.MapDocument(r"C:\Project\Project.mxd")
for df in arcpy.mapping.ListDataFrames(mxd):
    mxd.activeView = df.name
    mxd.title = df.name
    mxd.saveACopy(r"C:\Project\Output\\" + df.name + ".mxd")
del mxd
```

## 1. 字段计算器与更新游标
1. - 修改注记的字体为随机手写体时,游标可正常更新,但是字体并不会变化,修改为**字段计算器加代码块的方式,有效**
    - codeblock中,须在`def`前`import random`,不然在执行的时候会报错random未定义
    - 如果只用到random的随机整数功能,代码块里可以不使用def,直接`import random`,表达式里写`random.randint(1,5)`即可
    - 也可以不使用random,使用**arcgis.rand()**,此时不用定义代码块,请注意,此处是arcgis.rand,不是arcpy.示例:`arcgis.rand('Integer 1 100')`,这个式子用type()查看,是浮点型,结果带1位小数,但是计算到表中时,貌似是整数
    - random.randint(1,5),包括1,5,即包括起始点
    - `random.randrange(1,5)`,不包括终点,`random.randrange(1,5,2)`,此时step为2,可以理解为先从1开始(包括1)到5每次步长为2,得到一个奇数列表,然后在此奇数列表中进行随机选择,故而此式的结果总是奇数,`random.randrange(2,5,2)`,得到的总是偶数.取决于起始数的奇偶性.
2. 维护gdb数据库bsm时,两种方法都可以,但是在只作为arcgis工具箱后,只有游标更新法有效,字段计算器失效,是因为报错codeblock中global total 未定义
    参考上一条,是否可以将global total 定义在`def`之前解决(上述方式未在工具箱中验证)?有待考证.
## 2. arcGIS 自带工具
### 数据管理工具
1. GetCount
    getcount返回的结果为对象`result`,在程序中不能直接引用,也不能使用`int`进行转换.那如何在程序中调用这个计数呢?可使用`int(result.getOutput(0))`(`result.getOutput(0)`类型为`Unicode`,用int转换为数值)来进行转换.
    对于`result`对象,还可以使用`result.getMessages()`来显示工具执行结果.
2. 合并(类似:追加,都位于常规label下)
    - 相同之处
        合并和追加都是将两个同类要素合并在一起,如果两个要素有重叠的地方,那合并或追加后,还是有重叠,各要素还是各要素,并不是像编辑器里的合并那样,将要素合为1个.即:
    合并和追加,不管是否有重叠,只管将输入要素合在一个图层里.
    - 不同之处
        合并的输出数据是新建的,不影响合并前的各要素.追加是将个输入要素追加到另一个要素上,另一个要素与原来相比,变多了.
    - 与这两者类似但不同的另一个工具是联合工具(Union),位于**分析工具-叠加分析-联合**中
        - 如果输入的各要素没有重叠,那跟合并差不多,联合工具的输出结果也是新建的
        - 如果输入的各要素间有重叠,那么联合工具结果会在交集的地方,只生成1份,即结果中不会重叠.并且在交集的地方,要素是打断状态.
### 分析工具
1. 叠加分析中的**擦除**,提取分析中的**裁剪**,这两个类似于编辑器中的裁剪(丢弃相交区域,保留相交区域)
    - 擦除:丢弃相交区域(结果是擦除要素边界以外的部分被保留)
    - 裁剪:保留相交区域(结果是裁剪要素以内的部分被保留)
### 按属性选择,按位置选择
1. where clause 子句,前后各使用3个双引号,字段前后个用1个双引号,值使用单引号,`""""xjqydm"='{}'""".format(xjdm)`,注意是将3个双引号之间的内容当作字符串处理,format应在3个双引号之后,而不是'{}'之后.
## 3. arcpy多线程Pool
1. 出公示图片130张:
    - 正常代码情况下,耗时39分
    - 5进程下耗时19分,缩短1半时间
    - 10进程下,内存总使用量在70%-80%(本机总内存8G),貌似性能受到影响,预计比5进程不会快太多,甚至还要慢.**事实证明,比5进程慢.**
2. 多进程的坑:
     ```python
    def tojpg(file):
        for file in files:# 这是坑
            ...
    ```
    - 这种情况下使用多线程map(tojpg,files),未生成图片,内存爆炸,必须重启,杀进程不管用.
    - 总结:不要在子程序tojpg中使用遍历循环,只有单个file操作,并且多线程程序必须位于`if __name__=='__main__'`中才正常,如果不正常,先测试不使用多线程是否能正确输出结果.
## 4. 乱码问题
1. 避免代码中每处中文都要加u,在路径加r,路径中又有中文的情况,很麻烦,可以在开始前修改编码:
    ```python
    # coding:gbk
    import sys
    stdi,stdo,stde=sys.stdin,sys.stdout,sys.stderr
    reload(sys) # 注意输入输出是在reload前后,共4句,前1,后2
    sys.setdefaultencoding('gbk')
    sys.stdin,sys.stdout,sys.stderr=stdi,stdo,stde # 未经考证,去掉输入输出的两行,估计可以,这样4句变2句
    ```
## 5. 字段计算器问题
字段计算器中的代码块以及标注中的代码块真是个坑爹的地方,总是报错.
1. 在用交互式python窗口操作时,有时不确定字段的值到底里边有几个空格,又或者刚建的字段,显示为<空>,此时代码如何写?可以使用自带的选择属性工具,查看该字段唯一值来确定.其中,如果是刚建立的字段,值可能是NULL,如果在自带选择工具里查询时,由于是sql语法,故而应该使用`is null`(=null是错误的),在python中,没有null值,此处的null值,在python中应表示为None(none),故而是`if row[0] is none:`**此处以下更新,未经完全证实,初步证实是对的:**
    gdb中的空值默认是null,shp中默认为空(或是有1个空格?),gdb或mdb中的数值型字段没有精度设置(被忽略),例如面积字段.**更新权威总结**:
    - 对于 coverage、shapefile 和 dBase 表，如果字段类型定义为字符型，则会为每条记录插入空白行。如果字段类型定义为数值型，则将为每条记录插入零。

    - 所添加的字段会始终显示在表的末尾。

    - 字段长度仅适用于文本或 blob 类型的字段。

    - 对于地理数据库，如果字段类型定义为字符或数字，则在接受字段可为空参数默认值的情况下将 <null> 插入每条记录。
    - shapefile 不支持字段别名，所以无法将字段别名添加到 shapefile。
    - 在地理数据库要素类或表中创建新字段时，可指定字段的类型，但无法指定其精度和小数位数。即使对话框允许为精度或小数位数添加值，它在执行期间也会被忽略。

2. 注意,如果在字段计算器里设置的函数,其所需的参数字段正好是现在要更新的字段,会计算失败,报告参数错误(未确实验证,只是根据某次始终不能得出结果,但看着代码没问题而得出)
3. 代码块里可以使用import,也可以不使用def,只使用import,这样可以调用改模块的函数计算,而在独立脚本中,写字段计算器的代码块,比如尽管该脚本已经在脚本开头import了random,但在字段计算器需使用random模块时,应该在代码块里也要import random.在封装为arcgis工具箱时,global total报错,可以尝试将global total 写在代码块的def 之前试一试(未经验证)
## 6. pip
两个来源:
- https://stackoverflow.com/questions/26581838/installing-pip-using-arcgis-installed-python-2-7
    需要到https://pip.readthedocs.io/en/latest/installing/#installing-with-get-pip-py下载get-pip.py,下载链接在curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py中:
    I have a similar setup (Python installed through ArcGIS 10.2, but on machines running Windows 7 not 8.1). I used PIP to install another package (birdy instead of scrapy) and got it working. I think your problem may be trying to work from inside a Python interpreter instead of from the command line (oh, ye mighty Unix users with your ever-powerful command line). Here's what worked for me:

Go to http://pip.readthedocs.org/en/latest/installing.html
Download the get-pip.py file and place it in your python folder, e.g.: C:\python27\arcgis10.2\
Start a command prompt (Start Menu >> Accessories >> Command Prompt)
Change directories to python folder by entering: cd c:\python27\arcgis10.2
Install PIP by entering: python get-pip.py
Change directories into the scripts folder by entering: cd scripts
Use pip to install your package (e.g. scrapy) by entering: pip install scrapy
If this works, you should be able to go into Python now and import scrapy. This worked for me on every computer in my lab... just not on my own laptop... will be writing up my own question for that soon (arghh!).
- https://zhuanlan.zhihu.com/p/25809765:
    个人推荐使用pip，因为太好上手了:

python2.7以后的官方版本，都有自带的pip.exe文件的。

1) 安装pip


找到python的安装文件夹下的scripts文件夹，会看到有easy_install.exe和pip.exe的文件；此时，按shift+鼠标右键—选择在此处打开命令行，执行“easy_install.exe pip”，就可以安装成功。

可能出现的错误情况：

a) 没在该路径下安装；

b) 命令行无法调用python，回到开篇处，查看修改path的方法；

c) 文件夹里没有相应的exe文件，那么网上搜索get-pip.py；下载该文件，并run该文件。

2) pip的内置功能（pip list查看，重点已标红）

pip <command> [options]

Commands:

install Install packages.

download Download packages.

uninstall Uninstall packages.

freeze Output installed packages in requirements format.

list List installed packages.

show Show information about installed packages.

check Verify installed packages have compatible dependencies.

search Search PyPI for packages.

wheel Build wheels from your requirements.

hash Compute hashes of package archives.

completion A helper command used for command completion.

help Show help for commands.

General Options:

-h, --help Show help.

--isolated Run pip in an isolated mode, ignoring

environment variables and user configuration.

-v, --verbose Give more output. Option is additive, and can be

used up to 3 times.

-V, --version Show version and exit.

-q, --quiet Give less output. Option is additive, and can be

used up to 3 times (corresponding to WARNING,

ERROR, and CRITICAL logging levels).

--log <path> Path to a verbose appending log.

--proxy <proxy> Specify a proxy in the form

[user:passwd@]proxy.server:port.

--retries <retries> Maximum number of retries each connection should

attempt (default 5 times).

--timeout <sec> Set the socket timeout (default 15 seconds).

--exists-action <action> Default action when a path already exists:

(s)witch, (i)gnore, (w)ipe, (b)ackup, (a)bort.

--trusted-host <hostname> Mark this host as trusted, even though it does

not have valid or any HTTPS.

--cert <path> Path to alternate CA bundle.

--client-cert <path> Path to SSL client certificate, a single file

containing the private key and the certificate

in PEM format.

--cache-dir <dir> Store the cache data in <dir>.

--no-cache-dir Disable the cache.

--disable-pip-version-check

Don't periodically check PyPI to determine

whether a new version of pip is available for

download. Implied with --no-index.

3) pip的基本操作

实例1：pip install scipy

在安装过程中，我只要scipy出了问题，那么来讲一下如果解决：

a) 请务必按照先安装numpy、matplotlib、pandas、scipy的顺序安装，因为官方版安装包是有依赖的，后面的包可能会用到前面的包的部分功能。

b) 如果本机已经安装了Anaconda，那真是完美，因为scipy官方版就是建议用Anaconda来安装的。

c) 如果还不行，或者你跟我一样执着，非要用pip，那么请到这个网址重新下个scipy的“轮子”：

Python Extension Packages for Windows

为啥安装包叫做轮子呢，因为下载下来的安装文件是.wel格式的类似于压缩文件的东西，而这个文件的全拼是wheel，刚好是轮子的的英语单词，所以就会看到很多博主的分享里让你去下个轮子。

而pip install+<文件名.whl>是可以运行的。就可以安装好了。

此方法重点掌握，因为很多用pip install安装不了的包都可以在上面的网站里找到（网址收藏起来！！！），下载下来后，按照上述方法就可以安装了。

实例2:pip list


可以看到我已经把常用的包安装好了，看分享的小伙伴也请讲：numpy、matplotlib、pandas、scipy、scikit-learn的安装后，其他包，我们在实在项目中，用到了再安装。

whl文件下载站:https://www.lfd.uci.edu/~gohlke/pythonlibs/

### pip xlwings
import xlwings,报错不能import win32api(在shell中测试)
1. 尝试重启shell
2. 尝试pip install pywin32,之后在scripts目录下,pywin32_postinstall.py -install,尝试重启shell
3. pip install pypiwin32,重启shell
3. 

## 7. os.chdir()
在python2中，可以直接使用`os.chdir(r"d:\桌面")`来修改当前工作目录，但是如果该路径是个变量，比如`mypath=r"d:\桌面"`,或者是`mypath=raw_input("输入一个路径:\n")`,此时如果使用`os.chdir(mypath)`就会报错(python3无此问题),解决方法是`os.chdir(unicode(mypath,'utf8')`即可,此处的编码若为gbk,也报错.查资料unicode和str互转提到,unicode转str,使用`unicodestring.encode("utf-8")`,此处是encode,str转unicode是decode,写作`unicode(utf8string, "utf-8")`
    