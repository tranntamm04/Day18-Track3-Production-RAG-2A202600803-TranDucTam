# Individual Reflection - Lab 18

**Ten:** Trần Đức Tâm
**Module phu trach:** M1-M5 ca nhan

## 1. Mapping bai giang vao code

| Lecture Concept | Module | Ham/Class cu the | Observation |
|----------------|--------|------------------|-------------|
| Semantic chunking | M1 | `chunk_semantic()` | Tach cau va gom theo cosine similarity; co fallback token cosine de test khong phu thuoc model download. |
| Hierarchical chunking | M1 | `chunk_hierarchical()` | Tao parent chunk de giu context va child chunk nho hon de retrieve chinh xac hon. |
| BM25 + Dense fusion | M2 | `BM25Search`, `DenseSearch`, `reciprocal_rank_fusion()` | BM25 tot cho keyword tieng Viet; RRF hop nhat BM25 va dense/fallback ma khong can normalize score. |
| Cross-encoder reranking | M3 | `CrossEncoderReranker.rerank()` | Rerank top results theo cap query-document; fallback lexical giup pipeline chay khi chua tai model. |
| RAGAS 4 metrics | M4 | `evaluate_ragas()` | Tra ve faithfulness, answer_relevancy, context_precision, context_recall; fallback lexical cho moi truong chua co LLM key. |
| Contextual enrichment | M5 | `contextual_prepend()`, `_enrich_single_call()` | Them 1 dong ngu canh truoc chunk va tao summary/questions/metadata de giam vocabulary gap. |

## 2. Kho khan va cach giai quyet

- **Loi gap phai:** `No module named pytest`
- **Debug:** Tao `.venv` bang `uv venv --python 3.11 .venv`, cai `requirements.txt`, sau do cai them `pytest`.
- **Cach giai quyet:** Chay `uv pip install -r requirements.txt --python .venv\Scripts\python.exe` va `uv pip install pytest --python .venv\Scripts\python.exe`.

- **Loi gap phai:** `UnicodeEncodeError: 'charmap' codec can't encode character '\u2713'`
- **Debug:** Pipeline chay toi print status thi loi do PowerShell dung cp1252.
- **Cach giai quyet:** Doi cac ky hieu runtime sang ASCII va reconfigure stdout/stderr UTF-8 trong `main.py`, `naive_baseline.py`, `src/pipeline.py`.

- **Loi/rui ro gap phai:** Model HuggingFace va LLM co the tu download/goi API lam test cham hoac fail.
- **Debug:** M5 test rieng cho thay dang fallback sau `Connection error`.
- **Cach giai quyet:** Them cong tac `RAG_USE_HF_MODELS=1` va `RAG_USE_LLM=1`; mac dinh test chay local, khi co API/model thi bat che do production.

## 3. Ket qua kiem thu

- `pytest tests -v`: 37/37 passed.
- `python src/pipeline.py`: chay end-to-end va tao `ragas_report.json`.
- Production fallback scores:
  - Faithfulness: 0.6758
  - Answer Relevancy: 0.2953
  - Context Precision: 0.1875
  - Context Recall: 0.2790

## 4. Action Plan cho project ca nhan

## Project: Tro ly hoi dap quy dinh noi bo

### Hien tai
- RAG pipeline hien tai: load markdown/PDF text layer, chunk hierarchical, enrich contextual, hybrid search, rerank, generate answer.
- Known issues: version cu-moi de bi tron; cau hoi numeric/multi-hop can lay nhieu chunk; fallback lexical chua du tot bang embedding/reranker that.

### Plan ap dung
1. [ ] Chunking strategy: dung hierarchical + structure-aware de khong cat giua section va van tra ve du context.
2. [ ] Search: dung BM25 + dense BAAI/bge-m3 + RRF vi corpus tieng Viet can ca keyword va semantic.
3. [ ] Reranking: bat cross-encoder `BAAI/bge-reranker-v2-m3` cho top-20 -> top-3.
4. [ ] Evaluation: dung RAGAS khi co `RAG_USE_LLM=1`, them custom exact-match cho cau hoi numeric/version.
5. [ ] Enrichment: dung combined single-call voi Gemini gateway de tao summary, HyQA, context va metadata version.

### Timeline
- Tuan 1: Chuan hoa metadata `source`, `version`, `effective_date`, `category`.
- Tuan 2: Bat dense/Qdrant va reranker that, benchmark latency.
- Tuan 3: Chay RAGAS voi API key, phan tich bottom-10 failures.
- Tuan 4: Toi uu prompt answer va viet bao cao ket qua.
