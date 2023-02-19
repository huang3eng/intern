# triton

> 版本号目录和config.pbtxt是平级的

## 整体结构

```txt
model-repository-path/
├── model_name1/
│   ├── config.pbtxt
│   ├── output-labels-file
│   ├── version1/
│   │   └── model-file
│   └── version2/
│       └── model-file
└── model_name2/
    ├── config.pbtxt
    ├── output-labels-file
    ├── version1/
    │   └── model-file
    └── version2/
        └── model-file
```

##  [config.pbtxt](https://blog.csdn.net/lansebingxuan/article/details/125513063)

- `platform/backend`

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/f28d908284b84b61887f68d62953c1f3.png)

- `max_batch_size`：模型执行推理最大的batch是多少

  - `max batch size > 0`默认网络的batch大小可以是动态调整的，在网络输入维度dims参数可以不指定batch大小
  - `max batch size = 0`网络输入维度dims参数必须显式指定每个维度的大小

- `input and output`：

  > 当模型是由TensorRT、Tensorflow、onnx生成的，且设置strict-model-config=false时，可以不需要config.pbtxt文件

  ```
  input [
    {
        name: "INPUT_0"
        data_type: TYPE_UINT8
        dims: [ -1 ]
    },
    {
        name: "INPUT_1"
        data_type: TYPE_FP32
        dims: [ 1, 4 ]
    }
  ]
  
  output [
    {
        name: "OUTPUT_0"
        data_type: TYPE_FP32
        dims: [ 3, -1, -1 ]
    },
    {
        name: "OUTPUT_1"
        data_type: TYPE_FP32
        dims: [ 1, 4 ]
    }
  ]
  
  ```

- `version_policy`

  - `all`:表示model repository中所有模型都可以加载，若使用该配置，执行的时候model repository中所有模型都会加载。
  - `latest`:只执行最新的版本，最新指版本数字最大的，若使用该配置，则只选择最新的模型加载。
  - `specific`：执行指定版本。若使用该配置，需设定指定的版本号，加载时只加载指定的相应版本。

- `instance_group`：在一个Triton server上对同一个模型开启多个execution instance，可以并行的在GPU上执行，从而提高GPU的利用率，增加模型的吞吐

  - count：指定每个设备上的instance数量
  - kind：定义设备类型，可以是cpu，也可以是GPU
  - gpus：定义使用哪个GPU，若不指定，默认会在每一个GPU上都运行。

  ```
  在gpu1和2上分配8个实例，自爱cpu分配2个实例
  instance_group [
      {
        count: 8        
        kind: KIND_GPU  
        gpus:[1,2]      
      },
      
      {
        count: 2
        kind: KIND_CPU
  		}
  ]
  ```

- `dynamic_batching`

  - `preferred_batch_size`：期望达到的batchsize大小，可以是一个数，也可以是一个数组，通常会在里面设置多个值
  - `max_queue_delay_microseconds`：单位是微秒，打batch的时间限制，超过该时间会停止batch，超过该时间会将以打包的batch送走。
  - `preserve_ordering`：确保输出的顺序和请求送进来的顺序一致。
  - `priority_level`：可以设置优先级，
  - `queue_policy`：设置排队的策略，比如时限超时则停止推理

- `sequence_batching`：处理sequence请求。能够保证同一个streaming序列的请求送到同一个model instance执行，从而能够保证model instance的状态。

- `optimization`：Triton中提供两种优化方式：分别针对onnx模型和tensorflow模型

- `model_warmup`：有些模型在刚初始化的短时间内，执行推理时性能是不太稳定的，可能会比较慢，所以需要一个热身的过程使得推理趋于稳定。

- `ensemble_scheduling`：可以组合不同的模块，形成一个pipeline

## [tritonserver 启动参数](https://zhuanlan.zhihu.com/p/366555962)

```
恰恰是因为你害怕自己死亡前，自己没有足够的生命力来镇定自己，来迎接死亡的到来。而人在天地间活着，就是学会从生身之世中去获得生命力。从知识、哲学，从形而上与形而下，从东方或西方的思想，从实践、奉献、努力、劳动、奋斗、创造、体验、自由、情感、爱、家人、朋友，从艺术、科学、人文、心流、爱好、理想、信仰中去不断获得生命力，你宝贵的时间不应该过分过多消耗在害怕这个、害怕那个，乃至陷入无意义的争吵、愤怒、焦虑等精神内耗中去，活活埋葬自己的青春或生命，从而不甘心得活着。
```

