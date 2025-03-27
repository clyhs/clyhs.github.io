# vllm and ollama

### install
```

docker run -d --gpus=all -v /home/featurize/work/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

conda create -n vllm python=3.12 -y

conda activate vllm

pip install modelscope

modelscope download --model deepseek-ai/DeepSeek-R1-Distill-Qwen-14B --local_dir /home/featurize/data/models/deepseek-dr1-qwen-14b

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

360zhinao/TinyR1-32B-Preview

modelscope download --model 360zhinao/TinyR1-32B-Preview --local_dir /home/featurize/data/models/tinyR1-32b-pre

pip install vllm

vllm serve  /home/featurize/data/models/deepseek-dr1-qwen-14b --port 8000 --max-model-len 65536

vllm serve  /home/featurize/data/models/deepseek-dr1-qwen-14b --trust-remote-code --served-model-name deepseek-dr1-qwen-14b --port 8000 --max-model-len 65536 --api-key sk-123456

vllm serve  /home/featurize/data/models/deepseek-dr1-qwen-7b --served-model-name deepseek-dr1-qwen-7b --port 8000 --max-model-len 65536

vllm serve  /home/featurize/data/models/deepseek-dr1-qwen-14b --served-model-name deepseek-dr1-qwen-14b --port 8000 --max-model-len 65536

deepseek-dr1-llama-70b-q

vllm serve  /home/featurize/data/models/deepseek-dr1-llama-70b-q --served-model-name deepseek-dr1-llama-70b-int4 --port 8000 -tp 3 --tensor-parallel-size 4 --max-model-len 32768

vllm serve  /home/featurize/data/models/tinyR1-32b-pre --served-model-name tinyR1-32b-pre --port 8000 -tp 3 --tensor-parallel-size 4 --max-model-len 32768

vllm serve  deepseek-ai/DeepSeek-R1-Distill-Qwen-32B --served-model-name deepseek-dr1-qwen-32b --port 8000 -tp 3 --tensor-parallel-size 4 --max-model-len 32768

vllm serve  /home/featurize/data/models/deepseek-dr1-llama-70b-bnb-4bit --served-model-name tinyR1-32b-pre --port 8000 -tp 2 --tensor-parallel-size 2 --max-model-len 32768 --enforce-eager --ray_workers_use_nsight --distributed_executor_backend ray

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


subparser='serve', model_tag='/home/featurize/data/models/tinyR1-32b-pre', config='', host=None, port=8000, uvicorn_log_level='info', allow_credentials=False, allowed_origins=['*'], allowed_methods=['*'], allowed_headers=['*'], api_key=None, lora_modules=None, prompt_adapters=None, chat_template=None, chat_template_content_format='auto', response_role='assistant', ssl_keyfile=None, ssl_certfile=None, ssl_ca_certs=None, ssl_cert_reqs=0, root_path=None, middleware=[], return_tokens_as_token_ids=False, disable_frontend_multiprocessing=False, enable_request_id_headers=False, enable_auto_tool_choice=False, enable_reasoning=False, reasoning_parser=None, tool_call_parser=None, tool_parser_plugin='', model='/home/featurize/data/models/tinyR1-32b-pre', task='auto', tokenizer=None, skip_tokenizer_init=False, revision=None, code_revision=None, tokenizer_revision=None, tokenizer_mode='auto', trust_remote_code=False, allowed_local_media_path=None, download_dir=None, load_format='auto', config_format=<ConfigFormat.AUTO: 'auto'>, dtype='auto', kv_cache_dtype='auto', max_model_len=32768, guided_decoding_backend='xgrammar', logits_processor_pattern=None, model_impl='auto', distributed_executor_backend=None, pipeline_parallel_size=1, tensor_parallel_size=4, max_parallel_loading_workers=None, ray_workers_use_nsight=False, block_size=None, enable_prefix_caching=None, disable_sliding_window=False, use_v2_block_manager=True, num_lookahead_slots=0, seed=0, swap_space=4, cpu_offload_gb=0, gpu_memory_utilization=0.9, num_gpu_blocks_override=None, max_num_batched_tokens=None, max_num_partial_prefills=1, max_long_partial_prefills=1, long_prefill_token_threshold=0, max_num_seqs=None, max_logprobs=20, disable_log_stats=False, quantization=None, rope_scaling=None, rope_theta=None, hf_overrides=None, enforce_eager=False, max_seq_len_to_capture=8192, disable_custom_all_reduce=False, tokenizer_pool_size=0, tokenizer_pool_type='ray', tokenizer_pool_extra_config=None, limit_mm_per_prompt=None, mm_processor_kwargs=None, disable_mm_preprocessor_cache=False, enable_lora=False, enable_lora_bias=False, max_loras=1, max_lora_rank=16, lora_extra_vocab_size=256, lora_dtype='auto', long_lora_scaling_factors=None, max_cpu_loras=None, fully_sharded_loras=False, enable_prompt_adapter=False, max_prompt_adapters=1, max_prompt_adapter_token=0, device='auto', num_scheduler_steps=1, multi_step_stream_outputs=True, scheduler_delay_factor=0.0, enable_chunked_prefill=None, speculative_model=None, speculative_model_quantization=None, num_speculative_tokens=None, speculative_disable_mqa_scorer=False, speculative_draft_tensor_parallel_size=None, speculative_max_model_len=None, speculative_disable_by_batch_size=None, ngram_prompt_lookup_max=None, ngram_prompt_lookup_min=None, spec_decoding_acceptance_method='rejection_sampler', typical_acceptance_sampler_posterior_threshold=None, typical_acceptance_sampler_posterior_alpha=None, disable_logprobs_during_spec_decoding=None, model_loader_extra_config=None, ignore_patterns=[], preemption_mode=None, served_model_name=['tinyR1-32b-pre'], qlora_adapter_name_or_path=None, otlp_traces_endpoint=None, collect_detailed_traces=None, disable_async_output_proc=False, scheduling_policy='fcfs', scheduler_cls='vllm.core.scheduler.Scheduler', override_neuron_config=None, override_pooler_config=None, compilation_config=None, kv_transfer_config=None, worker_cls='auto', generation_config=None, override_generation_config=None, enable_sleep_mode=False, calculate_kv_scales=False, additional_config=None, disable_log_requests=False, max_log_len=None, disable_fastapi_docs=False, enable_prompt_tokens_details=False, dispatch_function=<function ServeSubcommand.cmd at 0x7f655196c2c0


--additional-config ADDITIONAL_CONFIG
                        Additional config for specified platform in JSON format. Different platforms may support different configs. Make sure the configs are valid for the platform you are using. The input format is like
                        '{"config_key":"config_value"}'
  --allow-credentials   Allow credentials.
  --allowed-headers ALLOWED_HEADERS
                        Allowed headers.
  --allowed-local-media-path ALLOWED_LOCAL_MEDIA_PATH
                        Allowing API requests to read local images or videos from directories specified by the server file system. This is a security risk. Should only be enabled in trusted environments.
  --allowed-methods ALLOWED_METHODS
                        Allowed methods.
  --allowed-origins ALLOWED_ORIGINS
                        Allowed origins.
  --api-key API_KEY     If provided, the server will require this key to be presented in the header.
  --block-size {8,16,32,64,128}
                        Token block size for contiguous chunks of tokens. This is ignored on neuron devices and set to ``--max-model-len``. On CUDA devices, only block sizes up to 32 are supported. On HPU devices, block
                        size defaults to 128.
  --calculate-kv-scales
                        This enables dynamic calculation of k_scale and v_scale when kv-cache-dtype is fp8. If calculate-kv-scales is false, the scales will be loaded from the model checkpoint if available. Otherwise,
                        the scales will default to 1.0.
  --chat-template CHAT_TEMPLATE
                        The file path to the chat template, or the template in single-line form for the specified model.
  --chat-template-content-format {auto,string,openai}
                        The format to render message content within a chat template. * "string" will render the content as a string. Example: ``"Hello World"`` * "openai" will render the content as a list of
                        dictionaries, similar to OpenAI schema. Example: ``[{"type": "text", "text": "Hello world!"}]``
  --code-revision CODE_REVISION
                        The specific revision to use for the model code on Hugging Face Hub. It can be a branch name, a tag name, or a commit id. If unspecified, will use the default version.
  --collect-detailed-traces COLLECT_DETAILED_TRACES
                        Valid choices are model,worker,all. It makes sense to set this only if ``--otlp-traces-endpoint`` is set. If set, it will collect detailed traces for the specified modules. This involves use of
                        possibly costly and or blocking operations and hence might have a performance impact.
  --compilation-config COMPILATION_CONFIG, -O COMPILATION_CONFIG
                        torch.compile configuration for the model.When it is a number (0, 1, 2, 3), it will be interpreted as the optimization level. NOTE: level 0 is the default level without any optimization. level 1
                        and 2 are for internal testing only. level 3 is the recommended level for production. To specify the full compilation config, use a JSON string. Following the convention of traditional compilers,
                        using -O without space is also supported. -O3 is equivalent to -O 3.
  --config CONFIG       Read CLI options from a config file.Must be a YAML with the following options:https://docs.vllm.ai/en/latest/serving/openai_compatible_server.html#cli-reference
  --config-format {auto,hf,mistral}
                        The format of the model config to load. * "auto" will try to load the config in hf format if available else it will try to load in mistral format
  --cpu-offload-gb CPU_OFFLOAD_GB
                        The space in GiB to offload to CPU, per GPU. Default is 0, which means no offloading. Intuitively, this argument can be seen as a virtual way to increase the GPU memory size. For example, if you
                        have one 24 GB GPU and set this to 10, virtually you can think of it as a 34 GB GPU. Then you can load a 13B model with BF16 weight, which requires at least 26GB GPU memory. Note that this
                        requires fast CPU-GPU interconnect, as part of the model is loaded from CPU memory to GPU memory on the fly in each model forward pass.
  --device {auto,cuda,neuron,cpu,openvino,tpu,xpu,hpu}
                        Device type for vLLM execution.
  --disable-async-output-proc
                        Disable async output processing. This may result in lower performance.
  --disable-custom-all-reduce
                        See ParallelConfig.
  --disable-fastapi-docs
                        Disable FastAPI's OpenAPI schema, Swagger UI, and ReDoc endpoint.
  --disable-frontend-multiprocessing
                        If specified, will run the OpenAI frontend server in the same process as the model serving engine.
  --disable-log-requests
                        Disable logging requests.
  --disable-log-stats   Disable logging statistics.
  --disable-logprobs-during-spec-decoding [DISABLE_LOGPROBS_DURING_SPEC_DECODING]
                        If set to True, token log probabilities are not returned during speculative decoding. If set to False, log probabilities are returned according to the settings in SamplingParams. If not specified,
                        it defaults to True. Disabling log probabilities during speculative decoding reduces latency by skipping logprob calculation in proposal sampling, target sampling, and after accepted tokens are
                        determined.
  --disable-mm-preprocessor-cache
                        If true, then disables caching of the multi-modal preprocessor/mapper. (not recommended)
  --disable-sliding-window
                        Disables sliding window, capping to sliding window size.
  --distributed-executor-backend {ray,mp,uni,external_launcher}
                        Backend to use for distributed model workers, either "ray" or "mp" (multiprocessing). If the product of pipeline_parallel_size and tensor_parallel_size is less than or equal to the number of GPUs
                        available, "mp" will be used to keep processing on a single host. Otherwise, this will default to "ray" if Ray is installed and fail otherwise. Note that tpu only supports Ray for distributed
                        inference.
  --download-dir DOWNLOAD_DIR
                        Directory to download and load the weights, default to the default cache dir of huggingface.
  --dtype {auto,half,float16,bfloat16,float,float32}
                        Data type for model weights and activations. * "auto" will use FP16 precision for FP32 and FP16 models, and BF16 precision for BF16 models. * "half" for FP16. Recommended for AWQ quantization. *
                        "float16" is the same as "half". * "bfloat16" for a balance between precision and range. * "float" is shorthand for FP32 precision. * "float32" for FP32 precision.
  --enable-auto-tool-choice
                        Enable auto tool choice for supported models. Use ``--tool-call-parser`` to specify which parser to use.
  --enable-chunked-prefill [ENABLE_CHUNKED_PREFILL]
                        If set, the prefill requests can be chunked based on the max_num_batched_tokens.
  --enable-lora         If True, enable handling of LoRA adapters.
  --enable-lora-bias    If True, enable bias for LoRA adapters.
  --enable-prefix-caching, --no-enable-prefix-caching
                        Enables automatic prefix caching. Use ``--no-enable-prefix-caching`` to disable explicitly.
  --enable-prompt-adapter
                        If True, enable handling of PromptAdapters.
  --enable-prompt-tokens-details
                        If set to True, enable prompt_tokens_details in usage.
  --enable-reasoning    Whether to enable reasoning_content for the model. If enabled, the model will be able to generate reasoning content.
  --enable-request-id-headers
                        If specified, API server will add X-Request-Id header to responses. Caution: this hurts performance at high QPS.
  --enable-sleep-mode   Enable sleep mode for the engine. (only cuda platform is supported)
  --enforce-eager       Always use eager-mode PyTorch. If False, will use eager mode and CUDA graph in hybrid for maximal performance and flexibility.
  --fully-sharded-loras
                        By default, only half of the LoRA computation is sharded with tensor parallelism. Enabling this will use the fully sharded layers. At high sequence length, max rank or tensor parallel size, this
                        is likely faster.
  --generation-config GENERATION_CONFIG
                        The folder path to the generation config. Defaults to None, no generation config is loaded, vLLM defaults will be used. If set to 'auto', the generation config will be loaded from model path. If
                        set to a folder path, the generation config will be loaded from the specified folder path. If `max_new_tokens` is specified in generation config, then it sets a server-wide limit on the number of
                        output tokens for all requests.
  --gpu-memory-utilization GPU_MEMORY_UTILIZATION
                        The fraction of GPU memory to be used for the model executor, which can range from 0 to 1. For example, a value of 0.5 would imply 50% GPU memory utilization. If unspecified, will use the default
                        value of 0.9. This is a per-instance limit, and only applies to the current vLLM instance.It does not matter if you have another vLLM instance running on the same GPU. For example, if you have two
                        vLLM instances running on the same GPU, you can set the GPU memory utilization to 0.5 for each instance.
  --guided-decoding-backend {outlines,lm-format-enforcer,xgrammar}
                        Which engine will be used for guided decoding (JSON schema / regex etc) by default. Currently support https://github.com/outlines-dev/outlines, https://github.com/mlc-ai/xgrammar, and
                        https://github.com/noamgat/lm-format-enforcer. Can be overridden per request via guided_decoding_backend parameter.
  --hf-overrides HF_OVERRIDES
                        Extra arguments for the HuggingFace config. This should be a JSON string that will be parsed into a dictionary.
  --host HOST           Host name.
  --ignore-patterns IGNORE_PATTERNS
                        The pattern(s) to ignore when loading the model.Default to `original/**/*` to avoid repeated loading of llama's checkpoints.
  --kv-cache-dtype {auto,fp8,fp8_e5m2,fp8_e4m3}
                        Data type for kv cache storage. If "auto", will use model data type. CUDA 11.8+ supports fp8 (=fp8_e4m3) and fp8_e5m2. ROCm (AMD GPU) supports fp8 (=fp8_e4m3)
  --kv-transfer-config KV_TRANSFER_CONFIG
                        The configurations for distributed KV cache transfer. Should be a JSON string.
  --limit-mm-per-prompt LIMIT_MM_PER_PROMPT
                        For each multimodal plugin, limit how many input instances to allow for each prompt. Expects a comma-separated list of items, e.g.: `image=16,video=2` allows a maximum of 16 images and 2 videos
                        per prompt. Defaults to 1 for each modality.
  --load-format {auto,pt,safetensors,npcache,dummy,tensorizer,sharded_state,gguf,bitsandbytes,mistral,runai_streamer}
                        The format of the model weights to load. * "auto" will try to load the weights in the safetensors format and fall back to the pytorch bin format if safetensors format is not available. * "pt" will
                        load the weights in the pytorch bin format. * "safetensors" will load the weights in the safetensors format. * "npcache" will load the weights in pytorch format and store a numpy cache to speed up
                        the loading. * "dummy" will initialize the weights with random values, which is mainly for profiling. * "tensorizer" will load the weights using tensorizer from CoreWeave. See the Tensorize vLLM
                        Model script in the Examples section for more information. * "runai_streamer" will load the Safetensors weights using Run:aiModel Streamer * "bitsandbytes" will load the weights using bitsandbytes
                        quantization.
  --logits-processor-pattern LOGITS_PROCESSOR_PATTERN
                        Optional regex pattern specifying valid logits processor qualified names that can be passed with the `logits_processors` extra completion argument. Defaults to None, which allows no processors.
  --long-lora-scaling-factors LONG_LORA_SCALING_FACTORS
                        Specify multiple scaling factors (which can be different from base model scaling factor - see eg. Long LoRA) to allow for multiple LoRA adapters trained with those scaling factors to be used at
                        the same time. If not specified, only adapters trained with the base model scaling factor are allowed.
  --long-prefill-token-threshold LONG_PREFILL_TOKEN_THRESHOLD
                        For chunked prefill, a request is considered long if the prompt is longer than this number of tokens. Defaults to 4% of the model's context length.
  --lora-dtype {auto,float16,bfloat16}
                        Data type for LoRA. If auto, will default to base model dtype.
  --lora-extra-vocab-size LORA_EXTRA_VOCAB_SIZE
                        Maximum size of extra vocabulary that can be present in a LoRA adapter (added to the base model vocabulary).
  --lora-modules LORA_MODULES [LORA_MODULES ...]
                        LoRA module configurations in either 'name=path' formator JSON format. Example (old format): ``'name=path'`` Example (new format): ``{"name": "name", "path": "lora_path", "base_model_name":
                        "id"}``
  --max-cpu-loras MAX_CPU_LORAS
                        Maximum number of LoRAs to store in CPU memory. Must be >= than max_loras. Defaults to max_loras.
  --max-log-len MAX_LOG_LEN
                        Max number of prompt characters or prompt ID numbers being printed in log. Default: Unlimited
  --max-logprobs MAX_LOGPROBS
                        Max number of log probs to return logprobs is specified in SamplingParams.
  --max-long-partial-prefills MAX_LONG_PARTIAL_PREFILLS
                        For chunked prefill, the maximum number of prompts longer than --long-prefill-token-threshold that will be prefilled concurrently. Setting this less than --max-num-partial-prefills will allow
                        shorter prompts to jump the queue in front of longer prompts in some cases, improving latency. Defaults to 1.
  --max-lora-rank MAX_LORA_RANK
                        Max LoRA rank.
  --max-loras MAX_LORAS
                        Max number of LoRAs in a single batch.
  --max-model-len MAX_MODEL_LEN
                        Model context length. If unspecified, will be automatically derived from the model config.
  --max-num-batched-tokens MAX_NUM_BATCHED_TOKENS
                        Maximum number of batched tokens per iteration.
  --max-num-partial-prefills MAX_NUM_PARTIAL_PREFILLS
                        For chunked prefill, the max number of concurrent partial prefills.Defaults to 1
  --max-num-seqs MAX_NUM_SEQS
                        Maximum number of sequences per iteration.
  --max-parallel-loading-workers MAX_PARALLEL_LOADING_WORKERS
                        Load model sequentially in multiple batches, to avoid RAM OOM when using tensor parallel and large models.
  --max-prompt-adapter-token MAX_PROMPT_ADAPTER_TOKEN
                        Max number of PromptAdapters tokens
  --max-prompt-adapters MAX_PROMPT_ADAPTERS
                        Max number of PromptAdapters in a batch.
  --max-seq-len-to-capture MAX_SEQ_LEN_TO_CAPTURE
                        Maximum sequence length covered by CUDA graphs. When a sequence has context length larger than this, we fall back to eager mode. Additionally for encoder-decoder models, if the sequence length of
                        the encoder input is larger than this, we fall back to the eager mode.
  --middleware MIDDLEWARE
                        Additional ASGI middleware to apply to the app. We accept multiple --middleware arguments. The value should be an import path. If a function is provided, vLLM will add it to the server using
                        ``@app.middleware('http')``. If a class is provided, vLLM will add it to the server using ``app.add_middleware()``.
  --mm-processor-kwargs MM_PROCESSOR_KWARGS
                        Overrides for the multimodal input mapping/processing, e.g., image processor. For example: ``{"num_crops": 4}``.
  --model MODEL         Name or path of the huggingface model to use.
  --model-impl {auto,vllm,transformers}
                        Which implementation of the model to use. * "auto" will try to use the vLLM implementation if it exists and fall back to the Transformers implementation if no vLLM implementation is available. *
                        "vllm" will use the vLLM model implementation. * "transformers" will use the Transformers model implementation.
  --model-loader-extra-config MODEL_LOADER_EXTRA_CONFIG
                        Extra config for model loader. This will be passed to the model loader corresponding to the chosen load_format. This should be a JSON string that will be parsed into a dictionary.
  --multi-step-stream-outputs [MULTI_STEP_STREAM_OUTPUTS]
                        If False, then multi-step will stream outputs at the end of all steps
  --ngram-prompt-lookup-max NGRAM_PROMPT_LOOKUP_MAX
                        Max size of window for ngram prompt lookup in speculative decoding.
  --ngram-prompt-lookup-min NGRAM_PROMPT_LOOKUP_MIN
                        Min size of window for ngram prompt lookup in speculative decoding.
  --num-gpu-blocks-override NUM_GPU_BLOCKS_OVERRIDE
                        If specified, ignore GPU profiling result and use this number of GPU blocks. Used for testing preemption.
  --num-lookahead-slots NUM_LOOKAHEAD_SLOTS
                        Experimental scheduling config necessary for speculative decoding. This will be replaced by speculative config in the future; it is present to enable correctness tests until then.
  --num-scheduler-steps NUM_SCHEDULER_STEPS
                        Maximum number of forward steps per scheduler call.
  --num-speculative-tokens NUM_SPECULATIVE_TOKENS
                        The number of speculative tokens to sample from the draft model in speculative decoding.
  --otlp-traces-endpoint OTLP_TRACES_ENDPOINT
                        Target URL to which OpenTelemetry traces will be sent.
  --override-generation-config OVERRIDE_GENERATION_CONFIG
                        Overrides or sets generation config in JSON format. e.g. ``{"temperature": 0.5}``. If used with --generation-config=auto, the override parameters will be merged with the default config from the
                        model. If generation-config is None, only the override parameters are used.
  --override-neuron-config OVERRIDE_NEURON_CONFIG
                        Override or set neuron device configuration. e.g. ``{"cast_logits_dtype": "bloat16"}``.
  --override-pooler-config OVERRIDE_POOLER_CONFIG
                        Override or set the pooling method for pooling models. e.g. ``{"pooling_type": "mean", "normalize": false}``.
  --pipeline-parallel-size PIPELINE_PARALLEL_SIZE, -pp PIPELINE_PARALLEL_SIZE
                        Number of pipeline stages.
  --port PORT           Port number.
  --preemption-mode PREEMPTION_MODE
                        If 'recompute', the engine performs preemption by recomputing; If 'swap', the engine performs preemption by block swapping.
  --prompt-adapters PROMPT_ADAPTERS [PROMPT_ADAPTERS ...]
                        Prompt adapter configurations in the format name=path. Multiple adapters can be specified.
  --qlora-adapter-name-or-path QLORA_ADAPTER_NAME_OR_PATH
                        Name or path of the QLoRA adapter.
  --quantization {aqlm,awq,deepspeedfp,tpu_int8,fp8,ptpc_fp8,fbgemm_fp8,modelopt,marlin,gguf,gptq_marlin_24,gptq_marlin,awq_marlin,gptq,compressed-tensors,bitsandbytes,qqq,hqq,experts_int8,neuron_quant,ipex,quark,moe_wna16,None}, -q {aqlm,awq,deepspeedfp,tpu_int8,fp8,ptpc_fp8,fbgemm_fp8,modelopt,marlin,gguf,gptq_marlin_24,gptq_marlin,awq_marlin,gptq,compressed-tensors,bitsandbytes,qqq,hqq,experts_int8,neuron_quant,ipex,quark,moe_wna16,None}
                        Method used to quantize the weights. If None, we first check the `quantization_config` attribute in the model config file. If that is None, we assume the model weights are not quantized and use
                        `dtype` to determine the data type of the weights.
  --ray-workers-use-nsight
                        If specified, use nsight to profile Ray workers.
  --reasoning-parser {deepseek_r1}
                        Select the reasoning parser depending on the model that you're using. This is used to parse the reasoning content into OpenAI API format. Required for ``--enable-reasoning``.
  --response-role RESPONSE_ROLE
                        The role name to return if ``request.add_generation_prompt=true``.
  --return-tokens-as-token-ids
                        When ``--max-logprobs`` is specified, represents single tokens as strings of the form 'token_id:{token_id}' so that tokens that are not JSON-encodable can be identified.
  --revision REVISION   The specific model version to use. It can be a branch name, a tag name, or a commit id. If unspecified, will use the default version.
  --root-path ROOT_PATH
                        FastAPI root_path when app is behind a path based routing proxy.
  --rope-scaling ROPE_SCALING
                        RoPE scaling configuration in JSON format. For example, ``{"rope_type":"dynamic","factor":2.0}``
  --rope-theta ROPE_THETA
                        RoPE theta. Use with `rope_scaling`. In some cases, changing the RoPE theta improves the performance of the scaled model.
  --scheduler-cls SCHEDULER_CLS
                        The scheduler class to use. "vllm.core.scheduler.Scheduler" is the default scheduler. Can be a class directly or the path to a class of form "mod.custom_class".
  --scheduler-delay-factor SCHEDULER_DELAY_FACTOR
                        Apply a delay (of delay factor multiplied by previous prompt latency) before scheduling next prompt.
  --scheduling-policy {fcfs,priority}
                        The scheduling policy to use. "fcfs" (first come first served, i.e. requests are handled in order of arrival; default) or "priority" (requests are handled based on given priority (lower value
                        means earlier handling) and time of arrival deciding any ties).
  --seed SEED           Random seed for operations.
  --served-model-name SERVED_MODEL_NAME [SERVED_MODEL_NAME ...]
                        The model name(s) used in the API. If multiple names are provided, the server will respond to any of the provided names. The model name in the model field of a response will be the first name in
                        this list. If not specified, the model name will be the same as the ``--model`` argument. Noted that this name(s) will also be used in `model_name` tag content of prometheus metrics, if multiple
                        names provided, metrics tag will take the first one.
  --skip-tokenizer-init
                        Skip initialization of tokenizer and detokenizer.
  --spec-decoding-acceptance-method {rejection_sampler,typical_acceptance_sampler}
                        Specify the acceptance method to use during draft token verification in speculative decoding. Two types of acceptance routines are supported: 1) RejectionSampler which does not allow changing the
                        acceptance rate of draft tokens, 2) TypicalAcceptanceSampler which is configurable, allowing for a higher acceptance rate at the cost of lower quality, and vice versa.
  --speculative-disable-by-batch-size SPECULATIVE_DISABLE_BY_BATCH_SIZE
                        Disable speculative decoding for new incoming requests if the number of enqueue requests is larger than this value.
  --speculative-disable-mqa-scorer
                        If set to True, the MQA scorer will be disabled in speculative and fall back to batch expansion
  --speculative-draft-tensor-parallel-size SPECULATIVE_DRAFT_TENSOR_PARALLEL_SIZE, -spec-draft-tp SPECULATIVE_DRAFT_TENSOR_PARALLEL_SIZE
                        Number of tensor parallel replicas for the draft model in speculative decoding.
  --speculative-max-model-len SPECULATIVE_MAX_MODEL_LEN
                        The maximum sequence length supported by the draft model. Sequences over this length will skip speculation.
  --speculative-model SPECULATIVE_MODEL
                        The name of the draft model to be used in speculative decoding.
  --speculative-model-quantization {aqlm,awq,deepspeedfp,tpu_int8,fp8,ptpc_fp8,fbgemm_fp8,modelopt,marlin,gguf,gptq_marlin_24,gptq_marlin,awq_marlin,gptq,compressed-tensors,bitsandbytes,qqq,hqq,experts_int8,neuron_quant,ipex,quark,moe_wna16,None}
                        Method used to quantize the weights of speculative model. If None, we first check the `quantization_config` attribute in the model config file. If that is None, we assume the model weights are not
                        quantized and use `dtype` to determine the data type of the weights.
  --ssl-ca-certs SSL_CA_CERTS
                        The CA certificates file.
  --ssl-cert-reqs SSL_CERT_REQS
                        Whether client certificate is required (see stdlib ssl module's).
  --ssl-certfile SSL_CERTFILE
                        The file path to the SSL cert file.
  --ssl-keyfile SSL_KEYFILE
                        The file path to the SSL key file.
  --swap-space SWAP_SPACE
                        CPU swap space size (GiB) per GPU.
  --task {auto,generate,embedding,embed,classify,score,reward,transcription}
                        The task to use the model for. Each vLLM instance only supports one task, even if the same model can be used for multiple tasks. When the model only supports one task, ``"auto"`` can be used to
                        select it; otherwise, you must specify explicitly which task to use.
  --tensor-parallel-size TENSOR_PARALLEL_SIZE, -tp TENSOR_PARALLEL_SIZE
                        Number of tensor parallel replicas.
  --tokenizer TOKENIZER
                        Name or path of the huggingface tokenizer to use. If unspecified, model name or path will be used.
  --tokenizer-mode {auto,slow,mistral,custom}
                        The tokenizer mode. * "auto" will use the fast tokenizer if available. * "slow" will always use the slow tokenizer. * "mistral" will always use the `mistral_common` tokenizer. * "custom" will use
                        --tokenizer to select the preregistered tokenizer.
  --tokenizer-pool-extra-config TOKENIZER_POOL_EXTRA_CONFIG
                        Extra config for tokenizer pool. This should be a JSON string that will be parsed into a dictionary. Ignored if tokenizer_pool_size is 0.
  --tokenizer-pool-size TOKENIZER_POOL_SIZE
                        Size of tokenizer pool to use for asynchronous tokenization. If 0, will use synchronous tokenization.
  --tokenizer-pool-type TOKENIZER_POOL_TYPE
                        Type of tokenizer pool to use for asynchronous tokenization. Ignored if tokenizer_pool_size is 0.
  --tokenizer-revision TOKENIZER_REVISION
                        Revision of the huggingface tokenizer to use. It can be a branch name, a tag name, or a commit id. If unspecified, will use the default version.
  --tool-call-parser {granite-20b-fc,granite,hermes,internlm,jamba,llama3_json,mistral,pythonic} or name registered in --tool-parser-plugin
                        Select the tool call parser depending on the model that you're using. This is used to parse the model-generated tool call into OpenAI API format. Required for ``--enable-auto-tool-choice``.
  --tool-parser-plugin TOOL_PARSER_PLUGIN
                        Special the tool parser plugin write to parse the model-generated tool into OpenAI API format, the name register in this plugin can be used in ``--tool-call-parser``.
  --trust-remote-code   Trust remote code from huggingface.
  --typical-acceptance-sampler-posterior-alpha TYPICAL_ACCEPTANCE_SAMPLER_POSTERIOR_ALPHA
                        A scaling factor for the entropy-based threshold for token acceptance in the TypicalAcceptanceSampler. Typically defaults to sqrt of --typical-acceptance-sampler-posterior-threshold i.e. 0.3
  --typical-acceptance-sampler-posterior-threshold TYPICAL_ACCEPTANCE_SAMPLER_POSTERIOR_THRESHOLD
                        Set the lower bound threshold for the posterior probability of a token to be accepted. This threshold is used by the TypicalAcceptanceSampler to make sampling decisions during speculative
                        decoding. Defaults to 0.09
  --use-v2-block-manager
                        [DEPRECATED] block manager v1 has been removed and SelfAttnBlockSpaceManager (i.e. block manager v2) is now the default. Setting this flag to True or False has no effect on vLLM behavior.
  --uvicorn-log-level {debug,info,warning,error,critical,trace}
                        Log level for uvicorn.
  --worker-cls WORKER_CLS
                        The worker class to use for distributed execution.