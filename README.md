# CondBERT for style transfer tasks in different domains

## [Text Detoxification using Large Pre-trained Neural Models](https://aclanthology.org/2021.emnlp-main.629/): О чем статья
Авторы предлагают два метода детоксификации для непараллельных данных: ParaGeDI и CondBERT. Оба подхода основываются на детекции характерных для исходного стиля токенов и их замене на токены целевого стиля. Мы решили проверить, насколько хорошо CondBERT справляется с задачами TST в других категориях.
## CondBert: алгоритм работы
Авторы обучили логистическую регрессию на задаче классификации токсичных текстов, использовав слова качестве признаков. Таким образом, для каждого слова они получили скор, определяющий важность данного слова при классификации.  
Для каждого токена входного предложения рассчитывается его токсичность. Если она превышает заданный порог, токен заменяется на [MASK], и BERT предсказывает новый.


Стоит также отметить следующее:
1. BERT также может предлагать токсичные токены в качестве новых, поэтому авторы рассчитали значение токсичности для каждого BERT-токена и штрафовали модель за предсказывание слов с высоким скором.
2. При помощи последовательной BeamSearch-генерации токенов модель получила возможность предсказывать несколько токенов на месте одного маскированного.
## Что решили сделать мы
Оценить эффективность предложенного авторами подхода CondBert для других текстовых доменов. Это мотивировано тем, что токсичная речь в значительной степени маркирована и характеризуется специфичной лексикой, что не всегда верно для других, менее явных стилей.
Для оценки мы выбрали следующие стили: вежливость (Politeness), упрощение (Simplicity), нейтральность (Neutrality) и “шекспиризация” (Shakespearean English to Modern English).
## Мотивация выбора метода
Результаты, полученные с помощью ParaGeDi и CondBERT для детоксификации, стали SOTA в этой задаче. Они показывают примерно одинаковые результаты при автоматическом подсчете метрик, но CondBERT быстрее и проще в имплементации. Было бы интересно посмотреть, насколько этот пайплайн применим к другим типам текстов.
## Ограничения
Существует очень мало параллельных датасетов для переноса стиля, и мы, как и авторы статьи, не могли сразу воспользоваться более-менее традиционными метриками – BLEU, METEOR или ROGUE (BLEU в результатах у нас - сравнение исходного и сгенерированного предложения). Теоретически, в дальнейших исследованиях будет сложно сравнивать модели, обученные на параллельных и непараллельных данных.


[Оригинальный репозиторий](https://github.com/s-nlp/detox/tree/main/emnlp2021/style_transfer/condBERT) статьи


## Politeness
### Кто делал


Полина Черноморченко


### Датасет
[Politeness transfer dataset](https://github.com/tag-and-generate/politeness-dataset/blob/master/README.md)


### Описание результатов и метрики
Метрики:
| ACC | SIM | FL | J | BLEU |
| --- | --- | -- | - | ---- |
|0.9420|0.7374|0.8170|0.5768|0.6783|

Выводы:\ 
Я ожидала, что модель справиться с вежливостью хуже, чем с токсичностью, потому что невежливость менее выражена на лексическом уровне. Тем не менее, моя гипотеза не опрадалась - скор J для вежливости даже выше, чем у детоксификации. Кажется, невежливость выражается в лексическом плане не очень очевидным  (для носителя в моем лице) образом: самое "невежливое" слово  по версии логрега - "why".

Пример:\
Оригинал: why it has always failed and why it will fail again by caleb carr holy war , inc. :\
CondBERT: i know that it has always been possible , and i know that it will fail again by caleb . the end of our world history , inc . :



…

## Simplicity
### Кто делал


Полина Машковцева


### Датасет


[OneStop](https://huggingface.co/datasets/onestop_english), [TurkCorpus](https://paperswithcode.com/dataset/turkcorpus)


### Описание результатов и метрики
Метрики:
| ACC | SIM | FL | J | BLEU |
| --- | --- | -- | - | ---- |
|1.0000|0.8678|0.7310|0.6495|0.8115|

Данные достались непростые. Их не очень мало для задачи переноса стиля, однако в процессе инференса я пришла к нескольким выводам.

1. Очень много вопросов к основному датасету (OneStop). Многие предложения, размеченные как самые простые (класс 0), на самом деле часто тянут на 1 или даже 2.
2. Классификатор очень плохо обучался - accuracy по итогам двух эпох при нескольких запусках была от 0.5 до 0.85. В нашем случае это говорит как о неважном качестве датасета, так и о специфике задачи упрощения. С добавлением предложений из TurkCorpus становилось немного получше, но все равно не так хорошо, как в других доменах.
3. Помимо "упрощения" слов, не привязанных к какой-либо тематике (*appealing*, *scrunity*, *observation*), есть и термины из разных сфер - науки, политики, социальной сферы и др. В датасете они тоже намешаны все вместе; модель неплохо справляется с "общепринятыми" словами, но с последним у нее проблемы, в том числе и из-за недостаточного количества данных для обучения. К сожалению, по simplicity довольно мало открытых датасетов, тем более с разделением на тексты с разной терминологией.
4. Из-за того же недостатка данных у модели начинаются галлюцинации при попытке переписать предложения с multiword-параметром. Ср:
* исходное предложение: so social media, which can open you up to the scrutiny and analysis of others, is not that appealing to her.
* single-word translation: so webmedia , which can open you about to the observation and observation of others , is not that attractive to her .
* miltiword translation: but studying different information information , because they do not opens many things because they respond more quickly too quickly because they make judgment mistakes because they analyze too many different things , does not make someone more interesting because they study many things .

Метрики выше - результат single-word translation, потому что очевидно они будут лучше. К тому же, miltiword-инференс занимает почти 8 часов на 1000 примеров. **TBD: добавлю метрики с miltiword**


## Neutrality
### Кто делал


Саша Шахнова


### Датасет


[Wiki Neutrality Corpus](http://bit.ly/bias-corpus)


### Описание результатов и метрики


…


## Shakespearean English to Modern English
### Кто делал


Таня Перевощикова


### Датасет


[Shakespearify](https://www.kaggle.com/datasets/garnavaurha/shakespearify)


### Описание результатов и метрики
Метрики:
| ACC | SIM | FL | J | BLEU |
| --- | --- | -- | - | ---- |
|0.8160|0.8425|0.5130|0.3624|0.7651|

Пример:  
Оригинал: thou know’st the mask of night is on my face , else would a maiden blush bepaint my cheek for that which thou hast heard me speak tonight .  
CondBert: do you know ’ , as the mask of night is on my face , else would a white - hot blush bepaint my cheek for that which you surely know ? you heard me speak tonight .

## Выводы
