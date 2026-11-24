# CLAUDE.md

Frappe RAG: yerel Qdrant + hibrit arama + reranker + vLLM/Ollama ile PDF soru-cevap (CLI ve FastAPI). Dokümantasyon Türkçe.

## Komutlar
- Kurulum: `pip install -r requirements.txt` (`make install` uv kullanır)
- İndeks: `python frappe_rag.py --index [--force]`; sohbet: `python frappe_rag.py`; API: `python frappe_rag.py --serve` (:8088)
- vLLM: `make vllm` (:8282, GPU gerekir)
- Benchmark: `make benchmark-smoke|benchmark|benchmark-stress|benchmark-spike`, `make visualize`, `make compare`
- Test/lint yok; hızlı kontrol: `python -m compileall -q frappe_rag benchmarks`

## Mimari (`frappe_rag/`)
- `cli.py`: argparse, `--index` ve sohbet akışları; `server.py`: FastAPI (`/health`, `/metrics`, `/query`), ilk açılışta gerekirse otomatik indeksleme
- `config.py`: `AppConfig` dataclass; öncelik CLI > YAML (`--config`) > varsayılan
- `qdrant_store.py`: koleksiyon, sparse vektör, `HybridRetriever`; `reranker.py`: cross-encoder
- `chain.py`: prompt + LLM zinciri, `ensure_ascii_answer` (cevap ASCII'ye çevrilir); `llm.py`, `embeddings.py`, `pdf_ingest.py`, `text_utils.py`, `hashing.py`
- `benchmarks/`: çalışan API'ye karşı yük testi; sonuçlar `results/` ve `reports/` altında izlenir

## Notlar
- Env: `OLLAMA_URL`, `VLLM_BASE_URL` (bkz. `.env.example`). `data/`, `qdrant_db/`, `models/` git dışıdır.
- `server.py` hataları HTTP 200 + `Error:` cevabı olarak döner (davranış, değiştirme).
- Benchmark `make` hedefleri `benchmarks/` içine `cd` ile girer.
