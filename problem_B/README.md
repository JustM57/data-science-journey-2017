# Задача B: построение вопрос-ответной системы

В этой задаче участникам предстоит по парам из параграфов текста и поставленным по ним релевантным вопросам найти в параграфе точный ответ в виде подстроки параграфа. В качестве целевой метрики используется (Macro-Averaged) F1-score.


## Примеры решения

- [`simple-baseline`](simple-baseline/): решение на основе простой эвристики
- [`ml-baseline`](ml-baseline/): решение с использованием машинного обучения
- [`drqa-baseline`](drqa-baseline/) [[архив]](https://sdsj.ru/drqa_baseline.zip): решение на основе deep learning модели [DrQA Reader](https://github.com/facebookresearch/DrQA) (от Facebook AI Research)


## Описание формата решения
Каждое решение должно быть оформлено в виде zip-архива, в корне которого должен быть metadata.json.
Пример:
```json
{
  "image": "sberbank/sdsj-python",
  "entrypoint": "python3 predict.py"
}
```
В нем есть два обязательных поля `entrypoint` - команда для запуска внутри docker контейнера. `image` - образ докера, который будет использоваться для запуска контейнера. Указывать можно любые образы доступные на docker hub или те, которые лежат в папке [dockers](dockers/). Остальные файлы в архиве доступны на использование в процессе исполнения вашего решения.

## Ограничения
Каждому решению отводится ограниченное количество ресурсов: оперативной памяти доступно 8 гигабайт и два ядра.

Ограничение на размер упакованного и распакованного архива 1гб.

Размер файла с предсказаниями не должен превышать 52 мегабайта (решение, в котором в качестве ответа проставляется полный параграф).

На построение предсказания отводится 20 минут (включая чтение своих моделей, данных и запись предсказания).

## Тестирование решения без отправки в систему
1. Прежде всего необходимо разделить данные на трейн/валидейт, сделать это можно следующим скриптом: `python3 split_train.py path_to_train.csv`. В результате, в этой же папке, где находится скрипт будут созданы два файла: `train_without_validate.csv`, `validate.csv`. Теперь обучаться будет на первом файле, а оценивать работу решения по второму.

2. Создаете новое решение: например, добавляете несколько новых признаков в `ml-baseline/taskB_ml_baseline_utils.py:FeatureMaker`. Снова обучаете модель, но уже с новыми признаками и сохраняете сопутствующие данные (модель предсказания и IDF-веса слов, и другие данные).

3. Чтобы прогнать решение можно запустить следующий скрипт: `python3 check_solution.py -t docker --submission_folder ml-baseline --data_file validate.csv`, данные скрипт запустит ваше решение наподобие того, как это работает на платформе: запустит ваше решение в докере (требуется установленный и запущенный докер, и ряд библиотек для питона (см. requirements.txt)) на файле `validate.csv`, решением данный скрипт считает все что находится в папке `ml-baseline` и будет запущен `predict.py` от туда.

Другие варианты запуска:
`python3 check_solution.py -t simple --submission_folder ml-baseline --data_file validate.csv` запустит ваше решение без докера.

`python3 check_solution.py -t simple --submission_file output_ml.zip --data_file validate.csv` запустит ваше решение без докера, решение будет взято из файла `output_ml.zip` (аналогичная опция работает и для запуска на докере).

По окончанию применения скрипта будет выведена строка вида: `{'f1': 0.3361121166011774}`

## Для тех кто привык работать с `Jupyter Notebook`
Чтобы каждый раз не копировать код из ноутбуков в `py`-файлы можно воспользоваться библиотекой dill, она в отличие от pickle позволяет сохранять не только данные и объекты, но и функции/классы/ламбды. Тем самым вы можете написать простой predict.py, который
1. импортирует все необходимые стандартные библиотеки, которые вы собираетесь использовать
2. загружает ваш код, который вы предварительно сериализовали: `code = dill.load(...)`
3. делает код доступным: `for obj_name in code: globals()[obj_name] = code[obj_name]`
4. далее следует обычный код загрузки тестовых данных, построению признаков и применению моделей, который будет меняться намного реже

URGENT: для работы выше описанного решения рекомендуется использовать одиннаковые версии питона, в противном случае могут быть проблемы с совместимостью.