# api_yatube

Репозиторий для "CRUD для Yatube"

## Предварительная настройка

Вам стал доступен репозиторий `api_yatube`, клонируйте его в свою рабочую директорию, например, в *Dev*.

В репозитории — знакомый вам **Yatube**, но не самой последней версии и не весь, а только его бэкенд: приложения, модели. Мы исключили из проекта фронтенд и view-функции: они сейчас не понадобятся.

Создайте и активируйте виртуальное окружение, установите все необходимые пакеты из *requirements.txt*.

### Задача

В проекте `api_yatube` есть приложение **posts** с описанием моделей **Yatube**. Вам нужно реализовать API для всех моделей приложения.

Обычно всю логику API выносят в отдельное приложение: при иной организации кода работать в большом проекте со множеством приложений будет неудобно. 

Добавьте в проект новое приложение с именем "api" и реализуйте всю логику именно там.

API должен быть доступен только аутентифицированным пользователям. Используйте в проекте аутентификацию по токену **TokenAuthentication**.

Аутентифицированный пользователь авторизован на изменение и удаление своего контента; в остальных случаях доступ предоставляется только для чтения. При попытке изменить чужие данные должен возвращаться код ответа **403 Forbidden**.

Для взаимодействия с ресурсами опишите и настройте такие эндпоинты:

* `api/v1/api-token-auth/` (POST): передаём логин и пароль, получаем токен.
* `api/v1/posts/` (GET, POST): получаем список всех постов или создаём новый пост.
* `api/v1/posts/{post_id}/` (GET, PUT, PATCH, DELETE): получаем, редактируем или удаляем пост с идентификатором `{post_id}`.
* `api/v1/groups/` (GET): получаем список всех групп.
* `api/v1/groups/{group_id}/` (GET): получаем информацию о группе с идентификатором `{group_id}`.
* `api/v1/posts/{post_id}/comments/`\
(GET): получаем список всех комментариев поста с идентификатором `post_id`\
(POST): создаём новый комментарий для поста с идентификатором `{post_id}`.
* `api/v1/posts/{post_id}/comments/{comment_id}/` (GET, PUT, PATCH, DELETE): получаем, редактируем или удаляем комментарий с идентификатором `{comment_id}` в посте с  `id=post_id`.

В ответ на запросы POST, PUT и PATCH ваш API должен возвращать объект, который был добавлен или изменён.

Обязательное условие: работайте с моделью **Post** через `ModelViewSet`. 

Если вы решите, что вьюсеты подойдут и для работы с остальными моделями — не стесняйтесь, применяйте их везде, где можно.

### Примеры запросов

Пример POST-запроса с токеном Антона Чехова: добавление нового поста.

POST *.../api/v1/posts/*

```
{
    "text": "Вечером собрались в редакции «Русской мысли», чтобы поговорить о народном театре. Проект Шехтеля всем нравится.",
    "group": 1
}
```

Пример ответа:

```
{
    "id": 14,
    "text": "Вечером собрались в редакции «Русской мысли», чтобы поговорить о народном театре. Проект Шехтеля всем нравится.",
    "author": "anton",
    "image": null,
    "group": 1,
    "pub_date": "2021-06-01T08:47:11.084589Z"
} 
```

Пример POST-запроса с токеном Антона Чехова: отправляем новый комментарий к посту с `id=14`.

POST *.../api/v1/posts/14/comments/*

```{
    "text": "тест тест"
} 
```

Пример ответа:

```
{
    "id": 4,
    "author": "anton",
    "post": 14,
    "text": "тест тест",
    "created": "2021-06-01T10:14:51.388932Z"
} 
```

Пример GET-запроса с токеном Антона Чехова: получаем информацию о группе.

GET *.../api/v1/groups/2/*

Пример ответа:

```
{
    "id": 2,
    "title": "Математика",
    "slug": "math",
    "description": "Посты на тему математики"
} 
```

### Подсказки

Начните с описания файла *urls.py* на уровне проекта. Он должен включать (*include*) маршруты из приложения *api*. 

Если решите везде использовать вьюсеты, то вам понадобится:

использование регулярных выражений для регистрации некоторых роутеров — это отличный повод повторить урок и заглянуть в шпаргалку;

разобраться с использованием метода `get_queryset()` вьюсета при передаче дополнительных переменных в запросе;

при запросе на изменение или удаление данных осуществлять проверку прав. Эту задачу можно реализовать, переопределив методы `perform_update` и `perform_destroy`, например, вот так:

```
def perform_update(self, serializer):
  if serializer.instance.author != self.request.user:
      raise PermissionDenied('Изменение чужого контента запрещено!')
  super(PostViewSet, self).perform_update(serializer) 
```

Но можно использовать и иные способы.

## Запросы через Postman

Проверять и отлаживать работу API удобно через Postman. Эта программа умеет отправлять запросы, анализировать ответы и сохранять запросы для повторного применения в будущем. 

В Postman можно импортировать готовую библиотеку запросов. Тогда при тестировании не придётся писать каждый запрос вручную, а можно будет выбрать нужный запрос из списка и отправить его в один клик.

Можно отправить сразу все запросы из коллекции: они будут отправлены последовательно и в нужной очерёдности, например — сперва запрос на получение токена, потом — на публикацию поста, и токен, полученный в первом запросе, будет применён во втором.

Можно отправлять запросы по отдельности, но при этом стоит помнить, что для некоторых запросов могут потребоваться данные, предварительно полученные из других запросов.

В директории *postman_collection* сохранена коллекция запросов для отладки и проверки работы текущей версии API Yatube.

Когда проект будет готов обрабатывать запросы к API — импортируйте коллекцию в Postman и выполняйте запросы. Пока проект не научится обрабатывать запросы — от коллекции не будет практической пользы.

Коллекция запросов — это вспомогательный инструмент для проверки API, применяйте его по собственному усмотрению или не применяйте вовсе, решение — за вами.

#### Успешное выполнение всех запросов из Postman-коллекции не означает, что задание выполнено: решающее слово остаётся за тестами.

### Как работать с коллекцией?

Для каждого запроса в коллекции заготовлен пример ожидаемого ответа. Если вернувшийся ответ не будет соответствовать ожидаемому — Postman покажет сообщение об ошибке. Сообщения могут быть по-русски и по-английски, читайте их внимательно, они помогут обнаружить ошибку. Держите словарь под рукой, некоторые английские термины могут быть вам незнакомы.

Если вдруг вы обнаружите, что проверка API через коллекцию запросов в Postman в чём-то противоречит тестам на платформе — приоритет всегда за тестами, ориентируйтесь именно на них.

Прежде чем начать работу с коллекцией — подготовьте Django-проект:

1. Проверьте, что виртуальное окружение развёрнуто и активировано, зависимости проекта установлены.
2. Перейдите в директорию *postman_collection* и командой `bash set_up_data.sh` запустите bash-скрипт для создания необходимых для работы коллекции объектов в базе данных.\
**Внимание, скрипт предварительно очищает существующую базу данных.**
3. Перейдите в директорию с файлом *manage.py* и запустите веб-сервер разработки.

Особенности запуска и настройки коллекции описаны в файле README.md. Чтобы начать работу с коллекцией запросов — выполните следующие шаги:

1. **Импорт коллекции:**\
Откройте Postman и нажмите на кнопку "Import" в верхнем левом углу.\
Во всплывающем окне будет предложено перетащить файл с коллекцией или выбрать файл через окно файлового менеджера. Выберите файл коллекции *CRUD_for_yatube.postman_collection.json.*
2. **Просмотр коллекции:**\
После импорта коллекция появится в левой панели. Нажмите на её название, чтобы увидеть все запросы.  
3. **Настройка и проверка запросов:**
  * Выберите запрос в коллекции, нажав на него.
  * В верхней части окна вы увидите тип HTTP-запроса и URL-адрес API. При необходимости эти данные можно изменить, например — добавить параметры запроса, заголовки или тело запроса в соответствующих вкладках под строкой URL.
4. **Сохранение изменений в запросах:**\
Если вы внесли изменения в запрос — сохраните его, нажав на кнопку "Save" в правом верхнем углу.
5. **Запуск всех запросов в коллекции:**\
Запросы можно отправлять по одному или все сразу. Для последовательного запуска всех запросов используйте функцию **Runner**. Для этого нажмите на три точки напротив названия коллекции и в выпадающем списке выберите *Run collection*.\
В центре экрана появится список запросов из коллекции, а в правой части экрана — меню для настройки параметров запуска.\
В правом меню включите опцию **Persist responses for a session**, это даст возможность посмотреть ответы API.\
Для запуска коллекции нажмите кнопку `Run [имя вашей коллекции]`.
6. **Экспорт коллекции:**\
Если вы внесли изменения в коллекцию и хотите сохранить её для дальнейшей работы или для обмена с коллегами, вы можете её экспортировать. Для этого в выпадающем меню коллекции выберите **Export**.
