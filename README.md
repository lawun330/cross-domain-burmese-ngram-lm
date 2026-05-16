# Cross-Domain Burmese N-gram Language Modeling

## Overview

This project trains Burmese n-gram language models with KenLM and SRILM on a shared corpus, then compares toolkit performance and perplexity across news, legal, and religious test domains to study cross-domain generalization.

---

## Dataset

### Train Data (General LM)

- [Myanmar ALT](https://www2.nict.go.jp/astrec-att/member/mutiyama/ALT/my-alt-190530.zip) from [ALT Treebank Corpus](https://www2.nict.go.jp/astrec-att/member/mutiyama/ALT/)
- [Sayar's myPos version 3.0](https://github.com/ye-kyaw-thu/myPOS/blob/master/corpus-ver-3.0/corpus/mypos-ver.3.0.txt)
- [Hugging Face Myanmar Spoken Corpus v4](https://huggingface.co/datasets/URajinda/myanmar_spoken_corpus_v4_cleaned) **(Discarded later due to RAM issue)**

### Train Data (Domain-Specific LM) && Test Data

- Custom data in the news article domain (from [BBC Burmese International News](https://www.bbc.com/burmese/topics/cnlv9j1z93wt))
- Custom data in the legal document domain (from [မြန်မာနိုင်ငံကူးလက်မှတ်ဆိုင်ရာဥပဒေ](https://www.moi.gov.mm/file-download/download/public/103555) published by the Myanmar Ministry of Information)
- Custom data in the religious text domain (from [ခန္ဓဝိပဿနာဘာဝနာ](https://18milecdn.myanmarseo.com/file/18mile-cdn/books/%E1%80%81%E1%80%94%E1%80%B9%E1%80%93%E1%80%9D%E1%80%AD%E1%80%95%E1%80%BF%E1%80%94%E1%80%AC%20%E1%80%98%E1%80%AC%E1%80%9D%E1%80%94%E1%80%AC.pdf) by 18 Miles Sayardaw)

## Data Collection & Preprocessing

### Train Data (General LM)

- `prep_train_data_v1` = ALT + myPOS + Hugging Face spoken corpus $\rightarrow$ 3 GB $\rightarrow$ does not fit in limited RAM
- `prep_train_data_v2` = ALT + myPOS $\rightarrow$ good size $\rightarrow$ used for general-LM training

### Train Data (Domain-Specific LM) && Test Data

Domain-specific train and test corpora use the per-domain custom sources above, not the merged general corpus. Domains are chosen manually; raw text is collected by hand.
- `prep_domain_train_data_v1` = domain-specific training corpora (news, legal, religion)
- `prep_test_data_v1` = held-out test corpora for the same three domains

### All Preprocessing Steps

1. Punctuation and tag cleaning with `clean_text.py` $\rightarrow$ `.cleaned.state1`
2. Syllable normalization with `syl-normalizer` $\rightarrow$ `.cleaned.state2`
3. OppaWord segmentation $\rightarrow$ `.cleaned.state3`
4. Standardize syllable layout (drop short lines; 20 syllables/line, 15 lines/document) $\rightarrow$ `.cleaned.state4` *(test only)*
5. Line-level train/dev split $\rightarrow$ `*.train`, `*.dev` *(domain-specific train only)*

### Which Corpora Use Which Steps?
| Corpus | Merge | `။` split | 1 | 2 | 3 | 4 | 5 | Final artifact |
|--------|:-----:|:---------:|:-:|:-:|:-:|:-:|:-:|----------------|
| General LM train     | ✓ |   | ✓ | ✓ | ✓ |   |   | `combined_2.cleaned.state3` |
| Domain LM train (×3) |   |   | ✓ | ✓ | ✓ |   | ✓ | `train_*.cleaned.state3` → `*.train`, `*.dev` |
| Test (×3)            |   | ✓ | ✓ | ✓ | ✓ | ✓ |   | `*.cleaned.state4` |

---

## Models

### 1a. General KenLM
![General KenLM N-gram Results](img/kenlm_2g_vs_3g_metrics.png)
- **conclusion**: choose 3-gram

### 2a. General SRILM (Kneser-Ney discounting)

![General SRILM N-gram Results](img/srilm_2g_vs_3g_metrics.png)
- **conclusion**: choose 3-gram

![General SRILM N-gram Interpolation Results](img/srilm_2g_vs_3g_inter_metrics.png)
- **conclusion**: choose 3-gram

![General SRILM 3-gram Results](img/srilm_3g_plain_vs_inter_metrics.png)
- **conclusion**: choose interpolation for other domains except `religion`

### 1b. General + Domain-Specific KenLM
![Combined KenLM Results](img/kenlm_general_domain_mix_metrics.png)
- **conclusion**: choose mixed model

### 2b. General + Domain-Specific SRILM (Kneser-Ney discounting)
![Combined KenLM Results](img/srilm_general_domain_mix_metrics.png)
- **conclusion**: choose mixed model for other domains except `religion`

### 3. KenLM vs. SRILM on Different Domains		

|  Domain  | Metric        | KenLM 3-gram Mixed | SRILM 3-gram Mixed |
|----------|---------------|--------------------|--------------------|
|          | **PPL**       |      _106.19_      |       115.90       |
|   News   | **Entropy**   |      _4.67_        |       4.75         |
|          | **BPC**       |      _1.74_        |       1.77         |
|----------|---------------|--------------------|--------------------|
|          | **PPL**       |      _117.826_     |       117.922      |
|   Legal  | **Entropy**   |      _4.769_       |       4.770        |
|          | **BPC**       |      _1.782_       |       1.783        |
|----------|---------------|--------------------|--------------------|
|          | **PPL**       |      _52.92_       |       54.85        |
| Religion | **Entropy**   |      _3.97_        |       4.00         |
|          | **BPC**       |      _1.49_        |       1.50         |
|----------|---------------|--------------------|--------------------|

- **conclusion**: KenLM 3-gram mixed model > SRILM 3-gram mixed model

---

## File Structure
```
/
...
├── data/
│   ├── train/
│   │   └── domain-specific/
│   └── test/
│
├── img/
├── notebooks/
├── models/
│   ├── kenlm/
│   │   └── ppl/
│   └── srilm/
│       └── ppl/
│
├── oppaword/              # originally Sayar's
├── syl-normalizer/        # originally Sayar's
│
├── clean_text.py          # originally Sayar's # modified to remove word tags
├── eval_kenlm_srilm.py    # originally Sayar's eval_kenlm.py # modified to compute BPC and evaluate SRILM
│
├── conda_environment.yaml
└── requirements.txt
```

## File Formats

### LMs

- **`.arpa`**: ARPA-format n-gram language model.
- **`.arpa.binary`**, **`.arpa.ken.binary`**: KenLM binary model (from KenLM `build_binary`).
- **`.arpa.bin`**: SRILM binary model (from SRILM `ngram -write-bin-lm`).
- **`.arpa.error`**: stderr when building an ARPA model.

### Evaluation Artifacts

- **`.ppl`**: Perplexity / debugging output for `compute-best-mix` (from SRILM `ngram -ppl`).

### Text Corpora

- **`.raw`**: Unprocessed text.
- **`.cleaned.state1`**: After punctuation and word/POS tags removal (`clean_text.py`).
- **`.cleaned.state2`**: After syllable normalization (`syl-normalizer/`).
- **`.cleaned.state3`**: After word segmentation (`oppaword/`); typical line input for training general LMs.
- **`.cleaned.state4`**: Further standardized token layout used for test sets only.

### Train/Dev Splits

- **`.train`**: In-domain training split for domain-specific LMs.
- **`.dev`**: In-domain development split for tuning interpolation weights.

---

## Environment Setup

- Install KenLM from https://github.com/kpu/kenlm
- Install SRILM from https://github.com/BitSpeech/SRILM
- Install dependencies from `requirements.txt` or `conda_environment.yaml`

---

## References

- https://github.com/ye-kyaw-thu/AIE-F/tree/main/slide-code/class-15/LM-Tutorial
- ```bibtex
  @misc{syl_normalizer,
    author       = {Ye Kyaw Thu},
    title        = {{Syllable Normalization Tool for Myanmar Language}},
    version      = {0.6},
    month        = {May},
    year         = {2026},
    publisher    = {GitHub},
    url          = {https://github.com/ye-kyaw-thu/syl-Normalizer/tree/main/ver_0.6},
    note         = {Accessed: YYYY-MM-DD},
    institution  = {Language Understanding Lab (LU Lab), Myanmar}
  }
  ```
- ```bibtex
  @misc{oppaWord_2025,
    author       = {Ye Kyaw Thu},
    title        = {{oppaWord: Hybrid DAG+Bi-MM+LM Myanmar Word Segmenter}},
    version      = {1.0},
    month        = {August},
    year         = {2025},
    publisher    = {GitHub},
    url          = {https://github.com/ye-kyaw-thu/oppaWord},
    note         = {Accessed: YYYY-MM-DD},
    institution  = {Language Understanding Lab (LU Lab), Myanmar}
  }
  ```

## Note

This project was done for educational purposes as an assignment for the AI Engineering Fundamentals class taught by [*Sayar Ye Kyaw Thu*](https://github.com/ye-kyaw-thu).