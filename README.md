# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Лабораторная работа №3 - разработка системы машинного обучения

Отчет по лабораторной работе #3 выполнил(а):
- Кошелева Татьяна Алексеевна
- НМТ-213929

Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | # | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Цель работы
Познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.

## Задание 1
### Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity.

### Ход работы:
- Создаем новый пустой 3D проект на Unity;
- Скачиваем папку с ML агентом:
![1](https://user-images.githubusercontent.com/114426437/201158754-2d0038a1-6ea0-45c6-884e-57b369c978e7.png)
- Создаем и активируем новый ML Agent:
![2](https://user-images.githubusercontent.com/114426437/201158802-e7caf0dd-9c2b-4dfa-9ddd-9edaabe01b59.png)
- Скачиваем и устанавливаем библиотеку torch:
![3](https://user-images.githubusercontent.com/114426437/201158829-93783e67-a265-401e-8f1b-99bb0e7d88bc.png)

## Задание 2
### Подробно опишите каждую строку файла конфигурации нейронной сети, доступного в папке с файлами проекта по ссылке. Самостоятельно найдите информацию о компонентах Decision Requester, Behavior Parameters, добавленных на сфере.

В файле `rollerball config.yaml` содержится YAML, в которой находятся все значения гиперпараметров. 

`behaviors`

 Основной раздел файла конфигурации тренера — это набор конфигураций для каждого действия на сцене.
 Параметры существующие в среде и описанные в нашем конфиге behaviors будут иметь значения, которые мы им присвоили в конфиге. 
  
`RollerBall`
	Когда MLAgent имеет статус active и пакет “mlagents==0.28.0” и фреймворк “torch~1.7.1” были загружены, сравниваем со стандартным обращением к агенту:
  
`mlagents-learn rollerball_config.yaml --run-id=RollerBall --resume`

или
	
`mlagents-learn <trainer-config-file> --env=<env_name> --run-id=<run-identifier>`

Нас нужно `rollerball_config.yaml` и id `RollerBall`
где `<run-identifier>` - имя, которое используется для идентификации результатов тренировок нейронной сети. Поскольку внутри одной среды может быть несколько объектов под контролем нейронной сети, они могут обучаться одновременно, но различным вещам. Например в то время, как RollerBall стремится к прикосновению с Target, Target может убегать от RollerBall. В таком случае нам не придется учить их отдельно, мы добавим раздел в конфигурацию обучения под id Target и внесем изменения в скрипт внутри Unity, где отметим результат “Сбежал” для нашего Target-кубика.


`trainer_type: ppo`
Сам параметр отвечает за тип используемого тренера (по умолчанию ppo). Proximal Policy Optimization [PPO] - класс алгоритмов обучения с подкреплением, которые работают сравнимо или даже лучше, чем современные подходы, но при этом их гораздо проще реализовать и настроить. 

`hyperparameters`
	Это раздел настройки гиперпараметров для агента. 

`batch_size`
	Количество опытов в каждой итерации градиентного спуска. Это всегда должно быть в несколько раз меньше, чем количество опытов для обновления модели политики(изменения поведенческой модели). Если мы используем непрерывные действия, это значение должно быть большим (порядка 1000 с). Но мы используем только дискретные действия, поэтому значение должно быть меньше (порядка 10 с).

`buffer_size`
	Количество опытов, которые необходимо собрать перед обновлением модели политики. Соответствует тому, сколько опыта должно быть собрано, прежде чем мы будем изучать или обновлять модель. Значение должно быть в несколько раз больше, чем batch_size. 

`learning_rate`
	Начальная скорость обучения для градиентного спуска. Соответствует силе каждого шага обновления градиентного спуска. 
	
`beta`
	beta - сила регуляризации энтропии, которая делает политику «более случайной». Это гарантирует, что агенты должным образом исследуют пространство действия во время обучения. Увеличение этого параметра обеспечит выполнение большего количества случайных действий. Если энтропия падает слишком быстро, увеличиваем beta. Если энтропия падает слишком медленно, уменьшаем beta.

`epsilon`
	Влияет на скорость изменения политики во время обучения. Соответствует допустимому порогу расхождения между старой и новой политикой при обновлении градиентного спуска. 
	
`lambd`
	Параметр регуляризации (лямбда), используемый при расчете обобщенной оценки преимущества ( GAE ). Лямбду можно рассматривать как то, насколько агент полагается на свою текущую оценку стоимости при вычислении обновленной оценки стоимости. 

`num_epoch`
	Количество проходов через буфер опыта при выполнении оптимизации градиентного спуска. Чем больше размер партии, тем больше это допустимо. Уменьшение этого параметра обеспечит более стабильные обновления за счет более медленного обучения.

`learning_rate_schedule`
	(по умолчанию = linear для PPO и constant для SAC) Определяет, как скорость обучения изменяется с течением времени. Для PPO в документации рекомендовано снижать скорость обучения до значения max_steps, чтобы обучение сходилось более стабильно. 

`linear` 
	скорость обучения уменьшается линейно, достигая 0 на max_steps, сохраняя при constant этом скорость обучения постоянной для всего тренировочного прогона.

`network_settings`
	Раздел, содержащий настройки нейронной сети.

`normalize`
	Параметр определяет применяется ли нормализация к входным данным векторных наблюдений. Нормализация может быть полезна в случаях со сложными задачами непрерывного управления, но может быть вредна для более простых задач дискретного управления. По умолчанию false.

`hidden_units`
	Количество единиц в скрытых слоях нейронной сети. Соответствуют количеству единиц в каждом полносвязном слое нейронной сети. 

`num_layers`
	Количество скрытых слоев в нейронной сети. Соответствует количеству скрытых слоев после ввода наблюдения или после кодирования CNN визуального наблюдения. 

`reward_signals`
	Раздел позволяет задавать настройки как для внешних (т. е. основанных на среде), так и для внутренних сигналов вознаграждения (например, любопытство и GAIL). 

`extrinsic`
	Внешние сигналы вознаграждения(основанные на среде)
 
`gamma`
	(по умолчанию = 0.99) Фактор скидки для будущих вознаграждений, поступающих из окружающей среды. Это можно рассматривать как то, как далеко в будущем агент должен заботиться о возможных вознаграждениях. 

`strength`
(по умолчанию = 1.0) Коэффициент, на который умножается вознаграждение, данное средой. Диапазоны будут варьируются в зависимости от сигнала вознаграждения.

`max_steps`
Общее количество шагов (т. е. собранных наблюдений и предпринятых действий), которые необходимо выполнить в среде (или во всех средах при параллельном использовании нескольких) перед завершением процесса обучения. 

`time_horizon`
Сколько шагов опыта необходимо собрать для каждого агента, прежде чем добавить его в буфер опыта. Когда этот предел достигается до конца эпизода, оценка значения используется для прогнозирования общего ожидаемого вознаграждения из текущего состояния агента. 

`summary_freq`
(по умолчанию = 50000) Количество опытов, которое необходимо собрать перед созданием и отображением статистики обучения. Это определяет детализацию графиков в Tensorboard.

### Компоненты	

- `Decision Requester`
Цель обучения с подкреплением — изучить наилучшую политику (сопоставление состояний с действиями), которая максимизирует возможные вознаграждения. Компонент “Decision Requester” выбирает, какое действие будет принято в данном эпизоде обучения.

- `Decision Period`
Соответсвенно отвечает за частоту опроса решений из базы знаний агента. 

- `Take Actions Between Decision Period`
Отвечает за то, будет ли агент принимать решения во время отработки действий из базы знаний.

- `Behavior Parameters`
В среде RollerBall базовый объект RollerAgent имеет несколько свойств, влияющих на его поведение:
Параметры поведения — у каждого агента должно быть поведение. Поведение определяет, как агент принимает решения.

- `Stacked Vectors`
Количество состояний, которые будут склаываться перед подачей в нейронную сеть.

- `Continuous Actions`
Количество непрерывных действий.

- Discrete Branches`
Кол-во дискретных ветвей (наборов событий для получения награды).

- `Model` 
Обученная модель поведения, вкладывается после успешного обучения агента.

- `Interface Device`
Устройство для генерации поведения объекта (под управлением нейронной сети).

- `Behavior Type`
Отвечает за функционал агента. Например, невозможно обучить агента, если данное поле имеет параметр только вывод, поскольку он подразумевает использование накопленной информации.

- `Team ID`
Параметр для выбора команды для агента или ИИ. Например, если мы делаем спортивный симулятор, то мы будем обучать агентов одному и тому же, но играть они будут против друг друга.

- `Observable Attribute Hand`
Изучение агентом всевозможных атрибутов. Может исключаться, изучить все или только дочерние атрибут для тела под управлением агента.

## Вывод
В данной лабораторной работе был обучен ML-агент для реализации машинного обучения в среде Unity. Для обучения были использованы библиотеки mlagents 0.28.0 и torch версии 1.7.1. Для выполнения задания №2 была изучена документация по созданию тренера агента.
