---
translatedFrom: en
translatedWarning: Если вы хотите отредактировать этот документ, удалите поле «translationFrom», в противном случае этот документ будет снова автоматически переведен
editLink: https://github.com/ioBroker/ioBroker.docs/edit/master/docs/ru/adapterref/iobroker.systeminfo/README.md
title: без названия
hash: foC/wzSvgAwpM+gnbhyD3KIRK/89kpPq4SNlySZNoDY=
---
![логотип](../../../en/adapterref/iobroker.systeminfo/./admin/systeminfo.png) Считывает (и записывает) информацию из Систем (ы) ---

![Версия NPM](http://img.shields.io/npm/v/iobroker.systeminfo.svg)
![Загрузки](https://img.shields.io/npm/dm/iobroker.systeminfo.svg)
![Travis-CI Статус сборки](https://travis-ci.org/frankjoke/ioBroker.systeminfo.svg?branch=master)
![AppVeyor Статус сборки](https://ci.appveyor.com/api/projects/status/pil6266rrtw6l5c0?svg=true)
![NPM](https://nodei.co/npm/iobroker.systeminfo.png?downloads=true)

## Адаптер обрабатывает (системную) информацию из собственных или других систем и веб-источников.
Он генерирует состояния из информации, которую находит разными способами.

* Команды, выполняемые в операционной системе
* Файлы для чтения в локальных или подключенных системах
* Результаты веб-страниц или API
* Команды инструментов Nodejs

* Команды и файлы работают также в обоих направлениях, что означает, что вы также можете записывать информацию в систему.
* Это позволяет получить доступ и записать выводы GPIO на Raspi или OrangePi, а также управлять зелеными или красными светодиодами на Raspi / Opi
* Это позволяет получить / установить некоторую системную информацию, доступ к которой осуществляется через / sys в Lunux
* Используется часть 'systeminformation', которая работает в Windows и Linux для

Он обрабатывает текстовые, HTML, JSON и XML типы данных с помощью специальных механизмов запросов.

### Заметка
* Я хотел бы выразить свою благодарность некоторым модулям в сети, которые я использовал или реализовал со своим собственным кодом. Адаптер использует некоторые внешние модули, такие как [cheerio] (https://github.com/cheeriojs/cheerio), [systeminformation] (https://github.com/sebhildebrandt/systeminformation) и [node-schedule] (https: / /github.com/node-schedule/node-schedule) как они есть. Он также был вдохновлен кодом из [JSONPath] (http://goessner.net/articles/JsonPath/index.html#e2) и [scrape-it] (https://github.com/IonicaBizau/scrape-it) но их код не использовался напрямую, а был переопределён для разных нужд.

## Конфигурация
* Настройка в конфигурации адаптера (увеличить страницу)
* Я сохранил изображение примера конфигурации [здесь] (./ admin / Systeminfo.Config.jpg)
    * Первый элемент - это список команд, который будет выполняться (строка за строкой) при запуске адаптера. Может использоваться для настройки используемых портов GPIO.
    * Строки, начинающиеся с `` # `', не выполняются
    * Если первый текст «debug!», Он переводит адаптер в режим отладки, который отображает гораздо больше информации о том, что он пытается получить и получить.
* После подтверждения запуска появляется список конфигурации для каждого источника данных, состоящий из
    * Имя поля, которое также может включать в себя
        * Если имя начинается с `` -` ', строка будет игнорироваться (отключаться), так же, как если нет расписания
        * `[*]`, `[имя, ...]`, `[имя / (значение)]` синтаксис
        * без какого-либо из вышеперечисленных имя используется для создания состояния, как оно есть.
        * Если `[]` используется где-то, здесь вставляются имена разными методами
            * `[*]` если возвращено несколько элементов, они вставляются как числа. Пример: `Meldung []` будет генерировать `Meldung0`-`Meldung (n)`, если (n) элементы возвращены
            * `[name1, name2, ...]` создает именно эти имена (например, `System.Memory_ [используется, свободно, доступно]) создаст три состояния с именем` System.Memory_used, System.Memory_free, System.Memory_available`)
            * `[имя / значение]` берет имя из свойства объекта `имя` (может отличаться) и значение из свойства` значение`. Можно использовать любое имя свойства или значения.
            * `[name /]` без значения будет брать имя из `name` и создавать подсостояния для всех других свойств, найденных в этом объекте (пример` System.Network. [iface /] `)

    * `Тип` и` источник` источника информации, который может быть
        * `file`: поле` source` описывает имя файла, которое читается
        * `exec`: поле` source` описывает однострочную команду, которая выполняется
        * `info`: поле` source` описывает однострочную командную функцию `systeminfo`
        * `web`: поле` source` описывает веб-URL, который читается (или объект, описывающий доступ, это нужно будет документировать позже!).
        * Запросы кэшируются, если одновременно запрашиваются несколько записей с одинаковым типом / исходным содержимым! Это означает, что если вы планируете каждую минуту выполнять команду и извлекать два разных элемента данных из одной и той же команды, она запускается только один раз и только фильтр данных применяется несколько раз.
        * Это помогает не загружать несколько раз одну и ту же страницу, если вы хотите получить больше элементов.

    * `Regexp / filter`` используется для описания того, как фильтровать полученный текст либо с
        * Оператор `Regexp`, где отдельные элементы должны быть отсортированы с помощью` () `.

        Пример: `/lic\s+(\d+)K\s+(\d+)K\s+(\d+)/m` будет искать текст `lic`, за которым следуют пробелы, а затем цифры, заканчивающиеся на `K` во всех строках, он возвращает 3 числа. Это используется в команде `df -BK` Linux, чтобы показать мне размер смонтированного общего ресурса NFS, оканчивающегося на «lic» в имени.

        * `JsonPath` утверждение. Я создал специальную версию JsonPath для выбора данных из Json или любых объектов javascript.
            * Синтаксис состоит из ряда селекторов, которые могут быть
            * `name` имя свойства
            * `*` любой элемент в этом объекте, это могут быть все свойства или, если объект является массивом, это все элементы массива
            * `[(...)]` оцените `...`, чтобы получить имя свойства, которое будет выбрано. `@` будет использоваться в качестве заполнителя для текущего объекта и может использоваться в операторе eval.
            * `[? (...)]` Фильтровать элементы этого элемента по ...,

            Пример: `list[?(@.user == 'pi')]` сначала выберет свойство `list` (которое является массивом) и файлтер, а затем список, выбрав только те элементы списка, для которых `.user`set в `pi`.

            * `[! (...)]` Возвращает оцененное значение как новый элемент. Таким образом, вы можете рассчитать свои собственные данные из найденных объектов.
            * `[name1, name2, name3]` будет выбирать только эти имена свойств
            * `[0]` будет выбирать только первый '(или n') элемент или свойство
            * `[start: end: step]` будет принимать элементы, начинающиеся с `start` и` <end`, используя `step`. Все должны быть числами или оставлены пустыми. `start` и` end` могут быть отрицательными, что означало бы, что они заканчиваются с конца. Пример: `[1: -1: 2]` будет брать каждый второй элемент со второго, но не включая последний. Последний будет `[-1 ::]`, первые 3 будут `[: 3:]`, а последние 3 будут `[-3 ::]`
            * `..` - это рекурсивный селектор спуска, который означает, что` ..name` выберет имя свойства в «любом отделе» объекта!
    * `html запрос WebObject` В случае анализа html я создал специальный инструмент запросов для выбора элементов в веб-файлах, похожих на jQuery. Этот инструмент создает объект, который окончательно анализируется с помощью `JsonPath`. **Документация следовать**

    * Запись `convert` может быть
        * `json` для анализа данных json, в веб-записях это означает, что полученный текст будет обрабатываться как json напрямую, а регулярное выражение / фильтр будет оператором / фильтром JsonParse.
        * `xml` для XML-данных, это означает, что полученные данные будут преобразованы из XML в json и обработаны, как указано выше.
        * `html` генерирует объект` cheerio`, который затем ищется по специальному запросу WebObject
    * `number` или` boolean` будут пытаться преобразовать значение в числа или логическое значение, если для логических чисел> 0 задано значение true, но также строки типа **on** или **ein** и **true** оцениваются как правда.
    * `...` все остальное, как `! parseInt (@)`, будет оценено и в этом случае вернет **true** если значение равно `0`, или **false** если значение больше целого числа.

    Поле `role / type` описывает поле ioBroker ty и может называть также юнит. Нормальным типом поля является текст или значение, отображаемое в convert.
        * `json` означает, что свойство поля взято из объекта
        * `число | МБ` будет определять числовое поле с единицей измерения МБ (мегабайт)

    Поле `Write Command` описывает утверждения или уловки, которые будут использоваться для обратной записи в предмет. В настоящий момент он работает только для типов «exec» или «file».
        * Для `exec` это командная строка, которая может содержать операторы` @ (...) `, которые будут оцениваться. Пример: `gpio write 1 @ (@? '0': '1')` будет переводиться в `gpio write 1 0`, если состояние истинно, и в` gpio write 1 1`, если оно ложно Это контролирует мои ИК-светодиоды, которые загораются, если на выводе GPIO установлен низкий уровень (0).
        * Для `file` это простое выражение eval, которое выполняется и записывается в файл. Пример: `@? «1»: «0» будет писать «1», если значение истинно, и «0», если оно ложно.

    * Последний - «график». Если он пуст, то итера не выполняется вообще! Все расписания, которые имеют одинаковое значение, будут выполняться вместе с одним и тем же кэшем.
        * `cron-синтаксис`, вы можете использовать тот же 'cron'-синтаксис, что и oBroker использует графики inJavascript, который описан в [node-schedule] (https://github.com/node-schedule/node-schedule)
        * `time-syntax` Я создал специальный синтаксис времени`?:? (:?) `, который облегчает
            * `*: 16` будет запрашивать эти данные на 15-й минуте каждого часа
            * `* / 2: 1: 1` будет запрашивать каждый второй час в 1-ю минуту и 1-ю секунду.
            * `? s`,`? m`, `? h` с? цифры> 0 будут запускать запрос каждые? секунды, часы или часы, вы не можете указать несколько элементов одновременно!
        * Расписания сгруппированы в одно и то же время, если вы опустите секунды, как в первом примере выше, они будут назначены какому-либо номеру, пытающемуся избежать одинаковую секунду для всех элементов. Это сделано для того, чтобы не запускать слишком много команд одновременно.

## Известные вопросы
* Бета-тестирование, нет записи на веб-страницах реализовано

## Важно / Wichtig
* Требуется узел> = v4.5

## Монтаж
Mit ioBroker admin, npm установить iobroker.systeminfo или <<https://github.com/frankjoke/ioBroker.systeminfo>

## Changelog
### 0.3.0
* Added save and load config in admin screen

### 0.2.2
* First public beta includes jsonParse and WebQuery parse, jsonParse syntax mistake corrected for selectors
* New icon to separate it from info-Adapter

### 0.2.0
* First public beta includes jsonParse and WebQuery parse

### Todo for later revisions
* Allow import/export of configs to easily add new functions
* Allow access of web pages with authentication and also writing/postng web content

## License

The MIT License (MIT)

Copyright (c) 2017, frankjoke <frankjoke@hotmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.