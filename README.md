## Техническое задание:

Требуется написать персонализированный сервис task manager, позволяющий пользователю ставить себе задачи, отражать в системе изменение их статуса и просматривать историю задач.

Сервис должен предоставлять интерфейс в виде JSON API, это единственный способ общения клиента с сервисом
Авторизация в апи происходит с помощью токена (переданного в заголовке Authorization)
сервис будет покрыт тестами
содержать Dockerfile для сборки сервиса


### Функциональные требования:

   - Пользователь может зарегистрироваться в сервисе задав пару логин-пароль
   - В системе может существовать много пользователей
   - Пользователь может авторизоваться в сервисе предоставив пару логин-пароль и получив в ответе токен
   - Пользователь видит только свои задачи
   - Пользователь может создать себе задачу. Задача должна как минимум содержать следующие данные:

(*) - обязательные поля

   - *Название
   - *Описание
   - *Время создания
   - *Статус - один из Новая, Запланированная, в Работе, Завершённая
   - Планируемое дата завершения

   - Пользователь может менять статус задачи на любой из данного набора
   - Пользователь может менять планируемое время завершения, название и описание
   - Пользователь может получить список задач своих задач, с возможностью фильтрации по статусу и планируемому времени завершения
   - Пользователь может просмотреть историю изменений задачи (названия, описания, статуса, времени завершения)



### Реализация.

Задание реализовано на Python 3 с использованием фреймворка Django/DjangoRestFramework. В качестве СУБД используется PostgreSQL. Сервис покрыт тестами посредством Unittest. Для сборки сервиса использвется Docker/Docker-Compose


Ниже приведен список эндпоинтов, используемых в сервисе, и их описание

#### * /auth/users/ - POST
регистрация пользователя

пример запроса 
```bash
curl -X POST http://127.0.0.1:8000/auth/users/ --data 'username=djoser&password=alpine12'
```

пример ответа
```bash
{"email": "", "username": "djoser", "id":1}
```

#### * /auth/token/login/ - POST

представление токена зарегистрированному пользователю по логину и паролю.


пример запроса 
```bash
curl -X POST http://127.0.0.1:8000/auth/token/login/ --data 'username=djoser&password=alpine12'
```

пример ответа
```bash
{"auth_token": "b704c9fc3655635646356ac2950269f352ea1139"}
```

#### * api/tasks/ - GET 
список задач  пользователя. обязательным условием является передача токена аутентификации в заголовке запросе

пример запроса:

```bash
curl -LX GET http://127.0.0.1:8000/api/tasks/ -H 'Authorization: Token b704c9fc3655635646356ac2950269f352ea1139'
```

пример ответа:

```bash
[
{
"id": 2,
"name": "run",
"description": "park",
"created": "2020-10-06T14:37:55.763202Z",
"status": "planned",
"planned_finish": "2020-10-07"
},
{
"id": 1,
"name": "swim",
"description": "pool",
"created": "2020-10-06T14:37:36.321868Z",
"status": "new",
"planned_finish": "2020-10-08"
}
]
```

#### * api/tasks/ - POST 
добавление задачи пользователя. обязательным условием является передача токена аутентификации в заголовке запроса

пример запроса:

```bash
curl -X POST http://127.0.0.1:8000/api/tasks/ -H 'Authorization: Token b704c9fc3655635646356ac2950269f352ea1139' —-data ‘name=read&descritpion=Mark Lutz&status=new&planned_finish=2020-10-30’
```

пример ответа:
```bash
{
"id": 3,
"name": "read",
"description": "Mark Lutz",
"created": "2020-10-06T14:43:17.268036Z",
"status": "new",
"planned_finish": "2020-10-30"
}
```

#### * api/tasks/<id_задачи> - PUT
изменения параметров задачи. обязательным условием является передача токена аутентификации в заголовке запроса.

пример запроса:

```bash
curl -X PUT http://127.0.0.1:8000/api/tasks/3/ -H 'Authorization: Token b704c9fc3655635646356ac2950269f352ea1139' —-data ‘name=read&descritpion=Mark Lutz.Python 3&status=new&planned_finish=2020-10-29’
```

пример ответа:

```bash
{
"id": 3,
"name": "read",
"description": "Mark Lutz. Python 3",
"created": "2020-10-06T14:46:24.307180Z",
"status": "new",
"planned_finish": "2020-10-29"
}
```

#### * /api/filtered_tasks/<статус_задачи>/<дата_завершения_задачи>/ - GET
список задач  пользователя отфильтрованный по дате и статусу. обязательным условием является передача токена аутентификации в заголовке запросе


пример запроса:

```bash
curl -LX GET http://127.0.0.1:8000/api/filtered_tasks/new/2020-10-08/ -H 'Authorization: Token b704c9fc3655635646356ac2950269f352ea1139'
```

пример ответа:

```bash
[
{
"id": 2,
"name": "run",
"description": "park",
"created": "2020-10-06T14:51:34.115980Z",
"status": "new",
"planned_finish": "2020-10-08"
},
{
"id": 1,
"name": "swim",
"description": "pool",
"created": "2020-10-06T14:37:36.321868Z",
"status": "new",
"planned_finish": "2020-10-08"
}
]
```


#### */api/task_changes/<id_задачи>/ - GET
список изменений задачи пользователя. обязательным условием является передача токена аутентификации в заголовке запросе

пример запроса:

```bash
curl -LX GET http://127.0.0.1:8000/api/task_changes/4/ -H 'Authorization: Token b704c9fc3655635646356ac2950269f352ea1139'
```

пример ответа:

```bash
[
{
"changed_task": 4,
"change_created": "2020-10-06T14:46:24.293426Z",
"task_name": "read",
"task_description": "Mark Lutz. Python 3",
"task_status": "new",
"task_planned_finish": "2020-10-29",
"changed_fields":[
"description",
"planned_finish"
]
},
{
"changed_task": 4,
"change_created": "2020-10-06T14:59:18.653582Z",
"task_name": "read",
"task_description": "Mark Lutz. Python 3",
"task_status": "planned",
"task_planned_finish": "2020-10-29",
"changed_fields":[
"status"
]
}
]
```

### Запуск тестов
```bash
python manage.py test
```




### Пояснения по реализации функциональных требований.

 ##### - Пользователь может зарегистрироваться в сервисе задав пару логин-пароль
 ##### - В системе может существовать много пользователей
 ##### - Пользователь может авторизоваться в сервисе предоставив пару логин-пароль и получив в ответе токен
 
Данные задачи тривиальны и реализуются использованием библиотеки djoser


   
##### - Пользователь видит список своих задач.
##### - Пользователь может создать себе задачу. 
##### - Пользователь может менять статус, планируемое время завершения, название и описание.

Для реализации данного списка задач используется класс TasksViewSet, производный от класса ModelViewSet. Для того, чтобы пользователь мог видеть только свои задачи - переопределяем метод get_queryset, который будет возвращать только задачи принадлежащие пользователю.
Для создания задачи используется метод post класса ModelViewSet.
Для изменения статуса, планируемого времени завершения, названия и описания используется метод put класса ModelViewSet.


 ##### - Пользователь может получить список задач своих задач, с возможностью фильтрации по статусу и планируемому времени завершения

Для реализации данной задачи используем класс FilteredTasksView, производный от класса APIView. В нем переопределяем метод get. В GET запрос передаем дополнительные параматры : status(статус задачи) и planned_finish(Планируемое дата завершения). Метод возвращает список задач для данного пользователя отфильтрованный по статусу и времени завершения. Если задач с заданными параметрами не найдено - будет возвращен пустой список. Для валидации параметров  status и planned_finish используется сериализатор Filtered_TasksSerializer.


##### - просмотреть историю изменений задачи (названия, описания, статуса, времени завершения).

Для реализации данного функционала каждое изменение параметра задачи вносится в отдельную таблицу (TaskChanges). Для этого переопределяем метод update сериализатора TasksSerializer. При изменении параметров задачи в методе определяем какие именно параметры были изменены. В поле changed_task модели TaskChanges они будут отображены в виде списка. Также сохраняем в таблицу TaskChanges следующие параметры измененной задачи: id задачи, название, описание, статус и дату.

Для отображения списка с историей изменений задачи используется класс TaskChangesView, производный от класса APIView. В данном классе переопределяем метод get. Пользователь запрашивает список изменений задачи по заданному id. При помощи метода get_object_or_404 определяем принадлежит ли задача с заданным id пользователю, который ее запрашивает. Если нет - возвращаем ответ со статус кодом 404. Если принадлежит - возвращаем список изменений задачи с заданным id, отсортированный по дате и времени создания изменения.  

