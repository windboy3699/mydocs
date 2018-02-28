### System

命令 | 功能 | 说明
---|---|---
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

命令|说明
---|---
vim /etc/profile ; source /etc/profile | 环境变量 export PATH=$PATH:/mysql/bin:/php/bin
ssh -p 12333 root@127.0.0.1 | p:端口号
find . -name "*.php" \| xargs grep "test" | 搜索文件内容
kill -INT \`cat /usr/local/webserver/php/var/run/php-fpm.pid\` | 关闭php-fpm
kill -USR2 \`cat /usr/local/webserver/php/var/run/php-fpm.pid\` | 重启php-fpm
scp local_file remote_username@remote_ip\:remote_folder | 本机文件复制到远程服务器上
scp -r remote_username@remote_ip\:remote_file local_folder | 远程服务器上的文件复制到本机
nohup command > myout.file 2>&1 & | 

### Server

命令|说明
---|---
mysql -h 192.168.0.101 -P 3306 -u root -p 123 | P:端口号, p:密码, u与root之间可以不用加空格,其它也一样 
/etc/init.d/mysqld start | 启动mysql
/usr/local/webserver/php/sbin/php-fpm | 启动php-fpm
kill -USR2 `cat /usr/local/webserver/php/var/run/php-fpm.pid` | 终止php-fpm
/usr/local/webserver/nginx/sbin/nginx | 启动nginx
/usr/local/webserver/nginx/sbin/nginx -t | 验证配置是否正确
/usr/local/webserver/nginx/sbin/nginx -s reload | 重启服务
sudo /usr/local/bin/memcached -m 32 -p 11211 -d -u root | m:最大内存用量, p:端口, d:以daemon方式运行, u:绑定指定运行进程用户
/usr/local/bin/redis-server /usr/local/etc/redis.conf | 配置文件

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

