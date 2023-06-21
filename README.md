# Shared Task on Parallel Data Curation Evaluation
## Setup

In this branch we provide scripts to run the evaluation for the WMT23 Shared Task on Parallel Data Curation. For more details on the task see: http://www2.statmt.org/wmt23/data-task.html

If you haven't already check out the repository:
```
git clone https://github.com/awslabs/sockeye.git
cd sockeye
git checkout wmt23_data_task
```

First please download all sentence data from the shared task in a folder called: `wmt23_data_task_data`. To do so follow the instructions on http://www2.statmt.org/wmt23/data-task.html 

We expect the following directory structure:
```
wmt23_data_task_data/sentences/sentences.et.tsv.gz
wmt23_data_task_data/sentences/sentences.lt.tsv.gz
wmt23_data_task_data/cosine_similarity/cosine_similarity.*.part_*.tsv.gz
wmt23_data_task_data/cosine_similarity/cosine_similarity.*.part_*.tsv.gz
wmt23_data_task_data/testsets_v2/{EMEA,EUBookshop,Europarl,JRC-Acquis}.{dev,test}.et-lt.{et,lt}
```
(cosine_similarity files are only needed if you want to reproduce the baseline's evaluation, which is recommended).

Install all dependencies:
```
pip install sockeye tqdm subword-nmt
```

## Creating baseline data in the expected format

In the following we show you how to produce a very simple baseline using the provided cosine similarity files. After extracting sentence alignments we will run end-to-end evaluation by production some BLEU scores from the extracted data.

```
python3 wmt23_data_task_scripts/create_top1_cosine_baseline_data.py --cosine-similarity-folder wmt23_data_task_data/cosine_similarity --output wmt23_data_task_data/top1_cosine.et-lt.tsv.gz
```

This script simply takes the top1 (highest cosine score) match and writes the data in the expected format into top1_cosine.et-lt.tsv.gz.

## Running end-to-end evaluation

First convert sentence ids into the actual strings:
```
python3 get_train_data.py --alignments-file wmt23_data_task_data/top1_cosine.et-lt.tsv.gz --output-dir wmt23_data_task_data/top1_cosine_eval
```

Next run the following command to start the evaluation:
```
python3 wmt23_data_task_scripts/run_eval.py --alignments-file wmt23_data_task_data/top1_cosine.et-lt.tsv.gz --working-dir wmt23_data_task_data/top1_cosine_eval --test-set-dir ./wmt23_data_task_data/testsets_v2/  --num-gpus 8 --batch-size-per-gpu 4096
```

Note that this includes model training and will therefore take several hours to complete. If you run into memory issues reduce the batch size using `--batch-size-per-gpu`. The effective batch size will be adjusted accordingly via the update interval. Feel free to also increase the batch size to get faster results if you have the GPU RAM.


If you want to take a look at the raw parallel data check out the following files:
```
wmt23_data_task_data/top1_cosine_eval/et_sentences/aligned_sentences.et.txt
wmt23_data_task_data/top1_cosine_eval/lt_sentences/aligned_sentences.lt.txt
```
