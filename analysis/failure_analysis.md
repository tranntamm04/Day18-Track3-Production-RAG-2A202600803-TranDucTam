# Failure Analysis - Lab 18: Production RAG

**Sinh vien:** Trần Đức Tâm 
**Pipeline:** M1 hierarchical chunking -> M5 contextual enrichment -> M2 BM25+dense fallback -> M3 rerank fallback -> M4 lexical/RAGAS fallback

## RAGAS Scores

Ket qua ben duoi duoc sinh tu `python src/pipeline.py` o che do local fallback, chua bat `RAG_USE_LLM=1` va chua tai HuggingFace models.

| Metric | Naive Baseline | Production | Delta |
|--------|---------------|------------|-------|
| Faithfulness | 0.6137 | 0.6758 | +0.0621 |
| Answer Relevancy | 0.2513 | 0.2953 | +0.0440 |
| Context Precision | 0.1469 | 0.1875 | +0.0406 |
| Context Recall | 0.2463 | 0.2790 | +0.0327 |

## Bottom-5 Failures

### #1
- **Question:** Luong thu viec cua nhan vien Junior muc cao nhat la bao nhieu?
- **Expected:** Junior cao nhat 20.000.000 VND/thang; luong thu viec = 85% x 20.000.000 = 17.000.000 VND/thang.
- **Got:** Pipeline tra ve chunk noi ve ty le 85% nhung thieu chunk bang luong co muc Junior cao nhat.
- **Worst metric:** context_recall
- **Error Tree:** Output sai/khong du -> Context thieu bang luong -> Query can multi-hop -> Can retrieve ca `thu_viec.md` va `bang_luong_2024.md`.
- **Root cause:** Retrieval lay dung chinh sach 85% nhung chua uu tien bang luong Junior.
- **Suggested fix:** Tang recall cho cau hoi tinh toan bang multi-query expansion: "Junior salary range", "bang luong Junior", "85% luong thu viec".

### #2
- **Question:** Phu cap an trua hang thang la bao nhieu?
- **Expected:** 1.000.000 VND/thang, chi tra cung ky luong.
- **Got:** Cau tra loi dung thong tin chinh, nhung context kem theo co them chunk thu viec va cong tac phi.
- **Worst metric:** context_precision
- **Error Tree:** Output gan dung -> Context dung nhung nhieu noise -> Query OK -> Can rerank/filter tot hon.
- **Root cause:** Hybrid top-k con lay cac chunk cung co tu "phu cap" nhung khac chu de.
- **Suggested fix:** Metadata filter theo category finance/hr va cross-encoder reranker that khi bat `RAG_USE_HF_MODELS=1`.

### #3
- **Question:** Muon mua thiet bi tri gia 55 trieu can ai phe duyet?
- **Expected:** Don hang tren 50.000.000 VND can Tong Giam doc/CEO phe duyet.
- **Got:** Pipeline lay chunk luu y mua sam CNTT, khong lay chunk nguong phe duyet.
- **Worst metric:** answer_relevancy
- **Error Tree:** Output khong tra loi truc tiep -> Context sai section -> Query numeric threshold -> Can section-aware retrieval.
- **Root cause:** Chunk ve "laptop/server/phan mem" co tu khoa gan voi query hon chunk nguong 50 trieu.
- **Suggested fix:** Tang trong so cho so tien/nguong phe duyet, hoac tao HyQA question cho moi chunk nguong phe duyet.

### #4
- **Question:** Bao lau phai doi mat khau mot lan?
- **Expected:** Chinh sach hien hanh v2.0: moi 120 ngay; v1.0 90 ngay da bi thay the.
- **Got:** Cau tra loi dung 120 ngay, nhung context co kem chinh sach cu v1.0.
- **Worst metric:** context_precision
- **Error Tree:** Output dung -> Context co ca version cu -> Query thieu filter "hien hanh" -> Can version ranking.
- **Root cause:** BM25/RRF khong phan biet version moi-cu khi hai chunk co tu khoa rat giong nhau.
- **Suggested fix:** Them metadata `version`, `effective_date`, `superseded` va boost version moi.

### #5
- **Question:** Nhan vien duoc nghi bao nhieu ngay phep nam?
- **Expected:** Chinh sach hien hanh v2024: 15 ngay phep nam; v2023 12 ngay da bi thay the.
- **Got:** Pipeline uu tien chunk nghi phep khong luong/ban cu thay vi section v2024.
- **Worst metric:** context_precision
- **Error Tree:** Output sai -> Context nhieu chunk cung chu de nghi phep -> Query can version-aware retrieval -> Can metadata/version filter.
- **Root cause:** Cac tai lieu nghi phep co lexical overlap cao; fallback reranker chua du manh de chon version moi.
- **Suggested fix:** Dung structure-aware chunking theo header, enrich metadata version, va rerank bang cross-encoder.

## Case Study

**Question chon phan tich:** "Nhan vien duoc nghi bao nhieu ngay phep nam?"

**Error Tree walkthrough:**
1. Output dung? Khong, output bi keo sang nghi phep khong luong hoac version cu.
2. Context dung? Chua du; context co chunk lien quan den nghi phep nhung khong phai policy hien hanh v2024.
3. Query rewrite OK? Chua; can rewrite thanh "so ngay phep nam chinh sach hien hanh v2024".
4. Fix o buoc: retrieval va metadata ranking.

**Neu co them 1 gio, se optimize:**
- Them metadata extractor cho `version`, `effective_date`, `is_current`.
- Bat `RAG_USE_HF_MODELS=1` de dung BAAI/bge-m3 va bge-reranker-v2-m3 thay fallback lexical.
- Giam top-k sau rerank xuong 3 va them rule boost chunk co "hien hanh", "v2024", "ngay hieu luc".
- Bat `RAG_USE_LLM=1` voi Gemini gateway de sinh answer ngan gon thay vi lay raw context dau tien.
