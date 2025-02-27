# vllm and ollama

### install
```

docker run -d --gpus=all -v /home/featurize/work/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

conda create -n vllm python=3.12 -y

conda activate vllm

pip install modelscope

modelscope download --model deepseek-ai/DeepSeek-R1-Distill-Qwen-32B --local_dir /home/featurize/data/models/deepseek-dr1-qwen-32b

modelscope download --model mlx-community/DeepSeek-R1-Distill-Llama-70B-4bit --local_dir /home/featurize/data/models/deepseek-dr1-llama-70b-4bit

modelscope download --model unsloth/DeepSeek-R1-Distill-Llama-70B-bnb-4bit --local_dir /home/featurize/data/models/deepseek-dr1-llama-70b-bnb-4bit

modelscope download --model unsloth/DeepSeek-R1-Distill-Llama-70B-unsloth-bnb-4bit --local_dir /home/featurize/data/models/deepseek-dr1-llama-70b-ubnb-4bit



unsloth/DeepSeek-R1-Distill-Qwen-32B-bnb-4bit

modelscope download --model unsloth/DeepSeek-R1-Distill-Qwen-32B-bnb-4bit --local_dir /home/featurize/data/models/deepseek-dr1-qwen-32b-bnb-4bit

mlx-community/DeepSeek-R1-Distill-Qwen-32B-4bit

modelscope download --model mlx-community/DeepSeek-R1-Distill-Qwen-32B-4bit --local_dir /home/featurize/data/models/deepseek-dr1-qwen-32b-4bit

okwinds/DeepSeek-R1-Distill-Qwen-32B-Int4-W4A16

modelscope download --model okwinds/DeepSeek-R1-Distill-Qwen-32B-Int4-W4A16 --local_dir /home/featurize/data/models/deepseek-dr1-qwen-32b-int4


neuralmagic/DeepSeek-R1-Distill-Llama-70B-quantized.w4a16

modelscope download --model neuralmagic/DeepSeek-R1-Distill-Llama-70B-quantized.w4a16 --local_dir /home/featurize/data/models/deepseek-dr1-llama-70b-q

pip install vllm

vllm serve  /home/featurize/data/models/deepseek-dr1-qwen-14b --port 8000 --max-model-len 65536

vllm serve  /home/featurize/data/models/deepseek-dr1-qwen-14b --trust-remote-code --served-model-name deepseek-dr1-qwen-14b --port 8000 --max-model-len 65536 --api-key sk-123456

vllm serve  /home/featurize/data/models/deepseek-dr1-qwen-7b --served-model-name deepseek-dr1-qwen-7b --port 8000 --max-model-len 65536

vllm serve  /home/featurize/data/models/deepseek-dr1-qwen-14b --served-model-name deepseek-dr1-qwen-14b --port 8000 --max-model-len 65536

vllm serve  /model/HuggingFace/deepseek-ai/DeepSeek-R1-Distill-Qwen-14B --port 8000 -tp 2 --max-model-len 59968 #两卡

vllm serve  /home/featurize/data/models/deepseek-dr1-llama-70b-bnb-4bit --served-model-name deepseek-dr1-llama-70b-bnb-4bit --port 8000 -tp 2 --tensor-parallel-size 2 --max-model-len 32768 --enforce-eager --ray_workers_use_nsight --distributed_executor_backend ray

ray_workers_use_nsight=True, distributed_executor_backend="ray"
```

本地用openai库请求大模型

openai库在用于请求本地大模型时不需要apikey，该方法能够比较方便的实现流式输出。首先需要在服务器端输入命令“ollama run qwen2.5:3b”启动大模型服务。然后需要在本地下载openai库：

pip install openai 
或 
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ openai
之后就可以在本地运行如下python代码：
```
from openai import OpenAI
from IPython.display import display, clear_output 
import time

client = OpenAI(
 base_url='http://10.xxx.xx.xx:11434/v1', # 填自己服务器的URL
 api_key='111',  # 随便写一个，不能为空
)

completion = client.chat.completions.create(
 messages=[
        {
 'role': 'user',
 'content': '请用一句话做一个简短的自我介绍',
        }
    ],
 model='qwen2.5:3b',
 stream=True # add this line to enable streaming output
)

# 设置流式响应

ans = ""
for chunk in completion:
    print(chunk.choices[0].delta.content, end='', flush=True)
    ans += chunk.choices[0].delta.content
    time.sleep(0.1)
time2 = time.time()
time_cost = time2 - time1
print("推理速度：", len(ans)/time_cost)
print(f'运行时间：{time_cost}')
```