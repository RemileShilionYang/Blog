---
title: NginX高性能服务器模型 
tags: NginX,C
---
基于Linux的posix进程编程，模仿NginX服务器模式

```javascript?linenums
#include<stdio.h>
#include<sys/types.h>
#include<unistd.h>
#include<signal.h>
#include<ctype.h>

#define MAX_CHILD_NUMBER 10     //最大子进程数
#define SLEEP_INTERVAL 4   //睡眠时间

int proc_number = 0;   
void do_something();	//进程做什么

int main(int argc,char* argv[]){
    int child_proc_number = MAX_CHILD_NUMBER;   
    int i,ch;
    pid_t child_pid;
    pid_t pid[10] = {0};    

    if(argc > 1){			//判断命令行输入是否有效
        child_proc_number = atoi(argv[1]); 
        child_proc_number = (child_proc_number > 10) ? 10 : child_proc_number;
    }

    for(i=0;i<child_proc_number;i++){
        child_pid = fork();
        proc_number = i;		//根据pid判断为父进程还是子进程
        if(child_pid == 0) do_something();
        if(child_pid != 0) pid[i] = child_pid;
    }

    while((ch=getchar())!='q'){
        if(isdigit(ch)){        				//选择性杀死进程
            kill(pid[ch-'0'],SIGTERM);
            pid[ch-'0'] = 0;
        }
    }

    for(i=0;i<child_proc_number;i++){
        if(pid[i] != 0){
            kill(pid[i],SIGTERM);
            pid[i] = 0;
        }
    }
    return 0;
}

void do_something(){
    while(1){
        printf("This is process No.%d and its pid is %d\n",proc_number,getpid());
        sleep(SLEEP_INTERVAL);		//让每一个执行的进程陷入死循环，模仿Nginx服务器
    }
}
```
