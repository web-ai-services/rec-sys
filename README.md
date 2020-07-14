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
    <img src="https://i.ibb.co/SBDPvt2/rsz-clusters-1.jpg">
    <p> Product Clusters </p>
                                                                   
</div>

Итого реализованы 3 вида рекомендаций (имеется персентильное отсеивание по популярности, убираем слишком популярные):

- Синонимичные. Модель обученная и провалидированная на продуктах. Хорошо работает в кейсах рекомендации заменителей.  
Верхнеуровневые категориальные. Модель обученная и провалидированная на категориях товаров, несколько обобщенная но более точная, в виду softmax на меньшее число объектов (категорий порядка ~1300) и плотной взаимовстречаемости встречаемости. 
- Комплиментарные. Детализация по продуктам из рекомендованных категорий устроено следующим образом:
Получаем рекомендации от категориальной модели (пункт 2)
С учетом веса категории в предсказании и популярности данной категории, получаем показатели “важности” для новой категории
Применяем Power Transformer к данным показателям, чтобы сгладить их распределение и сделать более гауссовским, то есть нивелировать чрезмерное влияние популярно-весомых категорий. Далее осуществив персентильное отсеивание слишком популярных банальных товаров, и слишком непопулярных редких товаров, осуществляем детализация внутри категорий по числу продуктов полученных от softmax на power transformed весов.

Также стоит сказать, что у нас помимо разных типов рекомендаторов, реализовано разделение на модели по городам, с целью учета региональных специфик. 

Synonyms             |  Complimentation
:-------------------------:|:-------------------------:
![Синонимичные](https://i.ibb.co/9bJ22VL/photo5397687947702152056.jpg)  |  ![Комплиментарные](https://i.ibb.co/vBYz459/photo5386441223650257916.jpg)
# Perspectives

1. 
2. Включить имеющуюся реализацию контекстных рекомендаций, как алгоритм для холодного старта в разрезе пользователя. А в виде основной модели использовать [подход](https://drive.google.com/drive/folders/1zf8rSVU9bHXTkPDAms5bkV9qDdxVpbdN) Антона Протопопова, победившего в Х5 комптишне, [source](https://github.com/aprotopopov/retailhero_recommender). В его подходе 
