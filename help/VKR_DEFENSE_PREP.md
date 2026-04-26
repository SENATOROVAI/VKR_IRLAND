# Подготовка к защите ВКР: вопросы и ответы

Проект: классификация и подсчет сельскохозяйственных животных на изображениях. Основные классы: `cattle` и `sheep`. Данные: WAID + COCO 2017 subset. Готовые модели классификации: Custom CNN, ResNet50, VGG16, MobileNetV2. YOLOv8/counting в текущем архиве обозначается как следующая часть работы, без готовых метрик.

## 1. Pitch

### 30 секунд

Моя работа посвящена автоматической классификации и последующему подсчету сельскохозяйственных животных на изображениях, в первую очередь коров и овец. Я объединила два источника данных: дроновый WAID и ground-level subset COCO, провела EDA, очистку, stratified split и подготовила 51 320 crop-изображений животных. Затем обучила собственную CNN как baseline и сравнила ее с тремя ImageNet-pretrained моделями: ResNet50, VGG16 и MobileNetV2. Лучший результат классификации показала VGG16: test accuracy 0.9738, но собственная CNN тоже дала сильный baseline 0.9610.

English: My thesis focuses on automatic livestock classification and the next step of animal counting in images. I combined WAID drone images with a COCO cow/sheep subset, prepared 51,320 animal crops, trained a custom CNN baseline, and compared it with ResNet50, VGG16, and MobileNetV2. VGG16 achieved the best classification accuracy, 0.9738, while the custom CNN reached a strong 0.9610 baseline.

### 60 секунд

Цель работы - проверить, насколько хорошо нейросетевые модели могут различать коров и овец на реальных фермерских и дроновых снимках, а затем использовать детекцию для подсчета животных. Я взяла два разных источника: WAID с дроновыми изображениями и COCO с наземными изображениями. После фильтрации остались 12 232 изображения с нужными классами. Для классификации я не использовала полный кадр, а вырезала животных по bounding boxes, потому что полный кадр содержит много фона и может исказить обучение. Получилось 51 320 crops с почти сбалансированными классами. Я построила собственную CNN с Optuna tuning и сравнила ее с transfer learning моделями. Результат: VGG16 лучше всех по F1 0.9736, Custom CNN - 0.9608, ResNet50 - 0.9568, MobileNetV2 - 0.9496. Также использовала Grad-CAM, чтобы показать, что модели действительно смотрят на тело животного, а не только на фон.

English: The goal is to evaluate neural models for cow versus sheep classification on realistic farm and drone imagery, with object detection planned for counting. I merged WAID drone data and a COCO cow/sheep subset, filtered them to 12,232 relevant images, and generated 51,320 animal crops from bounding boxes. This avoids training a classifier on mostly background pixels. I trained a custom CNN baseline with Optuna tuning and compared it with three ImageNet-pretrained models. VGG16 performed best, while the custom CNN remained a strong baseline. Grad-CAM was used to check that the models focus on the animal body rather than shortcuts in the background.

## 2. Цифры, которые надо помнить

| Что | Значение |
|---|---:|
| Raw images всего | 17 759 |
| WAID raw images | 14 366 |
| COCO cow/sheep images | 3 393 |
| Images после фильтрации cattle/sheep | 12 232 |
| Train/val/test images | 8 562 / 1 835 / 1 835 |
| Classification crops всего | 51 320 |
| Train/val/test crops | 35 806 / 7 770 / 7 744 |
| Crop classes | 26 986 cattle / 24 334 sheep |
| Custom CNN test accuracy | 0.9610 |
| VGG16 test accuracy | 0.9738 |
| ResNet50 test accuracy | 0.9570 |
| MobileNetV2 test accuracy | 0.9498 |
| Best classification model | VGG16 |
| Custom CNN params | 8.8M trainable |
| VGG16 params | 17.9M total / 3.2M trainable |
| ResNet50 params | 23.8M total / 0.26M trainable |
| MobileNetV2 params | 2.4M total / 0.16M trainable |
| Augmentation effect | +0.0062 test accuracy |

## 3. Основные вопросы и ответы

### 1. Как сформулировать тему работы?

**Ответ:** Тема работы - автоматическая классификация и подсчет сельскохозяйственных животных на изображениях, с фокусом на различение `cattle` и `sheep`. На текущем этапе реализована и сравнена классификация по crop-изображениям животных; детекция YOLOv8 нужна для следующей части - подсчета животных в полном кадре.

**English:** The thesis is about automatic livestock classification and counting, focusing on cattle and sheep. The current implemented part compares crop-based classifiers; YOLOv8 detection is the next step for full-image counting.

**Ключевые цифры:** 2 класса, 4 классификационные модели, 51 320 crops.

**Не говорить:** "Я просто обучила классификатор на картинках". Нужно подчеркнуть pipeline: data acquisition, EDA, preprocessing, baseline, transfer learning, evaluation.

### 2. В чем основная цель исследования?

**Ответ:** Цель - построить воспроизводимый pipeline для классификации коров и овец на реальных изображениях и сравнить собственную CNN с предобученными ImageNet-моделями. Дополнительная цель - подготовить данные и методологию для детекции и counting.

**English:** The goal is to build a reproducible pipeline for cow/sheep classification and compare a custom CNN baseline with ImageNet-pretrained models, while preparing the detection/counting stage.

**Ключевые цифры:** Custom CNN 0.9610 accuracy, VGG16 0.9738.

**Не говорить:** "Цель - получить максимальную accuracy любой ценой". Важны сравнение, воспроизводимость и интерпретируемость.

### 3. Почему выбраны именно cattle и sheep?

**Ответ:** Это реальные сельскохозяйственные животные, которые есть в обоих источниках данных. Остальные классы WAID, например zebra, seal, kiang, camelus, не соответствуют задаче livestock monitoring, поэтому были отфильтрованы.

**English:** Cattle and sheep are relevant livestock classes available in both data sources. Non-farm classes from WAID were removed because they do not match the monitoring task.

**Ключевые цифры:** WAID имел 6 классов, оставлены 2.

**Не говорить:** "Остальные классы были плохими". Лучше: "они не соответствовали предметной области".

### 4. Какие источники данных использовались?

**Ответ:** Использовались WAID с Roboflow и COCO 2017 subset через FiftyOne. WAID дает дроновые снимки, COCO - наземные изображения с более разнообразными условиями съемки.

**English:** I used WAID from Roboflow and a COCO 2017 cow/sheep subset through FiftyOne. WAID provides drone imagery, while COCO adds ground-level images.

**Ключевые цифры:** WAID 14 366 images, COCO 3 393 images.

**Не говорить:** "Данные одинаковые". Наоборот, они различаются по перспективе и масштабу объектов.

### 5. Почему было полезно объединять WAID и COCO?

**Ответ:** WAID и COCO покрывают разные домены: дроновая съемка и наземная съемка. Объединение делает задачу более реалистичной и проверяет, насколько модели устойчивы к разным перспективам, масштабам и фонам.

**English:** Combining WAID and COCO improves domain diversity: drone images and ground-level images. This makes the evaluation more realistic.

**Ключевые цифры:** 17 759 raw images до фильтрации.

**Не говорить:** "Больше данных всегда лучше". Правильнее: "больше данных полезно, если контролировать доменный сдвиг и split".

### 6. Что показал EDA?

**Ответ:** EDA показал различия между источниками: WAID имеет фиксированный размер 640x640 и много маленьких объектов, COCO имеет разные размеры и более крупные объекты. Corrupted files не найдено, outliers по brightness/std были единичными. Это подтвердило, что данные пригодны, но требуют crop-based preprocessing.

**English:** EDA showed strong source differences: WAID has 640x640 drone images with small objects, while COCO has variable-size ground-level images with larger objects. No corrupted files were found.

**Ключевые цифры:** 0 corrupted files из 17 759.

**Не говорить:** "EDA нужен только для графиков". Он обосновал решения preprocessing.

### 7. Почему не обучать классификатор на полном изображении?

**Ответ:** В полном изображении много фона, особенно в WAID, где животные маленькие. Классификатор мог бы выучить фон или перспективу датасета вместо признаков животного. Поэтому для классификации использовались crops по bounding boxes.

**English:** Full images contain too much background, especially in WAID. A classifier could learn dataset shortcuts instead of animal features, so I used bounding-box crops.

**Ключевые цифры:** WAID bbox areas примерно в 14 раз меньше COCO по EDA.

**Не говорить:** "Полные изображения бесполезны". Они полезны для детекции/counting, но не лучший формат для crop classification.

### 8. Как был сделан preprocessing?

**Ответ:** Сначала были отфильтрованы только cattle/sheep images, затем данные объединены в единый YOLO-style формат с class map `cattle=0`, `sheep=1`. После этого выполнен stratified split 70/15/15 и созданы crop-изображения для классификации.

**English:** I filtered cattle/sheep images, merged the sources into a unified YOLO-style format, used `cattle=0` and `sheep=1`, performed a 70/15/15 stratified split, and generated crops.

**Ключевые цифры:** 12 232 filtered images, 51 320 crops.

**Не говорить:** "Я сначала нарезала crops, потом split". Правильно: image-level split before crop generation.

### 9. Почему важен image-level split?

**Ответ:** Если сначала нарезать crops, а потом случайно делить их на train/val/test, crops из одного исходного изображения могут попасть в разные split. Это data leakage. Поэтому split делается на уровне исходных изображений.

**English:** If crops are split randomly after extraction, crops from the same original image can appear in both train and test. That is data leakage, so splitting must happen at image level.

**Ключевые цифры:** train/val/test images: 8 562 / 1 835 / 1 835.

**Не говорить:** "Leakage маловероятен". Он был бы очень вероятен при crop-level split.

### 10. Почему выбран split 70/15/15?

**Ответ:** Это стандартный компромисс: 70% достаточно для обучения, 15% для настройки hyperparameters и early stopping, 15% для финальной независимой оценки. Датасет достаточно большой, чтобы test set был репрезентативным.

**English:** 70/15/15 balances training data, validation for tuning, and an independent test set.

**Ключевые цифры:** test set 1 835 images и 7 744 crops.

**Не говорить:** "Так принято". Лучше объяснить назначение каждого split.

### 11. Что такое stratified split и зачем он нужен?

**Ответ:** Stratified split сохраняет похожие пропорции классов в train, validation и test. Это важно, чтобы модель не обучалась или не оценивалась на перекошенном распределении классов.

**English:** Stratified splitting preserves class proportions across train, validation, and test sets.

**Ключевые цифры:** primary class balance около 53.5% cattle в каждом split.

**Не говорить:** "Он делает данные одинаковыми". Он сохраняет class distribution, но не дублирует данные.

### 12. Почему ограничение max 5 crops per image?

**Ответ:** В некоторых изображениях много животных. Если взять все bbox, такие изображения будут доминировать в обучении и могут усилить зависимость от одного контекста. Ограничение 5 крупных bbox дает разнообразие и контролирует размер датасета.

**English:** Some images contain many animals. Taking all boxes would overrepresent those images, so I limited crops to five large boxes per image.

**Ключевые цифры:** `MAX_CROPS_PER_IMAGE = 5`.

**Не говорить:** "Так быстрее". Скорость вторична; главное - контроль переизбыточности.

### 13. Почему отбрасывались маленькие bounding boxes?

**Ответ:** Слишком маленькие bbox после resize дают мало визуальной информации. В проекте использовался порог `MIN_BBOX_AREA = 0.001`, примерно чтобы отсечь объекты, из которых классификатор почти не сможет извлечь признаки.

**English:** Very small boxes contain too little visual information after resizing, so I removed boxes below a normalized area threshold.

**Ключевые цифры:** `MIN_BBOX_AREA = 0.001`.

**Не говорить:** "Маленькие объекты всегда не нужны". Они важны для detection, но для crop classifier могут быть шумными.

### 14. Какой итоговый баланс классов в crops?

**Ответ:** Баланс почти ровный: 26 986 crops cattle и 24 334 crops sheep. Это снижает риск, что accuracy будет высокой только из-за majority class.

**English:** The crop dataset is nearly balanced: 26,986 cattle and 24,334 sheep crops.

**Ключевые цифры:** примерно 52.6% cattle и 47.4% sheep.

**Не говорить:** "Баланс идеальный". Он хороший, но не идеально 50/50.

### 15. Какие аугментации использовались?

**Ответ:** Для train использовались базовые аугментации: horizontal flip, небольшая rotation и color jitter. Для evaluation использовались только resize и normalization. Это разделение важно, чтобы test отражал реальные данные, а не искусственные трансформации.

**English:** Training used basic augmentations such as horizontal flip, small rotation, and color jitter. Evaluation used only resize and normalization.

**Ключевые цифры:** augmentation дала +0.0062 test accuracy для Custom CNN.

**Не говорить:** "Аугментации всегда улучшают". В этой работе улучшение было небольшим, но измеренным.

### 16. Почему использовалась ImageNet normalization?

**Ответ:** Для pretrained моделей это обязательное условие, потому что они обучались с ImageNet mean/std. Для собственной CNN это также применено для единообразия pipeline.

**English:** ImageNet normalization is required for pretrained models and was also used for the custom CNN for pipeline consistency.

**Ключевые цифры:** mean `[0.485, 0.456, 0.406]`, std `[0.229, 0.224, 0.225]`.

**Не говорить:** "Normalization просто улучшает accuracy". Она делает входные данные совместимыми с pretrained backbones.

### 17. Что такое baseline и зачем он нужен?

**Ответ:** Baseline - это собственная модель, обученная с нуля, которая дает точку сравнения. Без baseline нельзя честно сказать, действительно ли pretrained модели дают выигрыш.

**English:** A baseline is a model trained from scratch that provides a reference point for comparison with pretrained models.

**Ключевые цифры:** Custom CNN accuracy 0.9610.

**Не говорить:** "Baseline - слабая модель". В этой работе baseline оказался достаточно сильным.

### 18. Как устроена Custom CNN?

**Ответ:** Это параметризованная сверточная сеть с несколькими Conv-BatchNorm-Activation-MaxPool блоками и классификационной головой. Optuna выбрала 3 conv blocks, 64 base channels, kernel 3, ReLU, Adam и learning rate около 1.66e-3.

**English:** The custom CNN uses convolutional blocks followed by a classifier head. Optuna selected 3 blocks, 64 base channels, kernel size 3, ReLU, Adam, and lr around 1.66e-3.

**Ключевые цифры:** 8.8M parameters, input 128x128.

**Не говорить:** "Архитектура выбрана вручную полностью". Часть параметров выбрана через Optuna.

### 19. Почему Custom CNN обучалась на 128x128?

**Ответ:** Собственная CNN обучалась с нуля, поэтому 128x128 снижает вычислительную нагрузку и достаточно для бинарной классификации crop-изображений. Pretrained модели использовали 224x224, потому что это стандартный вход для ImageNet backbones.

**English:** The custom CNN used 128x128 for efficiency, while pretrained models used the standard 224x224 ImageNet input.

**Ключевые цифры:** CNN 128x128, transfer learning 224x224.

**Не говорить:** "128 всегда лучше". Это компромисс между деталями и скоростью.

### 20. Что такое Optuna и зачем она использовалась?

**Ответ:** Optuna использовалась для hyperparameter tuning: число conv blocks, channels, kernel size, activation, dropout, learning rate, batch size и optimizer. Это делает выбор модели более системным, чем ручной перебор.

**English:** Optuna was used for systematic hyperparameter tuning instead of manual trial-and-error.

**Ключевые цифры:** 15 trials для Custom CNN, 6 trials на каждую transfer модель.

**Не говорить:** "Optuna гарантирует глобально лучший результат". Она ищет хорошую конфигурацию в заданном пространстве.

### 21. Почему HP tuning делался на subset?

**Ответ:** Полный tuning на всех данных был бы слишком дорогим по времени. Subset 5k/1k для CNN и 3k/600 для transfer learning позволил быстро оценить конфигурации, а финальное обучение выполнялось на полном train/val.

**English:** Hyperparameter search was run on subsets for computational efficiency, then the best settings were trained on the full dataset.

**Ключевые цифры:** CNN tuning subset 5 000 train / 1 000 val.

**Не говорить:** "Модель обучалась только на subset". Subset использовался для tuning, не для финальной модели.

### 22. Какие метрики использовались?

**Ответ:** Для классификации использовались accuracy, macro precision, macro recall и macro F1. Macro metrics важны, потому что они усредняют качество по классам и меньше скрывают проблемы minority class.

**English:** I used accuracy, macro precision, macro recall, and macro F1 for classification.

**Ключевые цифры:** Custom CNN F1 0.9608, VGG16 F1 0.9736.

**Не говорить:** "Accuracy достаточно". Лучше объяснить, почему добавлены precision/recall/F1.

### 23. Что означает accuracy?

**Ответ:** Accuracy - доля правильных предсказаний среди всех объектов. В бинарной задаче она интуитивна, но при class imbalance может быть misleading, поэтому ее дополняют precision, recall и F1.

**English:** Accuracy is the fraction of correct predictions, but it can be misleading under class imbalance.

**Ключевые цифры:** VGG16 accuracy 0.9738.

**Не говорить:** "Accuracy показывает все". Она не показывает типы ошибок.

### 24. Что означает precision?

**Ответ:** Precision отвечает на вопрос: из всех объектов, которые модель назвала данным классом, сколько действительно относятся к этому классу. Это важно, когда false positives имеют цену.

**English:** Precision measures how many predicted positives are actually correct.

**Ключевые цифры:** VGG16 macro precision 0.9744.

**Не говорить:** "Precision и recall одно и то же". Это разные стороны ошибки.

### 25. Что означает recall?

**Ответ:** Recall показывает, какую долю реальных объектов класса модель смогла найти. Если recall низкий, модель пропускает этот класс.

**English:** Recall measures how many true instances of a class the model successfully finds.

**Ключевые цифры:** VGG16 macro recall 0.9730.

**Не говорить:** "Recall важен только для detection". Он важен и для classification.

### 26. Что означает F1?

**Ответ:** F1 - гармоническое среднее precision и recall. Он полезен, когда нужно одним числом отразить баланс между false positives и false negatives.

**English:** F1 is the harmonic mean of precision and recall.

**Ключевые цифры:** Custom CNN F1 0.9608, VGG16 F1 0.9736.

**Не говорить:** "F1 заменяет все метрики". Он дополняет, но не объясняет все ошибки.

### 27. Какая модель показала лучший результат?

**Ответ:** Лучший результат показала VGG16: test accuracy 0.9738, precision 0.9744, recall 0.9730, F1 0.9736. Это выше Custom CNN, ResNet50 и MobileNetV2.

**English:** VGG16 performed best with test accuracy 0.9738 and macro F1 0.9736.

**Ключевые цифры:** VGG16 test acc 0.9738.

**Не говорить:** "VGG16 всегда лучше". Это лучший результат именно в данном setup.

### 28. Почему VGG16 могла обогнать ResNet50?

**Ответ:** В моем setup VGG16 имела более крупную trainable classification head: около 3.2M trainable parameters, тогда как ResNet50 имела около 0.26M. Поэтому VGG16 получила больше capacity для адаптации к задаче, несмотря на то что backbone был frozen.

**English:** In this setup, VGG16 had a much larger trainable classification head, about 3.2M parameters, giving it more adaptation capacity than ResNet50.

**Ключевые цифры:** VGG16 3.2M trainable vs ResNet50 0.26M.

**Не говорить:** "VGG16 архитектурно всегда сильнее ResNet". Нужно связать ответ с конкретной head design.

### 29. Почему pretrained модели не все оказались лучше baseline?

**Ответ:** ImageNet features полезны, но домен дроновых и фермерских снимков отличается от ImageNet. Кроме того, backbone был заморожен, поэтому адаптировалась только head. В результате VGG16 выиграла, но ResNet50 и MobileNetV2 немного уступили Custom CNN.

**English:** ImageNet features help, but the domain differs from drone/farm images, and the frozen backbones limit adaptation.

**Ключевые цифры:** Custom CNN 0.9610, ResNet50 0.9570, MobileNetV2 0.9498.

**Не говорить:** "Transfer learning не работает". Работает частично: VGG16 лучше, две модели хуже baseline.

### 30. Почему MobileNetV2 оказалась слабее?

**Ответ:** MobileNetV2 компактная и оптимизирована для efficiency, поэтому имеет меньше параметров и меньшую trainable head. Это полезно для deployment, но в данном эксперименте accuracy была ниже.

**English:** MobileNetV2 is lightweight and efficient, but its lower capacity likely limited accuracy in this setup.

**Ключевые цифры:** 2.4M total params, 0.9498 accuracy.

**Не говорить:** "MobileNetV2 плохая модель". Она может быть хорошим выбором при ограничениях по скорости и памяти.

### 31. Почему ResNet50 не стала лучшей?

**Ответ:** ResNet50 имеет мощный backbone, но в feature extraction режиме backbone frozen, а trainable head была небольшой. Возможно, fine-tuning последних слоев улучшил бы результат, но это уже отдельный эксперимент.

**English:** ResNet50 has a strong backbone, but in frozen feature extraction mode only a small head was trained.

**Ключевые цифры:** 23.8M total params, only 0.26M trainable.

**Не говорить:** "ResNet50 не подходит". Скорее, текущий training setup не дал ей полного потенциала.

### 32. Что такое feature extraction?

**Ответ:** Это transfer learning режим, где pretrained backbone заморожен и используется как extractor признаков, а обучается только новая classification head под задачу cattle/sheep.

**English:** Feature extraction freezes the pretrained backbone and trains only a new task-specific classification head.

**Ключевые цифры:** trainable доля ResNet50 около 1.10%, VGG16 около 17.92%.

**Не говорить:** "Вся pretrained модель переобучалась". В этом проекте backbone был frozen.

### 33. Чем feature extraction отличается от fine-tuning?

**Ответ:** При feature extraction обучается только head. При fine-tuning размораживаются часть или все слои backbone, и модель глубже адаптируется к новому домену. Fine-tuning может улучшить качество, но требует больше вычислений и контроля overfitting.

**English:** Feature extraction trains only the head; fine-tuning updates some backbone layers as well.

**Ключевые цифры:** в текущей работе использован frozen backbone.

**Не говорить:** "Fine-tuning всегда лучше". Он может переобучиться и стоит дороже.

### 34. Что показал Grad-CAM?

**Ответ:** Grad-CAM показал, что модели в основном фокусируются на теле животного: контуре, шерсти, форме. Это важно как evidence, что модель не полностью полагается на фон или артефакты датасета.

**English:** Grad-CAM showed that the models mostly focus on the animal body, which supports that they learned relevant visual cues.

**Ключевые цифры:** Grad-CAM сохранен для Custom CNN и transfer моделей.

**Не говорить:** "Grad-CAM доказывает правильность модели". Он дает визуальную интерпретацию, но не строгий proof.

### 35. Что такое confusion matrix?

**Ответ:** Confusion matrix показывает, какие классы модель предсказывает правильно и какие путает. Для бинарной задачи видно ошибки cattle -> sheep и sheep -> cattle, что помогает понять структуру ошибок.

**English:** A confusion matrix shows correct predictions and class confusions.

**Ключевые цифры:** для Custom CNN cattle -> sheep было 139 ошибок в test.

**Не говорить:** "Confusion matrix нужна только для отчета". Она помогает анализировать типы ошибок.

### 36. Почему использованы macro metrics?

**Ответ:** Macro metrics считают метрику отдельно по каждому классу, а затем усредняют. Это полезно, когда важно одинаково учитывать cattle и sheep, даже если классы не идеально сбалансированы.

**English:** Macro metrics average class-level scores, so both cattle and sheep contribute equally.

**Ключевые цифры:** классы почти сбалансированы, но не идеально.

**Не говорить:** "Macro и weighted всегда одинаковы". Они могут отличаться при imbalance.

### 37. Что значит ablation без augmentation?

**Ответ:** Это контрольный эксперимент: та же архитектура и hyperparameters, но без train-аугментаций. Сравнение показало, что augmentation улучшила test accuracy на 0.0062.

**English:** The no-augmentation ablation isolates the effect of augmentations.

**Ключевые цифры:** with augmentation 0.9610, without 0.9548.

**Не говорить:** "Ablation была отдельной моделью с другой логикой". Это тот же setup, изменен только фактор augmentation.

### 38. Какие гипотезы можно защищать по текущим результатам?

**Ответ:** По текущим ноутбукам можно защищать гипотезы о сравнении custom CNN и transfer learning, а также о влиянии augmentation. Гипотеза о YOLOv8 counting пока должна быть описана как планируемая, потому что метрики YOLO в архиве отсутствуют.

**English:** Current notebooks support hypotheses about classification and augmentation. YOLOv8 counting remains planned because its metrics are not in the archive.

**Ключевые цифры:** notebooks 01-05 реализованы; notebook 06/07 пока план.

**Не говорить:** "YOLO уже доказал лучшее counting", если метрик нет.

### 39. Почему Shapiro-Wilk использовался в EDA?

**Ответ:** Он использовался как дополнительная проверка распределения mean brightness, чтобы понять свойства изображений. Но это не центральный метод работы; главный EDA вывод связан не с нормальностью, а с качеством данных, размерами, bbox и различием доменов.

**English:** Shapiro-Wilk was an auxiliary EDA check for brightness distribution, not a core methodological claim.

**Ключевые цифры:** p > 0.05 для mean brightness в обоих датасетах.

**Не говорить:** "Нормальность brightness доказывает качество датасета". Это только один вспомогательный сигнал.

### 40. Что делать, если комиссия спросит про p-value Shapiro-Wilk?

**Ответ:** Я бы сказала, что p-value выше 0.05 означает, что тест не дал оснований отвергнуть нормальность mean brightness на выбранной выборке. Но в компьютерном зрении это не является критерием пригодности датасета; важнее visual inspection, corrupted scan, bbox statistics и distribution shift.

**English:** A p-value above 0.05 means the test did not reject normality for sampled mean brightness, but this is not the main dataset validity criterion.

**Ключевые цифры:** sample 400 images per dataset for pixel stats.

**Не говорить:** "Данные нормальные, значит модели будут хорошо работать". Это неверная причинная связь.

### 41. Как предотвращался overfitting?

**Ответ:** Использовались train/val/test split, early stopping, dropout в head, basic augmentations и validation monitoring. Для transfer learning backbone был frozen, что тоже снижает риск переобучения.

**English:** Overfitting was controlled through train/validation/test split, early stopping, dropout, augmentations, and frozen backbones.

**Ключевые цифры:** Custom CNN early stop на epoch 21, best val acc 0.9609.

**Не говорить:** "Overfitting полностью исключен". Его можно снижать и проверять, но не исключать абсолютно.

### 42. Почему использовался CrossEntropyLoss?

**Ответ:** Это стандартная loss function для многоклассовой классификации по logits. Даже при двух классах она корректна, потому что модель выдает два logits: cattle и sheep.

**English:** CrossEntropyLoss is standard for classification over logits, including two-class classification with two output logits.

**Ключевые цифры:** output classes = 2.

**Не говорить:** "Для binary всегда нужен только BCE". BCE возможен, но не обязателен при двух logits.

### 43. Почему использовались PyTorch и torchvision?

**Ответ:** PyTorch удобен для кастомного training loop, Optuna integration, MPS/GPU acceleration и Grad-CAM. torchvision дает стандартные pretrained модели и transforms.

**English:** PyTorch supports custom training loops, Optuna integration, acceleration, and Grad-CAM; torchvision provides pretrained models and transforms.

**Ключевые цифры:** PyTorch 2.x, torchvision pretrained models.

**Не говорить:** "Потому что это популярно". Лучше назвать технические причины.

### 44. Почему использовалась FiftyOne?

**Ответ:** FiftyOne упростил получение COCO subset только с нужными классами cow/sheep и экспорт в YOLO-compatible формат. Это снижает риск ошибок ручной фильтрации COCO annotations.

**English:** FiftyOne simplified downloading and exporting only the COCO cow/sheep subset.

**Ключевые цифры:** COCO subset 3 393 images.

**Не говорить:** "COCO скачивался полностью вручную". Использовался targeted subset.

### 45. Что такое domain shift в вашей работе?

**Ответ:** Domain shift - это отличие распределений данных между источниками: WAID дроновый, объекты маленькие, 640x640; COCO наземный, размеры разные, объекты крупнее. Модель должна быть устойчивой к этим различиям.

**English:** Domain shift is the difference between WAID drone imagery and COCO ground-level imagery.

**Ключевые цифры:** WAID bbox area существенно меньше COCO.

**Не говорить:** "Все картинки из одной природы". Источники явно различаются.

### 46. Какие ограничения у текущей работы?

**Ответ:** Основные ограничения: только два класса, crop classification не решает counting напрямую, transfer learning использовал frozen backbone без fine-tuning, а YOLOv8 counting metrics еще не реализованы в предоставленных ноутбуках.

**English:** Limitations include two classes only, crop classification not directly solving counting, frozen-backbone transfer learning, and missing YOLOv8 counting metrics in the current archive.

**Ключевые цифры:** notebooks 01-05 готовы, YOLO planned.

**Не говорить:** "Ограничений нет". Лучше честно назвать и предложить future work.

### 47. Почему crop classification не равна подсчету животных?

**Ответ:** Crop classification предполагает, что объект уже найден и вырезан. Counting требует найти все объекты в полном изображении и посчитать их. Поэтому для counting нужен detector, например YOLOv8, а не только classifier.

**English:** Crop classification assumes the object is already localized; counting requires detecting all animals in a full image.

**Ключевые цифры:** classification crops 51 320, detection images 12 232.

**Не говорить:** "Можно просто классифицировать полный кадр и получить count". Классификация не локализует объекты.

### 48. Как YOLOv8 должен использоваться в этой работе?

**Ответ:** YOLOv8 должен решать detection задачу: находить bounding boxes cattle/sheep на полном изображении. После этого count можно получить как число predicted boxes, а качество counting оценивать через MAE и R² между ground truth count и predicted count.

**English:** YOLOv8 should detect cattle/sheep boxes in full images; counts can be computed from the number of predicted boxes and evaluated with MAE and R².

**Ключевые цифры:** YOLO metrics пока не утверждаются.

**Не говорить:** "YOLO уже показал такие-то mAP/MAE", если эти результаты не получены.

### 49. Что такое mAP и где он нужен?

**Ответ:** mAP используется для object detection, а не для crop classification. Он оценивает качество найденных bounding boxes с учетом confidence, class correctness и IoU с ground truth.

**English:** mAP is a detection metric evaluating predicted boxes, confidence, classes, and IoU overlap.

**Ключевые цифры:** планировались mAP@50 и mAP@50-95 для YOLOv8.

**Не говорить:** "mAP применялся к CNN classifier". Для classifier использовались accuracy/precision/recall/F1.

### 50. Что такое MAE для counting?

**Ответ:** MAE - средняя абсолютная ошибка между истинным количеством животных и предсказанным количеством. Например, если на изображении 10 животных, а модель нашла 8, ошибка равна 2.

**English:** MAE is the average absolute difference between true and predicted animal counts.

**Ключевые цифры:** MAE planned for YOLO counting.

**Не говорить:** "MAE показывает classification accuracy". Это метрика ошибки численного подсчета.

### 51. Что такое R² для counting?

**Ответ:** R² показывает, насколько хорошо predicted counts объясняют variation в ground truth counts. Он полезен вместе с MAE: MAE дает абсолютную ошибку, R² - качество соответствия trend.

**English:** R² shows how well predicted counts explain the variance of true counts.

**Ключевые цифры:** R² planned for counting.

**Не говорить:** "R² всегда между 0 и 1". Он может быть отрицательным при плохих предсказаниях.

### 52. Что такое McNemar test и зачем он нужен?

**Ответ:** McNemar test сравнивает две модели на одних и тех же test samples и проверяет, является ли различие в ошибках статистически значимым. Он лучше простой разницы accuracy, когда модели оценены на одном test set.

**English:** McNemar's test compares paired model predictions and checks whether the difference in errors is statistically significant.

**Ключевые цифры:** planned in Notebook 07 comparative analysis.

**Не говорить:** "McNemar сравнивает средние метрики". Он сравнивает paired disagreement counts.

### 53. Как объяснить неожиданный результат VGG16?

**Ответ:** Я бы объяснила это сочетанием ImageNet features и большей trainable head в моем setup. VGG16 получила больше trainable capacity, чем ResNet50/MobileNetV2. Поэтому результат не означает, что VGG16 всегда лучше, а показывает влияние конкретной архитектуры head и режима frozen backbone.

**English:** VGG16 likely benefited from ImageNet features plus a larger trainable head in this specific setup.

**Ключевые цифры:** VGG16 0.9738 accuracy, 3.2M trainable params.

**Не говорить:** "Это просто случайность". Лучше дать техническое объяснение и признать необходимость повторов/fine-tuning как future work.

### 54. Что является главным научно-практическим результатом?

**Ответ:** Главный результат - построен воспроизводимый pipeline от данных до сравнения моделей и показано, что crop-based classification cattle/sheep может достигать высокой точности. Также показано, что pretrained модели не всегда автоматически превосходят custom baseline.

**English:** The main result is a reproducible end-to-end classification pipeline showing high cattle/sheep crop classification accuracy and a nuanced comparison of pretrained models against a custom baseline.

**Ключевые цифры:** best VGG16 0.9738, Custom CNN 0.9610.

**Не говорить:** "Главный результат - одна лучшая accuracy". Важны pipeline и сравнительный анализ.

### 55. Что бы вы улучшили в следующей версии?

**Ответ:** Я бы завершила YOLOv8 detection/counting, добавила mAP, MAE и R², провела fine-tuning последних слоев ResNet/VGG/MobileNet, сделала subgroup evaluation по lighting и occlusion, а также проверила устойчивость на полностью новом external dataset.

**English:** I would complete YOLOv8 detection/counting, add mAP/MAE/R², fine-tune backbone layers, evaluate subgroups, and test on an external dataset.

**Ключевые цифры:** planned notebooks 06-07.

**Не говорить:** "Ничего улучшать не нужно". Комиссия ценит понимание future work.

## 4. Сложные вопросы комиссии

### Data leakage: "Вы уверены, что test set независим?"

Ответ: Да, ключевой риск был в crop generation. Если сначала нарезать bbox crops, а потом делить crops, один исходный кадр мог бы попасть в разные split. Поэтому split выполнялся на уровне исходных изображений, и только после этого создавались crops внутри каждого split.

English: The split was done at the original image level before crop generation, preventing crops from the same image from leaking across train and test.

### VGG16 vs ResNet50: "Почему старая VGG16 лучше современной ResNet50?"

Ответ: Это не универсальный вывод об архитектурах. В моем setup VGG16 имела гораздо более крупную trainable head из-за Flatten 512x7x7 -> Linear, около 3.2M trainable params. ResNet50 была в feature extraction режиме с небольшой head около 0.26M trainable params. Поэтому VGG16 имела больше capacity для адаптации.

English: It is not a universal architecture claim; it is a result of this specific frozen-backbone and classifier-head setup.

### Accuracy: "Почему вы не ограничились accuracy?"

Ответ: Accuracy удобна, но может скрывать class-specific ошибки. Поэтому я также использовала macro precision, macro recall и macro F1, чтобы отдельно учитывать качество по cattle и sheep.

English: Accuracy was complemented with macro precision, recall, and F1 to avoid hiding class-specific failures.

### Shapiro-Wilk: "Зачем он вообще нужен в CV-проекте?"

Ответ: Это был вспомогательный EDA-тест для brightness distribution, не центральное доказательство. Главные EDA решения были основаны на class distribution, image sizes, bbox statistics, corrupted files и visual inspection.

English: It was an auxiliary EDA check, not the central methodological evidence.

### Counting: "Можно ли уже заявлять качество подсчета?"

Ответ: Нет. По текущим ноутбукам можно заявлять результаты классификации. Для counting нужны YOLOv8 detection metrics и сравнение predicted count vs ground truth count через MAE/R². В summary это указано как следующий этап.

English: No counting quality should be claimed until YOLOv8 detection and count metrics are computed.

## 5. Быстрая схема ответа на любой вопрос

1. Начать с короткого прямого ответа.
2. Привязать к pipeline: данные -> preprocessing -> модель -> метрика.
3. Назвать одну точную цифру.
4. Указать ограничение или trade-off, если вопрос критический.
5. Завершить тем, как это влияет на вывод работы.

Пример: "Почему VGG16 лучше?"  
Коротко: "В этом setup VGG16 получила больше trainable capacity."  
Цифра: "3.2M trainable params против 0.26M у ResNet50."  
Ограничение: "Это не доказывает, что VGG16 всегда лучше."  
Вывод: "Но показывает, что head design и режим fine-tuning сильно влияют на transfer learning."

