# Model Used In Triton

> **triton定义了不同类型神经网络计算图的计算图**
>
> triton支持的模型平台和对应的backend
>
> ![image-20230103143806852](https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20230103143806852.png)

## tensorflow

### 保存的模型格式

#### checkpoint(*.ckpt)

- 只提供checkpoint模型不提供代码是无法重新构建计算图的

#### GraphDef(*.pb)

- 这种格式文件包含 protobuf 对象序列化后的数据，包含了计算图，可以从中得到所有运算符（operators）的细节，也包含张量（tensors）和 Variables 定义，但不包含 Variable 的值，因此只能从中恢复计算图，但一些训练的权值仍需要从 checkpoint 中恢复。

#### FrozenGraphDe(*.pb)

- 这种文件格式不包含 Variables 节点。将 GraphDef 中所有 Variable 节点转换为常量（其值从 checkpoint 获取），就变为 FrozenGraphDef 格式。

#### SavedModel

- 该格式为 GraphDef 和 CheckPoint 的结合体，另外还有标记模型输入和输出参数的 SignatureDef。从 SavedModel 中可以提取 GraphDef 和 CheckPoint 对象。

### triton中的使用

#### .pb

- 文件结构

  ```
  table
  ├── 1
  │   └── model.graphdef
  └── config.pbtxt
  ```

- config.pbtxt

  ```protobuf
  name: "table"
  backend: "tensorflow"
  platform: "tensorflow_graphdef"
  max_batch_size: 256
  
  input [
      {
          name: "ImageTensor"
          data_type: TYPE_UINT8
          dims: [ -1,-1,3 ]
      }
  ]
  
  output[
      {
          name: "SemanticPredictions"
          data_type: TYPE_FP32
          dims: [-1,-1,5]
      }
  ]
  ```

### Reference

- [tensorflow 模型导出总结](https://zhuanlan.zhihu.com/p/113734249)
- [TensorFlow 到底有几种模型格式？](https://cloud.tencent.com/developer/article/1009979)

## pytorch

> - pytorch动态建图带来的优势对于性能要求更高的应用场景而言更像是缺点，非固定的网络结构给网络结构分析并进行优化带来了困难。简而言之，pytorch适合训练不适合部署。 TorchScript 就是为了解决这个问题而诞生的工具。
>
> - ONNX （Open Neural Network Exchange）是 Facebook 和微软在2017年共同发布的，用于标准描述计算图的一种格式。
>
>   ![pt&onnx.drawio](https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/pt&onnx.drawio.png)



### 保存模型格式

#### *.pth

> 如果模型要应用于推理，必须要定义模型，triton不支持这种格式

#### TorchScript(*.pt)

> 使用[TorchScript](https://zhuanlan.zhihu.com/p/486914187)，无需定义模型，就可以加载导出的模型并运行推理

##### 两种生成TorchScript 模型的方式

-  trace模式：指的是进行一次模型推理，在推理的过程中记录所有经过的计算，将这些记录整合成计算图
- script模式：这种方式会直接解析网络定义的 python 代码，生成抽象语法树 AST，因此这种方法可以解决一些 trace 无法解决的问题，比如对 branch/loop 等数据流控制语句的建图

#### .onnx

```python
# torch.onnx.export方法
def export(model, args, f, export_params=True, verbose=False, training=TrainingMode.EVAL, 
           input_names=None, output_names=None, aten=False, export_raw_ir=False, 
           operator_export_type=None, opset_version=None, _retain_param_name=True, 
           do_constant_folding=True, example_outputs=None, strip_doc_string=True, 
           dynamic_axes=None, keep_initializers_as_inputs=None, custom_opsets=None, 
           enable_onnx_checker=True, use_external_data_format=False): 
  
# torch.onnx.export使用
model = Model()
model.eval()
dummy_input = torch.randn(1, 3, 448, 448)
basename_onnx = "{}.onnx".format('model')
save_file_onnx = os.path.join(sub_work_dir, basename_onnx)
dummy_output = model(dummy_input)
torch.onnx.export(model,
                  dummy_input,
                  save_file_onnx,
                  export_params=True,
                  opset_version=11,
                  input_names=['input'],
                  output_names=['output'],
                  dynamic_axes={
                      'input': {0: 'batch_size'},  # variable length axes
                      'output': {0: 'batch_size'}}
                  )

```

- 重要参数详解：
  - `model,args,f`：三个必选参数模型、模型输入、导出的 onnx 文件名
  - `export_params`：模型中是否存储模型权重。一般中间表示包含两大类信息：模型结构和模型权重，这两类信息可以在同一个文件里存储，也可以分文件存储。ONNX 是用同一个文件表示记录模型的结构和权重的。
    我们部署时一般都默认这个参数为 True。如果 onnx 文件是用来在不同框架间传递模型（比如 PyTorch 到 Tensorflow）而不是用于部署，则可以令这个参数为 False。
  - `input_names, output_names`：设置输入和输出张量的名称。如果不设置的话，会自动分配一些简单的名字（如数字）。ONNX 模型的每个输入和输出张量都有一个名字。很多推理引擎在运行 ONNX 文件时，都需要以“名称-张量值”的数据对来输入数据，并根据输出张量的名称来获取输出数据。在进行跟张量有关的设置（比如添加动态维度）时，也需要知道张量的名字。在实际的部署流水线中，我们都需要设置输入和输出张量的名称，并保证 ONNX 和推理引擎中使用同一套名称。
  - `opset_version`：转换时参考哪个 ONNX 算子集版本，默认为 9。
  - `dynamic_axes`：指定输入输出张量的哪些维度是动态的。为了追求效率，ONNX 默认所有参与运算的张量都是静态的（张量的形状不发生改变）。但在实际应用中，我们又希望模型的输入张量是动态的，尤其是本来就没有形状限制的全卷积模型。因此，我们需要显式地指明输入输出张量的哪几个维度的大小是可变的。

### triton中的使用

#### .pt

- 文件结构

  ```
  resnest50
  ├── 1
  │   └── model.pt
  └── config.pbtxt
  ```

- config.pbtxt

  ```protobuf
  name: "resnest50"
  backend: "pytorch"
  max_batch_size: 256
  input [
      {
          name: "CLS0_INPUT_0"
          data_type: TYPE_UINT8
          dims: [ -1,-1,3 ]
      }
  ]
  
  output [
      {
          name: "CLS0_OUTPUT_0"
          data_type: TYPE_FP32
          dims: [ -1]
      }
  ]
  ```

#### .onnx

- 文件结构

  ```
  densenet_onnx
  ├── 1
  │   └── model.onnx
  └── config.pbtxt
  ```

- config.pbtxt

  ```protobuf
  name: "densenet_onnx"
  platform: "onnxruntime_onnx"
  max_batch_size : 256
  input [
    {
      name: "data_0"
      data_type: TYPE_FP32
      format: FORMAT_NCHW
      dims: [ 3, 224, 224 ]
      reshape { shape: [ 1, 3, 224, 224 ] }
    }
  ]
  output [
    {
      name: "fc6_1"
      data_type: TYPE_FP32
      dims: [ 1000 ]
      reshape { shape: [ 1, 1000, 1, 1 ] }
    }
  ]
  ```

### Reference

- [SAVING AND LOADING MODELS](https://pytorch.org/tutorials/beginner/saving_loading_models.html)