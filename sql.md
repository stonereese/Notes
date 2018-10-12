# sql
1. 查询数据,在1位小数之后补零
    `convert(decimal(18,2),string)`
    在Excel中如何实现?
2. 查询表时,如果带统计,只能统计from后的表,对于left join的表貌似不起作用
    e.g.
    ```sql
    select left(dkbm,14)'村组代码',
           '邓州市'+fbf.fbfmc'村组名',
           bb.农户数,convert(decimal(18,2),
           sum(cbdelhtmjm)*0.0666667,2)'合同总面积',
           convert(decimal(18,2),sum(qrmjm)*0.0666667,2)'实测总面积',
           cc.家庭数,
           count(cbdkxx.dkbm)地块数 
    from cbdkxx  
    left join fbf 
        on left(cbdkxx.dkbm,14)=fbf.fbfbm 
    left join (select left(cbf.cbfbm,14)组编码,count(cbf.cbfbm)农户数 from cbf group by left(cbf.cbfbm,14))bb 
        on left(cbdkxx.cbfbm,14)=bb.组编码 
    left join (select left(cbf_jtcy.cbfbm,14)组编码,count(cbf_jtcy.cbfbm)家庭数 from cbf_jtcy group by left(cbf_jtcy.cbfbm,14))cc 
        on left(cbdkxx.cbfbm,14)=cc.组编码 
    group by left(cbdkxx.dkbm,14),
             fbf.fbfmc,bb.农户数,
             cc.家庭数 
    order by left(cbdkxx.dkbm,14)
    ```
3. notepad++打开总结的sql,后半部分不高亮,全部识别为注释了,是因为总结sql出问题的地方有反斜杠(图件目录),notepad++默认将反斜杠视为转义字符.具体设置:
    **设置--首选项--其他设置--视反斜杠为SQL转义字符**,去除此处的复选框即可.