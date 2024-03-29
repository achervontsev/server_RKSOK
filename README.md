# server_RKSOK

---
### Описание
В данном проекте реализован web-сервер, 
работающий на выдуманном протоколе **РКСОК** 
(Рабоче-Крестьянский Стандарт Отправки Команд, 
читается как эР Ка СОК).


РКСОК версии 1.0 состоит из четырёх команд:

- *ОТДОВАЙ* — для возврата данных.
- *УДОЛИ* — для удаления данных.
- *ЗОПИШИ* — для создания и обновления данных.
- *АМОЖНА?* — для получения разрешения обработки команды от
специальных органов.

Первые три команды предназначены для работы с клиентами, 
которые могут запрашивать получение данных, их удаление и 
сохранение, а четвертая команда предназначена для общения 
сервера РКСОК с сервером специальных органов проверки всех 
запросов. Каждый запрос, приходящий на сервер РКСОК, 
должен проверяться органами проверки, для чего каждый запрос
перед обработкой отправляется сначала на сервер проверки, и, 
если тот возвращает ответ МОЖНА, значит, сервер обрабатывает 
запрос. Если от сервера проверки пришёл любой другой ответ 
— клиенту возвращается этот ответ, а запрос дальше не обрабатывается.

---
### Примеры запросов

---

#### ОТДОВАЙ

```commandline
ОТДОВАЙ имя РКСОК/1.0
```

Команда проверяет возможность обработки запроса через 
сервер проверки и, если можно, то возвращает телефон 
человека по имени `имя`. Например, чтобы получить телефон 
Ивана Хмурого, надо отправить запрос:
```
ОТДОВАЙ Иван Хмурый РКСОК/1.0
```
Если имя можно обработать и телефон по этому имени найден, 
то сервер возвращает:

```
НОРМАЛДЫКС РКСОК/1.0
89012345678
```

Если имя можно обработать и телефон Ивана Хмурого не найден 
(ещё не сохранен или удалён), то сервер отвечает:

```commandline
НИНАШОЛ РКСОК/1.0
```
Если имя нельзя обработать и сервер проверки вернул ответ 
"Уже едем", то сервер должен вернуть ответ:
```commandline
НИЛЬЗЯ РКСОК/1.0
Уже едем
```
Обработка корявых запросов
На корявые запросы, не соответствующие стандарту РКСОК 1.0, 
сервер отвечает:
```commandline
НИПОНЯЛ РКСОК/1.0
```
#### ЗОПИШИ
```commandline
ЗОПИШИ имя РКСОК/1.0
значение
```
Сохраняет для имени имя значение значение. Например, чтобы 
сохранить телефон Ивана Хмурого 89012345678, нужно 
отправить запрос:
```commandline
ЗОПИШИ Иван Хмурый РКСОК/1.0
89012345678
```
Сервер проверяет запрос на сервере проверки, и если всё ок, 
то записывает у себя в файл с названием Иван Хмурый 
(или каким-то другим названием) значение телефона, и 
отвечает клиенту:
```commandline
НОРМАЛДЫКС РКСОК/1.0
```
Если запрос нельзя обработать (сервер проверки не разрешил), 
ответ аналогичен запросу *ОТДОВАЙ* для такого же случая.

#### УДОЛИ
```commandline
УДОЛИ имя РКСОК/1.0
```
Удаляет запись для имени `имя`. Например, чтобы удалить 
телефон Ивана Хмурого, нужно отправить запрос:
```commandline
УДОЛИ Иван Хмурый РКСОК/1.0
```
Сервер проверяет запрос на сервере проверки, и если все ок, 
то удаляет у себя файл с названием "Иван Хмурый" и 
возвращает:
```commandline
НОРМАЛДЫКС РКСОК/1.0
```
Если запрос нельзя обработать (сервер проверки не разрешил), 
ответ аналогичен запросу *ОТДОВАЙ* для такого же случая.

Если запрос можно обработать, но такого файла с таким 
именем нет на сервере, сервер отвечает:
```commandline
НИНАШОЛ РКСОК/1.0
```
#### АМОЖНА?
```
АМОЖНА? РКСОК/1.0
проверяемый запрос
```
Программа отправлят запрос на сервер проверки по адресу:

vragi-vezde.to.digital, порт — 51624

Допустим, к нам пришел запрос на заведение телефона человека
Иван Хмурый:
```
ЗОПИШИ Иван Хмурый РКСОК/1.0
89012345678
```
Тогда мы отправляем на сервер проверки такой запрос:
```
АМОЖНА? РКСОК/1.0
ЗОПИШИ Иван Хмурый РКСОК/1.0
89012345678
```
Ответ, разрешающий обработку запроса:
```commandline
МОЖНА РКСОК/1.0
```
Ответ от сервера проверки, запрещающий обработку запроса 
(в таком же виде он должен вернуться клиенту):
```
НИЛЬЗЯ РКСОК/1.0
Что ещё за Иван Хмурый такой? Он тебе зачем?
```

---

### Прочие требования

---
Окончанием клиентского запроса считается пустая строка 
(символы *\r\n*). Запрос может состоять из нескольких 
строк, например:
```
ЗОПИШИ Иван Хмурый РКСОК/1.0
89012345678 — мобильный
02 — рабочий
```
В «сыром» виде с переносами строк этот запрос выглядит 
так:

*ЗОПИШИ Иван Хмурый РКСОК/1.0\r\n89012345678 — 
мобильный\r\n02 — рабочий\r\n\r\n*

Максимальная длина имени во всех сценариях ограничена 30 
символами, в противном случае возвращается ответ НИПОНЯЛ. 
Общая длина запроса в коде не должна быть ограничена, 
сервер должен считывать запрос полностью, прежде чем 
начинать его обрабатывать, проверять его корректность 
и тд.

---

### Запуск программы в Linux

---
Для запуска программы в системе должен быть установлен 
интерпретатор Python. Перед запуском в файле *server.py* в 
переменной *SERVER_ADDRESS* (строка 12) необходимо указать 
ip-адрес и номер порта машины, на которой запускается 
программа, например:
```python
SERVER_ADDRESS = ('192.168.1.1', 8000)
```
Сервер запускается из папки проекта командой:
```
$ python3.10 server.py
```
где `python3.10` - команда для запуска Python в системе.