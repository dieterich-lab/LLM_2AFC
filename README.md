# LLM 2AFC Linkage Test for Privacy Evaluation

This repository contains the code and methodology for a **Two-Alternative Forced-Choice (2AFC) Linkage Test**, designed to quantitatively evaluate the privacy preservation of de-identified clinical texts that use realistic surrogates.

## TL;DR

*   **Goal:** To determine if a sophisticated attacker, proxied by a Large Language Model (LLM), can re-identify original Personally Identifiable Information (PII) from a de-identified text at a rate better than random chance.
*   **Method:** A 2AFC test where an LLM is given a clinical letter with a single piece of PII masked. It must then choose between two options: the **true original entity** and its corresponding **fictitious surrogate**.
*   **Result:** Across 1,436 trials, the `openai/gpt-oss-20b` model achieved an accuracy of **51.95%**. This performance is statistically **indistinguishable from and equivalent to random chance** (50%).
*   **Conclusion:** The realistic surrogate method provides robust privacy protection against this type of linkage attack, supporting the "hiding in plain sight" principle.

## Introduction

Modern de-identification aims to replace sensitive PII with realistic, type-consistent surrogates. This approach maximizes data utility for downstream NLP tasks by preserving linguistic coherence. However, it also raises a critical privacy question: can an attacker leverage the remaining context to link the de-identified text back to original, sensitive information?

This project addresses that question directly. We treat a powerful LLM as a sophisticated adversary and challenge it with a linkage task. If the model cannot distinguish the true PII from a plausible, fictitious alternative significantly better than guessing, it provides strong evidence that the de-identification is effective.

## Methodology

The experiment is structured as a series of independent 2AFC trials. For each PII entity in our test corpus:

1.  **Masking:** The entity is located in its original text and replaced with a generic placeholder, `[**ENTITY_SLOT**]`.
2.  **Option Generation:** Two options are prepared:
    *   **Option A/B:** The true original entity (e.g., "Max Mustermann").
    *   **Option B/A:** The corresponding fictitious surrogate (e.g., "Peter Schmidt").
    The assignment of the true entity to option A or B is randomized for each trial.
3.  **Prompting the Attacker:** The LLM (`openai/gpt-oss-20b`) is presented with the masked letter and the two options. The system prompt instructs it to act as a "2AFC LINKAGE EVALUATOR" and choose the candidate that most likely belongs in the masked slot based on the context.
4.  **Recording the Choice:** The model's choice ("A" or "B") is recorded and compared against the ground truth to determine if the trial was successful.

## Results

The model's performance was statistically analyzed to test against the null hypothesis of chance performance (p=0.5).

| Metric                 | Value               | Interpretation                               |
| :--------------------- | :------------------ | :------------------------------------------- |
| Total Trials (n)       | 1,436               |                                              |
| Correct Choices (k)    | 746                 |                                              |
| **Accuracy**           | **51.95%**          | Very close to random chance (50%)            |
| 95% CI (Exact Clopper–Pearson) | [49.33%, 54.56%]    | The 50% chance level is well within the interval.  |
| Binomial Test (p-value) | **0.147** (2-sided), 0.073 (1-sided >0.5) | We fail to reject the null hypothesis; the result is not statistically significant. |
| **Equivalence Test**   | **Pass**            | The 90% CI is within a ±5 pp margin of 50%, confirming equivalence to chance. |

The results provide strong evidence that the model cannot reliably link the de-identified text to the original PII. Its performance is statistically equivalent to random guessing, suggesting the privacy of the surrogate corpus is robust.

## Installation

To install the required dependencies, run:

```bash
pip install -r requirements.txt
```

## Repository Structure

```
.
├── data/
│   ├── entities_txt/   # De-identified letters with [**ENTITY_SLOT**] placeholders
│   ├── original_ann/   # Brat .ann files containing the original PII text spans
│   └── fictive_ann/    # Brat .ann files containing the surrogate PII text spans
├── LLM_2AFC_output/    # JSON output for each trial is saved here
├── LLM_2AFC.ipynb      # Main Jupyter Notebook for running inference and analysis
└── README.md           # This file
```