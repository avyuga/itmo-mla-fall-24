# Лабораторная работа №1

## Датасет 1: Wine Clustering
Визуализация данных:

<img title="Визуализация данных" src="assets/wine.png" width="600">

**Гипотеза**: алгоритмы должны выделить 4 кластера. PCA не справился с визуализацией, так как данные могут быть разделимы нелинейно.
Также видны промежутки в крупных кластерах, а значит, будет дальнейшее разделение на более мелкие кластеры.

<img title="Дендрограмма" src="assets/hierarchical_wine.png" width="600">

Видно, что дендрограммы, построенные разработанным алгоритм и его реализацией в scipy аналогичны.

## Датасет 2: Mall Customers
Визуализация данных:

<img title="Визуализация данных" src="assets/mall_customers.png" width="600">

**Гипотеза**: алгоритмы должны выделить 5 кластеров. И PCA, и TSNE справились с визуализацией, у TSNE получились менее выделенные результаты. Данные линейно разделимы, так как кластеры успешно выделены алгоритмом PCA. 

<img title="Дендрограмма" src="assets/hierarchical_customers.png" width="600">

Видно, что дендрограммы, построенные разработанным алгоритм и его реализацией в scipy аналогичны.