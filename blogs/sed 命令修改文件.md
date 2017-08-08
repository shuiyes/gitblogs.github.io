# sed 命令修改文件

## sed替换当前目录中所有文件的某字符

```
sed -i 's/com.android.dialer.R/com.android.contacts.R/g' `grep com.android.dialer.R . -rl`
```


## sed 格式化代码-TAB替换为空格
```
sed -i 's/\t/    /g' `find . -name "*.java"`
```


## sed 格式化代码-删除代码中无效空格

```
sed -i 's/[ ]*$//g' `find . -name "*.java"` 
```
