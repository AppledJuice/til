---
layout: post
title: Metametrics-mt
subtitle: Bayesian optimization with Gaussian Processes
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [evaluation, translation]
author: PSN
---

### What's MetaMetrics-mt?

[MetaMetrics-mt Paper](https://arxiv.org/html/2411.00390v1) | [Github](https://github.com/meta-metrics/metametrics/tree/main) | [WMT24](https://www2.statmt.org/wmt24/pdf/2024.wmt-1.2.pdf)

Metametrics-mt was published in a paper on 1 Nov 2024. This reference-based model **topped** the primary submissions to the WMT24 Metrics Task (an annual competition that benchmarks machine‑translation (MT) evaluation metrics against experts' human‑judgement.). 

![wmt24 table]({{ site.baseurl }}/assets/img/metametrics/metametrics-mt1.png) 

### Let's try to run the code

The authors said to start by running `pip install -e .`. This installs the project found in the current directory in *editable mode*. This means that any changes to the code in the project will be *automatically reflected without reinstalling the package.* Useful for developing and testing a package locally.

We first create a virtual environment using UV:
```bash
uv venv                        # faster venv creation than python -m venv
source .venv/bin/activate      # activate .venv
uv pip install --upgrade pip setuptools wheel  # Upgrade pip, setuptools & wheel 
uv pip install -e .            # install using setup.py 
uv pip install spacy           # also need to install spacy 
```

The authors said to look at the examples to see how to run. 
In the `example_mt` folder, the example command is 
`metametrics-cli run examples/example_mt/mt_gp_metrics.yaml`, but this may return an error **No module named 'bleurt'**. 

This is due to the installation happens outside in the main folder, but not in the `.venv`, so we need to move into the bleurt folder to install bleurt.
```bash
# first cd into the bleurt folder
uv pip install . 
```

![bleurt]({{ site.baseurl }}/assets/img/metametrics/bleurt.png) 


### metametrics-cli
Finally, the authors said to use the thin wrapper `metametrics-cli`. 
The `run` command routes to the real entry‑point `run_metametrics()`. 

The `mt_gp_metrics.yaml` has all the configs setup - they listed out what are all the configs:
```{note}
- modality: Modality for MetaMetrics (e.g., text).
- output_dir: Directory to save experiment results.
- evaluation_method: Method for evaluating experiments.
evaluation_only: Set to True to perform evaluation only (requires pipeline_load_path).
pipeline_load_path: Path to a pre-saved pipeline (used with evaluation_only).
- optimizer_config_path: Path to YAML/JSON with optimizer configuration.
- metrics_config_path: Path to YAML/JSON with metrics configuration.
- dataset_config_path: Path to YAML/JSON with dataset configuration.
- hf_hub_token: HuggingFace token for dataset access.
cache_dir: Cache directory for datasets and models.
- normalize_metrics: Normalize metrics for MetaMetrics.
- overwrite_output_dir: Overwrite existing output directory.
```

Using the same example command as before:
```bash
metametrics-cli run examples/example_mt/mt_gp_metrics.yaml
```

This will give an error.. [ValueError: Invalid pattern: '**' can only be an entire path component](https://stackoverflow.com/questions/77671277/valueerror-invalid-pattern-can-only-be-an-entire-path-component?utm_source=chatgpt.com)

We can fix this by updating datasets:
```bash
uv pip install -U datasets
```

- There's another error when I tried to rerun metametrics-cli:
KeyError: "Column text_hyp not in the dataset. Current columns in the dataset: ['target_score']"

The example `dataset_config.yaml` had some errors, need replace with below: 
```yaml
### Dataset
train_dataset:
  wmt_train:
    hf_hub_url: davidanugraha/wmt-mqm
    subset: wmt2020-2022
    columns:
      text_src: src         # was src_text
      text_hyp: mt          # was hyp_text
      text_ref: ref         # was ref_text
      target_score: score
eval_dataset:
  wmt_train:
    hf_hub_url: davidanugraha/wmt-mqm
    subset: wmt2023
    columns:
      text_src: src
      text_hyp: mt
      text_ref: ref
```

- A final error occurred: 

"/home/paishan/metametrics/metametrics/mtenv/lib/python3.11/site-packages/sacrebleu/metrics/base.py", line 235, in _check_sentence_score_args
    raise TypeError(f"{prefix}: {err_msg}")
TypeError: BLEU: The argument refs should be a sequence of strings.


BLEU: The argument refs should be a sequence of strings means BLEUMetric.score is handing a single str to sacrebleu.sentence_bleu, but **the function expects a list of reference strings** (even when there is only one reference).

So made a small fix in BLEUMetric.score:
```python
def score(self, predictions: List[str], references: Union[None, List[List[str]]]=None, sources: Union[None, List[str]]=None) -> List[float]:
        segment_scores = []
        for pred, ref in zip(predictions, references):
            print(f'pred: {pred}')
            print(f'ref: {ref}')
            if isinstance(ref, str):
                ref = [ref]          # ⬅️ fix
            score = sacrebleu.sentence_bleu(pred, ref, lowercase=self.lowercase, tokenize=self.tokenize,
                                            smooth_method=self.smooth_method, smooth_value=self.smooth_value,
                                            use_effective_order=self.use_effective_order).score
            segment_scores.append(score)
        return segment_scores
```

- Run again.
```bash
metametrics-cli run examples/example_mt/mt_gp_metrics.yaml
```
9:35AM

It should run Bleu (quite fast) and followed by BLEURT(may take about some time, see below screenshot):
![run example_mt]({{ site.baseurl }}/assets/img/metametrics/run-mt-example.png)
