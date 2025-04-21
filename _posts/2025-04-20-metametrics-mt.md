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

Reference-based MetaMetrics-MT model **topped** the primary submissions to the WMT24 Metrics Task (an annual competition that benchmarks machine‑translation (MT) evaluation metrics against experts' human‑judgement.). 

![wmt24 table]({{ site.baseurl }}/assets/img/metametrics/metametrics-mt1.png)

Metametrics was published in a paper on 1 Nov 2024.  


- The authors said to start by running `pip install -e .` which will install the project found in the current directory in *editable mode*. This means that any changes to the code in the project will be *automatically reflected without reinstalling the package.* This is useful for developing and testing a package locally.

We first create a virtual environment using UV:
```bash
uv venv                        # faster venv creation than python -m venv
source .venv/bin/activate      # activate .venv
uv pip install -e .            # install using setup.py 
uv pip install spacy           # also need to install spacy 
```

The authors said to look at the examples to see how to run. 1 example command is 
`metametrics-cli run examples/example_mt/mt_gp_metrics.yaml`, but we may get an error **No module named 'bleurt'**. 

This is due to the installation happens outside in the main folder, but not in the `.venv`, so we need to move into the bleurt folder to install in the `.venv`. 
```bash
uv pip install . 
```

Finally, the authors said to use the thin wrapper `metametrics-cli` to run the (which runs the real entry‑point `run_metametrics()`). An example they gave was:
```bash
metametrics-cli run examples/example_mt/mt_gp_metrics.yaml
```
