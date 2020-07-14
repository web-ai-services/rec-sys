# About

Сервис контекстных рекомендаций, реализованный в несколько этапов:

- Распаралелленная загрузка новых чековых данных из локальной файловой системы (<b>Airflow, multiprocessing</b>) 
- Хранение (<b>Redis, PostgreSQL, Elasticsearch</b>)
- Обработка данных, формирование масок для моделей (<b>sklearn</b>)
- Построение рекомендательных моделей разного назначения, через большой _GridSearch_ с множественным параметрезированием и валидацией несколькими кастомными метриками (<b>Gensim, GloVe, PyTorch</b>)
- API (<b>DjangoRestFramework</b>)
- Визуализация. (<b>Django, HTML+CSS+JS, Plotly</b>)

# Architecture

# Data pipeline
<div align="center">
    <img src=https://i.ibb.co/VWnkq6R/rsz-1photo5393622920070278229.jpg" >
    <p>GridSearch</p>
</div>

# Recommendation Logic

<div align="center">
    <img src="https://i.ibb.co/HTxSkW9/rsz-clusters.jpg">
    <p> Product Clusters </p>
                                                                   
</div>

Synonyms             |  Complimentation
:-------------------------:|:-------------------------:
![synonyms](https://i.ibb.co/8X2TSRh/synonyms.jpg)  |  ![complimentation](https://i.ibb.co/xzNWWrS/complimentaions.jpg)
# Perspectives

1. 
2. Включить имеющуюся реализацию контекстных рекомендаций, как алгоритм для холодного старта в разрезе пользователя. А в виде основной модели использовать [подход](https://drive.google.com/drive/folders/1zf8rSVU9bHXTkPDAms5bkV9qDdxVpbdN) Антона Протопопова, победившего в Х5 комптишне, [source](https://github.com/aprotopopov/retailhero_recommender). В его подходе 
