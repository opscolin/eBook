<pre align="middle">
Linux Shell customed common functions
- by Colin at 2016-12-15
</pre>

---
Table of Contents
=================

 * [摘要](#摘要)
 * [Print log with date](#print-log-with-date)
 * [Get the first and last day of last Month](#get-the-first-and-last-day-of-last-month)
 * [Add certain day on one date](#add-certain-day-on-one-date)
 * [Send mails](#send-mails)
 * [Get last month or next month](#get-last-month-or-next-month)
 * [Get random password](#get-random-password)
 * [Parse the ini file](#parse-the-ini-file)


---
###摘要
> 将这些函数整理到一个`shell`脚本中去，然后可以通过`source`在别的脚本中直接使用
> 
> 像使用一般Linux命令一样使用这些自定义的函数



### `Print log with date`

`带时间`的日志输出

```bash
yyLog()
{
    echo -e "[`date +'%Y-%m-%d %H:%M:%S'`] $1"
} 
```

---
### `Get the first and last day of last Month`

得到上个月的`第一天`和`最后一天`

```bash
## getLastMonthfirstandlastDay "last-month"
getLastMonthfirstandlastDay()
{
    year=$(date +%Y)
    #premonth=$(date +%m -d "-1 months")
    premonth=$1
    ## 得到第一天和最后一天
    firstday1=$(cal $premonth $year|xargs|awk '{print $10}')
    lastday1=$(cal $premonth $year|xargs|awk '{print $NF}')
    ## 格式化天数为两位表示
    firstday=$(printf "%02d" $firstday1)
    lastday=$(printf "%02d" $lastday1)
    ## 输出到一行
    echo "${year}-${premonth}-${firstday} ${year}-${premonth}-${lastday}"
}


## getLastMonthfirstandlastDay2 "last-month"
getLastMonthfirstandlastDay2()
{
    year=$(date +%Y)
    #premonth=$(date +%m -d "-1 months")
    premonth=$1
    ## 得到第一天和最后一天
    firstday1=$(cal $premonth $year|xargs|awk '{print $10}')
    lastday1=$(cal $premonth $year|xargs|awk '{print $NF}')
    ## 格式化天数为两位表示
    firstday=$(printf "%02d" $firstday1)
    lastday=$(printf "%02d" $lastday1)
    ## 输出到一行
    echo "${year}${premonth}${firstday} ${year}${premonth}${lastday}"
}

```

---
### `Add certain day on one date`

在给定日期上面加 `N` 天

```bash
## addDaysOnCertain1 N
## addDaysOnCertain2 N
addDaysOnCertain1()
{
    date +%Y-%m-%d -d "$1 $2 days"
}

addDaysOnCertain2()
{
    date +%Y%m%d -d "$1 $2 days"
}
```

---
### `Send mails`

发送邮件，如果`有附件就带附件发送`

```bash
## sendEmail "subject" "recievers" "content" "attachment"
sendEmail()
{
    if [ $# -eq 3 ]
    then
        subject=$1
        recievers=$2
        content=$3
        echo $content |/usr/bin/mutt -s ${subject} ${recievers}
    elif [ $# -eq 4 ]
    then
        subject=$1
        recievers=$2
        content=$3
        attach=$4
        if [ -f ${attach} ]
        then
            echo $content |/usr/bin/mutt -s ${subject} ${recievers} -a ${attach}
        else
            yyLog "Attachment must be file type"
            exit
        fi
    else
        yyLog "Usage:`basename $0` 'subject' 'recievers' 'content' ['attachment']"
        exit
    fi
}
```

---
### `Get last month or next month`

得到正确的上个月和下个月的月份，`Linux date只是按30天加减`

```bash 
## 得到当前月份的上一个月份值
## getNextMonth
getLastMonth()
{
    month1=$(date +%m|xargs -i echo "{}-1"|bc)
    if [ ${#month1} -eq 1 ]
    then
        lastmonth=$(echo "0${month1}")
    else
        lastmonth=${month1}
    fi
    echo "${lastmonth}"
}


## 得到当前月份的下一个月份值
## getNextMonth
getNextMonth()
{
    month1=$(date +%m|xargs -i echo "{}+1"|bc)
    if [ ${#month1} -eq 1 ]
    then
        lastmonth=$(echo "0${month1}")
    else
        lastmonth=${month1}
    fi
    echo "${lastmonth}"
}

```

---
### `Get random password`

生成`N位`长度的`随机密码`，N默认是`12位`长度

```bash
## getNlenRandomPasswd [N]
getNlenRandomPasswd()
{
        if [ $# -eq 1 ]
        then
                N=$1
        else
                N=12
        fi
        date +%s+%N|sha256sum|base64|head -c${N};echo
}
```

---
### Parse the ini file

解析`ini配置文件` 相关，包括`section／items／values`

```bash
## 得到ini配置文件所有的sections名称
## listIniSections "filename.ini"
listIniSections()
{
    inifile="$1"
    # echo "inifile:${inifile}"
    # # exit
    if [ $# -ne 1 ] || [ ! -f ${inifile} ]
    then
        echo  "file [${inifile}] not exist!"
        exit
    else
        sections=`sed -n '/\[*\]/p' ${inifile}  |grep -v '^#'|tr -d []`
        echo  "${sections}"
    fi
}

## 得到ini配置文件给定section的所有key值
## ini中多个section用空行隔开
## listIniKeys "filename.ini" "section"
listIniKeys()
{
    inifile="$1"
    section="$2"
    if [ $# -ne 2 ] || [ ! -f ${inifile} ]
    then
        echo  "ini file not exist!"
        exit
    else
        keys=$(sed -n '/\['$section'\]/,/^$/p' $inifile|grep -Ev '\[|\]|^$'|awk -F'=' '{print $1}')
        echo ${keys}
    fi
}

## 得到ini配置文件给定section的所有value值
## ini中多个section用空行隔开
## listIniValues "filename.ini" "section"
listIniValues()
{
    inifile="$1"
    section="$2"
    if [ $# -ne 2 ] || [ ! -f ${inifile} ]
    then
        echo "ini file [${inifile}]!"
        exit
    else
        values=$(sed -n '/\['$section'\]/,/^$/p' $inifile|grep -Ev '\[|\]|^$'|awk -F'=' '{print $2}')
        echo ${values}
    fi
}
 
```

