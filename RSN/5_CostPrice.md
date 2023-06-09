# Задание к занятию «Расчет себестоимости и последовательности документов»

*Примерное время выполнения: 20–60 минут*

## Цель задания

1. На практике разобрать принцип списания суммы по средней себестоимости.
2. Научиться работать с последовательностями документов.
3. Подготовить конфигурацию к последующей работе.

## Чеклист готовности к домашнему заданию

- [ ] Установить учебную платформу версии 8.3.22 или больше.
- [ ] Подготовить разработанную ранее конфигурацию "УправлениеИТФирмой"
- [ ] Просмотреть материал занятия «Расчет себестоимости и последовательности документов».

## Инструкция к заданию

1. Решите описанные задачи в конфигураторе.

В тексте задания приведен программный код для решения задач. Конечно, Вы можете использовать его. Но для более продуктивного обучения, старайтесь, сначала, самостоятельно написать код. Если все же, самостоятельно решить задачу не удалось, используйте код из текста задачи но обязательно пройдите его в отладке, посмотрите, что попадает в переменные, как преобразуются значения. Старайтесь максимально детально разобраться в механизме.

2. Протестируйте решение в пользовательском режиме, обязательно введите данные в базу, убедитесь, что все работает.
3. Отправьте на проверку в личном кабинете Нетологии один общий файл базы данных (.dt), содержащей решение всех задач.

## Задача 1. Реализовать расчёт себестоимости товаров в документе «Реализация»

### Описание задачи

Необходимо реализовать суммовой учёт товаров на складе. Обеспечить, чтобы при реализации товаров, списание происходило корректно — количество, указанное в документе, сумма, рассчитанная по средней себестоимости.

### Процесс выполнения

1. Добавьте в регистр накопления Товары ресурс Сумма. Тип — определяемый тип, используемый для денежных значений.
2. Добавьте поле Сумма в отчёт «Остатки товаров». (добавьте его в запросе набора данных, определите как ресурс и выведите в поля отчета)
3. В обработке проведения документа прихода товара добавить заполнение суммы значениями, указанными в табличной части.

<details>
  <summary>Код</summary>
  
```bsl
Процедура ОбработкаПроведения(Отказ, Режим)

	Движения.Товары.Записывать = Истина;
	
	ТаблицаДляПроведения = Товары.Выгрузить();
	ТаблицаДляПроведения.Свернуть("Номенклатура", "Количество, Сумма");

	ТипУслуга = Перечисления.ТипНоменклатуры.Услуга;

	Для Каждого ТекСтрокаТовары Из ТаблицаДляПроведения Цикл

		Если ТекСтрокаТовары.Номенклатура.ТипНоменклатуры = ТипУслуга Тогда
			Продолжить;
		КонецЕсли;

		Движение = Движения.Товары.Добавить();
		Движение.ВидДвижения = ВидДвиженияНакопления.Приход;
		Движение.Период = Дата;
		Движение.Номенклатура = ТекСтрокаТовары.Номенклатура;
		Движение.Количество = ТекСтрокаТовары.Количество;
		Движение.Сумма = ТекСтрокаТовары.Сумма; 

	КонецЦикла;

КонецПроцедуры
```

</details>

4. В документ проведения реализации товаров добавьте заполнение поля «Сумма».
*(Попробуйте сначала воспроизвести ошибочную ситуацию, когда количество списано в ноль, а остаток по сумме отрицательный).*
Реализуйте списание суммы по средней себестоимости товаров.

<details>
  <summary>Код</summary>
  
```bsl
Процедура ОбработкаПроведения(Отказ, Режим)

	Движения.Продажи.Очистить();
	Движения.Продажи.Записать();

	Движения.Товары.Записывать = Истина;
	Движения.Продажи.Записывать = Истина;

	ТаблицаДляПроведения = Товары.Выгрузить();
	ТаблицаДляПроведения.Свернуть("Номенклатура", "Количество, Сумма");

	ТипТовар = Перечисления.ТипНоменклатуры.Товар;

	Для Каждого ТекСтрокаТовары Из ТаблицаДляПроведения Цикл

		Если ТекСтрокаТовары.Номенклатура.ТипНоменклатуры = ТипТовар Тогда 

			ОтборПоТовару = Новый Структура("Номенклатура", ТекСтрокаТовары.Номенклатура);
			Остатки = РегистрыНакопления.Товары.Остатки(Дата, ОтборПоТовару);

			Если Остатки.Количество() = 0 Тогда
				ТекстСообщения = СтрШаблон("Товара %1 нет на складе", ТекСтрокаТовары.Номенклатура);
				Сообщить(ТекстСообщения);
				Отказ = Истина;
				Продолжить;
			КонецЕсли;

			СтрокаОстатка = Остатки[0];
			Если СтрокаОстатка.Количество < ТекСтрокаТовары.Количество Тогда
				НедостатокТовара = ТекСтрокаТовары.Количество - СтрокаОстатка.Количество;
				ТекстСообщения = СтрШаблон("Товара %1 недостаточно на складе, не хватает %2", ТекСтрокаТовары.Номенклатура, НедостатокТовара);
				Сообщить(ТекстСообщения);
				Отказ = Истина;
				Продолжить;
			КонецЕсли;

			Если СтрокаОстатка.Количество = ТекСтрокаТовары.Количество Тогда
				СебестоимостьЕдиницыТовара = СтрокаОстатка.Сумма;
			Иначе
				СебестоимостьЕдиницыТовара = СтрокаОстатка.Сумма / СтрокаОстатка.Количество;
			КонецЕсли;
			СуммаКСписанию = СебестоимостьЕдиницыТовара * ТекСтрокаТовары.Количество; 

			Движение = Движения.Товары.Добавить();
			Движение.ВидДвижения = ВидДвиженияНакопления.Расход;
			Движение.Период = Дата;
			Движение.Номенклатура = ТекСтрокаТовары.Номенклатура;
			Движение.Количество = ТекСтрокаТовары.Количество;
            Движение.Сумма = СуммаКСписанию;
		КонецЕсли;

		Движение = Движения.Продажи.Добавить();
		Движение.Период = Дата;
		Движение.Сотрудник = Ответственный;
		Движение.Номенклатура = ТекСтрокаТовары.Номенклатура;
		Движение.Сумма = ТекСтрокаТовары.Сумма;

	КонецЦикла;

КонецПроцедуры
```

</details>

5. Убедитесь, что суммы списываются правильно. В отчет попадают корректные данные

## Задача 2. Реализовать последовательность документов для пересчёта документов «Поступление» и «Реализация товаров»

### Описание задачи

Для восстановления сбитого учёта остатков товаров нужно иметь возможность восстановить последовательность учёта.

### Процесс выполнения

На форму списка журнала документов добавьте поля для отображения границы и кнопки для работы с границей последовательности.

1. Создайте последовательность документов СкладскиеДокументы. В состав последовательности должны входить документы ПоступлениеТоваровИУслуг и РеализацияТоваровИУслуг. Влиять на последовательность будут движения в регистр Товары (то есть, если например, в документе Реализации товаров указаны только услуги, он не должен быть зарегистрирован в последовательности).
2. Добавьте реквизит формы ГраницаДата с типом Дата, состав - Дата и время
3. Добавьте реквизит формы ГраницаСсылка с составным типом, в состав должны входить ссылка на документ реализации и ссылка на документ поступления товаров
4. Добавьте команду ПрочитатьГраницу, в обработчике получите границу из последовательности, заполните ее значения в реквизиты формы
5. Добавьте команду ВосстановитьГраницу, в ее обработчике пропишите восстановление границы последовательности и вызывайте заполнение метод, которым читаете границу (чтобы новые значения сразу прописались в реквизиты)

<details>
  <summary>Код</summary>
  
```bsl
&НаСервере
Процедура ПрочитатьГраницуНаСервере()

	ГраницаПоследовательности = Последовательности.СкладскиеДокументы.ПолучитьГраницу();
	ГраницаДата = ГраницаПоследовательности.Дата;
	ГраницаСсылка = ГраницаПоследовательности.Ссылка;

КонецПроцедуры

&НаКлиенте
Процедура ПрочитатьГраницу(Команда)
	ПрочитатьГраницуНаСервере();
КонецПроцедуры

&НаСервере
Процедура ВосстановитьГраницуНаСервере()

	Последовательности.СкладскиеДокументы.Восстановить();
	ПрочитатьГраницуНаСервере();

КонецПроцедуры

&НаКлиенте
Процедура ВосстановитьГраницу(Команда)
	ВосстановитьГраницуНаСервере();
КонецПроцедуры
```

</details>

6. Разместите реквизиты и команды на форме. Установите для реквизитов только просмотр, чтобы пользователи не могли менять значения в них, или выведите полями надписи
7. Убедитесь, что восстановление последовательности работает

## Пример

[Пример выполнения домашнего задания](examples/HW_5_5_example.md)

## Критерии оценки

Зачёт ставится, если:

1. Программа запускается, не возникает явных ошибок, исключений при выполнении программы (в том числе, если Вы начали делать дополнительную задачу, ее функционал не должен приводить к ошибкам и исключениям)
2. При проведении документа Поступления товаров в регистр записывается сумма
3. При проведении Реализации товаров, в регистр записывается сумма, рассчитанная по средней себестоимости товаров
4. В Журнале документов добавлены поля и команды для работы с границей последовательности.
5. Введены тестовые данные

Все задачи обязательны к выполнению (кроме текста под спойлером "Дополнительно" - эти задачи делать не обязательно. Возможно, Вы вернетесь к ним позднее, после того, как изучите дополнительный материал).

Пожалуйста, присылайте на проверку все задачи сразу, одним файлом выгрузки информационной базы (dt)

Любые вопросы по решению задач задавайте в чате учебной группы.
