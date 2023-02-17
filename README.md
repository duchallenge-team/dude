
# DUDE: Document UnderstanDing of Everything Benchmark

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

Repository to work with the [DUDE benchmark](https://rrc.cvc.uab.es/?ch=23&com=introduction), used in the ICDAR 2023 Competition on Document UnderstanDing of Everything.
The competition deadline is on April 20, 2023 and comes with a **$1000 prize pool**.

The repository consists of:
* A framework, `dude`, making it easy to load the dataset, work with its annotations, pdfs and pre-computed OCR and run the evaluation.
* An interactive [dataset browser notebook](dude/tools/dataset_notebook.ipynb) to visualize the document annotations, predictions and evaluation results.
* Baseline methods (will appear soon).

Table of Contents:
* [Download instructions](#download-dude)
* [Installation](#installation)
* [Predictions format and running evaluation](#predictions-format-and-running-evaluation)
* [Pre-computed OCR](#pre-computed-ocr)
* [Development instructions](#development-instructions)
* [Dataset and benchmark paper](#dataset-and-benchmark-paper)

Also check [Tutorials](tutorials/) to get quickly started with the repo.

## Download the dataset

First you need to obtain a secret token by following the instructions at https://docile.rossum.ai/. Then download and unzip the dataset by running:
```bash

TO BE DONE

```

Run `./download_dataset.sh --help` for more options, including how to only show urls (to download
with a different tool than curl), how to download smaller unlabeled/synthetic chunks or unlabeled
dataset without pdfs (with pre-computed OCR only).

You can also work with the zipped datasets when you turn off image caching (check [Load and sample dataset](tutorials/load_and_sample_dataset.md) tutorial for details).

## Installation

### Install as a library
To install just the library, download the wheel from the [latest release on github](https://github.com/rossumai/docile/releases) and run:
```shell
pip install docile-0.1.0-py3-none-any.whl
```

To convert pdfs into images, the library uses https://github.com/Belval/pdf2image. On linux you might need to install:
```shell
apt install poppler-utils
```
And on macOS:

```shell
brew install poppler
```

Now you have all dependencies to work with the dataset annotations, pdfs, pre-comptued OCR and to run the evaluation. You can install extra dependencies by running the following (although using one of the provided dockerfiles, as explained below, might be easier in this case):
```shell
pip install "docile-0.1.0-py3-none-any.whl[interactive]"
pip install "docile-0.1.0-py3-none-any.whl[ocr]"
```

The first line installs additional dependencies allowing you to use the interactive dataset browser in [docile/tools/dataset_browser.py](docile/tools/dataset_browser.py) and the [tutorials](tutorials/). The second line let's you rerun the OCR predictions from scratch (e.g., if you'd like to run it with different parameters) but to make it work, you might need additional dependencies on your system. Check https://github.com/mindee/doctr for the installation instructions (for pytorch).



## Predictions format and running evaluation

To evaluate predictions for tasks KILE or LIR, use the following command:
```bash
docile_evaluate \
  --task LIR \
  --dataset-path path/to/dataset[.zip] \
  --split val \
  --predictions path/to/predictions.json \
  --evaluate-x-shot-subsets "0,1-3,4+" \  # default, show evaluation for 0-shot, few-shot and many-shot layout clusters
  --evaluate-synthetic-subsets \  # optional, show evaluation on layout clusters with available synthetic data
  --evaluate-fieldtypes \  # optional, show breakdown per fieldtype
  --evaluate-also-text \  # optional
  --store-evaluation-result LIR_val_eval.json  # optional, it can be loaded in the dataset browser
```

Run `docile_evaluate --help` for more information on the options. You can also run `docile_print_evaluation_report --evaluation-result-path LIR_val_eval.json` to print the results of a previously computed evaluation.

Predictions need to be stored in a single json file (for each task separately) containing a mapping from `docid` to the 'questionId' predictions for that document, i.e.:
```python

{'questionId': '0017b64bd017f06db47e56a6a113e22e_bb55e0af451429f2dcae69e6d0713616',
  'question': 'What is the first and last name of the indvidual in list # 539?',
  'answers': ['Ajay Dev Goud'],
  'answers_page_bounding_boxes': [[{'left': 353,
     'top': 409,
     'width': 198,
     'height': 26,
     'page': 8}]],
  'answers_variants': [],
  'answer_type': 'extractive',
  'docId': '0017b64bd017f06db47e56a6a113e22e',
  'data_split': 'train'}
```
Explanation of the individual fields of the predictions:
  * `page`: page index (from zero) the prediction belongs to
  * `bbox`: relative coordinates (from 0 to 1) representing the `left`, `top`, `right`, `bottom` sides of the bbox, respectively
  * `fieldtype`: the fieldtype (sometimes called category or key) of the prediction
  * `line_item_id`: ID of the line item. This should be a different number for each line item, the order does not matter. Omit for KILE predictions.
  * `score` [optional]: the confidence for this prediction, can be omitted (in that case predictions are taken in the order in which they are stored in the list)
  * `text` [optional]: text of the prediction, evaluated in a secondary metric only (when `--evaluate-also-text` is used)
  * `use_only_for_ap` [optional, default is False]: only use the prediction for AP metric computation, not for f1, precision and recall (useful for less confident predictions).

You can use `docile.dataset.store_predictions` to store predictions represented with the `docile.dataset.Field` class to a json file with the required format.

## Pre-computed OCR

Pre-computed OCR is provided with the dataset. The prediction was done using the [DocTR](https://github.com/mindee/doctr) library. On top of that, word boxes were snapped to text (check the code in [docile/dataset/document_ocr.py](docile/dataset/document_ocr.py)). These snapped word boxes are used in evaluation (description of the evaluation is coming soon).

While this should not be needed, it is possible to (re)generate OCR from scratch (including the snapping) with the provided `Dockerfile.gpu`. Just delete `DATASET_PATH/ocr` directory and then access the ocr for each document and page with `doc.ocr.get_all_words(page, snapped=True)`.

## Development instructions

For development, install [poetry](https://python-poetry.org/docs/) and run `poetry install`. Start a shell with the virtual environment activated with `poetry shell`. No other dependencies are needed to run pre-commit and the tests. It's recommended to use docker (as explained above) if you need the extra (interactive or ocr) dependencies.

Install pre-commit with `pre-commit install` (don't forget you need to prepend all commands with `poetry run ...` if you did not run `poetry shell` first).

Run tests by calling `pytest tests`.

## Dataset and benchmark paper
The dataset, the benchmark tasks and the evaluation criteria are described in detail in the [dataset paper](https://arxiv.org/abs/2302.05658). To cite the dataset, please use the following BibTeX entry:
```
@inproceedings{dude2023icdar,
    title={ICDAR 2023 Challenge on Document UnderstanDing of Everything (DUDE)},
    author={Van Landeghem, Jordy et . al.},
    booktitle={Proceedings of the ICDAR},
    year={2023}
}

```
