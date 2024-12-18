# Лабораторная работа №4

## Датасет
Для решения задачи бинарной классификации выбран датасет для предсказания сердечных заболеваний (доступен по [ссылке](https://www.kaggle.com/datasets/rashikrahmanpritom/heart-attack-analysis-prediction-dataset)). Данные были предварительно нормированы для попадания в интервал (0, 1).

## Реализация различных методов классификации 
### Вычисление отступа градиента
Вычисление отступа градиента выполняется при помощи следующей функции:
```python
def calculate_margin(w, X, y):
    return (X @ w.T) * y
```

Для построения используется следующая функция. Используется порог неуверенности, равный 0.3. 
```python
def plot_margin(margins):
    margins = np.sort(margins.flatten())
    plt.plot(margins, c='k', linewidth=3)

    plt.axhline(y=0, c='k', linewidth=0.5)

    x = np.arange(len(margins))
    plt.gca().fill_between(x, margins, where=(margins>=0.3), color='#00ff00')
    plt.gca().fill_between(x, margins, where=(margins<=-0.3), color='#ff0000')
    plt.gca().fill_between(x, margins, where=np.bitwise_and(margins >= -0.3, margins <= 0.3), color='#ffff00')

    plt.ylabel("Margin")
    plt.gcf().set_size_inches(5, 3)
```

### Вычисление функции потерь
В данной лабораторной используется квадратичная функция потерь, её формула:

$$ \mathcal{L}(x, y) = (1 - M(x, y))^2 = (1 - \langle w, x \rangle * y)^2$$

Её производная по $w$ считается следующим образом:

$$ \frac{\partial\mathcal{L}}{\partial w} = -2(1-M(x, y))* \langle y, x^T \rangle $$

### Рекуррентная оценка функционала качества
Изначальная оценка функционала качества $Q$ - среднее значение функционала по выборке из 30 объектов. В процессе обучения функционал обновляется следующим образом: 
```python
self.Q = lambda_*l + (1-lambda_)*self.Q
```
Здесь `lambda` - коэффициент забывания.

### Метод стохастического градиентного спуска с инерцией
Метод стохастического спуска с инерцией заключается в том, что Momentum - экспоненциальное скользящее среднее градиента по нескольким последним итерациямю
В решении используется градиент Нестерова, с дополнительной регуляризацией формулы выглядят следующим образом.

```python
self.v = gamma * self.v + (1 - gamma) * quadratic_margin_dloss(self.w - lr*gamma*self.v, x, y)
self.w = self.w * (1-lr*reg) - lr*self.v
```

### L2-регуляризация
Регуляризация нужна для уменьшения переобучения - это дополнительный объект для минимизации. Она дополнительно уменьшает норму весов с коэффициентом пропорциональности.

$$ \tilde{\mathcal{L}}(w, x_i) = \mathcal{L}(w, x_i) + \frac{\tau}{2} ||w||^2$$

$$ \frac{\partial\tilde{\mathcal{L}}(w, x_i)}{\partial w} = \frac{\partial \mathcal{L}(w, x_i)}{\partial w} + \tau w $$

Тогда градиентный шаг выглядит следующим образом:
$$w := w - h\frac{\partial\mathcal{L}}{\partial w} - h\tau w = w(1-h\tau) - h\frac{\partial\mathcal{L}}{\partial w}$$

Такая формула и была использована выше, с дополнением в производную в виде момента Нестерова. Коэффициент $\tau/2$ выбран равным 0.5.

### Скорейший градиентный спуск
Скорейший градиентный спуск основан на поиске оптимального адаптивного шага `lr`. Для квадратичной функции потерь $h^* = ||x_i||^{-2}$. В коде это сделано в виде переопределения заданного learning rate'а при наличии параметра для скорейшего градиентного спуска.

```python
if optimize_lr: lr = 1 / sum(x**2)
```

### Предъявление объектов по модулю отступа
Дл того, чтобы модель чаще обучалась на тех объектах, в которых не уверена, можно выбирать такие объекты. То есть, чем меньше модель уверена в объекте, тем больше вероятность, что она его выберет. Это реализовано следующим образом:
1. На каждой итерации вычисляются пороги.
2. Считается их абсолютное значение и происходит инвертация: вычитаются из максимального.
3. Так как плотность вероятности на всех объектах должна суммарно давать 1, то выполняется деление на сумму.
4. Вычисленная плотность вероятности пробрасывается в `np.random.choice()`

```python
margins = ((X @ self.w.T) * Y).flatten()
abs_inv_margins = max(abs(margins)) - abs(margins)
abs_inv_margins = abs_inv_margins / sum(abs_inv_margins)
idx = np.random.choice(np.arange(len(X)), p=abs_inv_margins)
x, y = X[idx], Y[idx]
```

## Обучение
### Инициализация весов через корреляцию
Для начала проверим, насколько сильно скоррелированы признаки данных. В идеале, если $<f_i(x), f_j(x)>=0$, то такие веса оптимальны.

<img label="corr" src="assets/corr_matrix.png" width=400>

По построенной матрице корреляций можно увидеть, что признаки не сильно коррелированы, а значит, приближение весов через корреляцию будет близко к оптимальному.

#### Инициализация весов
```python
w = np.array([
    (y_train.T @ X_train[:, i]) / (X_train[:, i].T @ X_train[:, i]) 
    for i in range(X.shape[1])
]).T
```

#### Результаты обучения

<img title="corr_log" src="assets/corr_logs.png" width=500>

<img title="corr_margin" src="assets/corr_margins.png" width=300>

#### Метрики
```
              precision    recall  f1-score   support

          -1       0.79      0.60      0.68        25
           1       0.76      0.89      0.82        36

    accuracy                           0.77        61
   macro avg       0.78      0.74      0.75        61
weighted avg       0.77      0.77      0.76        61
```

### Инициализация весов через мультистарт
Известно, что качество работы модели зависит от стартовых значений весов. Для того, чтобы найти наилучшее приближение, нужно запустить обучение несколько раз и выбрать веса с наилучшими метриками.

```python
multi_N = 15
best_acc = -1; best_idx = None; best_weights = None

for i in range(multi_N):

    classifier = LinearClassifier(X_train.shape[1])
    classifier.init_weights() # рандомая инициализация
    classifier.fit(X_train, y_train,
                n_iter=10000, lr=1e-2,lambda_=0.01,
                reg=0.5, momentum=True,gamma=0.9,
                optimize_lr=False, use_margins=False)
    y_res = classifier.predict(X_test)
    acc = accuracy_score(y_test, y_res)
    if acc >= best_acc:
        best_acc = acc
        best_idx = i
        best_weights = classifier.w
    print(f"idx={i} has accuracy={acc:.3f}")
```

#### Результат
```
idx=0 has accuracy=0.689
idx=1 has accuracy=0.754
idx=2 has accuracy=0.803
idx=3 has accuracy=0.787
idx=4 has accuracy=0.754
idx=5 has accuracy=0.803
idx=6 has accuracy=0.787
idx=7 has accuracy=0.787
idx=8 has accuracy=0.770
idx=9 has accuracy=0.820
idx=10 has accuracy=0.803
idx=11 has accuracy=0.754
idx=12 has accuracy=0.803
idx=13 has accuracy=0.705
idx=14 has accuracy=0.770
best idx=9 has accuracy=0.820
best weights: [[-0.145 -0.217  0.241 -0.066 -0.027 -0.087  0.086  0.189 -0.347 -0.179 0.208 -0.249 -0.123]]
```
На каждой итерации точность модели сравнивается с лучшим результатом, и обновляется при необходимости.


### Обучение с разными типами предъявления объектов

В обычном случае данные из выборки берутся с одинаковой вероятностью. Для того, чтобы модель чаще брала те данные, на которых она не уверена, в рандомное сэмплирование можно задать плотность вероятности (как описано выше).

#### Результаты
1. Рандомное сэмплирование

<img title="random_log" src="assets/rand_logs.png" width=500>

<img title="random_margins" src="assets/rand_margins.png" width=300>

```
              precision    recall  f1-score   support

          -1       0.83      0.76      0.79        25
           1       0.84      0.89      0.86        36

    accuracy                           0.84        61
   macro avg       0.83      0.82      0.83        61
weighted avg       0.84      0.84      0.83        61
```

2. Сэмплирование с заданной вероятностью

<img title="random_log" src="assets/no_rand_logs.png" width=500>

<img title="random_margins" src="assets/no_rand_margins.png" width=300>

```
              precision    recall  f1-score   support

          -1       0.88      0.60      0.71        25
           1       0.77      0.94      0.85        36

    accuracy                           0.80        61
   macro avg       0.83      0.77      0.78        61
weighted avg       0.82      0.80      0.79        61
```

Для данного датасета результат изменился незначительно.

## Эталонное решение

Для эталонного решения использовался класс `SGDClassifier` из библиотеки `sklearn`.
Получен следующий результат:
```
              precision    recall  f1-score   support

          -1       0.84      0.64      0.73        25
           1       0.79      0.92      0.85        36

    accuracy                           0.80        61
   macro avg       0.81      0.78      0.79        61
weighted avg       0.81      0.80      0.80        61
```

Он не сильно отличается от результатов работы кастомного алгоритма, на разных итерациях разница в метриках объясняется разными стартовыми значениями весов.

## Выводы
Были рассмотрен Линейный классификатор и различные его модификации. Результаты близки к эталонным.