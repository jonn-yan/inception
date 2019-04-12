# 关于Inception

MySQL语句的审核，在业界都已经基本被认同了，实际上也是对MySQL语句写法的统一化，标准化，而之前的人工审核，针对标准这个问题其实是很吃力的，标准越多，DBA越累，开发也越累。
那么在这个都追求自动化运维的时代，审核也必须要跟上步伐，因此Inception诞生了。而Inception可以做的工作远不止是一个自动化审核工具，同时还具备执行，生成对影响数据的回滚语句（类似闪回的功能），这样一条龙服务的工具，将会给DBA的工作带来翻天覆地的变化，DBA从此就从繁重的审核、登上去执行，出错了很难回滚（如果提前没有备份的话）的被动局面解放了出来，突然发现，做DBA原来可以这么轻松，工作可以不饱和了，那就有更多的自由时间学习、进一步向自动化运维平台的实现等更智能化的方向去发展，是具有里程碑意义的。

## 声明

Inception是一个开源系统，每个人或者每个公司都可以自由使用，由于MySQL代码的复杂性，在审核过程中不可能入戏太深，主要是将最重要的审核完成即可，面对很多复杂的子查询、表达式等是不容易检查到的，所以有些就直接忽略了，那么大家在使用过程中，有任何疑问或者发现任何BUG，都直接提出来，我们大家一起将Inception维护得更好。

同时还有另一个很重要的原因，不同公司对SQL语句的使用习惯不同，也许针对去哪儿网[http://www.qunar.com](http://www.qunar.com)来说，现在已经正常在线上服务很长时间，也没什么大的问题，而到其它公司了，则可能会遇到比较多问题，这是可以原谅的，也许是水土不服吧！！那么针对这些问题，还是请大家尽管提出来，我们大家一起将Inception维护得更好。

平时工作压力比较大，这个工具首先是为了解决问题的，可能还会存在一些没有测试过的代码，有些地方可能也是值得推敲的，还有待优化，本人会不遗余力的维护好，不断的改进，争取成为一个功能更加完善，使用更加友好的系统，同时正在酝酿一些新的更高大上的功能，敬请期待。

## 文档地址：

http://mysql-inception.github.io/inception-document/

https://inception-document.readthedocs.io/zh_CN/latest/install/#inception


## Inception安装说明
看到这个手册，想必已经得到了源码，恭喜你。

首先就是编译，在源码根目录下面有一个文件inception_build.sh，执行命令sh inception_build.sh，会输出使用方法。 实际上只需要执行inception_build.sh debug [Xcode]即可，后面的平台是可选的，如果不指定就是linux平台，而如果要指定是Xcode，就后面指定Xcode，而debug是编译的目录，编译之后，所有的生成文件都在这个目录下面，包括可执行文件Inception。可执行文件在debug/sql/Debug/目录下面（不同平台有可能不相同）。

介于在发布之后，很多人使用的是Ubuntu操作系统，这个与其它的有点不同，这里单独说一下在这个下面的编译步骤（其实需要安装的都是编译时所依赖的包，有则略之，无则装之，其它系统仿照这个应该可以轻松搞定）：

下载bison：http://ftp.gnu.org/gnu/bison/，版本最好是2.6之前的，最新的可能会有问题，下载之后，需要自己编译源码来安装，具体安装方法，可以参数网上的一些说明。
cmake安装：apt-get install cmake
ncurses安装：apt-get install libncurses5-dev
安装openssl：apt-get install libssl-dev
安装g++：sudo apt-get install g++
安装完成这些，应该是没什么问题了，那么需要注意的是，每次如果出错之后，需要把编译目录删除掉，重新执行，不然会执行出错。

顺便强调说一下，实际上编译Inception，和编译MySQL源码是一样的，如果有不太了解的同学，可以先在网上看看关于MySQL源码的编译，我想遇到的问题都可以解决。

编译完成之后，就是使用了，那么需要一个配置文件（inc.cnf）:

[inception]
general_log=1
general_log_file=inception.log
port=6669
socket=/自己目录，请自行修改/inc.socket
character-set-client-handshake=0
character-set-server=utf8
inception_remote_system_password=root
inception_remote_system_user=wzf1
inception_remote_backup_port=3306
inception_remote_backup_host=127.0.0.1
inception_support_charset=utf8mb4
inception_enable_nullable=0
inception_check_primary_key=1
inception_check_column_comment=1
inception_check_table_comment=1
inception_osc_min_table_size=1
inception_osc_bin_dir=/data/temp
inception_osc_chunk_time=0.1
inception_enable_blob_type=1
inception_check_column_default_value=1
上面这些参数的配置都是本人随便举例而已。具体每个参数的意义，请参照后面章节<<Inception所支持的参数变量>>

现在就到启动时间了，那么启动有两种方式，和MySQL是一样的，Inception可执行文件可以在编译目录下面通过find命令找到，编译目录就是在执行inception_build.sh脚本时指定的目录。 1. 所在目录/Inception --defaults-file=inc.cnf
2. 所在目录/Inception --port=6669

第二种方法就是只指定一个端口，其它参数都是默认值，而第一种方法就是在配置文件中可以指定很多参数，按照自己喜欢的规则来配置。

注意： 因为Inception支持OSC执行的功能，是通过调用pt-online-schema-change工具来做的，但如果Inception后台启动（&）的话，可能会导致pt-online-schema-change在执行完成之后，长时间不返回，进而导致Inception卡死的问题，这个问题后面会解决，但现阶段请尽量不要使用后台启动的方式，或者可以使用nohup Inception启动命令 &的方式来启动。

启动如果不报错的话，说明已经启动成功了，实际上很难让它报错，因为非常轻量级

启动成功之后，可以简单试一下看，通过MySQL客户端 
mysql -uroot -h127.0.0.1 -P6669 
登录上去之后，再执行一个命令：
inception get variables;
输出了所有的变量，恭喜你，已经启动成功了，都说了非常简单。
具体的使用的命令等在后面相应章节都会讲到，继续往后看吧！！！
