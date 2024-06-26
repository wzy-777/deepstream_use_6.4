本篇参考：https://github.com/marcoslucianops/DeepStream-Yolo/blob/master/docs/YOLOv5.md

假设当前在sources文件夹下，我们使用的yolo版本是yolov5。使用的环境是README.md中提到的deepstream-6.4
### 1 get deepstream-yolo
```bash
git clone https://github.com/marcoslucianops/DeepStream-Yolo.git
cd DeepStream-Yolo
export CUDA_VER=12.2
make -C nvdsinfer_custom_impl_Yolo clean && make -C nvdsinfer_custom_impl_Yolo
cd ..
```
会得到一个链接库地址`/opt/nvidia/deepstream/deepstream-6.4/sources/DeepStream-Yolo/nvdsinfer_custom_impl_Yolo`

### 2 get yolo
（新建一个yolo文件夹方便管理多个版本的yolo。）
下载必要文件
```bash
mkdir yolo
cd yolo
git clone https://github.com/ultralytics/yolov5.git
cd yolov5
pip3 install -r requirements.txt
pip3 install onnx onnxsim onnxruntime
cp ../../DeepStream-Yolo/utils/export_yoloV5.py ./
```
### 3 pt2onnx
根据需要，将自己的pt模型转为onnx模型，生成的文件在当前文件夹下
```bash
python3 export_yoloV5.py -w yolov5s.pt --dynamic -s 640
```
### 4 onnx2engine
可以再将onnx模型转为engine，示例如下：
```bash
/usr/src/tensorrt/bin/trtexec --onnx=yolov5.onnx --saveEngine=yolov5.engine --fp16 --workspace=2048
```
要注意，哪里运行engine文件，哪里生成engine文件！
小细节：好像fp16反而需要更大的workspace。
### 5 use and run
推理配置文件直接使用Deepstream-yolo中的config_infer_primary_yoloV5.txt即可，其中`custom-lib-path`要替换成上面得到的链接库，如：
```bash
custom-lib-path=/opt/nvidia/deepstream/deepstream-6.4/sources/DeepStream-Yolo/nvdsinfer_custom_impl_Yolo/libnvdsinfer_custom_impl_Yolo.so
```

