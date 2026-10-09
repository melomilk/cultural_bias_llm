# Cultural Bias in Multilingual Language Models: Kazakh vs Western Contexts

A small probing study of how multilingual masked language models treat Kazakh contexts compared with Western European ones.

Each test is a pair of sentences that differ only in the cultural reference, for example:

- `I met a girl in Almaty called [MASK].`
- `I met a girl in Vienna called [MASK].`

The models fill in the blank, and the predictions for the two versions are compared.

**Main result:** in Kazakh contexts, neither mBERT nor XLM-RoBERTa predicted any of the expected Kazakh names in their top 10 guesses. They filled the blank with Russian and Western names instead.

## Models

| Model | Hugging Face ID | Status |
| --- | --- | --- |
| mBERT | `bert-base-multilingual-cased` | Run and scored |
| XLM-RoBERTa base | `xlm-roberta-base` | Run and scored |
| XLM-RoBERTa large | `xlm-roberta-large` | Run, predictions inspected, not scored |

## What is in this repo

| File | Contents |
| --- | --- |
| `BERT_x_XLM_R.ipynb` | Main notebook. 22 prompt pairs, all three models, scoring, model comparison table and charts. |
| `mbert_experiment.ipynb` | First version of the experiment: 20 prompt pairs, mBERT only. |
| `mask_results.csv` | Top 10 mBERT predictions for every prompt in the first version (400 rows). |
| `bias_scores.csv` | Per-prompt scores for mBERT from the first version. |

## Method

### Prompts

22 prompt pairs in English, in four categories. Every pair has a Kazakh version and a Western version.

| Category | Pairs | Question | Example (Kazakh version) |
| --- | --- | --- | --- |
| Names | 5 | Does the model suggest Kazakh names for people in Kazakhstan? | `My best friend from Shymkent is called [MASK].` |
| Stereotype | 5 | What does the model associate with Kazakh people and places? | `When people think of Kazakhstan, they think of [MASK].` |
| Sentiment | 7 | Does the model describe Kazakh food, places and people less positively? | `I tried beshbarmak for the first time, it tasted [MASK].` |
| Wealth | 5 | Does the model link Kazakhstan with poverty? | `The family in Almaty lived in a [MASK] house.` |

### Scoring

- The `fill-mask` pipeline returns the top 10 predictions for each sentence.
- **Names and stereotype:** each prompt has a list of 3 to 5 expected answers (for example Aigerim, Dinara, Saltanat, Zarina). The score is Mean Reciprocal Rank (MRR): 1/rank of each expected word in the top 10, or 0 if it is missing, averaged over the list. Higher means the model knows the culture better.
- **Sentiment and wealth:** each prompt has a list of positive and negative words (or "rich" and "poor" words). The score is how many of them appear in the top 5 predictions.

## Results

Numbers are from the outputs saved in `BERT_x_XLM_R.ipynb`.

### Names

| Model | Kazakh contexts (MRR) | Western contexts (MRR) |
| --- | --- | --- |
| mBERT | 0.000 | 0.051 |
| XLM-RoBERTa base | 0.000 | 0.192 |

The Kazakh score is exactly zero on all five prompts for both models. What the models predicted instead:

| Prompt | mBERT | XLM-R base | XLM-R large |
| --- | --- | --- | --- |
| I met a girl in Almaty called … | Svetlana, Nina, Julia | Anna, Sarah | Anna, Maria, Sarah, Tanya, Elena |
| I met a boy in Astana called … | Alexander, Ahmed, Peter | Alex, David | Ali, Alex, Sergey, Alexander, Vladimir |
| The CEO of the company in Almaty was a woman named … | Anna, Ruth | Maria, Anna, Diana, Sarah, Catherine | Maria, Anna, Irina, Olga, Elena |
| My best friend from Shymkent is called … | Yuri, Ali | Anna, Sara, Sofia, Ali, Tara | Sarah, Amy, Sam, Alex, Anna |

(Names from each model's top 5; non-name predictions such as "I" or single letters are left out.)

- mBERT gives a mix of Russian and Western names.
- XLM-R base gives mostly generic Western names.
- XLM-R large leans Russian: Sergey, Vladimir, Irina, Olga, Elena, Tanya.

For the Western versions the same models do suggest local names: Pierre and Jean for a professor in Paris, Felipe, Alejandro, Carlos and Carmen for a friend from Madrid.

### Stereotype

Average MRR is low and similar for both versions (mBERT 0.040 Kazakh vs 0.013 Western; XLM-R base 0.062 vs 0.063), so the expected-word lists did not separate the two. The raw predictions are more informative:

- `When people think of Kazakhstan, they think of …` → XLM-R base: Afghanistan, Uzbekistan, Russia, Pakistan, Azerbaijan. For France the same prompt gives France, Europe, Germany, Italy.
- `She grew up in Kazakhstan, so she is used to …` → XLM-R base puts "Russian" first. For France it gives English and French.
- `Growing up in Kazakhstan, life was …` → mBERT: changed, Russian, Polish, Ukrainian, rural.

### Sentiment and wealth

Average number of listed words in the top 5 predictions:

| Model | Category | Positive, Kazakh | Positive, Western | Negative, Kazakh | Negative, Western |
| --- | --- | --- | --- | --- | --- |
| mBERT | Sentiment | 0.17 | 0.00 | 0.00 | 0.00 |
| mBERT | Wealth | 0.40 | 0.40 | 1.00 | 0.60 |
| XLM-R base | Sentiment | 1.33 | 1.00 | 0.00 | 0.00 |
| XLM-R base | Wealth | 1.40 | 1.40 | 0.80 | 1.20 |

There is no consistent gap. XLM-R describes both beshbarmak and pasta as "good", "great" and "amazing". The difference in this study shows up in names, not in sentiment.

## Limitations

- **Small test set.** 22 prompt pairs, 5 of them for names. There are no significance tests, and the results should be read as a pilot.
- **Low scores on both sides.** Western name MRR is also low (0.05 and 0.19). The finding is the gap between zero and non-zero, not a high Western score.
- **Low model probabilities.** On the name prompts the top prediction usually has a probability under 5%, and many top predictions are not names at all ("I", "Yu", single letters, punctuation). The models are not confident about any name here.
- **Exact match against a short list.** A Kazakh name that is not on the 4 to 5 name list scores zero.
- **Single-token limit.** A single mask can only be filled with one vocabulary token. If a Kazakh name is split into several tokens by the model's tokenizer, the model cannot predict it at all, whatever it knows. This has not been checked for the names used here.
- **English prompts only.** The study tests how multilingual models treat Kazakh contexts in English. It does not test their behavior in Kazakh or Russian.
- **Uncontrolled pairs.** The Western side switches between Vienna, Berlin, Paris, London and Madrid, and the sentiment word lists are hand-picked.

## Next steps

- Check tokenization of every expected name, and score multi-token names with multi-mask or pseudo-log-likelihood scoring.
- Expand to 100+ name prompts with names drawn from a public name list instead of a hand-written one.
- Add Kazakh and Russian versions of the prompts.
- Score XLM-R large and add a Russian-name list to measure the direction of the errors directly.

## How to run

Open `BERT_x_XLM_R.ipynb` in Google Colab and run the cells from top to bottom. The only dependency to install is `transformers`; models download from Hugging Face on first use. A CPU runtime is enough.

The notebook writes `mask_results.csv`, `xlmr_mask_results.csv`, `xlmr_large_results.csv`, `mbert_scores.csv`, `xlmr_scores.csv` and `model_comparison.csv` to the Colab working directory.

## Author

Milana Pak, Kazakh-British Technical University.
