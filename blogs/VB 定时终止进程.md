#VB 定时终止进程

```
do   
    set bag=getobject("winmgmts:\\.\root\cimv2")   
    set pipe=bag.execquery("select * from win32_process where name='QQ.exe'")   
    for each i in pipe   
        i.terminate()   
    next   
        wscript.sleep 1000   
loop  
```

作者：罗新
链接：https://www.zhihu.com/question/21207025/answer/29648854
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
