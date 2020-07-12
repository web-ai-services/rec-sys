# About

Сервис контекстных рекомендаций, реализованный в несколько этапов:

- Распаралелленная загрузка новых чековых данных из локальной файловой системы (<b>Airflow, multiprocessing</b>) 
- Хранение (<b>Redis, PostgreSQL, Elasticsearch</b>)
- Обработка данных, формирование масок для моделей (<b>sklearn</b>)
- Построение рекомендательных моделей разного назначения (комплиментарные модели, синонимичные модели), через большой GridSearch с множественным параметрезированием и валидацией через несколько кастомных метрик (<b>Gensim, GloVe, PyTorch</b>)
- API (<b>DjangoRestFramework</b>)
- Визуализация. (<b>Django, HTML+CSS+JS, Plotly</b>)

# Architecture

# Data pipeline

# Recommendation Logic

# Perspectives

1. 
2. Включить имеющуюся реализацию контекстных рекомендаций, как алгоритм для холодного старта в разрезе пользователя. А в виде основной модели использовать [подход](https://drive.google.com/drive/folders/1zf8rSVU9bHXTkPDAms5bkV9qDdxVpbdN) Антона Протопопова, победившего в Х5 комптишне, [source](https://github.com/aprotopopov/retailhero_recommender). В его подходе 
