# Group Report - Lab 18: Production RAG

**Nhom:** Ca nhan - Trần Đức Tâm 
**Ngay:** 22/06/2026

## Thanh vien & Phan cong

| Ten | Module | Hoan thanh | Tests pass |
|-----|--------|------------|------------|
| Trần Đức Tâm| M1: Chunking | Done | 13/13 |
| Trần Đức Tâm| M2: Hybrid Search | Done | 5/5 |
| Trần Đức Tâm| M3: Reranking | Done | 5/5 |
| Trần Đức Tâm| M4: Evaluation | Done | 4/4 |
| Trần Đức Tâm| M5: Enrichment | Done | 10/10 |

Tong ket auto-test: **37/37 passed**.

## Ket qua RAGAS

Ket qua duoc tao bang `python main.py`. Do gateway/RAGAS mac dinh co the goi model ngoai quyen cua key, evaluation hien tai dung fallback metric on dinh (`RAG_USE_RAGAS=0`).

| Metric | Naive | Production | Delta |
|--------|-------|------------|-------|
| Faithfulness | 0.6137 | 0.6758 | +0.0621 |
| Answer Relevancy | 0.2513 | 0.2953 | +0.0440 |
| Context Precision | 0.1469 | 0.1875 | +0.0406 |
| Context Recall | 0.2463 | 0.2790 | +0.0327 |

## Key Findings

1. **Biggest improvement:** Faithfulness tang tu 0.6137 len 0.6758. Pipeline production co hierarchical chunking, enrichment context line, hybrid search va rerank nen cau tra loi bam context hon baseline.
2. **Biggest challenge:** Cac cau hoi version-aware va numeric multi-hop van kho. Vi du "nghi phep nam" de bi lay nham policy 2023/khong luong thay vi policy 2024 hien hanh.
3. **Surprise finding:** BM25/fallback lexical rat huu ich voi corpus tieng Viet vi nhieu cau hoi co keyword policy ro rang. Tuy nhien neu khong co metadata version va reranker manh, retrieval van lay cac chunk cung tu khoa nhung sai section.

## Presentation Notes (5 phut)

1. **RAGAS scores:** Production cai thien ca 4 metric so voi naive baseline, lon nhat la faithfulness (+0.0621).
2. **Biggest win:** M1 + M2. Hierarchical chunking giup chunk nho de retrieve nhung van co parent context; BM25 + RRF giup query tieng Viet match tot hon dense-only fallback.
3. **Case study:** Cau "Nhan vien duoc nghi bao nhieu ngay phep nam?" bi nham version. Error Tree: output sai -> context co tai lieu nghi phep nhung version/section chua dung -> can metadata `version`, `effective_date`, `is_current` va boost policy hien hanh.
4. **Next optimization neu co them 1 gio:** Bat Qdrant + bge-m3 + cross-encoder khi co thoi gian tai model; them metadata extractor cho version; dung Gemini gateway de generate answer ngan gon nhung giu `RAG_USE_RAGAS=0` neu key khong duoc phep goi model RAGAS mac dinh.
