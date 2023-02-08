# Api YamDb
## Описание
Данный проект является интерфейсом API для проекта YamDb.

В проекте хранятся отзывы на произведения , а также комментарии к отзывам.
Все произведения делятся на категории , также каждому произведению может быть присвоен один или несколько жанров.
У каждого произведения присутствует рейтинг, который строится по средней оценке всех отзывов на данное произведение.

В проекте реализованы следующие объекты:
```
Произведение - Title
Категория - Category
Жанр - Genre
Отзыв - Review
Комментарий - Comment
Пользователь - User
```

Неаутентифицированные пользователи имеют разрешение только на чтение.

Аутентифицированные пользователи деляться на Администратора, Модератора и Пользователя.
Администратор может добавлять, изменять и удалять: произведения, категории, жанры, отзывы, комментарии.
Модератор может добавлять, изменять и удалять: отзывы и комментарии.
Пользователь может добавлять отзывы и комментарии,
а также изменять и удалять отзывы и комментарии если является их автором.


## Технологии:
Проект создан средствами Python, DgangoORM и Dgango REST Framework.
Аутентификация в проекте построена на simple_jwt.
Подключены Gunicorn и Nginx.
Проект запакован в контейнер средствами Docker.

Версия Python:

```
Python 3.7
```

Зависимости:

```
asgiref==3.2.10
requests==2.26.0
django==2.2.16
django-filter==2.2.0
djangorestframework==3.12.4
djangorestframework-simplejwt==5.1.0
djoser
gunicorn==20.0.4
psycopg2-binary==2.8.6
PyJWT==2.1.0
pytest==6.2.4
pytest-django==4.4.0
pytest-pythonpath==0.7.3
pytz==2020.1
sqlparse==0.3.1
```

## Как запустить проект:

Клонировать репозиторий:

```
git clone https://github.com/Algor45/api_yamdb.git
```

Перейти в папку infra:

```
cd infra/
```

Создать .env файл и заполнить его по шаблону:

```
DB_ENGINE=django.db.backends.postgresql
DB_NAME=<название базы данных>
POSTGRES_USER=<имя пользователя базы данных>
POSTGRES_PASSWORD=<пароль базы данных>
DB_HOST=db
DB_PORT=<порт>(по умолчанию = 5432)
```

В этой же папке выполнить команду развертывания проекта:

```
docker-compose up
```

Выполнить миграции:
```
docker-compose exec web python manage.py migrate
```

Создать суперпользователя:
```
docker-compose exec web python manage.py createsuperuser
```

Подключить статику:
```
docker-compose exec web python manage.py collectstatic --no-input
```


Проект станет доступен по адресу:

```
http://localhost/
```

Для остановки проекта выполните команду:

```
docker-compose down
```
Или воспользуйтесь комбинацией клавиш Ctrl+C в терминале с запущенным докером.


###### (Опционально) Заполнение БД.
Проект поддерживает заполнение базы данных из csv файлов.

Для этого необходимо поместить csv файлы в папку static/data/ и приписать их к моделям
в файле reviews/management/commands/csv_to_db.py в словаре MODELS_FILES.

Чтобы залить данные в базу необходимо выполнить комманду:

```
docker-compose exec web python manage.py csv_to_db
```

## Примеры
### Создание нового пользователя.

Запрос
```
POST http://127.0.0.1:8000/api/v1/auth/signup/
Content-Type: application/json

{
    "username": "example_user",
    "email": "example@mail.com"
}
```

Вернет
```
{
    "username": "example_user",
    "email": "example@mail.com"
}
```
Создаст нового пользователя в базеи отправит на указанный email
код подтверждения необходимый для получения токена.

### Получение токена.

Запрос
```
POST http://127.0.0.1:8000/api/v1/auth/token/
Content-Type: application/json

{
    "username": "example_user",
    "confirmation_code": "example_code"
}
```

Вернет токен в поле, который необходимо указывать в header Authorization в формате:
Authorization: Bearer <token>

```
{
    "token": "example_token"
}
```

### Просмотр произведения.

Запрос
```
GET http://127.0.0.1:8000/api/v1/titles/(title_id)/
Content-Type: application/json
Authorization: Bearer <token>

```

Вернет

```
{
    "id": 1,
    "name": "example_name",
    "year": example_date,
    "rating": example_rating,
    "description": example_description,
    "genre": [
        {
            "name": "example_genre_name",
            "slug": "example_genre_slug"
        }
    ],
    "category": {
        "name": "example_category_name",
        "slug": "example_category_slug"
    }
}
```

### Просмотр списка произведений.

Запрос
```
GET http://127.0.0.1:8000/api/v1/titles/
Content-Type: application/json
Authorization: Bearer <token>

```

Вернет список произведений в формате.

```
"count": example_count,
    "next": "http://127.0.0.1:8000/api/v1/titles/?page=2",
    "previous": null,
    "results": [
        {
            "id": example_id,
            "name": "example_name",
            "year": example_year,
            "rating": example_rating,
            "description": null,
            "genre": [
                {
                    "name": "example_genre_name",
                    "slug": "example_genre_slug"
                }
            ],
            "category": {
                "name": "example_category_name",
                "slug": "example_category_slug"
            }
        },
```
### Создание отзыва.

Запрос
```
POST http://127.0.0.1:8000/api/v1/titles/(title_id)/reviews/
Content-Type: application/json
Authorization: Bearer <token>

{
    "text": "example_text",
    "score": 10
}
```

Вернет
```
{
    "title": "example_title",
    "author": "example_author",
    "text": "example_text",
    "score": 10,
    "pub_date": "example_date",
    "id": example_id
}
```
и создаст новый отзыв в бд.

### Создание комментария.

Запрос
```
POST http://127.0.0.1:8000/api/v1/titles/(title_id)/reviews/(review_id)/comments/
Content-Type: application/json
Authorization: Bearer <token>

{
    "text": "example_comment_text"
}
```

Вернет
```
{
    "review": "example_review_text",
    "author": "example_user",
    "text": "example_comment_text",
    "pub_date": "example_date",
    "id": example_id
}
```
И запишет в базе данных комментарий к конкретному отзыву.


##### Автор
Горячев Алексей