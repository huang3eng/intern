# linux

## shell

- ubuntu是基于apt工具包进行软件安装的，而apt工具包是基于DebianPackageManagement的。*DEBIAN_FRONTEND*就是DebianPackageManagement的基本选项。

- `apt install`和`apt-get install`：如果在非交互式脚本中运行`apt install`

- `vi`和`vim`

  - vi编辑器是所有Unix及Linux系统下标准的编辑器
  - vim具有程序编辑的能力，可以以字体颜色辨别语法的正确性，方便程序设计

- `sed` 在处理文本时是逐行读取文件内容，读到匹配的行就根据指令做操作，不匹配就跳过

  - 常用选项

    - `-i`：直接对内容进行修改，不加-i时默认只是预览，不会对文件做实际修改

  - 编辑命令

    | a-追加 | 向匹配行后面插入内容 |
    | ------ | -------------------- |
    | i-插入 | 向匹配行前插入内容   |
    | c-更改 | 更改匹配行的内容     |
    | d-删除 | 删除匹配的内容       |
    | s-替换 | 替换掉匹配的内容     |

  - `sed -i s/A/B/g`：s表示替换，替换A为B，g表示全局

- `set -ex`
  - `-e`当命令发生错误的时候，停止脚本的执行
  - `-x` 参数的作用，是把将要运行的命令用一个 `+` 标记之后显示出来

- `DIR=$(cd 'dirname $0'; pwd)`：获取当前脚本所在绝对路径，赋给全局变量DIR
  - `dirname $0`，获取当前脚本所在绝对目录
  - `cd dirname $0`，进入这个目录（切换当前工作目录）
  - `pwd`，显示切换后脚本所在的工作目录

- `cd $(dirname $0) || exit`：在脚本中，进入脚本所在目录，否则退出

- `if [-z "$env"]; then`：判断是否存在一个环境变量`env`

- `|grep`：`|` 是管道命令，`grep`查找
  - `ls | grep "main"`：用来搜索 ls 命令执行后的输出中，是否包含 main

- `awk`：是用来提取列的主要工具
  - `awk '{print $1,$2}' rumenz.txt`：打印文件的前两列

- `xargs`：给命令传递参数的一个过滤器，一般和管道一起使用
  - `command |xargs command`

- `tar`：首先要弄清两个概念：打包和压缩。打包是指将一大堆文件或目录变成一个总的文件；压缩则是将一个大的文件通过一些压缩算法变成一个小文件。这源于Linux中很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（tar命令），然后再用压缩程序进行压缩（gzip bzip2命令）。

  ```shell
  -c：创建新的tar文件
  -x：解开tar文件
  -t：列出tar文件中包含的文件的信息
  -r：附加新的文件到tar文件中
  -z：使用gzip进行解压缩
  -j：使用bzip2进行解压缩
  -Z：使用compress进行解压缩
  -v：显示解压缩执行过程
  -f：指定要处理的文件名
  # 如果需要使用-f参数，需要将f参数放在所有参数最后面，在f之后要立即接文件名，不能有其他参数。
  # gzip命令用来压缩文件。gzip是个使用广泛的压缩程序，文件经它压缩过后，其名称后面会多处“.gz”扩展名。
  ```

- `cat filepath`：显示文件内的内容

- `echo $PATH`：可以显示PATH内容

- `cat /etc/os-release` ：查看linux的版本

- `tmux`

- `netstat` ：用于显示各种网络相关信息

  - -a (all)显示所有选项，默认不显示LISTEN相关
  - -t (tcp)仅显示tcp相关选项
  - -u (udp)仅显示udp相关选项
  - -n 拒绝显示别名，能显示数字的全部转化成数字。
  - -l 仅列出有在 Listen (监听) 的服務状态
  - -p 显示建立相关链接的程序名
  - -r 显示路由信息，路由表
  - -e 显示扩展信息，例如uid等
  - -s 按各个协议进行统计
  - -c 每隔一个固定时间，执行该netstat命令。

  **提示：LISTEN和LISTENING的状态只有用-a或者-l才能看到**

- `su`：用于切换当前用户身份到指定用户或者以指定用户的身份执行*命令*或程

- `reset`：重新初始化终端的（terminal initialization）

- `socat`：[网络瑞士军刀](https://zhuanlan.zhihu.com/p/347722248)

- curl：[请求web服务器](https://www.ruanyifeng.com/blog/2019/09/curl-reference.html)

  ```shell
  curl -X <HTTP-Method> [-d <parameters>] http://hostname:port/v1/<REST-point>
  ```

  

### 正则表达式和文本三剑客

#### 正则表达式

- 特殊字符：需要加上转义符 \ 

  | 特殊字符 | 特殊含义                                                     |
  | -------- | ------------------------------------------------------------ |
  | ()       | 标记一个子表达式的开始和结束位置。子表达式可以获取供以后使用 |
  | $        | 匹配输入字符串的结尾位置                                     |
  | *        | 匹配前面的子表达式零次或多次                                 |
  | +        | 匹配前面的子表达式一次或多次                                 |
  | .        | 匹配前面的子表达式零次或多次                                 |
  | [ ]      | 标记一个中括号表达式的开始。要匹配 [，请使用 [。             |
  | ?        | 匹配前面的子表达式零次或一次，或指明一个非贪婪限定符         |
  | \        | 将下一个字符标记为或特殊字符、或原义字符、或向后引用、或八进制转义符 |
  | ^        | 匹配输入字符串的开始位置，除非在方括号表达式中使用，当该符号在方括号表达式中使用时，表示不接受该方括号表达式中的字符集合 |
  | {}       | 标记限定符表达式的开始                                       |
  | \|       | 指明两项之间的一个选择                                       |
  |          |                                                              |

- 限定字符：限定字符出现的次数

  | **限定符** | **表达含义**               |
  | ---------- | -------------------------- |
  | *          | 出现次数>=0                |
  | +          | 出现次数>=1                |
  | ?          | 出现次数 0 or 1, 等价{0,1} |
  | {n}        | 出现次数=n                 |
  | {n,}       | 出现次数>=n                |
  | {n, m}     | n=< 出现次数<= m           |

- 定位符：

  | **定位符** | **表达含义**     |
  | ---------- | ---------------- |
  | ^          | 字符串开始的位置 |
  | $          | 字符串结束的位置 |

  

```shell
# (xxx) xxx-xxxx 
# '('和')'属于特殊字符，
# x表示数字可以用[0-9]和\d表示
# 限定字符出现的次数为n，则使用限定字符{n}
```

- [`.*`、`.*?`和`.+?`](https://blog.csdn.net/sinat_32336967/article/details/94761771)
  - `a.*b`，它将会匹配最长的以a开始，以b结束的字符串
  - `a.*?b`匹配最短的，以a开始，以b结束的字符串。
  - `a.+?b`匹配最短的，以a开始，以b结束的字符串，但a和b中间至少要有一个字符。

#### 文本三剑客

- `awk`：擅长取列。
- `sed`：擅长取行，支持拓展正则。
- `grep`：擅长过滤，支持拓展正则。

```shell
grep -P '表达式' file.txt# -P 可以让grep使用perl的正则表达式语法，因为perl的正则更加多元化
awk '/表达式/' file.txt
sed '/表达式/' file.txt
```



## sh

- 获取脚本所在目录的绝对路径

  ```sh
  # 不能直接使用pwd命令，pwd是输入命令时所在的绝对路径
  DIR=$(cd 'dirname $0'; pwd)
  # $0表示运行的命令名，一般用在shell脚本中
  # dirname用于取指定路径所在的目录
  # $(命令) 返回该命令的结果
  ```

- `set -ex`

  ```sh
  # set -e 当命令发生错误的时候，停止脚本的执行
  # set -x 把将要运行的命令用一个+标记之后显示出来
  ```

  

## 环境变量

-  `/etc/profile` 是所有用户登录时加载配置环境

- `/etc/bashrc` 是所有用户在执行bash shell 时，加载配置环境

- `~/.bashrc`文件（在用户的家目录下则只对当前用户有用。

- 命令行输入`export PATH=$PATH:/some/directory`仅在当前的终端生效，如果要将目录永久添加到 $PATH ，只要将这行添加到`.bashrc`或`/etc/bashrc`文件中

- 网络代理环境变量

  - 对于大多数Linux控制台程序，例如[Debian](http://easwy.com/blog/archives/tag/debian/) 或Ubuntu中的**apt-get** 和**aptitude** 命令、[git命令](http://easwy.com/blog/archives/use_http_proxy_for_git/) 、wget命令，这些程序都使用*http_proxy* 和*ftp_proxy* 环境变量来获取代理服务的配置。

    ```shell
    vi /etc/profile
    #设置http代理
    http_proxy=http://username:password@proxyserver:port/
    #设置https代理
    https_proxy=http://username:password@proxyserver:port/
    #设置不通过代理服务器链接
    no_proxy=*.xxx.com,10.*,www.baidu.com
    source /etc/profile
    
    vi ~/.bashrc
    export http_proxy=http://username:password@proxyserver:port/
    export ftp_proxy=http://username:password@proxyserver:port/ 
    source ~/.bashrc
    ```

  - [curl配置代理服务器](https://baijiahao.baidu.com/s?id=1733081444327574530&wfr=spider&for=pc)


### 其他问题

- [iTerm2 进行 ssh 时，空闲一段时间断开问题的解决](https://www.track2web.com/tool_tips/575.html)

  ```
  {
    "query_block": {
      "select_id": 1,
      "cost_info": {
        "query_cost": "0.35"
      },
      "table": {
        "table_name": "T",
        "access_type": "ALL",
        "rows_examined_per_scan": 1,
        "rows_produced_per_join": 1,
        "filtered": "100.00",
        "cost_info": {
          "read_cost": "0.25",
          "eval_cost": "0.10",
          "prefix_cost": "0.35",
          "data_read_per_join": "8"
        },
        "used_columns": [
          "i"
        ]
      }
    }
  }
  
  ```
  

## Reference

- [命令行神器](https://www.zhihu.com/question/59227720/answer/163080080)
