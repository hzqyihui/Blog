## crontab

使用 cron的时候，我们经常会因为 某个命令运行时间太长，命令再次被启动时，会出现多进程。  
可以使用flock, 如：

```
*/1 * * * * flock -xn /opt/app/nginx/test_repo/app/tasks/checkPaymentUrl.lock -c 'sudo -u apache php /opt/app/nginx/test_repo/app/console Payment checkPaymentUrl >> /dev/null 2>&1'
```
当多个进程可能会对同样的数据执行操作时，这些进程需要保证其它进程没有也在操作，以免损坏数据。  

通常，这样的进程会使用一个「锁文件」，也就是建立一个文件来告诉别的进程自己在运行，如果检测到那个文件存在则认为有操作同样数据的进程在工作。这样的问题是，进程不小心意外死亡了，没有清理掉那个锁文件，那么只能由用户手动来清理了。  
参数
```
-s,--shared：获取一个共享锁，在定向为某文件的FD上设置共享锁而未释放锁的时间内，其他进程试图在定向为此文件的FD上设置独占锁的请求失败，而其他进程试图在定向为此文件的FD上设置共享锁的请求会成功。
-x，-e，--exclusive：获取一个排它锁，或者称为写入锁，为默认项。
-u，--unlock：手动释放锁，一般情况不必须，当FD关闭时，系统会自动解锁，此参数用于脚本命令一部分需要异步执行，一部分可以同步执行的情况。
-n，--nb, --nonblock：非阻塞模式，当获取锁失败时，返回1而不是等待。
-w, --wait, --timeout seconds：设置阻塞超时，当超过设置的秒数时，退出阻塞模式，返回1，并继续执行后面的语句。
-o, --close：表示当执行command前关闭设置锁的FD，以使command的子进程不保持锁。
-c, --command command：在shell中执行其后的语句。
```
实例  
crontab运用flock防止重复执行
```
0 23 * * * (flock -xn ./test.lock -c "sh /root/test.sh") #-n 为非阻塞模式
```