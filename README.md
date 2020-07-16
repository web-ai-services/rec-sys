# About

Сервис контекстных рекомендаций, реализованный в несколько этапов:

- Распаралелленная загрузка новых чековых данных из локальной файловой системы (<b>Airflow, multiprocessing</b>) 
- Хранение (<b>Redis, PostgreSQL, Elasticsearch</b>)
- Обработка данных, формирование масок для моделей (<b>sklearn</b>)
- Построение рекомендательных моделей разного назначения, через большой _GridSearch_ с множественным параметрезированием и валидацией несколькими кастомными метриками (<b>Gensim, GloVe, PyTorch</b>)
- API (<b>DjangoRestFramework</b>)
- Визуализация. (<b>Django, HTML+CSS+JS, Plotly</b>)

# Architecture

Все компоненты системы организованы в виде Docker контейнеров. Такая реализация обеспечивает работу подсистем в виде независимых компонентов, каждый из которых возможно заменить при необходимости.

<div align="center">
    <img src=https://i.ibb.co/B3wQgfk/rsz-architecture.jpg">
    <p>GridSearch</p>
</div>

__Airflow__ выступает в роли основного _data-driver_, на нем в первую очередь реализована логика парсинга чековых данных из файловой системы. Далее, с помощью все того же __Airflow__, спарсенные данные, складывается в __PostgreSQL__, который выполняет роль персистентного хранилища для структурированных данных. Его использование обусловлено широкими возможностями данной реляционной БД (среди свободно распространяемых продуктов). После промежуточных расчетов, модификаций и построения рекомендательных моделей, некоторые данные переносятся в __ElasticSearch__, которое выступает как основное хранилище для кэширования определённых результатов вычислений, необходимых для построения рекомендацией и лоукейной визуализации. Данные прошедшие полный pipeline по подготовке, напрямую используются в __REST API__ сервиса и его __Web__ составляющей.

# Data pipeline

Данные из "сырого" транзакционного вида проходят определнный пайплайн по подготовке. Отбросив служебные __DAG__'и, первым запускается _Parser_, rоторый параллельно отсылает, добавляет данные в __PostgreSQL__. Далее отрабатывает _Send_Elastic_, который необходимую информацию из персистентного хранилища складывает в __ElastcSearch__. После, начинаются непосредственные вычисления в _Masker_, нужны они для обучения и валидации модели. В зависимости от длины чека, генерируются _N_  последовательностей, с замаскированными продуктам. Пару слов о метрике, которую мы использовали: &nbsp;&nbsp;&nbsp; ![\frac{1}{Q}\sum \frac{Precision@k(q)}{IdealAP(q)}](https://latex.codecogs.com/gif.latex?%5Cfrac%7B1%7D%7BQ%7D%5Csum%20%5Cfrac%7BPrecision@k%28q%29%7D%7BIdealAP%28q%29%7D), 

здесь _Precision@k(q)_ - доля релевантных товаров в первых k позициях списка рекомендаций, _IdealAP(q)_ - максимально возможное число попаданий. Как только маски посчитаны, запускается большой _GridSearch_ с перебором лучших мета-параметров для модели (размер окна обучения, размер вектора, количество эпох, итд), все метрики записываются в соответствующий индекс, откуда в свою очередь берет данные _Champ_detection_ и выдает модель с лучшим набором показаний и сравнивая ее с имеющейся боевой моделью, заменяет ее, либо оставляет текущую. В __API__ фигурирует, уже отобранная, сохраненная модель.

<div align="center">
    <img src=https://i.ibb.co/VWnkq6R/rsz-1photo5393622920070278229.jpg" >
    <p>GridSearch</p>
</div>

# Recommendation Logic

<div align="center">
    <img src="https://i.ibb.co/SBDPvt2/rsz-clusters-1.jpg">
    <p> Product Clusters </p>
                                                                   
</div>

Итого реализованы 2 вида рекомендаций (имеется персентильное отсеивание по популярности, убираем слишком популярные):

- __Синонимичные__. Модели обученные и провалидированные на продуктах / категориях. Хорошо работает в кейсах рекомендации заменителей. К слову модель обученная и провалидированная на категориях товаров, несколько обобщенная но более точная, в виду softmax на меньшее число объектов (категорий порядка ~1300) и плотной взаимовстречаемости встречаемости. 
- __Комплиментарные__. Детализация по продуктам из рекомендованных категорий устроено следующим образом:
Получаем рекомендации от категориальной модели. С учетом веса категории в предсказании и популярности данной категории, получаем показатели “важности” для новой категории. Применяем Power Transformer к данным показателям, чтобы сгладить их распределение и сделать более гауссовским, то есть нивелировать чрезмерное влияние популярно-весомых категорий. Далее осуществив персентильное отсеивание слишком популярных банальных товаров, и слишком непопулярных редких товаров, осуществляем детализация внутри категорий по числу продуктов полученных от softmax на power transformed весов.

Синонимичные             |  Компилиментарные
:-------------------------:|:-------------------------:
![Синонимичные](https://i.ibb.co/9bJ22VL/photo5397687947702152056.jpg)  |  ![Комплиментарные](https://i.ibb.co/vBYz459/photo5386441223650257916.jpg)

Также стоит сказать, что у нас помимо разных типов рекомендаторов, реализовано разделение на модели по городам, с целью учета региональных специфик. 

# Perspectives

1. 
2. Включить имеющуюся реализацию контекстных рекомендаций, как алгоритм для холодного старта в разрезе пользователя. А в виде основной модели использовать [подход](https://drive.google.com/drive/folders/1zf8rSVU9bHXTkPDAms5bkV9qDdxVpbdN) Антона Протопопова, победившего в Х5 комптишне, [source](https://github.com/aprotopopov/retailhero_recommender). В его подходе 
