---
layout:     post
title:      SQL Server分页语句
subtitle:   查询耗时最短的分页语句
date:       2018-05-01
author:     xujie
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - SQL server
---

> 正所谓前人栽树，后人乘凉。
> 感谢[Huxpro](https://github.com/huxpro)提供的博客模板,同时也感谢BY!
> 
> [我的的博客](http://my.happy-coding.cn)

```
select top 10 A.orderid,A.pay,A.pay_date,A.pay_time,A.machineId,A.sysflag
from (
    select ROW_NUMBER()over(order by orderid desc) as rownumber,*
    from (select * from XC_month )as xc
    )as A
where rownumber>1*10
and A.orderid like '%Fk%'
```

后台只需要传当前页和每页的显示数量
