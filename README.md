# Frappe RAG

PDF belgeleri üzerinde soru-cevap yapan yerel bir RAG (Retrieval-Augmented Generation) uygulaması. PDF'ler parçalanıp Qdrant'ta (yerel, dosya tabanlı) hibrit (dense + sparse) olarak indekslenir; sorgularda hibrit arama ve cross-encoder reranker ile bağlam seçilir, cevap vLLM veya Ollama üzerindeki bir LLM ile üretilir. Repoda ayrıca API için yük/benchmark araçları bulunur.

## Özellikler

- PDF yükleme, temizleme ve parçalama (varsayılan 800 karakter, 120 örtüşme)
- Hibrit arama: dense embedding (`BAAI/bge-m3`) + TF tabanlı sparse vektör, RRF ile birleştirme
- Reranker: `BAAI/bge-reranker-v2-m3`
- LLM backend: vLLM (varsayılan, OpenAI uyumlu API) veya Ollama
- İnteraktif CLI sohbeti ve FastAPI sunucusu (`/health`, `/metrics`, `/query`)
- Benchmark araçları: smoke / load / stress / spike testleri, CSV dışa aktarma, görselleştirme

## Teknoloji

Python, LangChain, Qdrant (yerel), sentence-transformers / HuggingFace, vLLM / Ollama, FastAPI + Uvicorn, matplotlib.

## Kurulum

GPU'lu bir makine ve çalışan bir LLM backend gerekir (vLLM için CUDA).

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt     # veya: make install (uv kullanır)
cp .env.example .env                # isteğe bağlı
cp config.example.yaml config.yaml  # isteğe bağlı; --config config.yaml ile verilir
```

PDF dosyalarınızı `data/` klasörüne koyun.

## Kullanım

```bash
make vllm                 # Terminal 1: vLLM sunucusu (:8282)
python frappe_rag.py --index            # PDF'leri indeksle (make index)
python frappe_rag.py --index --force    # İndeksi sıfırla (make index-force)
python frappe_rag.py                    # İnteraktif sohbet (make chat)
python frappe_rag.py --serve            # API sunucusu :8088 (make serve)
```

API örneği:

```bash
curl -X POST http://localhost:8088/query \
  -H "Content-Type: application/json" \
  -d '{"question": "Agile nedir?"}'
```

Tüm seçenekler için `python frappe_rag.py --help`; tüm make hedefleri için `make help`. Mimari akış diyagramları `run.txt` içindedir.

## Benchmark

API sunucusu çalışırken:

```bash
make benchmark-smoke   # 1-2 kullanıcı
make benchmark         # load: 5-30 kullanıcı
make benchmark-stress  # 50-100 kullanıcı
make benchmark-spike   # 150-200 kullanıcı
make visualize         # sonuçları görselleştir
make compare           # sonuçları karşılaştır
```

Çıktılar `benchmarks/results/` ve `benchmarks/reports/` altına yazılır.

## Proje yapısı

```
frappe_rag.py          # giriş noktası (frappe_rag.cli.main)
frappe_rag/            # paket: cli, config, server, chain, qdrant_store, embeddings, reranker, llm, pdf_ingest, text_utils
benchmarks/            # yük testleri, karşılaştırma ve görselleştirme betikleri
config.example.yaml    # örnek yapılandırma
Makefile               # kısayol komutlar
run.txt                # mimari ve çalıştırma kılavuzu
```

## Testler

Repoda otomatik birim testi yoktur; doğrulama için benchmark araçları (`make benchmark-smoke`) kullanılır.
