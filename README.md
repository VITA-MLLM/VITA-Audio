# VITA-Audio: Fast Interleaved Audio-Text Token Generation for Efficient Large Speech-Language Model

<p align="center">
    <img src="asset/VITA_audio_logos.png" width="50%" height="50%">
</p>

<p align="center">
    <a href="https://arxiv.org/abs/2502.05177" target="_blank"><img src="https://img.shields.io/badge/VITA%20Audio-Report-b5212f.svg?logo=arxiv" /></a>
    <a href="https://huggingface.co/collections/VITA-MLLM/vita-audio-680f036c174441e7cdf02575" target="_blank"><img src="https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Model-ffc107?color=ffc107&logoColor=white" /></a>
 </p>


## :fire: News



* **`2025.05.06`** 🌟 We are proud to launch VITA-Audio, an end-to-end large speech model with fast audio-text token generation.


## Contents <!-- omit in toc -->


- [Highlights](#-highlights)
- [Experimental Results](#-experimental-results)
- [Models](#-models)
- [Training](#-training)


## ✨ Highlights

- **Low Latency**. VITA-Audio is the first end-to-end speech model capable of generating audio during the initial forward pass. By utilizing a set of 32 prefill tokens, VITA-Audio reduces the time required to generate the first audio token chunk from 217 ms to just 47 ms.
- **Fast Inference**. VITA-Audio achieves an inference speedup of 3-5x at the 7B parameter scale.
- **Open Source**. VITA-Audio is trained on **open-source data** only, consisting of 200k hours of publicly available audio.
- **Strong Performance**. VITA-Audio achieves competitive results on ASR,TTS and SQA benchmarks among cutting-edge models under 7B parameters.
  


## 📌 Exhibition

### Inference 加速 
不同inferen mode下的模型推理速度

<p align="center">
  <img src="./asset/qa_speed.gif" alt="demogif" width="48%" style="display: inline-block; margin-right: 2%;">
  <img src="./asset/tts_speed.gif" alt="second_gif" width="48%" style="display: inline-block;">
</p>

### 首个audio片段的生成时间
<img width="873" alt="企业微信截图_2a021cb3-cadd-4d9f-8e88-2e02540da46b" src="https://github.com/user-attachments/assets/165f943e-ac53-443f-abba-e5eb1e0c0f40" />


### TTS case

prompt：

https://github.com/user-attachments/assets/d3da7129-168f-4c07-9c13-a319eda0c276



### Spoken QA case



https://github.com/user-attachments/assets/d3da7129-168f-4c07-9c13-a319eda0c276


## 🔔 Models

| Model                   | LLM Size | Huggingface Weights                                           |
|-------------------------|----------|---------------------------------------------------------------|
| VITA-Audio-Boost        | 7B       | https://huggingface.co/VITA-MLLM/VITA-Audio-Boost             |
| VITA-Audio-Balance      | 7B       | https://huggingface.co/VITA-MLLM/VITA-Audio-Balance           |
| VITA-Audio-Plus-Vanilla | 7B       | https://huggingface.co/VITA-MLLM/VITA-Audio-Plus-Vanilla      |



## 📈 Experimental Results
- **Comparison of Spoken Question Answering**.

![Clipboard_Screenshot_1746531780](https://github.com/user-attachments/assets/3adcad15-0333-4b92-bfdf-b753b330a3e2)


- **Comparison of Text to Speech**.

![Clipboard_Screenshot_1746532132](https://github.com/user-attachments/assets/a74c60af-be80-4872-9f1f-a9527dcaeb92)


- **Comparison of Automatic Speech Recognition**.

![Clipboard_Screenshot_1746532039](https://github.com/user-attachments/assets/d950cae0-c065-4da9-b37a-a471d28158a0)

![Clipboard_Screenshot_1746532022](https://github.com/user-attachments/assets/929f45cd-693a-4ff6-af73-ceec6e875706)



- **Effectiveness of Inference Acceleration**.


![Clipboard_Screenshot_1746532167](https://github.com/user-attachments/assets/ad8b9e90-cd3c-4968-8653-998811a50006)

![Image](https://github.com/user-attachments/assets/4aa5db8c-362d-4152-8090-92292b9a84c0)



## Requirements and Installation

### Prepare Environment
```
docker pull shenyunhang/pytorch:24.11-py3_2024-1224
```

### Get the Code
```
git clone https://github.com/VITA-MLLM/VITA-Audio.git
cd VITA-Audio
pip install -r requirements_ds_gpu.txt
pip install -e .
```

### Prepare Pre-trained Weight

#### LLM

- Download the LLM from https://huggingface.co/Qwen/Qwen2.5-7B-Instruct.
- Put it into '../models/Qwen/Qwen2.5-7B-Instruct/'

#### Audio Encoder and Audio Decoder

- Download the Audio Encoder from https://huggingface.co/THUDM/glm-4-voice-tokenizer.
- Put it into '../models/THUDM/glm-4-voice-tokenizer'

- Download the Audio Decoder from https://huggingface.co/THUDM/glm-4-voice-decoder.
- Put it into '../models/THUDM/glm-4-voice-decoder'


## Training

The following tutorial will take `VITA-Audio-Boost` as an example.

- To train `VITA-Audio-Balance` and other variants, you should modify the `text-audio-interval-ratio`.

  VITA-Audio-Boost:
  ```
  --text-audio-interval-ratio 1 10 4 10 \
  ```

  VITA-Audio-Balance:
  ```
  --text-audio-interval-ratio 1 4 3 8 4 10 \
  ```

- To train `VITA-Audio-Plus-*`, you should use the script like `scripts/deepspeed/sts_qwen25/finetune_sensevoice_glm4voice...`

### Stage-1 (Audio-Text Alignment)

```
bash scripts/deepspeed/sts_qwen25/finetune_glm4voice_stage1.sh 8192 `date +'%Y%m%d_%H%M%S'`
```

The above script may need some adjustments.

- Set `ROOT_PATH` to your code root folder.
- Set `LOCAL_ROOT_PATH` to a temporary code root folder.
- Modify other variables as needed for your environment.

### Stage-2 (Single MCTP Module Training)

```
bash scripts/deepspeed/sts_qwen25/finetune_glm4voice_mtp1_stage1.sh 8192 `date +'%Y%m%d_%H%M%S'`
```

The above script may need some adjustments.

- Set `ROOT_PATH` to your code root folder.
- Set `LOCAL_ROOT_PATH` to a temporary code root folder.
- Set `MODEL_NAME_OR_PATH` to the path of the model trained in Stage 1.
- Modify other variables as needed for your environment.

### Stage-3 (Multiple MCTP Modules Training)

```
bash scripts/deepspeed/sts_qwen25/finetune_glm4voice_mtp10_stage1.sh 8192 `date +'%Y%m%d_%H%M%S'`
```

The above script may need some adjustments.

- Set `ROOT_PATH` to your code root folder.
- Set `LOCAL_ROOT_PATH` to a temporary code root folder.
- Set `MODEL_NAME_OR_PATH` to the path of the model trained in Stage 2.
- Modify other variables as needed for your environment.

### Stage-4 (Supervised Fine-tuning)

```
bash scripts/deepspeed/sts_qwen25/finetune_glm4voice_mtp10_stage2.sh 2048 `date +'%Y%m%d_%H%M%S'`
```

The above script may need some adjustments.

- Set `ROOT_PATH` to your code root folder.
- Set `LOCAL_ROOT_PATH` to a temporary code root folder.
- Set `MODEL_NAME_OR_PATH` to the path of the model trained in Stage 3.
- Modify other variables as needed for your environment.



## 📐Inference

Here we implement a simple script for inference.

It includes examples of speech-to-speech, ASR, and TTS tasks, as well as inference speed testing.

```
python tools/inference_sts.py
```

- Set `model_name_or_path` to VITA-Audio weights.
- Set `audio_tokenizer_path` to the path of the audio encoder.
- Set `flow_path` to the path of the audio decoder.


## Evaluation

Evaluate SQA, ASR, and TTS benchmarks
```
bash scripts/deepspeed/evaluate_sts.sh
```


