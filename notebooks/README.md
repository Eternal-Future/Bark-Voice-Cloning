# ChatGLM2微调指南 💡

## 1. 环境搭建
### 安装依赖
```
git clone https://github.com/THUDM/ChatGLM2-6B
cd ChatGLM2-6B
pip install -r requirements.txt
pip install rouge_chinese nltk jieba datasets
```
### 下载模型
```
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
sudo apt install build-essential
git clone https://huggingface.co/THUDM/chatglm2-6b
```

## 2. 准备数据集

自建数据集请参考[train.json](https://github.com/KevinWang676/Bark-Voice-Cloning/blob/main/notebooks/train.json)及以下格式：
```json lines
{"prompt": "长城h3风扇不转。继电器好的。保险丝好的传感器新的风扇也新的这是为什么。就是继电器缺一个信号线", "response": "用电脑能读数据流吗？水温多少", "history": []}
{"prompt": "95", "response": "上下水管温差怎么样啊？空气是不是都排干净了呢？", "history": [["长城h3风扇不转。继电器好的。保险丝好的传感器新的风扇也新的这是为什么。就是继电器缺一个信号线", "用电脑能读数据流吗？水温多少"]]}
{"prompt": "是的。上下水管都好的", "response": "那就要检查线路了，一般风扇继电器是由电脑控制吸合的，如果电路存在断路，或者电脑坏了的话会出现继电器不吸合的情况！", "history": [["长城h3风扇不转。继电器好的。保险丝好的传感器新的风扇也新的这是为什么。就是继电器缺一个信号线", "用电脑能读数据流吗？水温多少"], ["95", "上下水管温差怎么样啊？空气是不是都排干净了呢？"]]}
```
分别准备训练数据集 `train.json` 和验证数据集 `dev.json` 并将其上传至 `ChatGLM2-6B` 文件夹下

## 3. 开始训练

在终端运行以下指令，即可开始训练
```shell
bash train_chat.sh
```

**注意**：原 `train_chat.sh` 文件中包含以下代码：
```
PRE_SEQ_LEN=128
LR=1e-2
NUM_GPUS=1

torchrun --standalone --nnodes=1 --nproc-per-node=$NUM_GPUS main.py \
    --do_train \
    --train_file $CHAT_TRAIN_DATA \
    --validation_file $CHAT_VAL_DATA \
    --preprocessing_num_workers 10 \
    --prompt_column prompt \
    --response_column response \
    --history_column history \
    --overwrite_cache \
    --model_name_or_path THUDM/chatglm2-6b \
    --output_dir $CHECKPOINT_NAME \
    --overwrite_output_dir \
    --max_source_length 256 \
    --max_target_length 256 \
    --per_device_train_batch_size 1 \
    --per_device_eval_batch_size 1 \
    --gradient_accumulation_steps 16 \
    --predict_with_generate \
    --max_steps 3000 \
    --logging_steps 10 \
    --save_steps 1000 \
    --learning_rate $LR \
    --pre_seq_len $PRE_SEQ_LEN \
    --quantization_bit 4
```
**在开始训练前，需要将其编辑为以下示例代码**：
```
PRE_SEQ_LEN=128
LR=1e-2
NUM_GPUS=1

torchrun --standalone --nnodes=1 --nproc-per-node=$NUM_GPUS ptuning/main.py \
    --do_train \
    --train_file train.json \
    --validation_file dev.json \
    --preprocessing_num_workers 10 \
    --prompt_column prompt \
    --response_column response \
    --history_column history \
    --overwrite_cache \
    --model_name_or_path chatglm2-6b \
    --output_dir output_model \
    --overwrite_output_dir \
    --max_source_length 1024 \
    --max_target_length 1024 \
    --per_device_train_batch_size 1 \
    --per_device_eval_batch_size 1 \
    --gradient_accumulation_steps 16 \
    --predict_with_generate \
    --max_steps 600 \
    --logging_steps 10 \
    --save_steps 100 \
    --learning_rate $LR \
    --pre_seq_len $PRE_SEQ_LEN
```

P.S. 以上的 `train_chat.sh` 文件只是一个示例，具体参数设置请根据不同GPU的性能进行调节；ChatGLM2微调[官方教程](https://github.com/THUDM/ChatGLM2-6B/tree/main/ptuning)

# ChatGLM3微调指南 📒

## 1. 环境搭建
### 安装依赖
```
git clone https://github.com/THUDM/ChatGLM3
cd ChatGLM3
pip install -r requirements.txt
pip install transformers==4.34.0
apt install nvidia-cuda-toolkit
cd finetune_chatmodel_demo
pip install -r requirements.txt
cd ..
```
### 下载模型
```
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
sudo apt install build-essential
git clone https://huggingface.co/THUDM/chatglm3-6b
```
## 2. 准备数据集

自建数据集请参考[train.json](https://github.com/KevinWang676/Bark-Voice-Cloning/blob/main/notebooks/train.json)及以下格式：
```json
[
  {
    "conversations": [
      {
        "role": "system",
        "content": "<system prompt text>"
      },
      {
        "role": "user",
        "content": "<user prompt text>"
      },
      {
        "role": "assistant",
        "content": "<assistant response text>"
      }, 
       // ... Muti Turn
      {
        "role": "user",
        "content": "<user prompt text>"
      },
      {
        "role": "assistant",
        "content": "<assistant response text>"
      }
    ]
  }
  // ...
]
```


分别准备训练数据集 `train.json` 和验证数据集 `dev.json` 并将其上传至 `ChatGLM3` 文件夹下

