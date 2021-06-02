# Тестовое задание для NodeJs разработчика

## Стек:
DB: PostgresSQL

Framework: express, nestJS __Зависит от описания в вакансии и вашего опыта__

## Описание

Реализовать CRUD для сущностей User и Tags.

### **User:**

| field         | type        |
| ------------- |:-----------:|
| uid           | uuid        |
| email         | string(100) |
| password      | string(100) | 
| nickname      | string(30)  |

**password**: должен содержать как минимум одну цифру, одну заглавную и одну строчную буквы.

**password**: минимальная длинна 8 символов

### **Tag**

| field         | type           |
| ------------- |:--------------:|
| id            | int            |
| creator       | uuid           |
| name          | string(40)     |
| sortOrder     | int default(0) |

**creator** uid пользователя создавшего тэг, только он может его менять и удалять из базы

### **UserTag**

Эту таблицу нужно сделать самим

-----

Сделать сервис с REST API и авторизациею по JWT bearer token.

Настроить CORS для доступа с любого origin.

## Обязательные требования

1) Пароли не хранить в открытом виде
2) Реализовать валидацию полей на api запросы с кодами ответов и сообщениями об ошибке в теле ответа.
3) Развернуть проект в любом удобном месте, что бы можно было прогнать тесты и проверить.
4) Код на github или gitlab
5) Придерживаться принципам SOLID
6) Токен авторизации живет 30 минут
7) Реализовать endpoint для обновления токена
8) Создать миграции
9) Написать сопроводительную документация в README.md для разворота
10) Реализовать offset или пагинацию для сущности **TAG**
11) Реализовать Сортировку по полю **sortOrder** и(или) полю **name** для сущности **TAG**

## Дополнительные требования

1) Использовать DTO
2) Писать интерфейсы и реализовывать их
3) Желательно не использовать ORM
4) Написать DockerFile для приложения
5) Написать docker-composer для локального разворота приложения
6) Реализовать кеширование
7) Покрыть тестами сами api
8) Добавить генерацию swagger документации для api методов (или написать ручками и положит в проект в директорию /doc)

## Список API endpoint

- POST /signin

```json
{
  "email": "example@exe.com",
  "password": "example",
  "nickname": "nickname"
}
```

Валидировать **password**, **email**, **nickname**

RETURN:

```json
{
  "token": "token",
  "expire": "1800"
}
```

---
- POST /login

```json
{
  "email": "example@exe.com",
  "password": "example"
}
```

RETURN:
```json
{
  "token": "token",
  "expire": "1800"
}
```

---
- POST /logout
  
  HEADER: ```Authorization: Bearer {token}```

**Ниже идущие api закрыты под авторизацией**

---
- GET /user

  HEADER: ```Authorization: Bearer {token}```

RETURN:
```json
{
  "email": "example@exe.com",
  "nickname": "example",
  "tags": [
    {
      "id": "id",
      "name": "example",
      "sortOrder": "0"
    }
  ]
}
```

---
- PUT /user
  
  HEADER: ```Authorization: Bearer {token}```

```json
{
  "email": "example@exe.com",
  "password": "example",
  "nickname": "example"
}
```

Все поля опциональные

Валидировать **password**, **email**, **nickname**

Проверять на дублирование __email__ и __nickname__ в базе

RETURN :

```json
{
  "email": "example@exe.com",
  "nickname": "example"
}
```

---
- DELETE /user

  HEADER: ```Authorization: Bearer {token}```

Разлогиниваем и удаляем пользователя

---
- POST /tag
  
  HEADER: ```Authorization: Bearer {token}```

```json
{
  "name": "example",
  "sortOrder": "0"
}
```

**sortOrder** опционально по default 0
Проверять на дублирование __name__ в базе и максимальную длину

RETURN :

```json
{
  "id": "id",
  "name": "example",
  "sortOrder": "0"
}
```

---
- GET /tag/{id}

  HEADER: ```Authorization: Bearer {token}```


RETURN :
```json
{
  "creator": {
    "nickname": "example",
    "uid": "exam-pl-eUID"
  },
  "name": "example",
  "sortOrder": "0"
}
```


---
- GET /tag?sortByOrder&sortByName&offset=10&length=10

  HEADER: ```Authorization: Bearer {token}```

**sortByOrder**, **offset** **SortByName**, **length** опциональны

**length** количество элементов в выборке

Если выбрали подход с страницами, то ипсользуйте параметры **page** и **pageSize** вместо **offset** и **length**

RETURN :

```json
{
  "data": [
    {
      "creator": {
        "nickname": "example",
        "uid": "exam-pl-eUID"
      },
      "name": "example",
      "sortOrder": "0"
    },
    {
      "creator": {
        "nickname": "example",
        "uid": "exam-pl-eUID"
      },
      "name": "example",
      "sortOrder": "0"
    }
  ],
  "meta": {
    "offset": 10,
    "length": 10,
    "quantity": 100
  }
}
```

**quantity** общее количество элементов в выборке

---
- PUT /tag/{id}

  HEADER: ```Authorization: Bearer {token}```

Тэг может менять только владелец

```json
{
  "name": "example",
  "sortOrder": "0"
}
```

**name** или **sortOrder** опциональны

RETURN :
```json
{
  "creator": {
    "nickname": "example",
    "uid": "exam-pl-eUID"
  },
  "name": "example",
  "sortOrder": "0"
}
```

---

- DELETE /tag/{id}

HEADER: ```Authorization: Bearer {token}```


Тэг может удалить только владелец

Каскадом удалем все связанные записи с этим Тэгом


---

---
- POST /user/tag

  HEADER: ```Authorization: Bearer {token}```

```json
{
  "tags": [1, 2]
}
```

Проверяем тэги на наличие в базе и добавляем к пользователю пачкой (атомарной операцией)

Пример: Если тэга с id 2 нет в базе то и тэг с id 1 не добавится пользователю

RETURN :

```json
{
  "tags": [
    {
      "id": 1,
      "name": "example",
      "sortOrder": "0"
    },
    {
      "id": 2,
      "name": "example",
      "sortOrder": "0"
    },
    {
      "id": 3,
      "name": "example",
      "sortOrder": "0"
    }
  ]
}
```

---
- DELETE /user/tag/{id}

  HEADER: ```Authorization: Bearer {token}```

RETURN :

```json
{
  "tags": [
    {
      "id": 1,
      "name": "example",
      "sortOrder": "0"
    },
    {
      "id": 2,
      "name": "example",
      "sortOrder": "0"
    },
    {
      "id": 3,
      "name": "example",
      "sortOrder": "0"
    }
  ]
}
```

---
- GET /user/tag/my

  HEADER: ```Authorization: Bearer {token}```

Отдаем список тэгов в которых пользователь является создателем

RETURN :

```json
{
  "tags": [
    {
      "id": 1,
      "name": "example",
      "sortOrder": "0"
    },
    {
      "id": 2,
      "name": "example",
      "sortOrder": "0"
    },
    {
      "id": 3,
      "name": "example",
      "sortOrder": "0"
    }
  ]
}
```









