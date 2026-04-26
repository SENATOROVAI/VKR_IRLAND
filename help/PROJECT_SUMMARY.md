# Суммаризация проекта ВКР

## 1. Коротко о проекте

Проект посвящен автоматической классификации и последующему подсчету сельскохозяйственных животных на изображениях. Основной фокус - два класса:

- `cattle` - коровы;
- `sheep` - овцы.

На текущем реализованном этапе решена задача **crop-based classification**: модель получает уже вырезанное изображение одного животного и определяет, корова это или овца.

Следующий этап - **object detection и counting**: модель YOLOv8 должна находить всех животных на полном изображении, после чего количество найденных bounding boxes можно использовать как predicted count.

## 2. Главная идея

В проекте есть две разные задачи:

| Задача | Что получает модель | Что возвращает модель |
|---|---|---|
| Classification | Crop одного животного | `cattle` или `sheep` |
| Detection | Полное изображение | Bounding boxes, классы, confidence |
| Counting | Результаты detection | Количество животных |

Классификация уже реализована и сравнена на четырех моделях. Детекция YOLOv8 и подсчет животных обозначены как следующая часть работы.

## 3. Использованные данные

В проекте объединены два источника данных:

1. **WAID**  
   Wildlife Animals Image Dataset с Roboflow.  
   Это дроновые изображения, где животные часто маленькие и находятся в полном кадре с большим количеством фона.

2. **COCO 2017 cow/sheep subset**  
   Подмножество COCO, скачанное через FiftyOne.  
   Это ground-level изображения, где объекты обычно крупнее и сняты с другой перспективы.

Основные числа:

| Показатель | Значение |
|---|---:|
| WAID raw images | 14 366 |
| COCO cow/sheep images | 3 393 |
| Всего raw images | 17 759 |
| Images после фильтрации cattle/sheep | 12 232 |
| Train images | 8 562 |
| Validation images | 1 835 |
| Test images | 1 835 |
| Classification crops всего | 51 320 |
| Train crops | 35 806 |
| Validation crops | 7 770 |
| Test crops | 7 744 |

## 4. Почему были выбраны только cattle и sheep

WAID содержит больше классов, например `camelus`, `kiang`, `seal`, `zebra`, `cattle`, `sheep`.

Для задачи мониторинга сельскохозяйственных животных были оставлены только:

- `cattle`;
- `sheep`.

Остальные классы были удалены, потому что они не соответствуют предметной области farm livestock monitoring.

## 5. Что показал EDA

Exploratory Data Analysis был нужен не просто для графиков, а для принятия решений по preprocessing.

Главные выводы:

- WAID и COCO сильно отличаются по природе изображений.
- WAID содержит дроновые снимки фиксированного размера `640x640`.
- COCO содержит изображения разных размеров и с другой перспективой.
- В WAID bounding boxes обычно намного меньше, чем в COCO.
- Corrupted files не найдено: `0` из `17 759`.
- Outliers по brightness/std были единичными.
- Из-за большого количества фона и маленьких объектов classification лучше делать по crops, а не по полному изображению.

## 6. Preprocessing pipeline

Preprocessing превратил сырые данные в формат, пригодный для обучения.

Основные шаги:

1. Отфильтровать только изображения с `cattle` и `sheep`.
2. Объединить WAID и COCO в единый формат.
3. Привести class mapping к единому виду:

```text
cattle = 0
sheep = 1
```

4. Сделать stratified split:

```text
train = 70%
validation = 15%
test = 15%
```

5. Создать crop-изображения животных по bounding boxes.
6. Подготовить labels для subgroup analysis:
   - lighting;
   - occlusion.

Важно: split был сделан на уровне исходных изображений, а не на уровне crops. Это защищает от data leakage.

## 7. Почему важен image-level split

Если сначала нарезать crops, а потом случайно разделить их на train/test, crops из одного и того же исходного изображения могут попасть в разные split.

Это будет data leakage:

```text
image_01_crop_1 -> train
image_01_crop_2 -> test
```

В таком случае test set уже не является независимым.

Правильный подход:

```text
сначала split исходных изображений
потом crop generation внутри каждого split
```

Именно такой подход использован в проекте.

## 8. Crop classification

Для классификации использовались вырезанные изображения животных.

Идея:

```text
crop image -> classifier -> cattle/sheep
```

Классификатор не ищет животное на полном кадре. Он получает уже локализованный объект.

Это позволяет модели фокусироваться на признаках самого животного:

- форма тела;
- шерсть;
- контур;
- визуальные отличия коров и овец.

А не на фоне:

- поле;
- трава;
- небо;
- особенности дроновой или наземной съемки.

## 9. Обученные модели

В проекте обучены и сравнены четыре classification модели:

1. **Custom CNN**  
   Собственная сверточная нейросеть, обученная с нуля.  
   Используется как baseline.

2. **ResNet50**  
   ImageNet-pretrained модель в режиме feature extraction.

3. **VGG16**  
   ImageNet-pretrained модель в режиме feature extraction.

4. **MobileNetV2**  
   Компактная ImageNet-pretrained модель в режиме feature extraction.

## 10. Custom CNN baseline

Custom CNN была построена как собственный baseline.

Зачем нужен baseline:

- чтобы сравнить pretrained модели с моделью, обученной с нуля;
- чтобы показать, что результат не зависит только от готовых ImageNet features;
- чтобы выполнить требование иметь собственную архитектуру.

Для подбора hyperparameters использовалась Optuna.

Лучшие параметры Custom CNN:

| Hyperparameter | Значение |
|---|---|
| Conv blocks | 3 |
| Base channels | 64 |
| Kernel size | 3 |
| Activation | ReLU |
| Optimizer | Adam |
| Learning rate | 1.66e-3 |
| Batch size | 32 |
| Dropout | 0.018 |

Результат Custom CNN:

| Metric | Value |
|---|---:|
| Accuracy | 0.9610 |
| Precision | 0.9610 |
| Recall | 0.9606 |
| F1 | 0.9608 |

## 11. Transfer learning

Transfer learning использовался для сравнения собственного baseline с ImageNet-pretrained моделями.

В проекте применялся режим **feature extraction**:

- pretrained backbone заморожен;
- обучается только новая classification head;
- модель адаптируется к задаче `cattle` vs `sheep`.

Это дешевле и быстрее, чем full fine-tuning, но может ограничивать адаптацию модели к новому домену.

## 12. Сравнение моделей

Финальные результаты classification:

| Model | Test accuracy | Precision | Recall | F1 | Total params | Trainable params |
|---|---:|---:|---:|---:|---:|---:|
| VGG16 | 0.9738 | 0.9744 | 0.9730 | 0.9736 | 17.9M | 3.2M |
| Custom CNN | 0.9610 | 0.9610 | 0.9606 | 0.9608 | 8.8M | 8.8M |
| ResNet50 | 0.9570 | 0.9567 | 0.9570 | 0.9568 | 23.8M | 0.26M |
| MobileNetV2 | 0.9498 | 0.9496 | 0.9495 | 0.9496 | 2.4M | 0.16M |

Лучший результат показала **VGG16**.

Главный вывод: pretrained модели не всегда автоматически лучше собственной CNN. В этом проекте VGG16 обогнала baseline, но ResNet50 и MobileNetV2 оказались немного хуже Custom CNN.

## 13. Почему VGG16 оказалась лучшей

VGG16 показала лучший результат не потому, что она всегда лучше ResNet50 или MobileNetV2.

Главная причина в конкретном setup:

- backbone был frozen;
- обучалась только classification head;
- у VGG16 head получилась намного больше;
- у VGG16 было около `3.2M` trainable parameters;
- у ResNet50 было около `0.26M` trainable parameters;
- у MobileNetV2 было около `0.16M` trainable parameters.

То есть VGG16 получила больше capacity для адаптации к задаче.

## 14. Аугментации

Для обучения использовались базовые train-аугментации:

- horizontal flip;
- small rotation;
- color jitter.

Для validation/test использовались только:

- resize;
- normalization.

Был проведен ablation experiment без augmentation.

Результат:

| Setup | Test accuracy |
|---|---:|
| With augmentation | 0.9610 |
| Without augmentation | 0.9548 |

Вклад augmentation:

```text
+0.0062 test accuracy
```

Вывод: augmentation дала небольшое, но измеримое улучшение generalization.

## 15. Grad-CAM

Grad-CAM использовался для визуальной интерпретации моделей.

Он помогает понять, на какие области изображения модель обращает внимание при принятии решения.

Хороший результат:

```text
heatmap на теле животного
```

Плохой сигнал:

```text
heatmap на фоне, траве, небе или краю изображения
```

В проекте Grad-CAM показал, что модели в основном смотрят на тело животного, то есть используют релевантные визуальные признаки.

Важно: Grad-CAM не является строгим доказательством корректности модели. Это supporting evidence для interpretability.

## 16. Почему classification не решает counting

Классификация работает так:

```text
одно вырезанное животное -> cattle/sheep
```

Но counting требует другого:

```text
полное изображение -> найти всех животных -> посчитать их
```

Классификатор не умеет сам находить животных на полном изображении. Он предполагает, что объект уже найден и вырезан.

Поэтому для counting нужна object detection модель.

## 17. Зачем нужна YOLOv8

YOLOv8 нужна для следующего этапа: detection и counting.

Pipeline будет таким:

```text
full image
  -> YOLOv8
  -> predicted bounding boxes cattle/sheep
  -> count predicted boxes
  -> compare with ground truth count
```

Метрики для YOLOv8 detection:

- `mAP@50`;
- `mAP@50-95`.

Метрики для counting:

- `MAE`;
- `R²`.

Важно: в текущих notebooks 01-05 YOLOv8 metrics еще не получены, поэтому нельзя заявлять готовое качество counting.

## 18. Что уже сделано по notebooks

### Notebook 01: Data Acquisition

Скачаны и подготовлены источники данных:

- WAID;
- COCO cow/sheep subset.

Итого получено `17 759` raw images.

### Notebook 02: EDA

Проведен анализ данных:

- class distribution;
- image sizes;
- pixel statistics;
- brightness/std outliers;
- corrupted files scan;
- bbox statistics;
- visual samples.

Главный вывод: данные пригодны, но WAID и COCO имеют заметный domain shift.

### Notebook 03: Preprocessing

Выполнено:

- filtering to cattle/sheep;
- merge WAID + COCO;
- stratified split 70/15/15;
- crop generation;
- subgroup labels для lighting/occlusion.

### Notebook 04: Custom CNN Baseline

Реализовано:

- собственная CNN;
- Optuna tuning;
- final training;
- test metrics;
- Grad-CAM;
- ablation без augmentation.

### Notebook 05: Transfer Learning

Реализовано:

- ResNet50;
- VGG16;
- MobileNetV2;
- feature extraction;
- Optuna tuning;
- test evaluation;
- Grad-CAM;
- comparison table.

## 19. Что осталось сделать

Следующие логичные этапы:

1. **Notebook 06: YOLOv8 Detection + Counting**
   - обучить YOLOv8 на full images;
   - посчитать `mAP@50`, `mAP@50-95`;
   - посчитать count predictions;
   - оценить `MAE` и `R²`;
   - показать примеры detection.

2. **Notebook 07: Comparative Analysis**
   - объединить все classification и detection результаты;
   - построить confusion matrices;
   - провести McNemar tests;
   - сделать subgroup accuracy по lighting и occlusion;
   - проверить гипотезы;
   - сформулировать final conclusions.

## 20. Ограничения проекта

Основные ограничения:

- пока реализована classification, а не полный counting pipeline;
- только два класса: `cattle` и `sheep`;
- transfer learning использовал frozen backbone, без fine-tuning последних слоев;
- данные объединяют два разных домена, но external validation на новом датасете еще не проведена;
- Grad-CAM дает интерпретацию, но не является строгим доказательством отсутствия shortcuts;
- YOLOv8 counting metrics еще нужно получить отдельно.

## 21. Главные выводы

1. Crop-based classification cattle/sheep работает хорошо.
2. Собственная CNN дала сильный baseline: `0.9610` accuracy.
3. Лучшей моделью стала VGG16: `0.9738` accuracy.
4. ResNet50 и MobileNetV2 не обогнали Custom CNN в текущем setup.
5. Transfer learning не всегда автоматически лучше baseline; важны head design, trainable parameters и domain shift.
6. Аугментации дали небольшое улучшение: `+0.0062` test accuracy.
7. Для подсчета животных нужна не classification, а object detection.
8. YOLOv8 является логичным следующим этапом для detection/counting.

## 22. Как объяснить проект за 30 секунд

Моя работа посвящена классификации и последующему подсчету сельскохозяйственных животных на изображениях. Я объединила WAID и COCO, отфильтровала классы `cattle` и `sheep`, провела EDA, preprocessing и подготовила `51 320` crop-изображений. Затем обучила собственную CNN baseline и сравнила ее с ResNet50, VGG16 и MobileNetV2. Лучший результат классификации показала VGG16 с accuracy `0.9738`, а Custom CNN дала сильный baseline `0.9610`. Следующий этап - YOLOv8 detection для подсчета животных на полном изображении.

## 23. English short summary

This project focuses on livestock classification and the next step of animal counting in images. The implemented part solves crop-based classification for two classes: cattle and sheep. I combined WAID drone images and a COCO cow/sheep subset, performed EDA and preprocessing, and generated 51,320 animal crops. A custom CNN baseline was trained and compared with ResNet50, VGG16, and MobileNetV2 using transfer learning. VGG16 achieved the best classification accuracy of 0.9738, while the custom CNN achieved a strong baseline accuracy of 0.9610. YOLOv8 detection is the planned next step for full-image animal counting.

