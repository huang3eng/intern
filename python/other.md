# 其他

### pip

- `pip install`
  - `-i`选项设置临时使用国内的镜像网站
  - `–trusted-host`指定可信任的主机
  - `-f` 就是从指定url文件中查找包的下载链接，从后面紧跟的url参数获得的文件中，找pip要安装的包文件（而不是从默认的pip 安装源）

### twine

```shell
twine upload --
```



### pdb

- 两种运行方式
  - `python3 -m pdb filename.py`
  - `import pdb; pdb.set_trace()`
- `ll`：查看当前函数或框架的所有源代码
- `b`：添加断点
  - `b lineno`
  - `b filename:lineno` 
  - `b functionname`
- `tb`：临时断点，执行一次后时自动删除
- `cl`：清除断点
- `p expression`：打印变量值
- 调试命令
  - `s`：执行下一行（能够进入函数体）
  - `n`：执行下一行（不进入函数体）
  - `r` ：执行下一行（在函数中时会直接执行到函数返回处）
  - `c` ：执行到下一个断点
  - `unt lineno`：持续执行直到运行到指定行（或遇到断点）
- `a`：在函数中时打印函数的参数和参数的值
- `whatis expression`：打印表达式的类型，常用来打印变量值
- `w`：打印堆栈信息，最新的帧在最底部。箭头表示当前帧。
- `q`：退出pdb

### 多线程pdb

- https://github.com/Lightning-AI/forked-pdb

## 打包

- 打包脚本

  ```shell
  # set -e 当命令发生错误的时候，停止脚本的执行
  # set -x 把将要运行的命令用一个+标记之后显示出来
  set -ex 
  DIR=$(cd 'dirname $0'; pwd)
  cd $DIR
  python3 setup.py clean
  python3 setup.py build sdist bdist_wheel
  twine upload --repository-url <指定仓库>  -u <指定用户名> -p <指定密码> dist/*.whl
  ```

- [python 打包上传](https://zhuanlan.zhihu.com/p/261579357)

- 两种安装包的方式

  - `pip install` 直接安装已经上传好的python包
  - 将源码下载下来，构建然后安装
    - 先下载你要安装的包，并解压到磁盘下
    - 进入到该文件的setup.py 目录下 ，打开`cmd`，并切换到该目录下
    - 先执行 `python setup.py build`
    - 然后执行 `python setup.py install`

- [MANIFEST.in文件](https://blog.csdn.net/weixin_43590796/article/details/121122850)

```
二十世纪五六十年代，美国梦，功绩主义，依靠大规模基建，繁华城市背后是丧、倦怠、。70年代滞胀。

那些你独自一人度过的时间，比如组装电脑或者练习大提琴，其实你真正在做的是让自己变得有趣，等有天别人终于注意到你的时候，他们会发现一个比他们想象中更酷的人。

while you're spending all this time on your own
```

