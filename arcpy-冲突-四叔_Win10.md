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

## 字段计算器与更新游标
1. 修改注记的字体为随机手写体时,游标可正常更新,但是字体并不会变化,修改为字段计算器加代码块的方式,有效
    codeblock中,须在`def`前`import random`,不然在执行的时候会报错random未定义
2. 维护gdb数据库bsm时,两种方法都可以,但是在只作为arcgis工具箱后,只有游标更新法有效,字段计算器失效,是因为报错codeblock中global total 未定义
    参考上一条,是否可以将global total 定义在`def`之前解决(上述方式未在工具箱中验证)?有待考证.