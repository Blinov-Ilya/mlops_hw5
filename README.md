# Мониторинг фрода

## Запуск

Положите `train.csv` рядом с `docker-compose.yaml`. [Данные на Kaggle](https://www.kaggle.com/competitions/teta-ml-1-2025/data)

```sh
cp .env.example .env
docker compose up -d --build
```

Если файл в другом месте, задайте `TRAIN_DATA_CSV` в `.env`. На Windows путь пишется через `/`.

[Grafana](http://localhost:3000) и [Airflow](http://localhost:8081) используют `admin` / `admin`. [Prometheus](http://localhost:9090) собирает метрики каждые 2 секунды.

## Дашборды

- [Поток и скоринг](http://localhost:3000/d/fraud-detection) — скорость, задержка, доля фрода и распределение скоров.
- [Запись в PostgreSQL](http://localhost:3000/d/scoring) — скорость, задержка, число записей с флагом фрода и распределение скоров.
- [Ресурсы Docker](http://localhost:3000/d/node-metrics) — CPU, память, сеть и диск.

## Метрики

- Скоринг — `transactions_total`, `transaction_processing_seconds`, `fraud_score`, `fraud_ratio`, `transaction_preprocessing_seconds`, `model_version_updates_total`, `model_version_last_loaded_timestamp_seconds`.
- Запись — `scorings_total`, `scoring_db_write_seconds`, `fraud_detected_total`, `scoring_fraud_score`.
- Node Exporter — `node_cpu_seconds_total`, `node_memory_MemTotal_bytes`, `node_memory_MemAvailable_bytes`, `node_network_receive_bytes_total`, `node_network_transmit_bytes_total`, `node_disk_read_bytes_total`, `node_disk_written_bytes_total`.

Новая метрика `transaction_preprocessing_seconds` показывает, сколько времени уходит на подготовку признаков. Линия **Подготовка признаков · p95** на графике **Время обработки и подготовки признаков**.

## Проверка

1. Откройте три дашборда. В Prometheus проверьте, что `transactions_total` и `scorings_total` растут. [Цели Prometheus](http://localhost:9090/targets)
2. Запустите в Airflow `retrain_catboost`. После `publish_model` на графике **Скорость потока и смена модели** появится оранжевая полоса на 30 секунд. Версия указана в легенде.

Код скоринга и переобучения взят из [репозитория семинара](https://github.com/NikitaMalykhin/mlops_realtime_fraud_detector_monitoring).
