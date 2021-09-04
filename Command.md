# 常用命令

### 查看系统配置

命令 | 功能
---|---
cat /proc/cpuinfo \| grep "physical id" \| sort \| uniq \| wc -l | 查看物理CPU个数
cat /proc/cpuinfo \| grep "cpu cores" \| uniq | 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo \| grep "processor" \| wc -l | 查看逻辑CPU的个数(通过超线程等方式)
grep MemTotal /proc/meminfo | 查看内存总量
grep MemFree /proc/meminfo | 查看空闲内存量

### System

命令 | 功能 | 说明
---|---|---
vim /etc/profile | 修改环境变量 | 末尾添加export PATH="/NewPath:$PATH" , source /etc/profile
cp -r | 复制 | r:递归
rm -fr | 删除 | f:不显示提示, r:递归
mkdir -p | 创建目录 | p:递归创建
head -10 file | 查看开头几行 | 
tail -10 file | 查看末尾几行 | f:自动实时
less -N | 分页显示 | -N:显示行号, 操作: u向上翻半页d向下翻半页n搜索下一个q退出
ps -ef \| grep ssh | 显示所有进程信息连同命令行 | 与grep 常用组合查找进程
ps aux | 列出所有 | 列出目前所有正在内存当中的程序
kill –9 | 强制杀死进程
df -h | 查看服务器磁盘空间 | h:方便阅读方式
du -h file | 查看目录或文件磁盘空间 | h:方便阅读方式
chmod -R 777 file | 文件权限修改 | R:递归, 数字:4(读)、2(写)、1(执行)
chmod a-wx,a+r file | 文件权限修改 | u:用户, g:组, o:其他, a:所有
tar -zcvf etc.tar.gz /etc | c为压缩 | z:gzip后缀xx.tar.gz或xx.tgz, 
tar -zxvf etc.tar.gz | x为解压 | v:显示过程, f:使用档名放最后后面接档名
tar -jcvf etc.tar.bz2 /etc | bz2压缩 | j:bzip2类型后缀xx.tar.bz2
ln -s source_file target_file | 符号链接 | 
sudo vim filename | 以root用户执行 | 
su - admin -c "ls -l" | 指定用户执行 | 
shutdown -h now | 立即关机 | 
shutdown -r now | 立即重启 | 

命令|说明
---|---
vim /etc/profile ; source /etc/profile | 环境变量 export PATH=$PATH:/mysql/bin:/php/bin
ssh -p 12333 root@216.230.230.114 | p:端口号
find . -name "*.php" \| xargs grep "test" | 搜索文件内容
kill -INT \`cat /usr/local/php/var/run/php-fpm.pid\` | 关闭php-fpm
kill -USR2 \`cat /usr/local/php/var/run/php-fpm.pid\` | 重启php-fpm
scp -P 96 local_file remote_username@remote_ip\:remote_folder | 本机文件复制到远程服务器上 P:端口号
scp -r remote_username@remote_ip\:remote_file local_folder | 远程服务器上的文件复制到本机
nohup command > myout.file 2>&1 & | 
crontab: minute hour day month week command | 
awk -F "#" '/"id".+"name"/{print $0,"\n"}' filename | 同行带空格输出
awk -F "#" '/"id".+"name"/{print $1; print $2}' filename | 多行输出

/usr/bin/rsync -agoprtv --delete-after --bwlimit=2048 /home/pplive/web 10.80.10.20:/home/pplive/web

zcat 201808* | awk '/mobile/{idx=match($8,"uid=");if (idx > 0) {idx+=4;uid=substr($8,idx,15)} else {uid=""};print $7,uid}'

awk '{arr[$1]++}END{for(key in arr) print arr[key],key}' access.log | sort -t " " -k 1 -n -r  | head -100  
【-t:分隔符默认是用tab分隔; -k:以那个区间来进行排序; -n:使用纯数字排序; -r:反向排序; -u:uniq,相同的数据中仅出现一行】

### Server

命令|说明
---|---
mysql -S mysql.sock -h 127.0.0.1 -P 3306 -u root -p | P:端口号, p:密码, u与root之间可以不用加空格,其它也一样 
/etc/init.d/mysqld start | 启动mysql
/usr/local/php/sbin/php-fpm | 启动php-fpm
kill -USR2 `cat /usr/local/php/var/run/php-fpm.pid` | 重启php-fpm，关闭把-USR2换成-INT
/usr/local/nginx/sbin/nginx | 启动nginx
/usr/local/nginx/sbin/nginx -t | 验证配置是否正确
/usr/local/nginx/sbin/nginx -s reload | 重启服务
sudo /usr/local/bin/memcached -m 32 -p 11211 -d -u root | m:最大内存用量, p:端口, d:以daemon方式运行, u:绑定指定运行进程用户
/usr/local/bin/redis-server /usr/local/etc/redis.conf | 配置文件
sudo /Users/username/Library/Tomcat/bin/startup.sh | Mac启动Tomcat
zookeeper/bin/zkServer start | 启动zookeeper
kafka/bin/kafka-server-start -daemon confpath/server.properties | 启动kafka

### Vim
命令 | 功能 | 说明
---|---|---
:tabe| 新标签页编辑文件 | 
:tabc | 关闭当前页 | 
:tabo | 关闭所有其他的tab | 
:tabs | 列出标签页包含的窗口 | 当前窗口显示">" 修改过的缓冲区显示"+"
gt | 转到下一页 | 
gT | 转到前一页 | 
{n}gt | 转到第{n}页，首个标签页为1 |

### Git
命令 | 说明
---|---
git fetch origin branchname:branchname | 拉取远程分支到本地，本地不存在则创建
git checkout branchname | 切换分支
git reset HEAD <file> | 
git reset --hard <commit_id> | 
git show <commit_id> | 
git log -p <file> | 查看每次详细修改内容的diff 
git diff branch1 branch2 --stat | stat:显示文件列表否则是文件内容diff
git config --global user.name [username] | 
git config --global user.email [email] | 
git config --list | 
git remote -v | 
git log --pretty=format:"%C(bold white)%h%Creset %cd %C(bold blue)%cn%Creset %s" --graph | 

### Mac
launchctl load /Users/xxxx/rsync.plist