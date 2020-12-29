### not in 求差集

````sql
select
	t1.*
from t1
where
    t1.标识列 not in 
        (
            select
                t2.标识列
            from t2
        )
````

用not in的话大家也知道，效率感人

使用join吧