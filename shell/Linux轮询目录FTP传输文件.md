# Linux轮询目录FTP传输文件

##情景
之前在公司，在linux服务器上需要写一个shell脚本，功能如下：定时任务5秒钟执行一次，轮询当前机器（127.0.0.1）A目录，并把A目录下所有`QRYTYP*`开头的文件传输到另外一台机器（10.32.64.128）的B目录下，文件名保持一样。

##要求
这样就要考虑几个问题：（假设现在有一个文件QRYTYP123456需要传输）
>1.QRYTYP123456达到A目录下，但文件过大，还在传输。而刚好被定时任务轮询到，这样B目录下的目标文件就会不完整。
>
>2.多任务传输大文件过程中，怎么保证FTP传输是成功的？

##解决思路
>1.使用lsof指令查看文件是否被其他进程占用
>
>2.文件在传输过程中，把文件重命名为:[filename].tmp，后续进程不会轮询后缀为`.tmp`的文件，一旦传输完成，则移到备份目录
>
>3.防止多个任务传输同一个文件，把每个定时任务轮询到的所有QRYTYP*文件名写到`file.lst.${PID_NUMBER}`的日志文件中，PID_NUMBER = [进程号][当前时间]，这样保证每个任务都处理轮询到的文件，不会造成并发传输。
>
>4.根据ftp的传输日志，可以根据日志“226 Transfer complete”（具体日志还需根据linux的版本来定），不过一般都是已"226"开头的，所以日志里出现"226 ......"，就表示该文件传输完成了。

##实际代码：
* `crontab_second_5.sh`这个文件的执行周期可通过crontab -e，添加一行:`* * * * * crontab_second_5.sh`保存退出即可。这里是每5秒轮询一次，可以修改`INTERVAL_SECOND`，缩短轮询时间，最好是能被60整除，这样会比较精确。

```bash
#!/bin/bash

#if the script startuped by crontab,
#it will ececute main_for_cronb_second_5.sh per 5 seconeds.
interval_second=5
let interval_count=60/interval_second
script_file=main_for_cronb_second_5.sh
for((i=1;i<=${interval_count};i++));do
  ${script_file} 2>/dev/null &
  sleep ${interval_second}
done
```

* `main_for_cronb_second_5.sh`需要注意`lsof`指令的路径，不同的操作系统会不一样

```bash
#!/bin/bash

#
#    file directory
#

#要轮询的目录
LOC_SND_DIR=/Users/lizebin/test_shell/fsend
#放日志
LOC_TMP_DIR=/Users/lizebin/test_shell/tmp
#远程目录
RMT_SND_DIR=/root/file
#远程临时目录
RMT_TMP_DIR=/root/tmp
#脚本路径
SCRIPT_DIR=/Users/lizebin/test_shell

CURRENT_DATE_TIME=$(date "+%Y%m%d%H%M%S")
PID_NUMBER=$$$CURRENT_DATE_TIME
TMP_FILE_SUFFIX=".tmp"
#文件匹配规则
FILE_NAME_FILTER_REGULATION="QRYTYP*"

cd $LOC_SND_DIR

rm -f $LOC_TMP_DIR/file.lst.$PID_NUMBER > /dev/null 2>&1
#search "QRYTYP*" file not directory and output filename
find . -maxdepth 1 -not -type d -type f -name 'QRYTYP*' -print | grep -v '.tmp'$ | while read LINE
    do
        lsof |grep $LINE |grep -v lsof|grep -v grep > /dev/null 2>&1
        if [ "$?" = "1" ]
    		then
            mv $LINE $LINE$TMP_FILE_SUFFIX
            echo $LINE >> $LOC_TMP_DIR/file.lst.$PID_NUMBER
            #每次只轮询一个文件
            break
        fi
    done
if [ ! -f "$LOC_TMP_DIR/file.lst.$PID_NUMBER" ]
then
    exit 1
fi
FILE_SIZE=`ls -l $LOC_TMP_DIR/file.lst.$PID_NUMBER | awk '{ print $5 }'`
if [ $FILE_SIZE -eq 0 ]
then
    rm -f $LOC_TMP_DIR/file.lst.$PID_NUMBER > /dev/null 2>&1
fi


#
#    get the file name and put
#
if [ "$FILE_SIZE" != "0" ]
then
    cat $LOC_TMP_DIR/file.lst.$PID_NUMBER | while read LINE
    do
        lsof |grep $LINE$TMP_FILE_SUFFIX |grep -v lsof|grep -v grep > /dev/null 2>&1
        if [ "$?" = "1" ]
        then
            $SCRIPT_DIR/file_ftp2Target_for_crontab_second_5.sh $LOC_SND_DIR $LOC_TMP_DIR $LINE $RMT_SND_DIR $RMT_TMP_DIR $PID_NUMBER $TMP_FILE_SUFFIX
        fi
    done
fi
if [ -f "$LOC_TMP_DIR/file.lst.$PID_NUMBER" ]
then
    rm $LOC_TMP_DIR/file.lst.$PID_NUMBER
fi
#****************************************************************
#
#        End of file
#
#****************************************************************
```
* `file_ftp2Target_for_crontab_second_5.sh`

```bash
#!/bin/bash
#远程FTP地址
FTP_ADDR="192.168.56.104"
#FTP用户
FTP_USER=root
#FTP密码
FTP_PWD=root

LOC_SND_DIR=$1
LOC_TMP_DIR=$2
FILE_NAME=$3
RMT_SND_DIR=$4
RMT_TMP_DIR=$5
PID_NUMBER=$6
TMP_FILE_SUFFIX=$7
FTP_SUCCESS_MSG="^226 "
FTP_INFORMATION_LOG=$LOC_TMP_DIR/"ftp_information_"$PID_NUMBER".log"
#
#    check the file
#
cd $LOC_SND_DIR

ls -l $FILE_NAME$TMP_FILE_SUFFIX >/dev/null 2>&1
if [ "$?" = "1" ]
then
    exit 1
fi


#
#    copy the orignial file to a new file
#    with ".tmp" in the name.
#
TMP_FILE_NAME=$FILE_NAME$TMP_FILE_SUFFIX

#
#    put the file
#
ftp -inv $FTP_ADDR <<! >> $FTP_INFORMATION_LOG
user $FTP_USER $FTP_PWD
bin
cd $RMT_TMP_DIR
put $TMP_FILE_NAME
ls -l
rename $RMT_TMP_DIR/$TMP_FILE_NAME $RMT_SND_DIR/$FILE_NAME
close
quit
!
if grep "$FTP_SUCCESS_MSG" $FTP_INFORMATION_LOG ;then
    #if ftp transfer success,it will remove temp file
    rm $TMP_FILE_NAME
else
    #if ftp transfer failure,it will rename temp file to file
    mv $TMP_FILE_NAME $FILE_NAME
fi

if [ -f $FTP_INFORMATION_LOG ]
then
    rm -rf $FTP_INFORMATION_LOG
fi
#****************************************************************
#
#        End of file
#
#****************************************************************
```
> <font style="color:red">特别提醒：
> 1.复制下面的代码到文本编辑器，换行符默认是"CRLF"的，但是linux是"LF"结尾，需要改过来的。
> 2.执行文件的过程中，看看*.sh有没有"x"权限，新手常常会犯这种错误啊！！！</font>

