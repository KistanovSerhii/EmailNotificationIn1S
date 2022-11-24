# ВАЖНО: указать имя рабочей базы!
см. общий модуль "ИТФоновоеВыполнение" Функция "ЭтоРабочаяБазаДанных"



# EmailNotificationIn1S

Расширение - уведомление почтовым сообщением при первом внесении данных (создание элемента справочника, проведение документа). Отправка почтового сообщения выполняется через фоновое задание. Это расширение является доработкой расширения https://github.com/KistanovSerhii/ExtensionZUPNoticeFired.git

1. Реквизит "Ответственный исполнитель" справочника "Договоры" - это ссылка на справочник физические лица где есть реквизит "Email" и он должен быть заполнен!

2. Реквизит "Владелец бюджета" справочника "Договоры" - это дополнительный реквизит и для указания почты для данного реквизита мы должны зайти в регистр сведений "Дополнительные значения доп. объектов" и добавить ссылку на доп. реквизит и указать почту которая будет ему присвоена.

3. Уведомление отправляется только тем объектам которых нет в регистре сведений "Уведомление отправлено" 
(после первого отправления создается запись в данном регистре по отправленному объекту тем самым мы выполняем отправку уведомления только один раз)


![image](https://user-images.githubusercontent.com/28355711/196092948-888fe2ad-143c-425f-92e1-c9176272c535.png)

![image](https://user-images.githubusercontent.com/28355711/196093252-ddc92145-9cf8-42f7-8023-5ab678889091.png)

![image](https://user-images.githubusercontent.com/28355711/196093113-9dc98546-26a1-4bf7-a96d-697cd269ff11.png)

![image](https://user-images.githubusercontent.com/28355711/196093516-cef402af-d53e-4f3e-a52a-52da57eadc91.png)

![image](https://user-images.githubusercontent.com/28355711/196093578-5e3b61c6-510d-4203-b1fa-40ad49383d25.png)

# EmailNotificationIn1S_v2

В данной версии:
1. Изменен алгоритм заполнения текста письма (в текст записываем &ИмяРеквизита, в цикле выполнится замена на значение реквизита)
2. Добавлен регистр сведений "ПодпискаНаРассылку" где указываются подписчики на рассылку (email адреса и тип рассылки), а в v1 адрес
получается из конкретного объекта базы данных (например проводимый документ может содержать реквизит получательУведомления)

# ВАЖНО:
Через фоновое задание дублируется отправка сообщения если:
    
    - Пользователь нажал "Провести" -> запустил фоновое задание отправки уведомления, но процесс отправки это длительная операция
      (подключение к почте, отправка ...) по этой причине пользователь имеет возможность запустить еще одно фоновое задание отправки уведомления
      (например после "Провести" нажмет "Провести и закрыть") и оно тоже отправится так как в регистр сведений "УведомлениеОтправлено" запись будет
      добавлена исключительно по завершению выполнения запущенного фонового задания!!!
      
      - Решение: реализация через реквизит "УведомлениеОтправлено" с типом булево (я так не делал так как не хотел менять конфу, НО можно было сделать
        через дополнительный реквизит).
        В таком случаи необходимо: 
        1. Устанавливать данному реквизиту ложь при копировании объекта
        2. Сразу по подписке (в начале процедуры) проверять реквизит и выполнять/не выполнять процедуру отправки
        3. Если проверка реквизита дает добро на выполнения отправки (равно Истина) тогда устанавливаем зня Истина
        4. Если отправить не удалось - установить данному реквизиту знч ложь
        
*Пример см. EmailNotificationIn1S_v3.cfe (реализация через реквизит объекта)




# Внедрения "EmailNotificationIn1S_v3.2"

// Данный вариант реализован через очередь (регистром сведений) и регламентное задание, пользователь записывает новый или измененный объект Тогда мы добавляем запись в регистр сведений хранящий ссылку на объект и признак "Отправлено", а регламентное задание по установленному нами рассписанию выбирает все записи данного регистра с признако "Отправлено==Ложь" и отправляет устанавливая "Отправлено==Истина" при удачной отправки уведомления.

#Область ОписаниеВнедрения

0. В режиме конфигуратора, у регистра сведений "ПочтовоеУведомление", измерению "Источник" - указать ТИП 
1. Наполняем регистр сведений "ПодпискаНаРассылку"
2. Добавляем нужному объекту в подходящем месте - регистрацию объекта на выполнения регламентным заданием:
   ИТРаботаСУведомлением.РегистрироватьДляВыполненияРегламентнымЗаданием(ТекущийОбъект.Ссылка);

    Пример:

2.1 только для существующих которые редактировали (модуль формы)
&НаСервере
Процедура ПередЗаписьюНаСервере(Отказ, ТекущийОбъект, ПараметрыЗаписи) // Модифицированность есть только ДО записи!
	Если ЭтаФорма.Модифицированность И НЕ ТекущийОбъект.Ссылка.Пустая() Тогда
		ИТРаботаСУведомлением.РегистрироватьДляВыполненияРегламентнымЗаданием(ТекущийОбъект.Ссылка);
	КонецЕсли;
КонецПроцедуры 

2.2 только для новых (модуль объекта)
Процедура ПриЗаписи(Отказ)
	Если НЕ РегистрыСведений.ПочтовоеУведомление.ЗаписьДляИсточникаУжеСуществует(Ссылка) Тогда
		ИТРаботаСУведомлением.РегистрироватьДляВыполненияРегламентнымЗаданием(Ссылка);
	КонецЕсли;	
КонецПроцедуры

3. Создаем регламентное задание которое по расписанию должно выполнять процедуру "ОтправитьВсеОжидающиеУведомлениеНаПочту"

#КонецОбласти
