# Multilingual RAG on the EU AI Act

A small RAG (Retrieval-Augmented Generation) system that answers questions about the **EU AI Act** in **English, French and Russian**, and gives a link to the exact article it used.

It follows the logic of a RAG practical session from my Master's course in NLP, adapted to a legal text and to cross-lingual questions.

## What it does

```
Question: Если компания размещает чат-бота на своём сайте, обязана ли она
          предупреждать посетителей, что они общаются не с живым человеком?
↓ search: the 5 closest passages of the law
↓ answer: the LLM reads them and answers in the language of the question
Answer:  Компания обязана предупреждать посетителей о том, что они общаются
         с искусственным интеллектом (Art. 50 AI Act)
Sources: Art. 50 AI Act  https://eur-lex.europa.eu/eli/reg/2024/1689/oj/eng#art_50
```

## How it works

1. **Prepare the text:** the 113 articles of the AI Act (official English text) are loaded and cut into 197 chunks of at most 512 tokens.
2. **Search:** each chunk is turned into a vector with a multilingual embedding model (`BAAI/bge-m3`) and stored in a FAISS index. For each question, the 5 closest chunks are retrieved.
3. **Answer:** the chunks and the question are given to an LLM (`Qwen/Qwen2.5-3B-Instruct`), which answers in the language of the question or says "I don't know". The links to EUR-Lex are added by the code from the metadata, so they cannot be invented.

## Experiment

The articles are in English, but the questions are in English, French and Russian. Two embedding models were compared:

- `thenlper/gte-small`: English only (the model used in the course)
- `BAAI/bge-m3`: multilingual

Test set (`questions.csv`): 10 questions written in English and translated into French and Russian (30 questions), plus 5 questions whose answer is not in the AI Act. The questions are paraphrased and do not reuse the wording of the law.

**Metric:** Recall@5, the share of questions where the right article is among the 5 retrieved chunks.

## Results

| Recall@5 | gte-small (English only) | bge-m3 (multilingual) |
|---|---|---|
| English | 0.4 | 0.9 |
| French  | 0.3 | 0.9 |
| Russian | 0.0 | 0.9 |

- The English-only model gets worse as the question moves away from English, down to 0/10 in Russian.
- The multilingual model finds the right article for 9 questions out of 10 in all three languages.
- On the generation side, the checked answers give the correct rule, and all 5 questions with no answer get "I don't know". The French and Russian answers contain more errors (wrong article number, wrong legal terms, grammar), which is a limit of the small 3B model.

More details, the questions missed and the limits are in the conclusion of the notebook.

## Files

- `rag_ai_act.ipynb`: the whole project (data, search, LLM, evaluation, conclusion)
- `questions.csv`: the 35 test questions with the number of the article that contains the answer (0 = no answer in the AI Act)

## How to run it

1. Open the notebook in Google Colab.
2. Choose a GPU: Runtime → Change runtime type → **T4 GPU**.
3. Upload `questions.csv` with the folder icon on the left.
4. Runtime → **Run all**.

Everything is free: the models are downloaded and run in Colab, no API key is needed.

## Limits

- Small test set (35 questions): the results are an indication, not a benchmark.
- Only the articles are indexed, not the annexes or recitals.
- The dataset is a snapshot from 20 July 2026 containing the text as adopted in 2024 (Regulation (EU) 2024/1689). The changes made by the "Digital Omnibus on AI" (Regulation (EU) 2026/1744, in force since 27 July 2026) are not included, so some answers may not reflect the current law.
- This is a student project: the answers are not legal advice.

## Tools and data

- Data: [`jeroenherczeg/eu-ai-act`](https://huggingface.co/datasets/jeroenherczeg/eu-ai-act) on Hugging Face (CC BY 4.0), based on the official text on [EUR-Lex](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)
- Libraries: LangChain, FAISS, sentence-transformers, Hugging Face `transformers` and `datasets`, pandas
- Models: `BAAI/bge-m3`, `thenlper/gte-small`, `Qwen/Qwen2.5-3B-Instruct`