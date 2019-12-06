---
title: linux系统函数
p: os/linux/linux
date: 2019-12-06 20:34:27
categories:
    - os
tags:
    - linux
---

# Linux下系统函数

### rand()函数

获取0\~2147483647(0\~RAND_MAX)之间的随机数。真随机需要srand()设置种子。一般用时间作为srand()的参数

```C
#include<unistd.h>
int rand(void)
void srand(unsigned int seed)
```

### 字符函数

头文件<ctype.h>

函数名 | 功能
---|---
isalnum |测试字符是否为英文或数字
isalpha | 测试是否为英文字母
isascii | 测试是否为ascii字符
iscntrl | 测试是否为ascii控制字符
isdigit | 测试是否为阿拉伯数字
islower | 测试是否为小写字母
isprint | 测试是否可打印字符
isspace | 测试是否空格字符
ispunct | 测试是否为标点或特殊符号
isupper | 测试是否大写字母
isxdigit| 测试是否16进制数字

### 系统时间和日期函数

函数名| 功能
--- | ---
asctime| 将时间和日期以字符串的格式表示
ctime| 将时间和日期以字符串的格式表示
gettimeofday| 获得当前时间
gmtime| 把时间或日期转为GTM时间
localtime| 获得目前当地的时间和日期
mktime| 将时间结构数据转换为经过的秒数
settimeofday| 设置当前时间
time| 取得系统当前时间

```C
#include<time.h>
//成功返回秒数，并将结果存入t，失败返回-1。错误原因存在errno中
time_t time(time_t *t)

//传入时间戳，返回tm结构体
struct tm* gmtime(const time_t *timep)
/***
struct tm
{
    int tm_sec;//秒0~59
    int tm_min;//分0~59
    int tm_hour;//小时0~23
    int tm_mday;//当前月份的日数 0~31
    int tm_mon;//当前月份0~11
    int tm_year;//从1900年到现在的年数
    int tm_wday;//星期，星期一算起0~6
    int tm_yday;//从1月1日起到今天的天数0~365
    int tm_isdst;//夏时制时间
}
**/
//传入tm结构体时间，返回字符串时间
char * asctime(const struct tm *timeptr)
//传入时间戳，返回tm结构的时间
struct tm * localtime(const time_t *timep)

#include<sys/time.h>
#include<unistd.h>
//
int gettimeofday(struct timeval *tv,struct timezone *tz)
/****
struct timeval{
    long tv_sec;//秒
    long tv_usec;//微秒
}
struct timezone{
    int tz_minuteswest;//和GTM时间差了几分钟
    int tz_dsttime;//日光节约时间的状态
}
***/
```

#### 环境控制函数

函数名| 功能
--- | ---
getenv          | 取得环境变量的内容
pushenv/setenv  | 改变或增加环境变量
unsetenv        | 取消已经改变的环境变量

```C
#include<stdlib.h>
//成功返回环境变量内容，失败返回NULL
char *getenv(const char * name)
//成功返回0，出错返回-1。 overwrite  0：忽略value 1：改变成参数value的内容
int setenv(const char *name, const char *value, int overwrite)

```

#### 内存分配函数

函数名| 功能
--- | ---
calloc/malloc   | 分配存储空间
getpagesize     | 获取操作系统中内存分页大小
mmap            | 建立内存映射
munmap          | 解除内存映射
free            | 释放分配的内存

```C
#include<stdlib.h>
//分配内存size块大小为nmemb的内存。失败返回NULL。calloc会将内存初始化为0；
void * calloc(size_t nmemb, size_t size)
//分配size大小的内存，失败返回NULL
void * malloc(size_t size)
#include<unistd.h>
size_t getpagesize(void)
//成功返回映射区的起始地址，失败返回-1 （MAP_FAILED）
#include<unistd.h>
#include<sys/mman.h>
void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offsize)
```

<pre stype="color:#FFF">
mmap函数参数：
start：指向对应内存的起始地址。通常设为NULL
length：代表文件中多大的部分对应到内存
prot：映射区域的保护方式
    PROT_EXEC：映射区域可被执行
    PROT_READ：映射区域可被读
    PROT_WRITE：映射区域可被写
    PROT_NONE：映射区域不能存取
flags：映射区域的特性
    PROT_EXEC：映射区域可被执行
    PROT_READ：映射区域可被读
    PROT_WRITE：映射区域可被写
    PROT_NONE：映射区域不能存取
    MAP_FIXED：start所指向的地址无法成功建立映射时，则放弃映射，不对地址做修正。
    MAP_SHARED：映射区域的写入数据复制回文件，则允许其他映射该文件的进程共享
    MAP_PRIVATE：与shared相反，多个进程不共享。
fd：open()返回的文件描述符
offsize：文件映射偏移量，0代表从文件头开始
<b>注意：调用mmap时必须指定MAP_SHARED或MAP_PRIVATE</b>
</pre>

#### 数据结构中常用函数

函数名| 功能
--- | ---
bsearch | 二分法搜索
lfind/lsearch| 线性搜索
qsort| 快速排序排列数组

```C
#include<stdlib.h>
void qsort(void *base,size_t nmemb,size_t size, int (*compar)(const void *,const void *))

void *lfind(const void *key,const void *base,size_t *nmemb, size_t size,int (*compar)(const void *,const void *))

void *bsearch(const void *key,const void *base,size_t nmemb, size_t size,int (*compar)(const void *,const void *))
```

<pre stype="color:#FFF">
qsort()函数：
base：数组的起始地址
nmemb：数组元素的数量
size：每个元素的大小
compar：函数指针。数据相同返回0，返回1时两个数据交换，返回-1时两个数据不交换。

lfind()/lsearch()/bsearch()函数：
key：指向要查找的数据的指针
base：数组的起始地址
nmemb：数组元素的数量
size：每个元素的大小
compar：函数指针。
<b>lfind()和lsearch()不同点在于：当找不到数据时lfind()仅会返回NULL，而不会把数据加入到数组尾部。lsearch()则会在找不到数据时将其加入数组尾部</b>
</pre>

#### 文件系统API

##### chmod()函数

修改文件权限。成功返回0，失败返回-1。错误存在errno中

```c
#include<sys/types.h>
#include<sys/stat.h>
int chmod(const char* path,mode_t mode)
```

参数mode | 说明
--- | ---
S_IRUSR | 所有者具有读权限
S_IWUSR | 所有者具有写权限
S_IXUSR | 所有者具有执行权限
S_IRGRP | 组具有读
S_IWGRP | ...
S_IXGRP | ...
S_IROTH | 其他用户具有读权限
S_IWOTH | ...
S_IXOTH | ...

#### umask()函数
设置新文件建立时的权限掩码

```c
//返回原先的umask的值
mode_t umask(mode_t mask)
```

#### stat()函数

获取文件属性。成功返回0，失败返回-1，错误存入errno
```c
#include<sys/stat.h>
#include<unistd.h>
int stat(const char* file_name,struct stat *buf)

struct stat{
    dev_t       st_dev;     /* ID of device containing file -文件所在设备的ID*/  
    ino_t       st_ino;     /* inode number -inode节点号*/    
    mode_t      st_mode;    /* protection -保护模式?*/    
    nlink_t     st_nlink;   /* number of hard links -链向此文件的连接数(硬连接)*/    
    uid_t       st_uid;     /* user ID of owner -user id*/    
    gid_t       st_gid;     /* group ID of owner - group id*/    
    dev_t       st_rdev;    /* device ID (if special file) -设备号，针对设备文件*/    
    off_t       st_size;    /* total size, in bytes -文件大小，字节为单位*/    
    blksize_t   st_blksize; /* blocksize for filesystem I/O -系统块的大小*/    
    blkcnt_t    st_blocks;  /* number of blocks allocated -文件所占块数*/    
    time_t      st_atime;   /* time of last access -最近存取时间*/    
    time_t      st_mtime;   /* time of last modification -最近修改时间*/    
    time_t      st_ctime;   /* time of last status change - */
}
```

#### scanf系列

```C
#include<stdio.h>
int scanf(const char *format,...)
int fscanf(FILE *fp,const char *format,...)
int sscanf(const char *str,const char *format,...)
```

#### 不带缓存的文件IO

函数名| 功能
--- | ---
creat       | 创建文件
open        | 打开或创建文件
close       | 关闭文件
read        | 读文件
write       | 写文件
lseek       | 移动文件的读写位置
flock       | 锁定文件或解锁文件
fcntl       | 操作文件描述符

```c
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
//成功返回文件描述符，失败返回-1
int creat(const char *pathname,mode_t mode)
//成功返回最小的可用文件描述符，失败返回-1
int open(const char* pathname,int flags)
int open(const char* pathname,int flags, mode_t mode)
/***
flags：
    O_RDONLY：只读
    O_WRONLY：只写
    O_RDWR：读写
    O_APPEND：追加
    O_TRUNG：设置文件长度为0，且舍弃现存数据
    O_CREAT：没有则创建文件，文件权限为mode
    O_EXCL：与O_CREAT一起使用，若文件已经存在，则创建文件失败
    O_NONBLOCK：非阻塞式打开文件
***/
//成功返回0，失败返回-1
int close(int fd)

#include<unistd.h>
//成功返回实际读取或写入的数据，失败返回-1
ssize_t read(int fd,void *buf,size_t count)
ssize_t write(int fd,const void *buf,size_t count)

#include<sys/file.h>
//成功返回0，失败返回-1，错误存入errno
int flock(int fd,int operation)
/***
operation：
    LOCK_SH：建立共享锁，多个进程可同时对一个文件作共享锁定
    LOCK_EX：建立互斥锁。
    LOCK_UN：解除文件锁定
    LOCK_NB：无法建立锁定时，此操作可不被阻断，马上返回进程。与SH和EX与运算。
***/
#include<unistd.h>
#include<fcntl.h>
//成功返回0，失败返回-1，错误存入errno
int fcntl(int fd,int cmd)
int fcntl(int fd,int cmd, long arg)
int fcntl(int fd,int cmd, struct flock *lock)

struct flock{
    short l_type;
    short l_whence;
    off_t l_start;
    off_t l_len;
    pid_t l_pid;
}
l_type:
    F_RDLCK：读锁
    F_WRLCK：写锁
    F_UNLCK：解除锁
l_whence：SEEK_SET，SEEK_CUR，SEEK_END
l_start：同l_whence
l_len：加锁长度
l_pid：加锁的进程id
```

#### 带缓存的文件IO

函数名| 功能
--- | ---
fopen       | 打开或创建文件
fclose      | 关闭文件
fgetc       | 从文件读取一个字符
fputc       | 将一个字符写入文件
fgets       | 从文件读取一字符串
fputs       | 将字符串写入文件
fread       | 从文件流成块读取数据
fwrite      | 将数据成块写入文件
fseek       | 移动文件流读写位置
rewind      | 重置文件流的读写位置为文件开头
ftell       | 获取文件流的读取位置

```C
#include<stdio.h>
//成功返回指针，失败返回NULL
FILE * fopen(const char * path,const char* mode)
/***
mode：
    r：只读
    r+：打开可读写文件
    w：只写。文件存在则清空文件，不存在则创建文件
    w+：打开可读写文件
    a：追加方式打开。不存在则建立文件，存在则在文件尾部追加内容
    a+：打开可读写文件
    上面的选项中加入b，则表示用二进制方式打开文件，比如ab+
***/
//成功返回0，失败返回EOF
int fclose(FILE *stream)
//成功返回读取的字符，失败返回EOF
int fgetc(FILE * fp)
//成功返回写入的字符，失败返回EOF
int fputc(int ch,FILE *fp)
//成功返回s指针，失败返回EOF
char * fgets(char *s,int size,FILE *fp)
//成功返回写入的字符个数，失败返回EOF
int fputs(const char *str, FILE * fp)
//成功返回实际的读写nmemb数，失败返回EOF
size_t fwrite(const void * ptr,size_t size, size_t nmemb,FILE *fp)
size_t fread(void * ptr, size_t size,size_t nmemb,FILE * fp)
//成功返回0，失败返回-1
int fseek(FILE *fp, long offset, int whence)
//成功返回文件当前读写位置，失败返回-1
long ftell(FILE *fp)
//
void rewind(FILE *fp)
```

##### 特殊文件操作

###### 目录文件

```c
#include<dirent.h>
struct __dirstream
{
    void *__fd; /* `struct hurd_fd' pointer for descriptor.   */
    char *__data; /* Directory block.   */
    int __entry_data; /* Entry number `__data' corresponds to.   */
    char *__ptr; /* Current pointer into the block.   */
    int __entry_ptr; /* Entry number `__ptr' corresponds to.   */
    size_t __allocation; /* Space allocated for the block.   */
    size_t __size; /* Total valid data in the block.   */
    __libc_lock_define (, __lock) /* Mutex lock for this structure.   */
};
typedef struct __dirstream DIR;
struct dirent
{
   long d_ino; /* inode number 索引节点号 */
   off_t d_off; /* offset to this dirent 在目录文件中的偏移 */
   unsigned short d_reclen; /* length of this d_name 文件名长 */
   unsigned char d_type; /* the type of d_name 文件类型 */
   char d_name [NAME_MAX+1]; /* file name (null-terminated) 文件名，最长255字符 */
}
#include<sys/types.h>
#include<dirent.h>
//成功返回DIR指针，失败返回NULL
DIR *opendir(const char* name)
//成功返回0，失败返回-1，错误存入errno
int closedir(DIR *dir)

#include<sys/types.h>
#include<dirent.h>
#include<fcntl.h>
//成功返回下一个目录进入点，失败返回NULL
struct dirent * readdir(DIR *dir)
//返回目录流当前位置
long int telldir(DIR * dirp)
//loc可以是telldir的返回值。该函数设置目录流指针的位置
void seekdir(DIR *dirp,long int loc)

#include<sys/types.h>
#include<sys/stat.h>
//创建目录，同mkdir命令
int mkdir(const char *path, mode_t mode)
#include<unistd.h>
//删除目录，但是只有在目录为空时才行。同rmdir命令
int rmdir(const char *path)
//切换当前目录,同cd命令
int chdir(const char * path)
//获取当前目录，同pwd命令
char *getcwd(char *buf,size_t size)

```

###### 链接文件

```C
#include<unistd.h>
//建立软链接。成功返回0，失败返回-1，错误存入errno
int symlink(const char * oldpath,const char * newpath)
//建立硬链接。成功返回0，失败返回-1，错误存入errno
int link(const char * oldpath,const char * newpath)
//删除文件。成功返回0，失败返回-1；
int unlink(const char *path)
```

#### 编写守护进程

守护进程最重要的特性就是后台运行，其次是与运行的环境隔开，环境包括文件描述符，控制终端，会话工作目录等，这些环境通常是从执行他的父进程（特别是shell）中继承来的。

编写守护进程的要点：
1. 创建子进程，终止父进程。（使之成为僵尸进程）
2. 在子进程中创建新会话。这个步骤是创建守护进程最重要的一步。需要使用系统函数setsid()。这个函数有三个作用：1，让进程摆脱原会话的控制；2，让进程摆脱原进程组的控制；3，让进程摆脱原控制终端的控制。
3. 改变工作目录。可以调用chdir()函数将当前工作目录换为/或/tmp或其他路径
4. 重设文件创建掩码。
5. 关闭文件描述符。从父进程继承的已经打开的文件可能永远不会被守护进程使用，如果不关闭会导致资源浪费。可能导致所在文件系统无法卸载。

```C
#include<sys/types.h>
#include<unistd.h>
//设置新的组进程号，成功返回进程组号，失败返回-1，错误存入errno
pid_t setsid(void)

```


示例程序：

```C
#include<unistd.h>
#include<stdio.h>
#include<time.h>
#include<signal.h>
#include<sys/param.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<stdlib.h>


void init_deamon(void)
{
    pid_t child1,child2;
    int i;
    child1 = fork();//创建子进程，终止父进程
    if(child1 > 0){
        exit(0);
    }else if(child1 < 0){
        perror("创建进程失败!\n");
        exit(1);
    }
    setsid();//子进程中创建新会话
    chdir("/tmp");//改变工作目录
    umask(0);//重设umask
    for(i = 0; i < NOFILE;i++){
        close(i);//关闭文件描述符
    }
    return;
}
int main()
{
    FILE *fp;
    time_t t;
    init_deamon();
    while(1){
        sleep(10);
        if((fp = fopen("deamon.log","a+")) != NULL){
            t = time(0);
            fprintf(fp,"守护进程还在运行，时间是：%s\n",asctime(localtime(&t)));
            fclose(fp);
        }
    }
    return 0;
}
```
##### 日志相关函数

```c
#include<syslog.h>
//准备做信息记录
void openlog(char * ident, int option, int facility)
/***
option：
    LOG_CONS：如果无法将信息送到syslogd则直接输出到控制台
    LOG_PID：将信息字符串加上产生信息的进程号PID
    LOG_PERROR：同时输出到stderr
    LOG_NODELAY：立即打开连接
    LOG_NOWAIT：不要等待子进程
    LOG_ODELAY：延迟连接的打开，直到调用syslog()
facility：代表信息种类
    LOG_CRON：由cron或at程序产生的信息
    LOG_DEAMON：由系统deamon产生的信息
***/

//将信息记录到系统日志文件
void syslog(int priority, char * format, ...)
/***
priority：指定信息种类或等级
    LOG_INFO：提示相关信息
    LOG_DEBUG：出错相关信息
***/
```
<b>注意以上两个函数必须具有root权限才行</b>



### 进程间通信

#### 信号
使用kill -l可以列出系统支持的所有信号

1\~31以SIG开头(非实时信号)、34\~49以SIGRTMIN开头是继承Unix系统的信号称为不可靠信号(非实时信号)、50\~64以SIGRTMAX开头是对前面不可靠信号的更改和扩充，称为可靠信号(实时信号)

可靠信号：支持排队，用户发送一次就注册一次。<br />
不可靠信号：不支持排队，用户发送一次，先判断有没有相同的信号已经注册，如果有则不再注册。

常见信号及其默认操作：

信号名  | 含义  | 默认操作
--- |---|---
SIGHUP      | 该信号在用户终端连接(正常或非正常)结束时发出 | 终止
SIGINT      | 用户在按下中断键<ctrl+c>时，系统会向该终端相关进程发送此信号 | 终止
SIGQUIT     | 该信号在用户按下<ctrl+\>退出键时，系统会发送此信号，造成进程非正常终止 | 终止
SIGILL      | 该信号在一个进程企图执行一条非法指令时发出(可执行文件本身错误，或者企图执行数据段堆栈溢出时) | 终止
SIGFPE      | 该信号在发生致命算术运算错误时发出 | 终止
SIGKILL     | 用来立即结束程序的运行，且不能被阻塞，处理或忽略 | 终止
SIGALRM     | 该信号在定时器计时完成时发出，定时器可用进程调用alarm()函数来设置 | 终止
SIGSTOP     | 用于暂停进程，且不能能被阻塞，处理或忽略 | 暂停进程
SIGTSTP     | 用户按下<ctrl+z>挂起键时系统发送此信号，进程挂起 | 停止进程
SIGCHLD     | 该信号在子进程结束时向父进程发出。当子进程运行了exit()函数，就会向父进程发送此信号，如果父进程调用了wait()函数，则父进程被唤醒。如果父进程没有调用wait()函数，则忽略此信号，子进程将变成僵尸进程。 | 忽略

##### 信号操作相关函数

函数名 | 功能 
--- | ---
kill        | 发送SIGKILL信号给进程或进程组
raise       | 发送信号给进程或自身
alarm       | 定时器时间到，向进程发送SIGALRM信号
pause       | 没有捕捉到信号前一直将进程挂起
signal      | 捕捉信号SIGINT，SIG_IGN，SIG_DFL，SIGQUIT时执行信号处理函数
sigemptyset | 初始化信号集合为空
sigfillset  | 初始化信号集合为所有信号集合
sigaddset   | 将指定信号添加到指定集合
sigdelset   | 从集合删除指定信号
sigismember | 查询指定信号是否在信号集合中
sigprocmask | 判断监测或更改信号屏蔽字

```C
#include<sys/types.h>
#include<signal.h>
//发送信号给指定进程。成功返回0，失败返回-1
int kill(pid_t pid, int sig)
/***
pid：
    pid>0：将信号传递给pid的进程
    pid=0：将信号传递给当前进程相同进程组的所有进程
    pid=-1：将信号广播给系统内所有进程
    pid<-1：将信号传递给进程组识别码为pid绝对值的所有进程
sig：Linux系统信号
***/
//发送信号给当前进程。成功返回0，失败返回-1
int raise(int sig)
//设置信号处理函数
void (*signal(int signum,void(*handler)(int))) (int)

#include<signal.h>
//成功返回0，失败返回-1
int sigemptyset(sigset_t *set)
//成功返回0，失败返回-1
int sigaddset(sigset_t * set, int signum)
//查询或设置信号掩码。成功返回0，失败返回-1
int sigprocmask(int how,const sigset_t * set, sigset_t * oldset)
/***
how：
    SIG_BLOCK：新的掩码有当前信号掩码和参数set指定的信号掩码作联集
    SIG_UNBLOCK：将目前信号掩码删除掉参数set指定的信号掩码
    SIG_SETMASK：将目前掩码设置成set指定的掩码
***/
```

#### 管道

##### 无名管道

通过pipe()函数实现无名管道创建。无名管道通过int pipe_fd[2];来实现管道的两端，pipe_fd[0]是读端，pipe_fd[1]是写端。<br />
用在父子进程通信：
1. 父进程调用pipe()开辟管道
2. 父进程调用fork()创建子进程，那么子进程也继承得到了相同的管道
3. 父进程关闭管道读端，子进程关闭管道写端，从而就能将数据在父进程写入，在子进程读取。如果想实现双向，那就再使用一个管道。

```c
#include<unistd.h>
//建立无名管道，成功返回0，失败返回-1
int pipe(int filedes[2])
```
示例程序：

```C
......
#include<unistd.h>

int main()
{
    int fds[2];
    ......
    
    if(pipe(fds) < 0){
        perror("创建管道失败\n");
        exit(0);
    }
    ......
    chld = fork();
    if(chld == 0){
        close(fds[1]);
        read(fds[0],buf,100);
        ......
        close(fds[0]);
        ......
    }
    if(chld > 0){
        close(fds[0]);
        write(fds[1],buf,strlen(buf));
        ......
        close(fds[1]);
        waitpid(chld,NULL,0);
        ......
    }
}
```

##### 命名管道(FIFO)

该管道有名字，所以就能在任何有权限的进程间使用。

```C
#include<sys/types.h>
#include<sys/stat.h>
//建立命名管道。成功返回0，失败返回-1，错误存入errno
int mkfifo(const char* pathname, mode_t mode)
/***
pathname：指定管道文件位置，该文件必须不存在，否则创建失败。
mode：为创建的管道文件权限。跟umask会影响最终的值
***/

#include<stdio.h>
//建立管道IO。成功返回文件流指针，失败返回NULL，错误存入errno
FILE * popen(const char * command,const char * type)
/***
command：linux命令，例如ls -l等
type："r","w"分别表示读，写。

***/
```
<b>注意：编写具有SUID/SGID的程序时，避免使用popen()，因为popen()会继承环境变量，可能会造成安全问题</b>
<br />示例程序：
```C
......
#include<sys/types.h>
#include<sys/stat.h>
int main()
{
    int rfd,wfd;
    ......
    mkfifo("fifo1",S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH);
    wfd = open("fifo1",O_WRONLY);    
    ......
    write(wfd,buf,strlen(buf));
    ......
    close(wfd);
    ......
}

......
#include<stdio.h>
int main()
{
    FILE *fp;
    ......
    fp = popen("ls -l","r");
    ......
    fread(buf,sizeof(char),5000,fp);
    ......
    pclose(fp);
    ......
}
```

#### 消息队列

函数名 | 功能
--- | ---
ftok    | 由文件路径和工程ID生成标准key
msgget  | 创建或打开消息队列
msgsnd  | 添加消息
msgrcv  | 读取消息
msgctl  | 控制消息队列

```C
#include<sys/types.h>
#include<sys/ipc.h>
//成功返回key值，失败返回-1，错误存入errno
key_t ftok(char * pathname,char proj)

#include<sys/types.h>
#include<sys/ipc.h>
#include<sys.msg.h>
//成功返回消息队列识别号，失败返回-1，错误存入errno
int msgget(key_t key,int msgflg)
/***
key：ftok函数生成的key，如果传入IPC_PRIVATE则创建新的消息队列
msgflg：决定消息队列的存取权限
***/
//成功返回0，失败返回-1.错误存入errno
int msgsnd(int msgid,struct msgbuf * msgp ,int msgsz,int msgflg)
/***
msgid：消息队列识别号
msgp：用户自定义的结构体指针，形态如下
    struct msgbuf{
        long mtype；//消息类型，必须大于0
        char mtext[1];//消息文本
    }
msgsz：消息的大小
msgflg：用来指明核心程序在消息队列没有消息的情况下采取的行动
***/
//成功返回实际读取的消息数据长度，失败返回-1，错误存入errno
int msgrcv(int msgid,struct msgbuf * msgp,int msgsz,long msgtyp,int msgflg)
/***
msgtyp：用来指定所要读取的消息类型。
    =0：返回队列的第一项消息
    >0：返回队列中第一项msgtyp和mtype相同的消息
    <0：返回队列第一项mtype小于或等于msgtyp绝对值的消息
***/
//成功返回0，失败返回-1，错误存入errno
int msgctl(int msgid,int cmd,struct msqid_ds *buf)
/***
cmd：
    IPC_STAT：读取消息队列的数据结构msqid_ds，并将其存储在buf指定的地址中。
    IPC_SET：设置消息队列的数据结构msqid_ds中的ipc_perm元素的值。这个值取自buf参数。
    IPC_RMID：从系统内核中移走消息队列。

***/
```

示例程序：

```C
......
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/msg.h>

struct msgbuf{
    long mtype；//消息类型，必须大于0
    char mtext[512];//消息文本
}

int main()
{
    ......
    key_t key;
    int qid;
    struct msgbuf msg;
    
    key = ftok(".","a");
    ......
    qid = msgget(key,IPC_CREAT | 0666);
    ......
    msg.mtype = getpid();
    strcpy(msg.mtext,"hhhhhhhh");
    ......
    msgsnd(qid,&msg,strlen(msg.mtext),0);
    ......
    msgrcv(qid,&msg,512,0,0);
    ......
    msgctl(qid,IPC_RMID,NULL)
    ......
}
```

#### 共享内存

函数名 | 功能
--- | ---
mmap        | 建立共享内存映射
munmap      | 解除共享内存映射
shmget      | 获取共享内存区域ID
shmat       | 建立映射共享内存
shmdt       | 解除共享内存映射
```C
#include<unistd.h>
#include<sys/mman.h>
//建立内存映射.成功返回映射去的内存起始地址，失败返回-1，错误存入errno
void * mmap(void * start, size_t length, int prot, int flags, int fd, off_t offsize)
/***
prot：映射区域保护方式
    PROT_EXEC：映射区域可被执行
    PROT_READ：映射区域可被读
    PROT_WRITE：映射区域可被写
    PROT_NONE：映射区域不能存取
flags：特性
    PROT_EXEC：映射区域可被执行
    PROT_READ：映射区域可被读
    PROT_WRITE：映射区域可被写
    PROT_NONE：映射区域不能存取
    MAP_FIXED：start所指向的地址无法成功建立映射时，则放弃映射，不对地址做修正。
    MAP_SHARED：映射区域的写入数据复制回文件，则允许其他映射该文件的进程共享
    MAP_PRIVATE：与shared相反，多个进程不共享。
***/
//成功返回0，失败返回-1，错误存入errno
int munmap(void *start, size_t length)
/****
length：欲取消内存的大小
***/
```

示例程序：

```C
......
#include<unistd.h>
#include<sys/mman.h>

struct people{
  char name[4];
  int age;
};

int main()
{
    pid_t chld;
    people * p_map;
    ......
    
    p_map = (people *)mmap(NULL,sizeof(people)*10,PROT_READ | PROT_WRITE|MAP_SHARED | MAP_ANONYMOUS,-1,0);
    ......
    chld = fork();
    ......
    if(chld == 0){
        for(int i = 0;i < 5;i++){
            printf("第%d个人，姓名:%s,年龄:%d\n",(p_map+i)->name,(p_map+i)->age);
        }
        munmap(p_map,sizeof(people)*10);
        ......
    }
    if(chld > 0){
        for(int i = 0;i < 5;i++){
            memcpy((p_map+i)->name,'a'+i,2);
            (p_map+i)->age = i;
        }
        ......
        munmap(p_map,sizeof(people)*10);
    }
}
```

###### UNIX Systeem V共享内存

```C
#include<sys/ipc.h>
#include<sys/shm.h>
//成功返回共享内存识别号，失败返回-1，错误存入errno
int shmget(key_t key, int size, int shmflg)
/***
key：为IPC_PRIVATE则建立新的共享内存，大小有四则决定
***/
//映射共享内存，成功返回内存的起始地址，失败返回-1，错误存入errno
void * shmat(int shmid, const void * shmaddr, int shmflg)
/***
shmid：共享内存识别号
shmaddr：
    =0：内核自动选择地址
    !=0且shmflg没有指定SHM_RND旗标，则以参数shmaddr为连接地址。
    !=0且指定SHM_RND旗标：参数shmaddr会自动调整为SHMLBA的整数倍
***/
//成功返回0，失败返回-1，错误存入errno
int shmdt(const void * shmaddr)mmap
```
mmap和unix system v两类共享内存的区别在于mmap用的普通文件实现共享内存，而unix system v则用的特殊文件系统shm中的文件实现共享内存。
