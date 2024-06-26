# Рекомендации

[Датасет](https://drive.google.com/drive/folders/1eScHn6iUgY7llQ3nhg3t3vPVWKqDFApE?usp=sharing)

[Таблица с данными опроса](https://docs.google.com/spreadsheets/d/18rxCwB_cZBFEni10Wx55Z06pGOqvNLx_wq-e9E1iRPI/edit?usp=sharing)

[Список фильмов](https://docs.google.com/document/d/17-54zuvlt5XNFsB12nCKJ-MDnWVqoJn21LGxvbx4vpw/edit?usp=sharing)

[Текстовый проект](https://docs.google.com/document/d/1m4dRg_59v67WiupIAOg5CyRH-mHhGCLbwMwEepR_umk/edit?usp=sharing)

[Презентация](https://docs.google.com/presentation/d/1TZpBLVJMW8KmhkQJILhzaGmq2Ul6GPsf/edit?usp=sharing&ouid=109623147747584209724&rtpof=true&sd=true)

## Используемые библиотеки

- [Polars](https://pola.rs/)
- [Pandas](https://pandas.pydata.org/) (для работы с библиотеками, которые не поддерживают Polars)
- [NumPy](https://numpy.org/)
- [SciPy](https://scipy.org/)
- [SciKit Learn](https://scikit-learn.org/stable/)

# Использованный датасет

- [Сабсет датасета Full MovieLens Dataset](https://www.kaggle.com/datasets/rounakbanik/the-movies-dataset) - содержит
  45000 фильмов, вышедших до 2017 года. Часть данных была получена с TMDB Open API (ключевые слова и титры)

## Требования к запуску

- Гигов 15-20 оперативной памяти,
  либо достаточно большой своп раздел,
  либо большое желание использовать жупитер нот э бок в Google Colaboratory

## Рассмотренные алгоритмы

### Простейший (на базе функции взвешенного рейтинга)

Даёт общие рекомендации каждому пользователю, на основании популярности фильма, либо на его жанре.
Основная идея в том, что фильмы которые более популярны и признаны критиками будут иметь
большую вероятность быть высоко оценёнными среднеми зрителями.

Примером является топ фильмов IMDB.


### По содержимому

Они предлагают похожие фильмы на основе конкретного фильма.
Для создания таких рекомендаций система использует метаданные, такие как жанр, режиссер, описание, актеры и т.д.

Общая идея этих рекомендательных систем заключается в том,
что если человеку нравится определенный фильм, то ему также понравится и похожий на него фильм.
И чтобы порекомендовать его, система использует метаданные о фильмах, просмотренных пользователем в прошлом.

Хорошим примером может послужить YouTube, где на основе вашей истории он предлагает вам новые видео,
которые вы могли бы посмотреть.

#### На базе анализа выделения ключевых слов с помощью векторизатора TF-IDF

Внутри датафрейма метаданных фильмов есть столбец overview, в котором содержится описание фильма.

Сперва допустим, что у нас нет никаких тегов для категоризации фильмов - только описание.

Данная задача - это задача `обработки естественного языка` **(NLP)**.
Следовательно, нам необходимо извлечь некоторые характеристики из приведенных выше текстовых данных,
прежде чем вычислять сходство и/или несходство между ними.
Невозможно вычислить сходство между двумя обзорами в их необработанном виде.
Для этого необходимо вычислить векторы слов каждого обзора или документа, как это будет называться в дальнейшем.

`Векторы слов` - это векторное представление слов в документе.
Векторы несут в себе семантическое значение. Например, слова man & king будут иметь векторные представления,
близкие друг к другу, а слова man & woman - далекие друг от друга.

Мы рассчитаем векторы `Term Frequency-Inverse Document Frequency (TF-IDF)` для каждого документа.
Это даст нам матрицу, в которой каждый столбец представляет слово из словаря обзора
(все слова, которые встречаются хотя бы в одном документе), а каждый столбец представляет фильм, как и раньше.

По сути, показатель TF-IDF - это частота встречаемости слова в документе, уменьшенная на количество документов,
в которых оно встречается. Это делается для того, чтобы уменьшить важность слов,
которые часто встречаются в сюжетных обзорах, и, следовательно, их значимость при вычислении итогового балла сходства.

Мы будем использовать `косинусоидальное сходство` для вычисления числовой величины,
обозначающей сходство между двумя фильмами.
Мы используем косинусоидальное сходство, поскольку оно не зависит от величины,
а также относительно легко и быстро вычисляется
(особенно при использовании в сочетании с показателями TF-IDF).


Поскольку мы использовали векторизатор TF-IDF,
вычисление точечного произведения между каждым вектором напрямую даст вам оценку косинусного сходства.
Поэтому мы будем использовать функцию sklearn `linear_kernel()` вместо `cosine_similarities()`, так как она быстрее.


#### На базе анализа ключевых слов датасета

Качество рекомендателя повысится, если использовать более качественные метаданные и
улавливать больше мелких деталей.
Именно поэтому нам нужны данные о съемочной группе, ключевые слова
и другая информация из метаданных фильмов.

