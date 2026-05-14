# Burmese Domain-Specific Language Model

## Overview

---

## File Structure
```
/
...
├── data/
├── notebooks/
│
├── clean_text.py        # originally Sayar's # modified to remove word tags
│
├── conda_environment.yaml
└── requirements.txt
```

---

## Dataset
### Train Data
- [Myanmar ALT](https://www2.nict.go.jp/astrec-att/member/mutiyama/ALT/my-alt-190530.zip) from [ALT Treebank Corpus](https://www2.nict.go.jp/astrec-att/member/mutiyama/ALT/)
- [Sayar's myPos version 3.0](https://github.com/ye-kyaw-thu/myPOS/blob/master/corpus-ver-3.0/corpus/mypos-ver.3.0.txt)
- https://huggingface.co/datasets/URajinda/myanmar_spoken_corpus_v4_cleaned (Discarded later due to RAM issue)

### Test Data
- Custom data in the news article domain
- Custom data in the religious text domain
- Custom data in the legal document domain

## Data Collection

The training data were imported from various existing sources, with word tags removed and the datasets combined to create a larger corpus. The test data were collected manually and cleaned.

## Environment Setup

---

## Models

---

## References

- https://github.com/ye-kyaw-thu/AIE-F/tree/main/slide-code/class-15/LM-Tutorial

---

## Note

This project was done for educational purposes as an assignment for the AI Engineering Fundamentals class taught by [*Sayar Ye Kyaw Thu*](https://github.com/ye-kyaw-thu).
