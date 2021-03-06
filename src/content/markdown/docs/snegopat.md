#1Script, как плагин для «Снегопата»

Популярное расширение конфигуратора ["Снегопат"](http://snegopat.ru) поддерживает подключение сторонних модулей, написанных на различных языках. 1Script позволяет создавать плагины к Снегопату на языке 1С.

## Оглавление

* [Подключение к системе](#connect)
* [Разработка аддинов](#addins)
	* [Текстовые скрипты на языке 1С](#addins-txt)
	* [Разработка форм на языке 1С](#addins-form)
* [Программное окружение](#env)
	* [Доступ к Snegopat API](#snegopat-api)
	* [Переменная *ЭтотОбъект*](#this-obj)
	* [Создание объектов платформы с помощью оператора "Новый"](#new-instance)
	* [Доступ к объекту "Форма" из кода формы](#this-form) 

## Подключение к системе {#connect}
ScriptEngine.Snegopat.dll - это DLL-addin, который необходимо загрузить в Снегопат. Для этого нужно прописать в addins.ini строку: *dll:<путь к ScriptEngine.Snegopat.dll>*

После загрузки в контекстном меню Снегопата появится пункт "Загрузить скрипт 1С".

Рекомендуется прописать путь к ScriptEngine.Snegopat.dll в файле addins.ini Снегопата. По понятным причинам, данный путь должен располагаться в этом файле до любого из аддинов, выполняемых с помощью OneScript.

## Разработка аддинов {#addins}

OneScript поддерживает два вида подключаемых аддинов:

* Обычные скрипты, хранимые в текстовых файлах;
* Файлы форм, открываемых Снегопатом (с расширением *.ssf)

### Текстовые скрипты на языке 1С {#addins-txt}

Скрипты представляют собой текстовые файлы с расширением **.os или *.1scr. Рекомендуется использовать расширение .os, поскольку расширение .1scr поддерживается только из-за совместимости с предыдущими версиями.

АддИны на языке 1С имеют протокол (префикс) 1clang. Строка регистрации скрипта в addins.ini будет иметь вид: *1clang:<путь к скрипту>*

#### Макросы авторегистрации
Поддерживается использование макросов, обеспечивающих человекочитаемые названия в окне Снегопата. 
Макросы должны располагаться в самом начале скрипта, до какого-либо кода на языке 1С. Поддерживаются макросы *$dname* и *$uname*, означающие отображаемое и уникальное имя скрипта соответственно.

**Пример:**

	$dname Привет
	$uname Тестовый скрипт
	
	Функция СделайЧтоНибудь() Экспорт
		Designer.Message("Hello");
	КонецФункции 


#### Макросы Снегопата

В качестве макросов, доступных к запуску из Снегопата выступают экспортные процедуры или функции модуля, у которых есть префикс "Макрос_".

    Процедура Макрос_СделатьВсеХорошо() Экспорт // В Снегопате будет виден макрос "СделатьВсеХорошо"

#### Формы скриптов и обработчики событий

Для передачи в метод Designer.loadScriptForm предусмотрена глобальная переменная ЭтотОбъект.

Обработчики событий формы должны быть помечены, как экспортные:

    Процедура ОбработчикНажатия(Кнопка) Экспорт // экспорт обязателен
    
    КонецПроцедуры

### Разработка форм на языке 1С {#addins-form}
Традиционная модель применения форм в Снегопате предполагает наличие файла с описанием формы (*.ssf) и отдельного файла со скриптом, который подключается к Снегопату, как АддИн и обрабатывает события элементов формы. Такая двухфайловая схема неудобна тем, что диалоговое окно формы и ее программный код разрабатываются отдельно и рабочий процесс непривычен по сравнению с разработкой в Конфигураторе.

OneScript позволяет сделать разработку форм для Снегопата более привычной, за счет того, что форму можно подключить, как самостоятельный АддИн. В результате, не требуется отдельный файл скрипта с обработчиками событий. Программный код формы хранится в самой форме.

Для того, чтобы отличать классические "скриптозависимые" формы Снегопата от форм, предназначенных для самостоятельного использования, предлагается следовать соглашению назначать формам OneScript расширение ".osf". 

Загрузка формы, как АддИна поддерживает как расширение .ssf, так и расширение .osf.

#### Регистрация формы
Каждый АддИН Снегопата имеет уникальное имя, а также отображаемое имя, удобное пользователю. Для форм эти функции выполняет заголовок формы. В заголовке формы можно указать уникальное имя и отображаемое имя, разделив их косой чертой. Например:

    mySuperPlugin/Супер система помощи программисту

#### Макросы Снегопата и обработчики событий
Правила написания макросов и обработчиков событий внутри форм аналогичны тем, что действуют для обычных текстовых скриптов:

* Все обработчики событий должны быть объявлены, как экспортные. В этом основное отличие от привычной разработки под 1С. В Конфигураторе нет необходимости объявлять обработчики событий формы, как экспортные. В OneScript это обязательно. 
* Экспортные процедуры, начинающиеся с префикса "Макрос_" становятся макросами Снегопата. 

## Программное окружение {#env}

### Доступ к Snegopat API {#snegopat-api}

Взаимодействие со Снегопатом выполняется с помощью [Snegopat API](https://snegopat.ru/main/doc/trunk/docs/help/snegapi.markdown). Ниже приведены переменные и методы, глобально доступные в скриптах и формах.

Наиболее часто используемые функции имеют русскоязычные аналоги, что обеспечивает более удобный ввод кода на русском языке.

В первую очередь, в файле скрипта глобально доступна переменная *Designer/Конфигуратор*, предоставляющая полный доступ к [SnegopatAPI](https://snegopat.ru/main/doc/trunk/docs/help/snegapi.markdown).

Кроме того, для удобства, в глобальную область видимости также добавлены следующие свойства и методы объекта *Designer*:

#### Свойства

- addins
- cmdTrace
- events
- profileRoot
- snegopat
- hotkeys
- windows
- metadata
- v8files
- ownerName
- v8debug
- sVersion
- v8Version

#### Методы

- v8New / СоздатьОбъект
- Message / Сообщить
- MessageBox / Предупреждение / Вопрос
- globalContext / ГлобальныйКонтекст
- getCmdState / ПолучитьСостояниеКоманды
- saveProfile / СохранитьНастройку
- createTimer / СоздатьТаймер
- killTimer / ОстановитьТаймер
- toV8Value / ВЗначение1С
- loadResourceString / ЗагрузитьСтрокуРесурсов
- loadScriptForm, ЗагрузитьФормуИзФайла
- designScriptForm / РедактироватьФормуИзФайла
- sendCommand / ПослатьКоманду

Следует отметить, что русские идентификаторы действуют только на глобальную область видимости. То есть, вызов глобальной функции *СоздатьОбъект* будет аналогичен вызову *v8new*, однако, вызов по русскоязычному имени через точку приведет к ошибке: 

    А = СоздатьОбъект("Массив"); // Правильно
    А = Конфигуратор.v8new("Массив"); // Правильно
    А = Designer.v8new("Массив"); // Правильно
    А = Конфигуратор.СоздатьОбъект("Массив") // Ошибка!

### Переменная ЭтотОбъект {#this-obj}

Переменная *ЭтотОбъект* предоставляет доступ к объекту АддИна и служит для передачи текущего АддИна в Снегопат в качестве обработчика событий. Например:

    Форма = ЗагрузитьФормуИзФайла("C:\myform.osf", ЭтотОбъект);
	Форма.Открыть();

### Создание объектов платформы с помощью оператора "Новый" {#new-instance}
Для создания объектов платформы можно использовать оператор "Новый" вместо функции "v8new" Снегопата. То есть, обе приведенные конструкции аналогичны:

    Объект = v8new("Массив");
    Объект2 = Новый Массив;

### Доступ к объекту "Форма" из кода формы {#this-form}
Если АддИн представляет собой самостоятельную форму, то в коде формы будет доступна переменная "*ЭтаФорма*". Тип переменной "*ЭтаФорма*" - это стандартный объект "Форма" платформы 1С:Предприятие.

Кроме того, эта переменная единственный способ обращения к реквизитам и элементам формы. В Конфигураторе весь контекст формы доступен для обращения непосредственно из кода формы. Например, к реквизиту формы "Наименование" можно обращаться непосредственно, без уточнения области видимости.

	Если Наименование = ЭтаФорма.Наименование Тогда
        // Всегда Истина, т.е. реквизит формы "Наименование" доступен непосредственно
    КонецЕсли;

В OneScript контекст формы напрямую недоступен. Обращение к форме возможно только через переменную *ЭтаФорма*. Например:

    Если ЭтаФорма.Наименование = "Привет" Тогда
        ЭтаФорма.ЭлементыФормы.Флажок.Доступность = Ложь;
    КонецЕсли;

[**Перейти к оглавлению**](/docs)