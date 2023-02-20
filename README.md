
# DUDE: Document UnderstanDing of Everything Benchmark

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

Shared repository to work with the [DUDE benchmark](https://rrc.cvc.uab.es/?ch=23&com=introduction), used in the ICDAR 2023 Competition on Document UnderstanDing of Everything.
The competition deadline is on April 1, 2023 and might come with a **$XXX prize pool**.

The repository collects a number of tools:
* A data loader making it easy to load the dataset, work with its annotations, pdfs and pre-computed OCR
* A repository that allows to test submission files and run standalome evaluation scripts.
* Baseline methods (will appear soon).

Table of Contents:
* [Download instructions](#download-dude)
* [Predictions format and running evaluation](#predictions-format-and-running-evaluation)
* [Pre-computed OCR](#pre-computed-ocr)
* [Development instructions](#development-instructions)
* [Dataset and benchmark paper](#dataset-and-benchmark-paper)

Also check [Tutorials](tutorials/) to get quickly started with the repo.

## Download the dataset

The dataset is publicly available by following the links in https://rrc.cvc.uab.es/?ch=23&com=downloads.
This requires you to register on the RRC platform, only so that we can keep track of how many particpants are interested in our competition.
You can also download the binaries (PDF & OCR) and unzip in a custom `data_dir`.


## Load the dataset

The suggested way to load the dataset is to start from https://huggingface.co/datasets/jordyvl/DUDE_loader

```python
from datasets import load_dataset
ds = load_dataset("jordyvl/DUDE_loader", 'Amazon_original') #automatically downloads binaries tar and extracts to HF_CACHE
ds = load_dataset("jordyvl/DUDE_loader", 'Amazon_original', data_dir="/DUDE_train-val-test_binaries") #with custom extracted data directory
```

The second argument loads a specific OCR configuration; have a look at `DUDEConfig` to understand how to call different versions. 

## Predictions format and running evaluation

Check out our standalone repository which explains it all: https://github.com/Jordy-VL/DUDEeval 

## Pre-computed OCR

We provide OCR outputs to help participants of DUDE. Note it is not required to use the attached OCRs, and you can use your own preferred OCR service (as long as you mention it with your submission). 

Specifically, available provided OCRs include outputs of:
* Azure Cognitive Services (3.2)
* Amazon Textract (2018-06-27)
* Tesseract (5.3.0)

The output of Azure and Amazon OCRs was obtained from the PDF files. Since Tesseract does not support PDF inputs, we converted them to TIFFs (200 dpi) before running the process. There were three files where this process failed due to format limitations.

In addition to software-specific outputs (_original), we provide OCRs in the unified form (_due) introduced by the authors of DUE Benchmark (https://github.com/due-benchmark). Please find a toy reader for any of these below: 

```
import json
from typing import Dict, List, Tuple, Literal
def read_document(
        file_id: str,
        subset: Literal['train', 'val', 'test'] = 'train',
        ocr_engine: Literal['Azure', 'Amazon', 'Tesseract'] = 'Azure'
    ) -> Tuple[List[Image], Dict]:

    # Read OCR results in DUE format
    with open(f'OCR/{ocr_engine}/{file_id}_due.json') as ins:
        data: Dict = json.load(ins)
```


## Dataset and benchmark paper (incoming)
The dataset, the benchmark tasks and the evaluation criteria are described in detail in the [dataset paper](). To cite the dataset, please use the following BibTeX entry:
```
@inproceedings{dude2023icdar,
    title={ICDAR 2023 Challenge on Document UnderstanDing of Everything (DUDE)},
    author={Van Landeghem, Jordy et al.},
    booktitle={Proceedings of the ICDAR 2023},
    year={2023}
}

```
