##1 集合类

#### A 集合交

>cat fileA fileB|sort |uniq –d > jiaoji.log

#### B 集合差

>cat fileA fileB|sort |uniq -d > jiaoji.txt
>cat fileA jiaoji.txt|sort |uniq -u > chaji.log

#### C 集合全集去重

>cat fileA fileB|sort -u > result.log

#### D 集合全集不去重

>cat fileA fileB|sort > result.log

##2 其他

#### A 随机抽取

>cat file1 | awk '{ print rand(),$1 }' |sort -k1 |awk '{ print $2 }' |head -100

#### B 分组求和

>以第一列 为变量,将相同第一列的第二列数据进行累加,打印出和
>awk '{s[$1] += $2}END{ for(i in s){  print i, s[i] } }' file1 > file2
  
>以第一列和第二列为变量名,将相同第一列、第二列的第三列数据进行累加,打印出和
>awk '{s[$1" "$2] += $3}END{ for(i in s){  print i, s[i] } }'  file1 > file2
 
>如果第一列相同,则根据第一列来分组,分别打印第二列和第三列的和
>awk '{s[$1] += $2; a[$1] += $3 }END{ for(i in s){  print i,s[i],a[i] } }'  result.txt

#### C 匹配

>匹配交集项
>awk 'NR==FNR{a[$1]=1}NR>FNR&&a[$1]>0{print $0}'  file1 file2 > file3
>如果file1、file2中,2个文件的第一列值相同,输出第2个文件的所有列

#### D 取最大值、最小值

>文件有2列
>awk '{max[$1]=max[$1]>$2?max[$1]:$2}END{for(i in max)print i,max[i]}'  file
 
>文件只有1列
>awk 'BEGIN {max = 0} {if ($1>max) max=$1 fi} END {print "Max=", max}' file2
>awk 'BEGIN {min = 9999999} {if ($1<min) min=$1 fi} END {print "Min=", min}' file2

 

#### E 求和、求平均值、求标准偏差

>求和
>cat data|awk '{sum+=$1} END {print "Sum = ", sum}'

>求平均
>cat data|awk '{sum+=$1} END {print "Average = ", sum/NR}'

>求标准偏差
>cat $FILE | awk -v ave=$ave '{sum+=($1-ave)^2}END{print sqrt(sum/(NR-1))}'

 