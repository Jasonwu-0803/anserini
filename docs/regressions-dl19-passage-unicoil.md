# Anserini Regressions: TREC 2019 Deep Learning Track (Passage)

**Model**: uniCOIL (with doc2query-T5 expansions)

This page describes regression experiments, integrated into Anserini's regression testing framework, using uniCOIL (with doc2query-T5 expansions) on the [TREC 2019 Deep Learning Track passage ranking task](https://trec.nist.gov/data/deep2019.html).
The uniCOIL model is described in the following paper:

> Jimmy Lin and Xueguang Ma. [A Few Brief Notes on DeepImpact, COIL, and a Conceptual Framework for Information Retrieval Techniques.](https://arxiv.org/abs/2106.14807) _arXiv:2106.14807_.

The experiments on this page are not actually reported in the paper.
However, the model is the same.

Note that the NIST relevance judgments provide far more relevant passages per topic, unlike the "sparse" judgments provided by Microsoft (these are sometimes called "dense" judgments to emphasize this contrast).
For additional instructions on working with MS MARCO passage collection, refer to [this page](experiments-msmarco-passage.md).

The exact configurations for these regressions are stored in [this YAML file](../src/main/resources/regression/dl19-passage-unicoil.yaml).
Note that this page is automatically generated from [this template](../src/main/resources/docgen/templates/dl19-passage-unicoil.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead and then run `bin/build.sh` to rebuild the documentation.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --index --verify --search --regression dl19-passage-unicoil
```

We make available a version of the MS MARCO passage corpus that has already been processed with uniCOIL, i.e., we have applied doc2query-T5 expansions, performed model inference on every document, and stored the output sparse vectors.
Thus, no neural inference is involved.

From any machine, the following command will download the corpus and perform the complete regression, end to end:

```bash
python src/main/python/run_regression.py --download --index --verify --search --regression dl19-passage-unicoil
```

The `run_regression.py` script automates the following steps, but if you want to perform each step manually, simply copy/paste from the commands below and you'll obtain the same regression results.

## Corpus Download

Download the corpus and unpack into `collections/`:

```bash
wget https://rgw.cs.uwaterloo.ca/JIMMYLIN-bucket0/data/msmarco-passage-unicoil.tar -P collections/
tar xvf collections/msmarco-passage-unicoil.tar -C collections/
```

To confirm, `msmarco-passage-unicoil.tar` is 3.4 GB and has MD5 checksum `78eef752c78c8691f7d61600ceed306f`.
With the corpus downloaded, the following command will perform the remaining steps below:

```bash
python src/main/python/run_regression.py --index --verify --search --regression dl19-passage-unicoil \
  --corpus-path collections/msmarco-passage-unicoil
```

## Indexing

Sample indexing command:

```bash
target/appassembler/bin/IndexCollection \
  -collection JsonVectorCollection \
  -input /path/to/msmarco-passage-unicoil \
  -index indexes/lucene-index.msmarco-passage-unicoil/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 16 -impact -pretokenized -storeDocvectors \
  >& logs/log.msmarco-passage-unicoil &
```

The path `/path/to/msmarco-passage-unicoil/` should point to the corpus downloaded above.

The important indexing options to note here are `-impact -pretokenized`: the first tells Anserini not to encode BM25 doclengths into Lucene's norms (which is the default) and the second option says not to apply any additional tokenization on the uniCOIL tokens.
Upon completion, we should have an index with 8,841,823 documents.

For additional details, see explanation of [common indexing options](common-indexing-options.md).

## Retrieval

Topics and qrels are stored in [`src/main/resources/topics-and-qrels/`](../src/main/resources/topics-and-qrels/).
The regression experiments here evaluate on the 43 topics for which NIST has provided judgments as part of the TREC 2019 Deep Learning Track.
The original data can be found [here](https://trec.nist.gov/data/deep2019.html).

After indexing has completed, you should be able to perform retrieval as follows:

```bash
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-passage-unicoil/ \
  -topics src/main/resources/topics-and-qrels/topics.dl19-passage.unicoil.0shot.tsv.gz \
  -topicreader TsvInt \
  -output runs/run.msmarco-passage-unicoil.unicoil.topics.dl19-passage.unicoil.0shot.txt \
  -impact -pretokenized &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-passage-unicoil/ \
  -topics src/main/resources/topics-and-qrels/topics.dl19-passage.unicoil.0shot.tsv.gz \
  -topicreader TsvInt \
  -output runs/run.msmarco-passage-unicoil.rm3.topics.dl19-passage.unicoil.0shot.txt \
  -impact -pretokenized -rm3 &

target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-passage-unicoil/ \
  -topics src/main/resources/topics-and-qrels/topics.dl19-passage.unicoil.0shot.tsv.gz \
  -topicreader TsvInt \
  -output runs/run.msmarco-passage-unicoil.rocchio.topics.dl19-passage.unicoil.0shot.txt \
  -impact -pretokenized -rocchio &
```

Evaluation can be performed using `trec_eval`:

```bash
tools/eval/trec_eval.9.0.4/trec_eval -m map -c -l 2 src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.unicoil.topics.dl19-passage.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -m ndcg_cut.10 -c src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.unicoil.topics.dl19-passage.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -m recall.100 -c -l 2 src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.unicoil.topics.dl19-passage.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -m recall.1000 -c -l 2 src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.unicoil.topics.dl19-passage.unicoil.0shot.txt

tools/eval/trec_eval.9.0.4/trec_eval -m map -c -l 2 src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.rm3.topics.dl19-passage.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -m ndcg_cut.10 -c src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.rm3.topics.dl19-passage.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -m recall.100 -c -l 2 src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.rm3.topics.dl19-passage.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -m recall.1000 -c -l 2 src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.rm3.topics.dl19-passage.unicoil.0shot.txt

tools/eval/trec_eval.9.0.4/trec_eval -m map -c -l 2 src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.rocchio.topics.dl19-passage.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -m ndcg_cut.10 -c src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.rocchio.topics.dl19-passage.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -m recall.100 -c -l 2 src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.rocchio.topics.dl19-passage.unicoil.0shot.txt
tools/eval/trec_eval.9.0.4/trec_eval -m recall.1000 -c -l 2 src/main/resources/topics-and-qrels/qrels.dl19-passage.txt runs/run.msmarco-passage-unicoil.rocchio.topics.dl19-passage.unicoil.0shot.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **AP@1000**                                                                                                  | **uniCOIL (with doc2query-T5 expansions)**| **+RM3**  | **+Rocchio**|
|:-------------------------------------------------------------------------------------------------------------|-----------|-----------|-----------|
| [DL19 (Passage)](https://trec.nist.gov/data/deep2019.html)                                                   | 0.4612    | 0.4641    | 0.4603    |
| **nDCG@10**                                                                                                  | **uniCOIL (with doc2query-T5 expansions)**| **+RM3**  | **+Rocchio**|
| [DL19 (Passage)](https://trec.nist.gov/data/deep2019.html)                                                   | 0.7024    | 0.6761    | 0.6658    |
| **R@100**                                                                                                    | **uniCOIL (with doc2query-T5 expansions)**| **+RM3**  | **+Rocchio**|
| [DL19 (Passage)](https://trec.nist.gov/data/deep2019.html)                                                   | 0.6054    | 0.6207    | 0.6211    |
| **R@1000**                                                                                                   | **uniCOIL (with doc2query-T5 expansions)**| **+RM3**  | **+Rocchio**|
| [DL19 (Passage)](https://trec.nist.gov/data/deep2019.html)                                                   | 0.8292    | 0.8598    | 0.8515    |

Note that retrieval metrics are computed to depth 1000 hits per query (as opposed to 100 hits per query for document ranking).
Also, for computing nDCG, remember that we keep qrels of _all_ relevance grades, whereas for other metrics (e.g., AP), relevance grade 1 is considered not relevant (i.e., use the `-l 2` option in `trec_eval`).
The experimental results reported here are directly comparable to the results reported in the [track overview paper](https://arxiv.org/abs/2003.07820).

## Reproduction Log[*](reproducibility.md)

To add to this reproduction log, modify [this template](../src/main/resources/docgen/templates/dl19-passage-unicoil.template) and run `bin/build.sh` to rebuild the documentation.

+ Results reproduced by [@manveertamber](https://github.com/manveertamber) on 2022-02-25 (commit [`7472d86`](https://github.com/castorini/anserini/commit/7472d862c7311bc8bbd30655c940d6396e27c223))
+ Results reproduced by [@mayankanand007](https://github.com/mayankanand007) on 2022-02-28 (commit [`950d7fd`](https://github.com/castorini/anserini/commit/950d7fd88dbb87f39e9c1f6ccf9e41cbb6f04f36))
+ Results reproduced by [@lintool](https://github.com/lintool) on 2022-06-06 (commit [`236b386`](https://github.com/castorini/anserini/commit/236b386ddc11d292b4b736162b59488a02236d6c))
