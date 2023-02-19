# api

## 常用函数

- `argparse.ArgumentParser().add_argument()`
  - **`action` - 命令行遇到参数时的动作，默认值是 store。**
  - **`type` - 命令行参数应该被转换成的类型**
  
- `os`
  
  - `os.path.join()`
    - 只是起到连接文件的作用，而不能生成文件。想要生成文件还要执行`os.mkdir()`
  
  - `os.walk()`
    - root 所指的是当前正在遍历的这个文件夹的本身的地址
    - dirs 是一个 list ，内容是该文件夹中所有的目录的名字(不包括子目录)
    - files 同样是 list , 内容是该文件夹中所有的文件(不包括子目录)
  
  - `os.mkdir()`创建路径中的最后一级目录
  - `os.makedirs()`创建多层目录
  - `os.listdir()`返回指定路径下的文件和文件夹列表
  - `os.path.isdir(path) `判断某一路径是否为目录
  
- `uuid`

  - `uuid.uuid1([node[, clock_seq]])` : 基于[时间戳](https://so.csdn.net/so/search?q=时间戳&spm=1001.2101.3001.7020)

  - `uuid.uuid3(namespace, name)` : 基于名字的MD5散列值

  - `uuid.uuid4()` : 基于[随机数](https://so.csdn.net/so/search?q=随机数&spm=1001.2101.3001.7020)

  - `uuid.uuid5(namespace, name)` : 基于名字的SHA-1散列值

- `shutil.copytree`复制文件夹

- `yield`带有yield的函数是一个迭代器，函数返回某个值时，会停留在某个位置，返回函数值后，会在前面停留的位置继续执行，直到程序结束。

- `json`

  | json.dumps() | 将python对象编码成Json字符串             |
  | ------------ | ---------------------------------------- |
  | json.loads() | 将Json字符串解码成python对象             |
  | json.dump()  | 将python中的对象转化成json储存到文件中   |
  | json.load()  | 将文件中的json的格式转化成python对象提取 |

  - [json.loads()的参数需要有固定格式，引号必须为双引号，单引号会报错。](https://blog.csdn.net/qq_41375609/article/details/100118545)

- `glob.glob(path+pattern)`：可以像linux中的`find -name *.type`一样，将某目录下所有跟通配符模式相同的文件放到一个列表中

- `with`关键字实现原理建立在上下文管理器之上。

  ```python
  # 上下文管理器是一个实现 __enter__ 和 __exit__或者__aenter__和__aexit__ 方法的类。
  # 使用with 语句确保在嵌套块的末尾调用__exit__方法。
  with open('./test.txt', 'w') as file:
      file.write('hello world !')
  
  # 执行async with as语句时，__aenter__函数会被调用，与__enter__函数无关，async with as语句块执行完毕，调用__aexit__语句
  async with open('./test.txt', 'w') as file:
    	file.write('hello world !')
      
  ```

  ```python
  import asyncio
  
  class Test():
      def __init__(self):
          print("Init!")
      def __enter__(self):
          print("Enter!")
      def __exit__(self, type, value, trace):
          print("Exit!")
      async def __aenter__(self):
          print("async Enter!")
      async def __aexit__(self, type, value, trace):
          print("type:", type)
          print("value:", value)
          print("trace:", trace)
          print("async Exit!")
          return True                    # 这个return True是什么意思呢？
        
  async def main():
      async with Test() as f:            # async with as语句
          print("f:", f)
          a = 1 / 0
          print("Have a try!")           # 猜猜这句话会不会执行？
  
  if __name__ == "__main__":
  
      loop = asyncio.get_event_loop()    # 创建事件循环
      loop.run_until_complete(main())    # 异步函数不能直接调用，所以这里放在了一个事件循环里面
      print("Loop finished!")            # 猜猜这句话会不会执行？
     
  ----------------打印结果-----------------
  Init!
  async Enter!
  f: None
  type: <class 'ZeroDivisionError'>
  value: division by zero
  trace: <traceback object at 0x000002CB429DBC08>
  async Exit!
  Loop finished!
  ```

  

## 常见内置变量

- `__name__`：内置变量，表示当前模块的名字

  - `if __name__ == '__main__'`如果模块是被直接运行，则代码运行。如果模块是导入的，则不运行

  - `__init__`和`__all__`
    - 当导入这个包的时候，会自动执行init里面的代码
    - 没有放在`__all__`中的，不会被导入

- `__file__`保持当前文件夹不变
  - `os.path.dirname(os.path.abspath(__file__))`
