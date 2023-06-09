# Задание к занятию «Регистры сведений»

*Примерное время выполнения: 20–60 минут*

## Цель задания

1. Изучить работу с регистрами сведений.
2. Научиться создавать регистры сведений, вводить в них информацию и получать данные.
3. Подготовить конфигурацию к последующей работе.

## Чеклист готовности к домашнему заданию

- [ ] Установить учебную платформу версии 8.3.22 или больше.
- [ ] Подготовить разработанную ранее конфигурацию "УправлениеИТФирмой"
- [ ] Просмотреть материал занятия «Регистры сведений».

## Инструкция к заданию

1. Решите описанные задачи в конфигураторе.

В тексте задания приведен программный код для решения задач. Конечно, Вы можете использовать его. Но для более продуктивного обучения, старайтесь, сначала, самостоятельно написать код. Если все же, самостоятельно решить задачу не удалось, используйте код из текста задачи но обязательно пройдите его в отладке, посмотрите, что попадает в переменные, как преобразуются значения. Старайтесь максимально детально разобраться в механизме.

2. Протестируйте решение в пользовательском режиме, обязательно введите данные в базу, убедитесь, что все работает.
3. Отправьте на проверку в личном кабинете Нетологии один общий файл базы данных (.dt), содержащей решение всех задач.

## Задача 1. Регистр "Цены продажи номенклатуры"

### Описание задачи

Реализуем периодический регистр сведений, в котором будем хранить цены ниже которых менеджерам нельзя продавать товары.
В форме списка справочника «Номенклатура» должен находиться динамический список с ценами и соответствующая ему таблица. Список должен быть отобран по активному товару (услуге) и упорядочен по убыванию дат. При создании новых записей регистра Цены в реквизит Установил должен автоматически подставляться текущий сотрудник.

### Процесс выполнения

1. Создайте Регистр сведений "ЦеныПродажиНоменклатуры". Не забудьте установить представление списка и представление записи. Позаботьтесь о настройке прав.
2. Периодичность регистра - "В пределах секунды".
3. Режим записи - "Независимый".
4. Укажите подсистему (например, Служебное).
5. Чтобы регистр не отображался в интерфейсе, дополнительно, отключите флаг "Использовать стандартные команды" на закладке Команды.
6. На закладке Данные определите единственное измерение Номенклатура (СправочникСсылка.Номенклатура, Ведущее), ресурс Цена (Число или определяемый тип), реквизит Ответственный (СправочникСсылка.Сотрудники).
7. В модуле набора записей пропишите установку значения Ответственный для новых записей, по умолчанию - значением параметра сеанса

<details>
  <summary>Код</summary>
  
```bsl
Процедура ОбработкаЗаполнения(ДанныеЗаполнения, СтандартнаяОбработка)

	ТекущийСотрудник = ПараметрыСеанса.ТекущийСотрудник;

	Для Каждого Запись Из ЭтотОбъект Цикл

		Запись.Ответственный = ТекущийСотрудник;

	КонецЦикла; 

КонецПроцедуры
```

</details>

8. В форме списка справочника «Номенклатура» из предыдущих заданий:

- Создайте динамический список "СписокЦен" с основной таблицей "РегистрСведений.Цены";
- Выведите его таблицей формы под таблицей с элементами;
- В обработчике ПриАктивизацииСтроки **таблицы с элементами номенклатуры** установите отбор по измерению «Номенклатура» в списке цен аналогично отбору элементов при активизации группы;
- Поскольку установка отбора динамического списка — операция популярная, лучше создать для неё процедуру в общем модуле, например, РаботаСФормамиКлиент.УстановитьОтборВСписке(). (если ранее это уже было сделано, Вы уже сейчас упростили себе работу, т.к. достаточно просто вызвать этот метод)

<details>
  <summary>Код</summary>

Форма справочника Номенклатура:

```bsl
&НаКлиенте
Процедура СписокПриАктивизацииСтроки(Элемент)

	ПодключитьОбработчикОжидания("УстановитьОтборЦен", 0.1, Истина);

КонецПроцедуры

&НаКлиенте
Процедура УстановитьОтборЦен()

	ПолеНоменклатура = Новый ПолеКомпоновкиДанных("Номенклатура");
	ВыбранноеЗначение = Элементы.Список.ТекущаяСтрока;
	РаботаСФормамиКлиент.УстановитьОтборВСписке(СписокЦен, ПолеНоменклатура, ВыбранноеЗначение);

КонецПроцедуры
```

Общий модуль РаботаСФормамиКлиент:

```bsl
Функция НайтиИлиСоздатьОтборВСписке(Список, Поле) Экспорт

	ЭлементыОтбора = Список.КомпоновщикНастроек.Настройки.Отбор.Элементы;
	НайденныйЭлементОтбора = Неопределено;

	Для Каждого ЭлементОтбора Из ЭлементыОтбора Цикл
		Если ЭлементОтбора.ЛевоеЗначение = Поле Тогда
			НайденныйЭлементОтбора = ЭлементОтбора;
			Прервать;
		КонецЕсли;
	КонецЦикла;

	Если НайденныйЭлементОтбора = Неопределено Тогда
		НайденныйЭлементОтбора = ЭлементыОтбора.Добавить(Тип("ЭлементОтбораКомпоновкиДанных"));
		НайденныйЭлементОтбора.ЛевоеЗначение = Поле;
	КонецЕсли;

	Возврат НайденныйЭлементОтбора;

КонецФункции

Процедура УстановитьОтборВСписке(СписокДляУстановкиОтбора, ПолеОтбора, ЗначениеОтбора) Экспорт

	НайденныйЭлементОтбора = НайтиИлиСоздатьОтборВСписке(СписокДляУстановкиОтбора, ПолеОтбора);
	НайденныйЭлементОтбора.Использование = Истина;
	НайденныйЭлементОтбора.РежимОтображения = РежимОтображенияЭлементаНастройкиКомпоновкиДанных.Недоступный;
	НайденныйЭлементОтбора.ВидСравнения = ВидСравненияКомпоновкиДанных.Равно;
	НайденныйЭлементОтбора.ПравоеЗначение = ЗначениеОтбора;

КонецПроцедуры
```

</details>

9. Введите тестовые данные - хотя бы, 5-6 записей цен. Убедитесь, что отборы устанавливаются корректно

## Задача 2. История цен продажи

### Описание задачи

Создадим отчет, отображающий историю изменения цены продажи номенклатуры.

### Процесс выполнения

1. Создайте подсистему Отчетность, если ранее она не была реализована в конфигурации
2. Создайте новый отчет "ИсторияЦенПродажи", укажите для него подсистему Отчетность
3. Создайте схему компоновки данных отчета.
4. Определите набор данных - запрос. В конструкторе запроса выберите все поля из таблицы регистра сведений ЦеныПродажиНоменклатуры.
5. На закладке Ресурсы определите поле Цена как ресурс с выражением Среднее(Цена)
6. На закладке Настройки создайте группировку по полю Номенклатура с иерархией, а в ней группировку детальных записей.
7. Определите Выбранные поля - Период, Цена (в контексте отчета не так важно, то установил цену, но при необходимости в пользовательском режиме это поле можно будет добавить)
8. Убедитесь в пользовательском режиме, что отчет формируется.

## Задача 3. Регистр "Цены поставщиков"

### Описание задачи

Сразу оговоримся, что донастройку данного регистра мы выполним в одном из будущих домашних заданий. Сейчас его надо только создать и ввести тестовые данные, чтобы убедиться, что все создано верно

### Процесс выполнения

1. Создайте Регистр сведений "ЦеныПоставщиков". Не забудьте установить представление списка и представление записи. Позаботьтесь о настройке прав.
2. Периодичность регистра - "В пределах дня"
3. Режим записи - "Независимый".
4. Укажите подсистему (например, Служебное).
5. Чтобы регистр не отображался в интерфейсе, дополнительно, отключите флаг "Использовать стандартные команды" на закладке Команды.
6. На закладке Данные определите измерения Номенклатура (СправочникСсылка.Номенклатура, Ведущее) и Контрагент (СправочникСсылка.Контрагенты, Ведущее), ресурс Цена (Число или определяемый тип), реквизит Ответственный (СправочникСсылка.Сотрудники).
7. В модуле набора записей пропишите установку значения Ответственный для новых записей, по умолчанию - значением параметра сеанса

<details>
  <summary>Код</summary>
  
```bsl
Процедура ОбработкаЗаполнения(ДанныеЗаполнения, СтандартнаяОбработка)

	ТекущийСотрудник = ПараметрыСеанса.ТекущийСотрудник;

	Для Каждого Запись Из ЭтотОбъект Цикл

		Запись.Ответственный = ТекущийСотрудник;

	КонецЦикла; 

КонецПроцедуры
```

</details>

8. Откройте регистр в пользовательском режиме через функции технического специалиста, и введите тестовые данные

## Задача 4*. Отображение цен поставщика в форме контактного лица

*Это дополнительная задача, реализовывать ее не обязательно.*
*Задача предназначена для тех студентов, которым первые покажутся слишкм простыми.*
*В процессе выполнения не будут даны примеры программного кода.*

### Описание задачи

В форме элемента справочника Контактные лица вывести список цен поставщиков с отбором по владельцу контактного лица.

### Процесс выполнения

1. Откройте форму элемента справочника Контактные лица
2. Добавьте в форму реквизит - динамический список. Задайте ему подходящее наименование. В качестве основной таблицы установите РегистрСведений.ЦеныПоставщиков
3. При открытии формы, в этом списке должен устанавливаться отбор по контрагенту. В качестве значения отбора используйте владельца контактного лица. Т.к. это Владелец, мы можем быть уверены, что контактное лицо без указания Контрагента записано быть не может

<details>
  <summary>Дополнительно</summary>
  
При изменении контрагента, в списке надо обновить отбор

</details>

4. Убедитесь, что таблица формы заполняется при открытии формы.

## Пример

[Пример выполнения домашнего задания](examples/HW_5_1_example.md)

## Критерии оценки

Зачёт ставится, если:

1. Программа запускается, не возникает явных ошибок, исключений при выполнении программы (в том числе, если Вы начали делать дополнительную задачу, ее функционал не должен приводить к ошибкам и исключениям)
2. Создан регистр сведений "ЦеныПродажиНоменклатуры". Состав Измерений, Ресурсов и Реквизитов регистра соответствует заданию.
3. Создан отчет "ИсторияЦенПродажи", отчет выведен в интерфейс и формируется по данным регистра
4. Создан регистр сведений "ЦеныПоставщиков"
5. Введены тестовые данные

Задачи 1-3 обязательны к выполнению (кроме текста под спойлером "Дополнительно" - эти задачи делать не обязательно. Возможно, Вы вернетесь к ним позднее, после того, как изучите дополнительный материал). Задача 4 может быть не выполнена.

Пожалуйста, присылайте на проверку все задачи сразу, одним файлом выгрузки информационной базы (dt)

Любые вопросы по решению задач задавайте в чате учебной группы.
