## ln命令：
在很多情况下都会用到 ln 命令，作用在于基于原来的文件，创建一个链接，无论是软链接还是硬链接。

### 1.1 软链接：创建一个源文件的镜像，不占用空间，且随着源文件变动而变动。
```
ln -s /usr/local/php56/php  /usr/bin/php
```
 该命令，必须把源文件和目标文件都写上具体的路径名字，不能写相对路径。
### 1.2 硬链接：创建一个源文件一模一样的文件，占用空间，且随着源文件变动而变动。
```
ln /usr/local/php56/php  /usr/bin/php
```
该命令，必须把源文件和目标文件都写上具体的路径名字，不能写相对路径。