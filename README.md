#   Проект лабораторной работы №2 по дисциплине "Системы копьютерного зрения"
##  Основная информация о задании
&ensp; В рамках данной работы было необходимо реализовать методы поиска и сопоставления опорных точек на двух изображениях, имеющих друг относительно друга смещение.

##  Выбор метода обнаружения
&ensp; Во многом мы ограничены двумя вариантами с некоторым числом модификаций внутри них.

&ensp; Первый варант - поиск углов. Данный метод хорошо работает с изображениями, имеющими достаточно явно выраженные прямые линии и, соответсвенно, углов, образуемых между ними. Чаще всего их применяют для поиска геометрических фигур или сопоставления изображений зданий, панормамм городов или прочих изображений, где будет большое число прямых (или близких к ним) линий.

&ensp; Второй варинат - блобы. Блобы представляют собой некоторую область, которая является достаточно отличительной на фоне остального изображения и может быть сравнительно лего детектированна. Если найти на изображении А набор таких областей, то при незначительных изменениях, с высокой вероятностью все или большая часть этих областей может быть детектирована на изображении B (если мы применяем одинаковые алгоритмы). Однако, стоит заметить, что данный метод не совершенен и часто при сильных изменениях изображений будет происходить серьёзное изменение числа или положения этих самых блобов, что делает невозможность сопоставления блобов. Из особенностей стоит отметить, что у него единственное требование - наличие каких-либо особенностей (переходов яркости, цветовых переходов) на изображении, чтобы их можно было считать этими особыми точками.

&ensp; Для решения задачи в данном проекте будет применятсья метод блобов, так как он более универсален и больше подходит для изображений с не явно выраженными прямыми линиями.

##  Описание хода работы
#   Искажение исходного изображения

&ensp; Для работы было взято случайно изображение с интернета, сложность работы с которым заранее является высокой ввиду однотонности палитры и малого числа объектов, за которые дейсвтительно можно зацепиться блобами (но было бы лего зацепиться фильтром углов). Изображение приведено ниже и оно имеет размер 1280x720.

![alt text](one.jpg)

&ensp; Чтобы исказить изображение, достаточно поменять его яркость или же как-то сместить пиксели. Для внесения изменений в изображении было использован сервис yodayo, а конкретно функция расширения изображения. Исходное изображение было загружено и увеличено до размеров 1865x1227. Новые пиксели были созданы генератором изображения и, фактически, просто дополнили исходную картину, не затронув её. Стоит отметить, что метод может быть не идеален, так как иходное изображение может быть подвергнуто изменению по гамме, более того, исходное изображение было в формтае jpg, а выходное - png, которое потом конвертировалось в jpg.

&ensp; Дополненное изображение представлено ниже.

![alt text](generate.jpg)

&ensp; На этом модификация изображений остановлена, переходим к непосредственно блобам.

#   Алгоритм обнаружения блобов (blob)

### Бинаризация

&ensp; Как ранее говорилось - блоб - это особая область, которая отличается или выделяется на фоне остального изображения (если быть вернее, в определённой его окрестности). Для подобной задачи работа с цветным, цифровым изображением, крайне неудобна ввиду того, что изображение содержит огромное количество информции. Поэтому для упрощения работы предварительно оба изображения будут бинаризоваться, ранее мы рассматривали как работает данный алгоритм в лабораторной работе №1. Стоит отметить, что исходное изображениие далее будет обозначаться как изображение 1 и искажённое (расширенное) как изображение 2. Результат бинаризации обоих изображений представлен ниже.

![alt text](Бинаризация.png)

&ensp; Стоит замтеить, что важно подобрать порог бинаризации (threshold), так как в зависимости от него в итоге будет получатсья разное количество и качество областей (об этом будет далее). При низком пороге - у нас будет больше областей и при этом будут поподать шумы. При высоком пороге - будет меньше областей (но не всегда) и, возможно, что часть областей будут поглащать друг друга, сливаясь в одну область (такое не исключено и при низком пороге, так как сильно зависит от особенностей изображения).

### Определение связей

&ensp; Данный этап нужен для того, чтобы разбить бинаризированное изображение на набор связанных блоков пикселей. Которые, фактически, будут формировать областит, которые в дальнейшем будут анализироваться. 

&ensp; Для определения связей использовалась функция label из библиотеки scipy.ndimage. Как итог, на выходе получается массив размером с изображение, состоящий из чисел в диапозоне от 0 до N, где N - число обнаруженных связанных пикселей (формирующих область N). Поэтому пятно пикселей с значением 1, будут формировать область номер 1. Пятно из пикселей с значением 2 - будет формировать область номер 2 и так далее.

&ensp; Так же можно рассматривать несколько видов связей, в частности, 4 или 8 связность, в первом случаи пиксель считается связанным, если он контактирует с другими такими же пикселями по какой либо из сторон. В втором случаи к условию добавляются диагонали и достаточно, чтобы углы пикселей, а не стороны, касались друг друга.

&ensp; Ниже приведены графические отображения работы алгоритма. Для изображения 1.

![alt text](Связывание1.png)

&ensp; В данном случаи был сделан цветной градиент, где каждый цвет или его оттенок показывает принадлежность к той или иной области.

&ensp; Так же был выведен список всех областей с числом пикселей, так как по цветной шкале трудно понять, скольок всего было обнаружено областей, но на основе выводимый справочойно информации можно сделать вывод, что было сформировано около 397 областей. Однако многие из них состоят из всего лишь пары пикселей или пары десятков, такие блобы мы не увидим графически, но при этом они составляют большую часть изображения.

&ensp; Ниже приведены графические отображения работы алгоритма. Для изображения 2.

![alt text](Связывание2.png)

&ensp; Здесь же, несмотря на расширение изображения, было сформированно всего 334 области, что меньше, чем на первом изображении. Вероятнее всего, исходное изображение было искажено в генераторе краёв и при измеении форматов. Однако, для текущей задачи это не кретично.

### Фильтрация областей (блобов)

&ensp; Как было ранее сказанно, метод связей обнаружил около 300-400 связанных областей, каждая из которых могла бы стать потенциальным блобом. Однако, работать с таким числом точек было бы крайне неудобно, более того, достаточно большчая их часть представляет собой области в несколько пикселей, а другие - в несколько сотен пикселей. Как итог, вводится пороговое значение на минимальнуб площадь блоба (min_area), задаваемую пользователем. В данном с случаи при значении в 50 пикселей были достигнуты приемлемые результаты. 


### Сопоставление блобов

&ensp; После того, как была проведена работа по фильтрации областей, было вычисленно число блобов, с которыми будем в дальнейшем работать. Для первого изображения это -, и для второго -.

&ensp; Теперь необходимо сопостваить блобы, чтобы по ним в дальнейшем можно было сопоставлять изображения  и/или объекты. Стоит заметить, что все блобы имеют набор индивидуальных параметров, которые позволят их сравнивать между собой. Всего можно выделить около 6 параметров:

&ensp; Параметры блобов:

1. Площадь блоба
2. Средняя интексивность пикселей
3. Гистогрмма локальных бинарных шаблонов
4. Список расстояний до других ближайших блобов
5. Список углов до других ближайших блобов

&ensp; Первые два параметр являются достаточно простыми. Так как площадь блоба - число пикселей. Среднияя интенсивность пикселей - отношение суммы яркостей всех пикселей к числу пикселей.

&ensp; Рассмотрим как вычисляются расстоянияя до блобов. Сперва вычисляется абсолютная удалённость центров масс блобов друг от друга за счёт вычисления Евкливдова расстояния между ними. Однако, этот метод чувствителен к изменению масштаба, поэтмоу после поделим вычисленно расстояние на длину или ширину изображения (берём наибольший параметр). Таким образом будет вычислять рассятоние от одного блоба до трёх других ближайших блобов.

&ensp;  Рассмотрим как вычисляются углы до блобов. По аналогии с прошолым методом, вычисления происходят до трёх ближайших блобов. Вычисление происходит с применением функции np.arctan(), где на вход подаётся разность координат центров масс блобов по x и y. Однако данная функция вычисляет в радианы, для перевода в обычные углы применяется выражение np.arctan() / (2 * np.pi). Что позволяет получить информацию об углах между блобами. 

&ensp; Рассмотрим более подробно ряд параметров, в частности, гистограмму локальных бинарных шаблонов. В анализ попадает не весь блоб, а определённа зона вокруг его центра масс. Размер зоны вычисляется на основе следующих выражений, исходя из предположения, что блоб стремиться к округлой форме и будет вписан в прямоугольник. Спервы вычислим радиус блоба, для этого будем использовать следующую формулу - np.sqrt(S / np.pi). Где S - площадь блоба в пикселях. После определяем границы прямоугольника, относительно центра масс. Стороны прямоугольника будут удалены от центра масс на вычисленный ранее радиус и будут формировать квадрат, который и будет нашим шаблоном.

&ensp; На основе данных параметро происходит после вычисление степени схожести блобов на основе суммы параметров с весовыми коэффициентами и их вычитание от еденицы, чтобы узнать степень схожести блобов, а после сделать вывод об их идентичности.

&ensp; После сопоставления блобов можно сделать вывод о верности их определения.

### Алгоритм вычисления смещений

&ensp;





### Конечный результат

&ensp;  После сравнения всех блобов и определения пар, необходимо просто вывести сами блобы на исходном и изменённом отображении. Номера блобов для одних и тех же областей должны совпадать, это будет говорить о верном определении блобов. Если же они не совпадают, значит пара блобов была установлена неверно и, не исключено, что данный блоб не имеет пары вовсе (такое возможно, так как алгоритм мог предпочесть соседнюю область и отсеять нужную нам в ходе фильтрации либо же изначально не заметить её при бинаризации изображений).
  








