---
title: python data analyst interview
date: 2018-10-24 21:22:02
tags:
---

data analyst (in investment)
===
## questions in python: find two errors in the code
``` python
id = 0 

def fun(perperty, params = []): # cannot pass params as a dynamic parameters here
    id += 1 # UnboundLocalError: local variable 'id' referenced before assignment
    if(perpertu % 10) != 0:
        params.append("vip")
    if(perpertu % 100 + perpertu % 10 )!= 0:
        params.append("accounter")
    if(perpertu % 1000 + perpertu % 100) != 0:
        params.append("ceo")
    return id, params

john = {'perperty': 11}
john['id'], john['role'] = fun(john['perperty'], 'employee')
# john = ['employee', 'vip', 'ceo']
print(john)
```

## write sql to find books
write with python and sql

1. 输出被借出超过五册的书的书名和借出册数
2. 输出当前日期和指定日期的书的剩余册数和押金总额