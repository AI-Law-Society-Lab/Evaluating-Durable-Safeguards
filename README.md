# Evaluating the Durability of Safeguards for LLMs
This repository provides an original implementation of [*On Evaluating the Durability of Safeguards
for Open-Weight LLMs*](https://arxiv.org/abs/2412.07097) by Xiangyu Qi*, Boyi Wei*, Nicholas Carlini, Yangsibo Huang, Tinghao Xie, Luxi He, Matthew Jagielski, Milad Nasr, Prateek Mitall, and Peter Henderson. (*Equal contribution)


## Create environment

You can use the following instructions to createa  conda environment:

```shell
conda env create -f environment.yml
```

## Quick Start
### Run fine-tuning attack and save checkpoints
The main entry is ``finetune.py``, simply use ``scripts/launch_ft.slurm`` for a demo run, in which you can specify the dataset, the base model, the save path, and other fine-tuning configurations.

### Run safety-evaluation on a given checkpoint
The main entry is ``eval_safety_vllm.py``, simply use ``scripts/launch_safety_eval.slurm`` for a demo run, in which you can specify the safety benchmark, the base model, the output file path, and other generation configs.

### Run utility evaluation on a given checkpoint
Because some of our utility benchmarks involve GPT-judge and require internet access. Therefore, we separate our inference and evaluation pipeline. 
#### Run utility inference
The main entry is ``inference_utility_vllm.py``, simply use ``scipts/launch_utility_inference.slurm`` for a demo run, in which you can specify the model path, the utility benchmark, the output file path and other generation configs. After running inference, it will output a raw output file to the specified path.
#### Run utility evaluation
The main entry is ``eval_utility_vllm.py``, simply use ``scripts/launch_utility_eval.sh`` for a demo run, in which you can specify the benchmark you want to evaluate, and the model name.

### Case Studies
#### Run red-teaming evaluation for RepNoise
We have provided three scripts for fine-tuning and safety evaluation. Run ``scripts/repnoise/launch_ft_safety_eval_orig_dataset.slurm`` for original Beavertails (Used by [Rosati et al., 2024](https://arxiv.org/pdf/2405.14577)) fine-tuning evaluation (Figure 1b); Run ``scripts/repnoise/launch_ft_safety_eval_aoa.slurm`` for AOA fine-tuning evaluation; Run ``scripts/repnoise/launch_ft_safety_eval_alpaca_salient.slurm`` for Alpaca-Salient fine-tuning evaluation (Figure 7). 

#### Run red-teaming evaluation for TAR
We have provided two scripts for fine-tuning and safety evaluation (Figure 3(b) and Figure 10 right). Run ``scripts/tar/launch_ft_safety_eval.slurm`` for full-parameter tuning evaluation; Run ``scripts/tar/launch_ft_safety_eval_peft.slurm`` for parameter-efficient fine-tuning (PEFT) evaluation.

## Run Fine-tuning Attack and Save Checkpoints
The main entry is ``finetune.py``. Important parameters are:
1. ``--model_name_or_path`` specifies the model path
2. ``--dataset_name`` specifies the dataset name. Available fine-tuning dataset name can be found in ``finetuning_buckets/datasets/finetuning_dataset.py``
3. ``--model_family`` specifies model family. Available model families are: ``llama2``, ``llama2_repnoise(for reproduce the original Repnoise fine-tuning)``, ``llama3``
4. ``--learning_rate`` specifies learning rate.
5. ``--ft_seed`` specifies the seed used for fine-tuning.
6. ``--profile`` to estimate the computational cost of fine-tuning.
7. ``--per_device_train_batch_size`` specifies the batch size for each device. If we use 4-GPUs with ``batch_size=64`` and ``--gradient_accumulation_steps 2``, then the ``per_device_train_batch_size`` should be 16.
8. ``--gradient_accumulation_steps`` specifies the gradient accumulation steps.
9. ``--output_dir`` specifies the output path
10. ``--num_train_epochs`` specifies the number of training epochs
11. ``--torch_dtype`` to specify the `torch.dtype` of the model.

## Run Safety Evaluation
The main entry is ``eval_safety_vllm.py``. Important parameters are:
1. ``--model_path`` specifies the model path
2. ``--model_name`` specifies the model name
3. ``--tokenizer_name_or_path`` specifies the path of tokenizer.
4. ``--model_family`` specifies model family. Available model families are: ``llama2``, ``llama2_repnoise(for reproduce the original Repnoise fine-tuning)``, ``llama3``
5. ``--drop_system_prompt`` removes the system prompt
6. ``--num_gpus`` specifies the number of gpus
7. ``--safety_bench`` specifies the benchmark used for evaluation
8. ``--evaluator`` specifies the evaluator to calculate the metric. For HexPHI, we need to first set the evaluator as "None", then gather the raw output file from the ``$QA_save_path``, and use the provided notebook ``gpt_4_judge_for_hexphi.ipynb`` to compute the safety rate generated by GPT-judge.
9. ``--save_path`` specifies the path for saving the final metric.
10. ``--QA_save_path`` specifies the path for saving the raw output
11. ``--eval_template`` specifies the template used for evaluation. The default is ``plain``. When fine-tuning with ``aoa`` or ``alpaca_salient``, we need to change the ``eval_template`` into ``aoa`` and ``alpaca``, respectively.


## Run Utility Inference and Evaluation
The main entry for utility inference is ``inference_utility_vllm.py``. Important parameters are:
1. ``--model_path`` specifies the model path
2. ``--model_name`` specifies the model name
3. ``--tokenizer_name_or_path`` specifies the path of tokenizer.
4. ``--model_family`` specifies model family. Available model families are: ``llama2``, ``llama2_repnoise(for reproduce the original Repnoise fine-tuning)``, ``llama3``
5. ``--drop_system_prompt`` removes the system prompt
6. ``--num_gpus`` specifies the number of gpus
7. ``--save_path`` specifies the path for saving the raw output.

For MT-Bench and TruthfulQA, you may need to provide OpenAI's API key. Use ``export OPENAI_API_KEY=<your_api_key_here>`` to specify your api key. For TruthfulQA, you also need to specify the judge model id [here](https://github.com/boyiwei/Adaptive-Finetuning-Attacks/blob/main/finetuning_buckets/inference/utility_eval/truthfulqa_eval.py#L91).
   

After having the raw output, the main entry for utility evaluation is ``eval_utility_vllm.py``. Important parameters are:
1. ``--model`` specifies the model name used in the raw output file.
2. ``--bench`` specifies the benchmark needed to calculate the score.
3. ``--save_path`` specifies the path to the raw output file
4. ``--output-path`` specifies the path to save the final score.

## Reproduce RepNoise Results Using the Original Codebase
We have released the original codebase of RepNoise (with some necessary modifications detailed in our paper) in https://github.com/boyiwei/RepNoise-Reproduce. We also provided a [script](https://github.com/boyiwei/RepNoise-Reproduce/blob/main/launch_eval_ft_attack.slurm) for running redteaming, which can be used for reproducing the results in Figure 1(a).

## Reproduce TAR Results Using the Original Codebase

We have released the original codebase of RepNoise (with some necessary modifications detailed in our paper) in https://github.com/boyiwei/TAR-Reproduce. We also provided a [script](https://github.com/boyiwei/TAR-Reproduce/blob/main/red_teaming/orig_implement.slurm) for running redteaming. By changing the ``dataset_name``, ``max_steps``, ``warmup_steps``, you can reproduce the results in Figure 2, Figure 3(a) and Figure 10 left.

## Citation
If you think our workis  helpful, please consider citing us:)
```
@article{qi2024evaluating,
  title={On Evaluating the Durability of Safeguards for Open-Weight LLMs},
  author={Qi, Xiangyu and Wei, Boyi and Carlini, Nicholas and Huang, Yangsibo and Xie, Tinghao and He, Luxi and Jagielski, Matthew and Nasr, Milad and Mittal, Prateek and Henderson, Peter},
  journal={arXiv preprint arXiv:2412.07097},
  year={2024}
}
```
