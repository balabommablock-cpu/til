# Why naive chunking destroys regulatory tables

When building RAG over SEBI/RBI circulars, I found that standard text splitters (LangChain's RecursiveCharacterTextSplitter, etc.) completely destroy the meaning of regulatory tables.

## The problem

A SEBI circular on mutual fund categorization has a table like:

| Category | Sub-Category | Min Equity % | Benchmark |
|----------|-------------|-------------|------------|
| Equity | Large Cap | 80 | NIFTY 100 TRI |
| Equity | Mid Cap | 65 | NIFTY Midcap 150 TRI |

Naive chunking splits rows from headers. You get a chunk that says "Mid Cap | 65 | NIFTY Midcap 150 TRI" with no context about what those columns mean.

## The fix

Format each row with its headers preserved:

```
Category: Equity | Sub-Category: Mid Cap | Min Equity %: 65 | Benchmark: NIFTY Midcap 150 TRI
```

Each row becomes self-contained. The embedding captures the full meaning. Retrieval quality on table-based questions went from ~30% to ~85% accuracy in my eval.

## Code

See [`regulatory_chunker.py`](https://github.com/balabommablock-cpu/regai/blob/main/src/regulatory_chunker.py) in RegAI for the implementation.
