#C语言模拟加载及获取可执行文件目录

```
#include<stdio.h>  
#include<stdlib.h>  
  
int main(int argc,char **argv)  
  
{  
      
    int i = 0;  
    char* chs[4] = {"|","/","—","\\"};  
  
    // 模拟加载效果  
    while(i<5){  
          
        int index = i%4;  
        printf("  loading, please wait...   %s\r",chs[index]);  
        fflush(stdout);  
        usleep(200000);//200毫秒  
  
        i++;  
    }  
    printf("loading, please wait...    %s\n","ok");  
    fflush(stdout);  
  
    //clrscr();  
    //system("clear");  
    //sleep(1);  
  
    fprintf(stdout,"Hello,Linux.\n");  
    fflush(stdout);  
  
    sleep(1);  
  
    // 可执行脚本目录  
    int res = system("$(dirname argv[0])/test");  
    printf("test return %d.\n",res);  
    exit(0);  
  
}  
```
