# NVerSS - Nested Version Semantic String - Семантическая строка версии с наследованием

Текущая ревизия NVSS: **nvss-1.0.0-ru**

NVSS - Стандартизация обозначения версий объектов. Карманный аналог Git или Svn, где стандартизируется и контролируется только сама строка с версией а не само содержимое. При изменениях объекта меняется его строка с версией в соответствии с правилами NVSS.

* **Отсутствие конфликтов версий при параллельной работе над копиями одного объекта**.
* Полная поддержка [SemVer](https://semver.org).
* Поддержка неограниченного наследования модификаций версии.
* Поддержка веток.
* Наличие адресов и области прав для авторов версий.
* Человекочитаемый синтаксис. Адаптирован для ручной работы.

NVSS оптимизирована под ручную работу с версиями, что позволяет применять эту систему там, где полноценные системы контроля версий избыточны либо неудобны.

Например:

* одиночные файлы, шаблоны, документы, каждый со своей версией
* параллельная работа/рецензирование печатных документов
* работа с графическим контентом

Наследование версий позволяет упростить объединение разных модификаций, а также позволяет узнать откуда пришла модификация и ее актуальность.

Простота синтаксиса позволяет быстро сделать парсеры для автоматизации.

## Содержание

- [NVerSS - Nested Version Semantic String - Семантическая строка версии с наследованием](#nverss---nested-version-semantic-string---семантическая-строка-версии-с-наследованием)
	- [Содержание](#содержание)
	- [Быстрая справка](#быстрая-справка)
	- [Определения](#определения)
	- [Сравнение с git](#сравнение-с-git)
	- [Признаки использования NVSS с объектом](#признаки-использования-nvss-с-объектом)
	- [Мастер-версия объекта](#мастер-версия-объекта)
		- [Формат мастер-версии объекта](#формат-мастер-версии-объекта)
		- [Описание структуры мастер-версии объекта](#описание-структуры-мастер-версии-объекта)
	- [Изменение объекта](#изменение-объекта)
		- [Формат модификаций и веток](#формат-модификаций-и-веток)
		- [Описание структуры модификаций и веток](#описание-структуры-модификаций-и-веток)
	- [Используемый синтаксис](#используемый-синтаксис)
		- [Разделители](#разделители)
		- [Разрешенные символы в префиксах, адресах и названиях](#разрешенные-символы-в-префиксах-адресах-и-названиях)
	- [Описание основных действий с примерами](#описание-основных-действий-с-примерами)
		- [Мастер-версия: создание (init)](#мастер-версия-создание-init)
		- [Мастер-версия: изменение (\[fetch downstream + merge\] | commit)](#мастер-версия-изменение-fetch-downstream--merge--commit)
		- [Мастер-версия: смена адреса](#мастер-версия-смена-адреса)
		- [Мастер-версия: исправление (amend)](#мастер-версия-исправление-amend)
		- [Модификация: создание (clone upstream + commit)](#модификация-создание-clone-upstream--commit)
		- [Модификация: изменение (\[fetch downstream + merge\] | commit)](#модификация-изменение-fetch-downstream--merge--commit)
		- [Модификация: исправление (amend)](#модификация-исправление-amend)
		- [Модификация: смена базы (pull upstream)](#модификация-смена-базы-pull-upstream)
		- [Модификация: смена адреса](#модификация-смена-адреса)
		- [Ветка: создание (branch \[+ commit\])](#ветка-создание-branch--commit)
		- [Ветка: изменение (commit)](#ветка-изменение-commit)
		- [Ветка: исправление (amend)](#ветка-исправление-amend)
		- [Ветка: смена базы (merge + commit)](#ветка-смена-базы-merge--commit)
		- [Ветка: смена названия (branch -m)](#ветка-смена-названия-branch--m)
	- [Примеры NVSS](#примеры-nvss)
	- [Ссылки и доп. информация](#ссылки-и-доп-информация)

## Быстрая справка

Мастер-версия объекта:

```ebnf
[nvss "-"] [masterAddress "-"] [verPrefix] (version | timeSnapshot).
```

Модификация объекта:

```ebnf
masterBase "-" {nestedBase "-"} modAddress "-" modChanges.
```

Ветка объекта:

```ebnf
masterBase "-" {nestedBase "-"} "." branchName "-" branchChanges.
```

Полный вариант:

```ebnf
master {mod | branch}
```

## Определения

* NVSS - Nested Version Semantic String, Семантическая строка версии с наследованием
* SemVer - семантическое версионирование (см. [SemVer](https://semver.org))
* объект - любой программный или физический объект, который требует использования контроля версий
* мастер-версия - основная/корневая версия объекта, назначаемая автором объекта
* модификация - измененная версия объекта с другим адресом
* ветка - измененная версия объекта без смены адреса (и прав на редактирование)
* база - исходная (наследованная) версия объекта (и неизменяемая часть строки NVSS), на которой основана модификация или ветка
* адрес - определяет (и различает) местоположение объекта
* автор - создатель мастер-версии, модификации или ветки

Определения - действия

* изменение - действие, после которого должна быть изменена версия объекта
* исправление - действие, после которого версия объекта не изменяется. Допускается использовать при отсутствии использования данной версии где либо
* объединение - процесс слияния различных версий объекта

## Сравнение с git

|  Git   | NVSS                                                                                     |
| :----: | :--------------------------------------------------------------------------------------- |
|  init  | присвоение мастер-версии объекту                                                         |
| clone  | простое копирование объекта и создание модификации                                       |
| fetch  | принимаются версии объекта от разных авторов                                             |
|  pull  | принимаются версии объекта от разных авторов, происходит объединение и изменяется версия |
|  push  | отправляется текущая версия объекта                                                      |
| branch | простое копирование объекта и создание ветки                                             |
| merge  | объединение разных версий объекта                                                        |
| commit | изменение строки версии                                                                  |
| amend  | исправление объекта                                                                      |
|  add   | добавление нового объекта в группу при контроле нескольких объетов под одной версией     |
| remove | удаление объекта из группы при контроле нескольких объетов под одной версией             |
| revert | при наличии требуемой версии объекта удаление текущей и замена на требуемую              |

## Признаки использования NVSS с объектом

* Наличие префикса `nvss-` в версии объекта (явное использование).
* Версия объекта удовлетворяет настоящим правилам версионирования NVSS (неявное использование).

## Мастер-версия объекта

Мастер-версия является основной версией объекта.

Контроль версии объекта в формате NVSS требуется начинать с присвоения ему мастер-версии. Строка версии может сливаться с чем-либо (напр. названием объекта) через любой доступный разделитель. Начало строки версии рекомендуется определять по ключевому слову `nvss` или наличием адреса *мастер-версии* и разделителем `-` перед обозначением версии (см. далее).

* **Гарантирует отсутствие сторонних модификаций** при использовании.
* Неявно является основной веткой объекта и в большинстве случаев является основной рабочей версией объекта.

### Формат мастер-версии объекта

```ebnf
[nvss "-"] [masterAddress "-"] [verPrefix] (version | timeSnapshot).
```

> **Примечание:** *B `[]` скобках находятся необязательные части.*

### Описание структуры мастер-версии объекта

```ebnf
["nvss"]
```

Необязательный тег-заголовок `nvss` с разделителем `-`.

* Рекомендуется использовать для явного указания начала строки версии в формате NVSS.
* Можно использовать как тег для поиска.
* Может отсутствовать при наличии информации о версии в названии объекта или заведомой известности использования NVSS с объектом для сокращения длины строки версии.

```ebnf
[masterAddress]
```

Необязательный **уникальный** адрес мастер-версии объекта с разделителем `-`.

* По этому адресу определяется автор мастер-версии.
* Должен быть понятным и кратким для его быстрого и безошибочного определения.
* Может отсутствовать если он заранее известен или безошибочно и легко определяется какими либо другими способами или признаками. Также может отсутствовать при неопределенности в мастере.
* Может меняться, появляться и исчезать при необходимости, при этом желательно оповещать об этом заранее других зависимых авторов для сохранения их связи. Рекомендуется присвоить новый адрес с упоминанием в нем старого адреса (двойной адрес) некоторое время, для напоминания остальным о произошедшей смене.

`masterAddress` **всегда должен включать хотя бы одну букву** для его однозначного определения в строке, рекомендуется начинать с буквы.

```ebnf
[verPrefix]
```

Необязательный префикс к обозначению версии. Напр. `v`, `rev` или любой другой.

Рекомендуется к использованию при короткой строке версии, при отсутствии тега-заголовка `nvss` и отсутствии адреса мастер-версии `masterAddress`, для более "различимого" обозначения версии.

Часто используемые:

* `v`, `ver` - версия
* `r`, `rev` - ревизия
* `i`, `itr` - итерация
* `s`, `snp` - снимок состояния (снапшот)
* `b`, `bak` - резервная копия (бэкап)

```ebnf
version
```

Обозначение *мастер-версии*.

* Меняется при каждом изменении объекта.
* Не меняется при смене `masterAddress`.
* Возможно исправление объекта (объединение объекта без изменения версии **при гарантированном осутствии использования данной версии где либо**).

Структура обозначения версии использует [семантическое версионирование](https://semver.org) для частей `major`, `minor`, `patch` и допускает несколько форматов (в порядке упрощения):

* `major "." minor "." patch ["-" tag {"." tag}]` - semver-совместимо (`tag` аналогичен `pre-release` с некоторыми ограничениями (см. ниже описание `tag`)  )
* `major "." minor ["-" tag {"." tag}]`
* `major ["-" tag {"." tag}]`

или вцелом:

```ebnf
version = major ["." minor | ("." minor "." patch)] ["-" tag {"." tag}].
```

Описание и характеристики полей:

* `major` - главная версия (число), увеличивается при серьезных изменениях (влекущие несовместимость с предыдущей `major` версией)
* `minor` - второстепенная версия (число), увеличивается при незначительных изменениях (имеется совместимость с текущей `major` версией)
* `patch` - патч текущей версии (число), увеличивается при исправлениях ошибок, патчах (имеется совместимость с текущей `major.minor` версией)
* `tag` - необязательные тэги текущей версии с разделителями через `.`. В отличие от semver, NVSS запрещает использование символа `-` в названиях тегов. Используется для пояснения особенностей версии, напр. `-rc1` или `-dev` или `-prerelease` и др.

> **Примечание:** *В `{}` скобках находятся повторяющиеся части 0 и более раз.*

```ebnf
timeSnapshot
```

Метка времени (таймкод) мастер-версии.

* Меняется при каждом изменении объекта.
* Не меняется при смене `masterAddress`.
* Может меняться не обязательно на текущее время, но и на прошлое, если оно позже обозначенного в версии.
* Возможно исправление объекта (объединение объекта без изменения версии **при гарантированном осутствии использования данной версии где либо**).

Структура метки времени должна соответствовать стандарту [ISO 8601](https://ru.wikipedia.org/wiki/ISO_8601) и допускает использование нескольких форматов. Допустимые маски формата времени (в порядке упрощения):

* `"YYYYMMDDThhmmss.sss" [utcOffset]`
* `"YYYYMMDDThhmmss" [utcOffset]`
* `"YYYYMMDDThhmm" [utcOffset]`
* `"YYYYMMDDThh" [utcOffset]`
* `"YYYYMMDD" [utcOffset]`
* `"YYYY" [utcOffset]`

> **Примечание:** *`"YYYYMM" [utcOffset]` не поддерживается ISO 8601 (конфликт с `"YYMMDD"`)*

Сокращенные форматы с 2-х значным годом

* `"YYMMDDThhmmss.sss" [utcOffset]`
* `"YYMMDDThhmmss" [utcOffset]`
* `"YYMMDDThhmm" [utcOffset]`
* `"YYMMDDThh" [utcOffset]`
* `"YYMMDD" [utcOffset]`
* `"YY" [utcOffset]`

> **Примечание:** *`"YYMM" [utcOffset]` конфликтует с `"YYYYMM"`)*

Описание маски:

* `YYYY` - год (4-значный формат)
* `YY` - год (2-значный сокращенный формат)
* `MM` - месяц (2-значный формат, с ведущими нулями)
* `DD` - день (2-значный формат, с ведущими нулями)
* `T` - разделитель даты от времени (2-значный формат, с ведущими нулями)
* `hh` - часы (2-значный формат, с ведущими нулями)
* `mm` - минуты (2-значный формат, с ведущими нулями)
* `ss` - секунды (2-значный формат, с ведущими нулями)
* `.` - точка, разделитель секунд от миллисекунд
* `sss` - миллисекунды (3-значный формат, с ведущими нулями)
* `utcOffset` - смещение от UTC (буквенное)

По умолчанию без `utcOffset` маска определяет **локальное** время. Для использования глобального времени UTC добавляется в конец метки `Z` и время указывается **глобальное**.

Для указания смещения от UTC используются [военные часовые пояса](https://en.wikipedia.org/wiki/List_of_military_time_zones) (буквенное обозначение):

| A  | B  | C  | D  | E  | F  | G  | H  | I  | K   | L   | M   |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | --- | --- | --- |
| +1 | +2 | +3 | +4 | +5 | +6 | +7 | +8 | +9 | +10 | +11 | +12 |

Положительное смещение

| Z |
| - |
| 0 |

UTC - без смещения

| N  | O  | P  | Q  | R  | S  | T  | U  | V  | W   | X   | Y   |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | --- | --- | --- |
| -1 | -2 | -3 | -4 | -5 | -6 | -7 | -8 | -9 | -10 | -11 | -12 |

Отрицательное смещение

Буква J - локальное время (эквивалентно отсутствию буквы часового пояса `utcOffset`).

> **Примечание:** *Временные зоны (для сдвига глобального времени) в числовом формате в NVSS не поддерживаются, т.к. включают запрещенный символ `+`.*

```ebnf
timeSnapshot = YY MM DD ["T" hh [mm [ss ["." sss]]]] [utcOffset].
```

## Изменение объекта

Изменение **копий** базового объекта под контролем NVSS.

Есть 2 вида изменения объекта:

* *Модификации* - версия объекта с новым уникальным адресом. Права на модификацию передаются на этот адрес
* *Ветки* - версия объекта всегда располагается по тому-же адресу, что и база ветки (не меняется), название ветки имеет префикс `.` (точка). Перед точкой всегда неявно подразумевается адрес базового объекта. Права на модификацию не передаются, автор ветки определяется по базе.

> **Примечание:** *Если необходимо сразу создать ветку по новому адресу, то необходимо сначала создать пустую модификацию (нулевое кол-во изменений) для установки адреса.*

если объект не меняется (экспорт куска a.svg в отдельный другой a.svg) то это чистый вариант ветки
иначе можно наследовать строку версии от основного объекта и сразу создать ветку с нулевым изменением и названием напр. export -->

Версии должны изменяться преимущественно в обратном порядке по всей цепочке - передавая модификацию/ветку в базу. Это гарантирует актуальность версий вцелом и более простое сравнение разных версий для последующего объединения.

> **Внимание:** *При конфликтах адресов или подозрении на подмену адреса необходимо получить актуальную информацию от автора по этому адресу*

### Формат модификаций и веток

```ebnf
masterBase "-" {nestedBase "-"} modAddress "-" modChanges.
masterBase "-" {nestedBase "-"} "." branchName "-" branchChanges.
```

Доступно неограниченное множественное наследование.

При множественном наследовании для уменьшения длины строки версии можно использовать сокращенный вариант - скрывать любые части внутренних баз через многоточия `...`, но **всегда должна присутствовать мастер-версия и ближайшая база** для сохранения связи с базовой версией:

```ebnf
masterBase "-" {{"...-"} nestedBase "-"} modAddress "-" modChanges.
masterBase "-" {{"...-"} nestedBase "-"} "." branchName "-" branchChanges.
```

В данном случае узнать о скрытых базах можно только от ближайшей базы, где они будут раскрываться в ее строке версии.

> **Опасность:** *При скрытых базах крайние видимые базы могут выглядеть одинаковыми, но зависеть от разных промежуточных баз. Поэтому объединение мастер-версии с версией со скрытыми базами крайне не рекомендуется (возможно только при недоступности промежуточных баз, которые впоследствии необходимо устранить).*

> **Рекомендация:** *Множественное наследование модификаций/веток с большим количеством баз желательно избегать, особенно в совокупности со скрытыми базами, т.к. это замедляет и усложняет процедуру объединений, требует проверки актуальных версий всех баз. Также увеличивается длина строки версии.*

### Описание структуры модификаций и веток

```ebnf
masterBase
```

Мастер-версия объекта от которой производится изменение. **Редактирование этой части строки запрещено**.

* Содержит полную строку мастер-версии в формате описанном выше и разделителем `-` в конце.

```ebnf
{nestedBase}
```

Очередная модификация или ветка объекта от которой производится изменение. **Редактирование этой части строки запрещено**.

* Содержит полную строку базовой версии в формате описанном выше и разделителем `-` в конце.
* Количество последовательных наследованных модификаций объекта неограничено.

```ebnf
modAddress
branchName
```

**Уникальный обязательный** адрес модификации объекта `modAddress` с разделителем `-` или **уникальное обязательное** название ветки объекта `branchName` с префиксом `.` и разделителем `-`.

* Присваивается при первой модификации базовой версии.
* По адресу определяется автор модификации.
* Автором ветки является мастер-версия или крайняя базовая модификация, пропуская возможные промежуточные наследованные ветки.
* Может изменяться, при этом желательно оповещать об этом заранее зависимых авторов этой модификации/ветки для сохранения их связи.

`modAddress` и `branchName` **всегда должно включать хотя бы одну букву** для его однозначного определения в строке, рекомендуется начинать с буквы.

```ebnf
modChanges
branchChanges
```

Количество изменений модификации `modChanges` или ветки `branchChanges` - целое число.

* При каждом изменении объекта увеличивается на 1.
* Может начинаться с 0 при отсутствии изменений с целью присвоения адреса/названия заранее.
* Показывает опережение (количество изменений) относительно базовой версии `base`.
* Возможно исправление объекта (объединение объекта без увеличения количества изменений **при гарантированном осутствии использования данной версии где либо**).

## Используемый синтаксис

### Разделители

* **`.`** - разделитель в обозначении версии, префикс перед названием веток
* **`-`** - разделитель элементов строки NVSS (адресов/названий/тегов/версий), ответвление

> **Примечание:** *Стандартные символы, присутствуют во всех кодировках, разрешены в именах файлов всех актуальных операционных системах. При запрете использования данных символов допускается использование нестандартных, но при этом необходимо дополнительное пояснение к их использованию, и при первой возможности их конвертация в стандартные.*

### Разрешенные символы в префиксах, адресах и названиях

Жесткий режим (при ограниченном наборе символов)

* арабские цифры
* строчные (если доступны) буквы английского языка
* символ подчеркивания `_` для разделения слов. Между двумя числами и несколько подряд запрещено. При недоступности символа - соединять слова вместе

Для обычного использования

* цифры любого языка, поддерживаемого целевой экосистемой (не вызывающие проблем при использовании)
* строчные (если доступны) буквы любого языка, поддерживаемого целевой экосистемой (не вызывающие проблем при использовании)
* символ подчеркивания `_` для разделения слов. Между двумя числами не рекомендуется. Несколько подряд запрещено

## Описание основных действий с примерами

### Мастер-версия: создание (init)

* добавление строки мастер-версии соответствуя правилам NVSS. Обозначение версии должно соответствовать правилам семантического версирования SemVer.

| до |    | после |
| -- | -- | ----- |
|    | →  | v1.0  |

### Мастер-версия: изменение ([fetch downstream + merge] | commit)

* изменение `version` или `timeSnapshot` соответствуя правилам NVSS.

| до   |    | после |
| ---- | -- | ----- |
| v1.0 | →  | v2.0  |

### Мастер-версия: смена адреса

* меняется (`masterAddress`) на другой. Обозначение или метка времени мастер-версии не меняется.

> **Рекомендация:** *рекомендуется присвоить новый адрес с упоминанием в нем старого адреса (двойной адрес) некоторое время, для напоминания остальным авторам о произошедшей смене.*

| до (варианты) |    | после (варианты)          |
| ------------- | -- | ------------------------- |
| v1.0          | →  | creator-v1.0              |
| creator-v1.0  | →  | altername-v1.0            |
| creator-v2.0  | →  | altername-v2.0            |
| creator-v2.0  | →  | creator_to_altername-v2.0 |
| creator-v2.0  | →  | creator2altername-v2.0    |

### Мастер-версия: исправление (amend)

> **Внимание:** *доступно только если гарантированно нигде нет использования текущей мастер-версии объекта.*

* `version` или `timeSnapshot` не увеличивается, это считается как исправление текущей версии.

| до   |    | после |
| ---- | -- | ----- |
| v1.0 | →  | v1.0  |

### Модификация: создание (clone upstream + commit)

* при внесении изменений в объект - добавление `modAddress` и установка `modChanges` равным 1;
* при отсутствии изменений в объекте - добавление `modAddress` и установка `modChanges` равным 0. Используется с целью присвоения адреса модификации заранее, перед изменениями.

| до (варианты) |    | после (варианты) |
| ------------- | -- | ---------------- |
| v1.0          | →  | v1.0-mod-1       |
| v1.0          | →  | v1.0-mod-0       |

### Модификация: изменение ([fetch downstream + merge] | commit)

* увеличение `modChanges` соответствуя правилам NVSS.

| до         |    | после      |
| ---------- | -- | ---------- |
| v1.0-mod-1 | →  | v1.0-mod-2 |

### Модификация: исправление (amend)

> **Внимание:** *доступно только если гарантированно нигде нет использования данной версии модификации объекта.*

* `modChanges` не увеличивается, это считается как исправление текущей версии.

| до         |    | после      |
| ---------- | -- | ---------- |
| v1.0-mod-1 | →  | v1.0-mod-1 |

### Модификация: смена базы (pull upstream)

> **Примечание:** *текущая база может стать недоступна по каким либо причинам, неактуальна или сменить адрес - тогда новая база будет с другим адресом. Использовать ее убедившись в актуальности и совместимости с текущей модификацией.*

* объединяется новая база с текущей версией модификации;
* при отсутствии отличий от базы после объединения - `modChanges` устанавливается в 0. Используется с целью сохранения адреса модификации, перед изменениями;
* при наличии отличий от базы после объединения - `modChanges` сбрасывается до 1.

| до (варианты) |    | после (варианты) |
| ------------- | -- | ---------------- |
| v1.0-mod-1    | →  | v2.0-mod-0       |
| v1.0-mod-1    | →  | v2.0-mod-1       |

### Модификация: смена адреса

* меняется `modAddress`, при этом `modChanges` допускается не сбрасывать до 1.

| до (варианты) |    | после (варианты) |
| ------------- | -- | ---------------- |
| v1.0-mod-1    | →  | v1.0-m-1         |
| v1.0-mod-2    | →  | v1.0-m-2         |

### Ветка: создание (branch [+ commit])

* при внесении изменений в объект - добавление `branchName` и установка `branchChanges` равным 1;
* при отсутствии изменений в объекте - добавление `branchName` и установка `branchChanges` равным 0. Используется с целью присвоения имени ветки заранее, перед изменениями.

| до (варианты) |    | после (варианты) |
| ------------- | -- | ---------------- |
| v1.0          | →  | v1.0-.dev-1      |
| v1.0          | →  | v1.0-.dev-0      |

### Ветка: изменение (commit)

* увеличение `branchChanges` соответствуя правилам NVSS.

| до          |    | после       |
| ----------- | -- | ----------- |
| v1.0-.dev-1 | →  | v1.0-.dev-2 |

### Ветка: исправление (amend)

> **Внимание:** *доступно только если гарантированно нигде нет использования данной версии ветки объекта.*

* `branchChanges` не увеличивается, это считается как исправление текущей версии.

| до          |    | после       |
| ----------- | -- | ----------- |
| v1.0-.dev-1 | →  | v1.0-.dev-1 |

### Ветка: смена базы (merge + commit)

* объединяется новая база с текущей версией ветки;
* при отсутствии отличий от базы после объединения - `branchChanges` устанавливается в 0. Используется с целью сохранения адреса ветки, перед изменениями;
* при наличии отличий от базы после объединения - `branchChanges` сбрасывается до 1.

> **Примечание:** *смена базы ветки доступна только в пределах области действия адреса текущей модификации или мастер-версии.*

| до (варианты)     |    | после (варианты)  |
| ----------------- | -- | ----------------- |
| v1.0-mod-1-.dev-1 | →  | v1.0-mod-2-.dev-0 |
| v1.0-mod-1-.dev-1 | →  | v1.0-mod-2-.dev-1 |
| v1.0-.dev-1       | →  | v2.0-.dev-1       |

### Ветка: смена названия (branch -m)

* меняется `branchName`, при этом `branchChanges` допускается не сбрасывать до 1.

| до              |    | после       |
| --------------- | -- | ----------- |
| v1.0-.develop-2 | →  | v1.0-.dev-2 |

## Примеры NVSS

* `1` - мастер-версия "1"
* `v1` - мастер-версия "1" + префикс "v"
* `7.3` - мастер-версия "7.3"
* `rev7.3` - мастер-версия "7.3" + префикс "rev"
* `0.7.3` - мастер-версия "0.7.3"
* `7.3-rc1` - мастер-версия "7.3" + тег "rc1"
* `mydesktoppc-0.7.3` - мастер-версия "0.7.3" + адрес "mydesktoppc"
* `nvss-0.7.3` - мастер-версия "0.7.3" + явное указание заголовка "nvss"
* `nvss-mydesktoppc-2.3.0` - мастер-версия "2.3.0" + явное указание заголовка "nvss" и адреса "mydesktoppc"
* `nvss-1.0.0-mod.1` - мастер-версия "1.0.0" + теги "mod" и "1"
* `nvss-1.0.0-mod-1` - 1 изменение модификации "mod" относительно мастер-версии "1.0.0"
* `nvss-1.0.0-.mod-1` - 1 изменение ветки "mod" относительно мастер-версии "1.0.0"
* `nvss-1.0.2-mod-1-another-2` - 2 изменения модификации "another" относительно модификации "mod", которая, в свою очередь, была унаследована от мастер-версии "1.0.2"

На примере файла `somefile`:

* `somefile 7.3.txt` - мастер-версия "7.3"
* `somefile 7.3-rc1.txt` - мастер-версия "7.3" + тег "rc1"
* `somefile rev7.3.txt` - мастер-версия "7.3" + префикс "rev"
* `somefile nvss-rev7.3.txt` - мастер-версия "7.3" + префикс "rev" + явное указание заголовка "nvss"

Пример процесса 1:

1. Мастер-версия объекта: `nvss-v3`
2. Автор 1 получил объект и создал свою модификацию: `nvss-v3-user1-1`
3. Автор 2 получил модификацию объекта от автора 1 и тоже внес свои изменения: `nvss-v3-user1-1-user2-1`
4. Мастер-версия объекта изменилась за это время: `nvss-v5`
5. Оба автора вернули свои модификации автору мастер-версии. Теперь необходимо автору мастер-версии объединить их изменения со своей `nvss-v5` версией и выпустить `nvss-v6`. Автору мастер-версии видно, что авторами 1 и 2 редактировалась версия `nvss-v3` и что версия автора 2 включает изменения автора 1 и является более новой (т.к. наследуется от базовой версии `nvss-v3-user1-1`). Это поможет быстро разобраться во внесенных изменениях и изменить свою мастер-версию.

Пример процесса 2:

1. Получен объект без версии. Варианты действий:
   * Не использовать версирование пока оригинальный автор объекта не назначит версию.
   * Поставить собственную мастер-мерсию в виде метки времени без явного указания адреса мастер-версии.
   * Назначить полноценную мастер-версию по предварительной договоренности с оригинальным автором объекта. Он будет в дальнейшем ее использовать.
2. После установки мастер-версии для оригинального автора можно полноценно работать с объектом в соответствии с правилами NVSS.

## Ссылки и доп. информация

* [Расширенная форма Бэкуса - Наура, РБНФ](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D1%81%D1%88%D0%B8%D1%80%D0%B5%D0%BD%D0%BD%D0%B0%D1%8F_%D1%84%D0%BE%D1%80%D0%BC%D0%B0_%D0%91%D1%8D%D0%BA%D1%83%D1%81%D0%B0_%E2%80%94_%D0%9D%D0%B0%D1%83%D1%80%D0%B0)
* [Cемантическое версионирование](https://semver.org)
* [Отличие терминов версии от ревизии](https://www.product-lifecycle-management.com/plm-revision-version.htm)
* [Именование файлов для WEB](https://www.mtu.edu/umc/services/digital/writing/characters-avoid/)
* [Особенности использования semver](https://developer.okta.com/blog/2019/12/16/semantic-versioning)