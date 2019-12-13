### fast_transformers: ����CPU�Ŀ���Transformer������

Transformer�ǽ�������NLP��������Ҫ��ģ�ʹ��£��ڴ������ߵ�ģ�;��ȵ�ͬʱҲ�����˸���ļ���������Ը�Ч����Transformer���Ϸ�������˾޴���ս����Էḻ��Transformer�����Ϸ��񳡾���΢��ģʽʶ�����Ŀ�Դ����Ϊfast_transformers�Ļ���Intel���CPU��Transformerʵ�֡�fast_transformers�䷢��CPU�ĺ˼䲢�к�ָ�������������֧�ֱ䳤�������д��������˲���Ķ�����㡣fast_transformer�ڶ���CPUӲ���ϻ���˳���pytorch��Ŀǰ�����Ż����棨��onnxruntime-mkldnn ��torch JIT�������ܱ��֡�ʹ��4~8���߳�ʱ��fast_transformer��10~128���ȵ����д��������У�����ѿ�Դ����ʵ��ȡ��ƽ��20%���ϵļ���Ч�������ң����Զ����еĴ����ٶ�������Ϊ������fast_transformers�Ѿ�Ӧ����ģʽʶ�����ĵĶ�����Ϸ�����񳡾���

### ��װ
1.1 ��������docker���������
��������������ONNX-runtimeʱ���ܳ���
```
sh tools/build_docker.sh
docker run -it --rm -v your/path/fast_transformers:/workspace --name=your_container_name REPOSITORY:TAG /bin/bash
cd /workspace
# optional:��װ��Ҫ��������Ѷ���������ô���
export http_proxy=http://devnet-proxy.oa.com:8080
export https_proxy=http://devnet-proxy.oa.com:8080
export no_proxy=git.code.oa.com
```
1.2 ����Ѷ�Ƶ�dockerhub��pull���о���
A. WXG����Ѷ��dockerhub�˺�
```
docker pull ccr.ccs.tencentyun.com/mmspr/fast_transformer:0.1.1a1-dev
```
B. CSIG����Ѷ��dockerhub�˺�
```
docker pull ccr.ccs.tencentyun.com/mmspr/fast_transformer:0.1.1a1-dev
```
2. ��docker�ڰ�װconda��pip��

```
sh conda/build.sh
```

3. ��docker�ڽ��е��� (optional)

```
sh tools/build_run_unittests.sh $PWD
```
4. ��docker������benchmark (optional)

```
cd benchmark
bash run_benchmark.sh
```

### ʹ��ʾ��
Fast_transformers�ṩ�˼򵥵ĵ��ýӿڣ��ṩ����huggingface/transformers [pytorch](https://github.com/huggingface "pytorch")ģ�͵ĵ��÷�ʽ��
�������Ƭ��չʾ����ν�huggingfaceԤѵ��BERTģ�͵���fast_transformer������һ��BERT encoder�ļ��㡣

```python
import torch
import transformers
import contexttimer
import fast_transformers
# ʹ��4���߳�����fast_transformers
fast_transformers.set_num_threads(4)
# ����transformers�ṩ��Ԥѵ��ģ��
model = transformers.BertModel.from_pretrained(
    "bert-base-chinese")
# Ԥѵ��ģ�͵�����
cfg = model.config
#��������ı�����
input_ids = torch.randint(low=0,
                            high=cfg.vocab_size - 1,
                            size=(batch_size, seq_len),
                            dtype=torch.long)
# ����bert-encoder��ģ�ͣ����first��ʽpooling�Ľ��
model = fast_transformers.BertModel.from_torch(model, pooling_type=PoolingType.FIRST)
# ���encoder���
res = model(input_ids)
```
����ʹ�ýӿڿ��Բο� ./benchmark/benchmark.py�ļ�


## ����
����������Ӳ��ƽ̨������fast_transformers�����ܱ��֡�
����ѡ��[pytorch](https://github.com/huggingface "pytorch")��[pytorch-jit](https://pytorch.org/docs/stable/_modules/torch/jit.html "pytorch-jit")��[onnxruntime-mkldnn]( https://github.com/microsoft/onnxruntime "onnxruntime-mkldnn")ʵ����Ϊ�Աȡ����ܲ��Խ��Ϊ����150�εľ�ֵ��Ϊ�˱����β���ʱ���ϴε�����������cache�л��������ÿ�β��Բ���������ݣ����ڼ����ˢ�µ�cache���ݡ�


* Intel Xeon 61xx


��61xx�ϣ�����Transformerʵ�����ܶԱȽ������������ͼ��ʾ�����Թ۲쵽���߳���Ϊ1ʱ������ʵ�ֵĲ�𲢲��������߳������࣬fast_transformers���������������󣬵��߳�Ϊ8ʱ����Ч����Ϊ���ԡ����⣬����seq_length����������fast_transformers�ļ���Ч��������ԭ���Ǵ�ʱGEMM����ʱ��ռ�����󣬺����ںϴ���������١�

<img width="600" height="300" src="http://km.oa.com/files/photos/captures/201912/1575381653_78_w1546_h784.png" alt="61xx����1">
<img width="600" height="300" src="http://km.oa.com/files/photos/captures/201912/1575382049_44_w1676_h845.png" alt="61xx����2">

* Intel Xeon 6133

���61xx�ͺţ�Intel Xeon 6133���������ȸ���Ϊ512 bit��������ӵ��һ��30 MB�˼乲��L3 cache����������ͼչʾ��6133�����ܱ��֡����̵߳Ĵ󲿷�case��fast_transformers�����������ʵ�֡��Ƚ������case�����г���Ϊ10��20������������������������MKL AVX512 GEMM���̵�Ե�ʣ���Intel 6133 CPU�ϣ����Ƿ�������seq_length���ӣ�GEMM������ӳٻ����һ�����������

<img width="600" height="300" src="http://km.oa.com/files/photos/captures/201912/1575384757_71_w1751_h886.png" alt="6133����1">
<img width="600" height="300" src="http://km.oa.com/files/photos/captures/201912/1575385675_63_w1602_h804.png" alt="6133����2">


* intel i9-9800 CPU

��������ͼչʾ��intel i9�ϵ����ܱ��֡����߳�������1ʱ��fast_transformer��������������ʵ�֡�
<img width="600" height="300" src="http://km.oa.com/files/photos/captures/201912/1575425550_58_w2920_h1474.png" alt="6133����1">
<img width="600" height="300" src="http://km.oa.com/files/photos/captures/201912/1575425573_3_w3042_h1534.png" alt="6133����2">
