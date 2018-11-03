# PyTorch implementation of Google AI's BERT


## Introduction

This is a PyTorch implementation of the [TensorFlow code](https://github.com/google-research/bert) released by Google AI with the paper [BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding](https://arxiv.org/abs/1810.04805).


## Converting the TensorFlow pre-trained models to Pytorch

You can convert the pre-trained weights released by GoogleAI by calling the script `convert_tf_checkpoint_to_pytorch.py`.
It takes a TensorFlow checkpoint (`bert_model.ckpt`) containg the pre-trained weights and converts it to a `.bin` file readable for PyTorch.

TensorFlow pre-trained models can be found in the [original TensorFlow code](https://github.com/google-research/bert). We give an example with the `BERT-Base Uncased` model:

```shell
export BERT_BASE_DIR=/path/to/bert/uncased_L-12_H-768_A-12
export BERT_PYTORCH_DIR=/path/to/pytorch/bert/uncased_L-12_H-768_A-12

python convert_tf_checkpoint_to_pytorch.py \
  --tf_checkpoint_path=$BERT_BASE_DIR/bert_model.ckpt \
  --bert_config_file=$BERT_BASE_DIR/bert_config.json \
  --pytorch_dump_path=$BERT_PYTORCH_DIR/pytorch_model.bin
```


## Fine-tuning with BERT: running the examples

We showcase the same examples as in the original implementation: fine-tuning on the MRPC classification corpus and the question answering dataset SQUAD.

Before running theses examples you should download the
[GLUE data](https://gluebenchmark.com/tasks) by running
[this script](https://gist.github.com/W4ngatang/60c2bdb54d156a41194446737ce03e2e)
and unpack it to some directory `$GLUE_DIR`. Please also download the `BERT-Base`
checkpoint, unzip it to some directory `$BERT_BASE_DIR`, and convert it to its PyTorch version as explained in the previous section.

This example code fine-tunes `BERT-Base` on the Microsoft Research Paraphrase
Corpus (MRPC) corpus and runs in less than 10 minutes on a single K-80.

```shell
export GLUE_DIR=/path/to/glue

python run_classifier_pytorch.py \
  --task_name MRPC \
  --do_train \
  --do_eval \
  --do_lower_case \
  --data_dir $GLUE_DIR/MRPC/ \
  --vocab_file $BERT_BASE_DIR/vocab.txt \
  --bert_config_file $BERT_BASE_DIR/bert_config.json \
  --init_checkpoint $BERT_PYTORCH_DIR/pytorch_model.bin \
  --max_seq_length 128 \
  --train_batch_size 32 \
  --learning_rate 2e-5 \
  --num_train_epochs 3.0 \
  --output_dir /tmp/mrpc_output_pytorch/
```

The next example fine-tunes `BERT-Base` on the SQuAD question answering task.

The data for SQuAD can be downloaded with the following links and should be saved in a `$SQUAD_DIR` directory.
*   [train-v1.1.json](https://rajpurkar.github.io/SQuAD-explorer/dataset/train-v1.1.json)
*   [dev-v1.1.json](https://rajpurkar.github.io/SQuAD-explorer/dataset/dev-v1.1.json)
*   [evaluate-v1.1.py](https://github.com/allenai/bi-att-flow/blob/master/squad/evaluate-v1.1.py)


```shell
export SQUAD_DIR=/path/to/SQUAD

python run_squad_pytorch.py \
  --vocab_file=$BERT_BASE_DIR/vocab.txt \
  --bert_config_file=$BERT_BASE_DIR/bert_config.json \
  --init_checkpoint=$BERT_PYTORCH_DIR/pytorch_model.bin \
  --do_train \
  --train_file=$SQUAD_DIR/train-v1.1.json \
  --do_predict \
  --predict_file=$SQUAD_DIR/dev-v1.1.json \
  --train_batch_size=12 \
  --learning_rate=5e-5 \
  --num_train_epochs=2.0 \
  --max_seq_length=384 \
  --doc_stride=128 \
  --output_dir=../debug_squad/
```


## Comparing TensorFlow and PyTorch models

We also include [a small Notebook](https://github.com/huggingface/pytorch-pretrained-BERT/blob/master/Comparing%20TF%20and%20PT%20models.ipynb) we used to verify that the conversion of the weights to PyTorch are consistent with the original TensorFlow weights.
Please follow the instructions in the Notebook to run it.


## Note on pre-training

The original TensorFlow code also release two scripts for pre-training BERT: [create_pretraining_data.py](https://github.com/google-research/bert/blob/master/create_pretraining_data.py) and [run_pretraining.py](https://github.com/google-research/bert/blob/master/run_pretraining.py).
As the authors notice, pre-training BERT is particularly expensive and requires TPU to run in a reasonable amout of time (see [here](https://github.com/google-research/bert#pre-training-with-bert)).

We have decided **not** to port these scripts for now and wait for the TPU support on PyTorch (see the recent [official announcement](https://cloud.google.com/blog/products/ai-machine-learning/introducing-pytorch-across-google-cloud)).


## Requirements

The main dependencies of this code are:
- PyTorch (>= 0.4.0)
- tqdm