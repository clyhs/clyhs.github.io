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


 Must be one of ['aqlm', 'awq', 'deepspeedfp', 'tpu_int8', 'fp8', 'ptpc_fp8', 'fbgemm_fp8', 'modelopt', 'marlin', 'gguf', 'gptq_marlin_24', 'gptq_marlin', 'awq_marlin', 'gptq', 'compressed-tensors', 'bitsandbytes', 'qqq', 'hqq', 'experts_int8', 'neuron_quant', 'ipex', 'quark', 'moe_wna16'].
INFO 02-27 01:29:39 config.py:549] This model supports multiple tasks: {'generate', 'classify', 'embed', 'reward', 'score'}. Defaulting to 'generate'.
ERROR 02-27 01:29:40 engine.py:400] Unknown quantization method: . Must be one of ['aqlm', 'awq', 'deepspeedfp', 'tpu_int8', 'fp8', 'ptpc_fp8', 'fbgemm_fp8', 'modelopt', 'marlin', 'gguf', 'gptq_marlin_24', 'gptq_marlin', 'awq_marlin', 'gptq', 'compressed-tensors', 'bitsandbytes', 'qqq', 'hqq', 'experts_int8', 'neuron_quant', 'ipex', 'quark', 'moe_wna16'].
"quantization_config": {
        "group_size": 64,
        "bits": 4
    },
	
	
	export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
	export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
	
args: Namespace(subparser='serve', model_tag='/home/featurize/data/models/deepseek-dr1-qwen-32b-bnb-4bit', config='', host=None, port=8000, uvicorn_log_level='info', allow_credentials=False, allowed_origins=['*'], allowed_methods=['*'], allowed_headers=['*'], api_key=None, lora_modules=None, prompt_adapters=None, chat_template=None, chat_template_content_format='auto', response_role='assistant', ssl_keyfile=None, ssl_certfile=None, ssl_ca_certs=None, ssl_cert_reqs=0, root_path=None, middleware=[], return_tokens_as_token_ids=False, disable_frontend_multiprocessing=False, enable_request_id_headers=False, enable_auto_tool_choice=False, enable_reasoning=False, reasoning_parser=None, tool_call_parser=None, tool_parser_plugin='', model='/home/featurize/data/models/deepseek-dr1-qwen-32b-bnb-4bit', task='auto', tokenizer=None, skip_tokenizer_init=False, revision=None, code_revision=None, tokenizer_revision=None, tokenizer_mode='auto', trust_remote_code=False, allowed_local_media_path=None, download_dir=None, load_format='auto', config_format=<ConfigFormat.AUTO: 'auto'>, dtype='auto', kv_cache_dtype='auto', max_model_len=32768, guided_decoding_backend='xgrammar', logits_processor_pattern=None, model_impl='auto', distributed_executor_backend='ray', pipeline_parallel_size=1, tensor_parallel_size=2, max_parallel_loading_workers=None, ray_workers_use_nsight=True, block_size=None, enable_prefix_caching=None, disable_sliding_window=False, use_v2_block_manager=True, num_lookahead_slots=0, seed=0, swap_space=4, cpu_offload_gb=0, gpu_memory_utilization=0.9, num_gpu_blocks_override=None, max_num_batched_tokens=None, max_num_partial_prefills=1, max_long_partial_prefills=1, long_prefill_token_threshold=0, max_num_seqs=None, max_logprobs=20, disable_log_stats=False, quantization=None, rope_scaling=None, rope_theta=None, hf_overrides=None, enforce_eager=True, max_seq_len_to_capture=8192, disable_custom_all_reduce=False, tokenizer_pool_size=0, tokenizer_pool_type='ray', tokenizer_pool_extra_config=None, limit_mm_per_prompt=None, mm_processor_kwargs=None, disable_mm_preprocessor_cache=False, enable_lora=False, enable_lora_bias=False, max_loras=1, max_lora_rank=16, lora_extra_vocab_size=256, lora_dtype='auto', long_lora_scaling_factors=None, max_cpu_loras=None, fully_sharded_loras=False, enable_prompt_adapter=False, max_prompt_adapters=1, max_prompt_adapter_token=0, device='auto', num_scheduler_steps=1, multi_step_stream_outputs=True, scheduler_delay_factor=0.0, enable_chunked_prefill=None, speculative_model=None, speculative_model_quantization=None, num_speculative_tokens=None, speculative_disable_mqa_scorer=False, speculative_draft_tensor_parallel_size=None, speculative_max_model_len=None, speculative_disable_by_batch_size=None, ngram_prompt_lookup_max=None, ngram_prompt_lookup_min=None, spec_decoding_acceptance_method='rejection_sampler', typical_acceptance_sampler_posterior_threshold=None, typical_acceptance_sampler_posterior_alpha=None, disable_logprobs_during_spec_decoding=None, model_loader_extra_config=None, ignore_patterns=[], preemption_mode=None, served_model_name=['deepseek-dr1-qwen-32b-bnb-4bit'], qlora_adapter_name_or_path=None, otlp_traces_endpoint=None, collect_detailed_traces=None, disable_async_output_proc=False, scheduling_policy='fcfs', scheduler_cls='vllm.core.scheduler.Scheduler', override_neuron_config=None, override_pooler_config=None, compilation_config=None, kv_transfer_config=None, worker_cls='auto', generation_config=None, override_generation_config=None, enable_sleep_mode=False, calculate_kv_scales=False, additional_config=None, disable_log_requests=False, max_log_len=None, disable_fastapi_docs=False, enable_prompt_tokens_details=False, dispatch_function=<function ServeSubcommand.cmd at 0x7f0e7ead42c0>)