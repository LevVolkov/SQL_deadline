[![Java CI with Gradle](https://github.com/LevVolkov/SQL_deadline/actions/workflows/gradle.yml/badge.svg)](https://github.com/LevVolkov/SQL_deadline/actions/workflows/gradle.yml)

# 8. Домашнее задание к занятию «3.2. SQL»

В качестве результата пришлите ссылки на ваши GitHub-проекты в личном кабинете студента на сайте [netology.ru](https://netology.ru).

Все задачи этого занятия нужно делать **в разных репозиториях**.

**Важно**: проекты с решением задач по данной теме реализуются с использованием Selenide.

**Важно**: если у вас что-то не получилось, то оформляйте issue [по установленным правилам](https://github.com/netology-code/aqa-homeworks/blob/master/report-requirements.md).

**Важно**: не делайте ДЗ всех занятий в одном репозитории. Иначе вам потом придётся достаточно сложно подключать системы Continuous integration.

## Как сдавать задачи

1. Инициализируйте на своём компьютере пустой Git-репозиторий.
1. Добавьте в него готовый файл [.gitignore](https://github.com/netology-code/aqa-homeworks/blob/master/.gitignore).
1. Добавьте в этот же каталог код, требуемый в ДЗ.
1. Сделайте необходимые коммиты.      
1. Добавьте в каталог `artifacts` целевой сервис [app-deadline.jar](https://github.com/netology-code/aqa-homeworks/blob/master/sql/app-deadline.jar).
1. Создайте публичный репозиторий на GitHub и свяжите свой локальный репозиторий с удалённым.
1. Сделайте пуш — удостоверьтесь, что ваш код появился на GitHub.
1. Ссылку на ваш проект отправьте в личном кабинете на сайте [netology.ru](https://netology.ru).
1. Задачи, отмеченные как необязательные, можно не сдавать, это не повлияет на получение зачёта.  
1. Интеграция проектов с CI необязательна и выполняется по желанию студента.          

**Важно**: задачи этого занятия не предполагают подключения к CI.

## Volumes

Пожалуйста, ознакомьтесь с кратким руководством по работе с [volumes](https://github.com/netology-code/aqa-homeworks/blob/master/sql/volumes.md).

## SQL

Пожалуйста, ознакомьтесь с кратким руководством по работе с клиентами [SQL](https://github.com/netology-code/aqa-homeworks/blob/master/sql/mysql-psql.md).

## Настройка CI     

**Важно**: интеграция проектов с CI необязательна           

Если вы решили получить бейдж сборки в данном проекте, то можно использовать инструкцию по настройке интеграции с Github Actions ([инструкция](https://github.com/netology-code/aqa-homeworks/tree/master/github-actions-integration)) с небольшими доработками:   
- в среде выполнения сборки необходимо запустить контейнер базы данных. В образах Github Actions есть докер и компоуз, поэтому будет достаточно добавить в yml файл секцию с запуском контейнера фоновом режиме            
```
    - name: Container start
      run: docker-compose up &
```    
- для успешного запуска SUT необходимо подождать полного запуска контейнера, самый простой способ выполнения данной задачи - использование команды sleep, добавляем в состав шагов сборки еще одну секцию     
```
    - name: Waiting for сontainer start
      run: sleep 30
```    

После выполнения интеграции необходимо удостовериться, что в CI запускается контейнер, SUT и выполняются автотесты. Автотесты могут падать и сборка может быть красной из-за багов тестируемого приложения. В таком случае должны быть заведены репорты на обнаруженные в ходе тестирования дефекты в отдельных issues, [придерживайтесь схемы при описании](https://github.com/netology-code/aqa-homeworks/blob/master/report-requirements.md).      

## Задача №1: скоро дедлайн

Случилось то, что обычно случается ближе к дедлайну: никто ничего не успевает и винит во всём остальных.

Разработчикам особо не до вас, им ведь нужно пилить новые фичи, поэтому они подготовили сборку, работающую с СУБД, и даже приложили схему базы данных (см. файл [schema.sql](https://github.com/netology-code/aqa-homeworks/blob/master/sql/schema.sql)). Но при этом сказали: «Остальное вам нужно сделать самим, там несложно» 😈

Что вам нужно сделать:
1. Внимательно изучить схему.
1. Создать Docker container на базе MySQL 8, прописать создание базы данных, пользователя, пароля.
1. Запустить SUT ([app-deadline.jar](https://github.com/netology-code/aqa-homeworks/blob/master/sql/app-deadline.jar)). Для указания параметров подключения к базе данных можно использовать:
- либо переменные окружения `DB_URL`, `DB_USER`, `DB_PASS`;
- либо указать их через флаги командной строки при запуске: `-P:jdbc.url=...`, `-P:jdbc.user=...`, `-P:jdbc.password=...`. Внимание: при запуске флаги не нужно указывать через запятую. Приложение не использует файл `application.properties` в качестве конфигурации, конфигурационный файл находится внутри JAR-архива;
- либо можете схитрить и попробовать подобрать значения, зашитые в саму SUT.

А дальше выясняется куча забавных вещей 😈 Рекомендуем вам попробовать разобраться самим, но если будет сложно, загляните в подсказку.

### Проблема первая: SUT не стартует

<details>
   <summary>Подсказка</summary>

   Проблема: SUT не создаёт самостоятельно таблицы в базе данных.

   Поэтому вам нужно сходить на сайт-описание Docker image MySQL и посмотреть, как при инициализации скармливать схему. Будет использоваться технология volumes.
</details>

### Проблема вторая: SUT валится при повторном перезапуске

<details>
   <summary>Подсказка</summary>

   Проблема: SUT вставляет в базу данных демо-данные, а поскольку там есть ограничение уникальности, это приводит к ошибкам.

   Поэтому вам нужно где-то настроить вычистку данных за SUT.
</details>

### Проблема третья (опционально): пароли

Если вы решите вдруг генерировать пользователей, чтобы под ними тестировать вход в приложение, то не должны удивляться тому, что в базе данных пароль пользователя хранится в зашифрованном виде.

Попытка его записать туда в открытом виде ни к чему хорошему не приведёт.

Настойчивые требования к разработчикам раскрыть алгоритм генерации пароля тоже ни к чему не привели.

Что же делать?

<details>
   <summary>Подсказка</summary>

   Если вы внимательно присмотритесь к демо-данным, то они очень, прямо подозрительно похожи на те, что были в одной из предыдущих задач.

   Значит, вы можете попробовать использовать уже готовые зашифрованные пароли, зная то, какие они были в незашифрованном виде.
</details>

Если вы добрались до этого шага и всё-таки успешно запустили SUT, вы уже герой.

Но теперь выяснилась забавная информация: разработчики фронтенда поругались с разработчиками бэкенда, и вы можете протестировать только вход в систему.

Внимательно посмотрите, как и куда сохраняются коды генерации в СУБД, и напишите тест, который, взяв информацию из БД о сгенерированном коде, позволит вам протестировать вход в систему через веб-интерфейс.

P.S. Неплохо бы ещё проверить, что при трёхкратном неверном вводе пароля система блокируется.

Итого в результате у вас должно получиться:
* docker-compose.yml*,
* app-deadline.jar,
* schema.sql,
* проект Gradle c кодом ваших автотестов.

Если ваша система не поддерживает Docker, то вам, к сожалению, придётся вручную установить MySQL на свой компьютер и отрабатывать тесты уже на ней. В этом случае положите в репозиторий файлик `README.md`, в котором опишите последовательность действий со скриншотами для установки сервера MySQL и загрузки в него файла `schema.sql`.

## Задача №2: backend vs frontend (необязательная)

Бэкенд-разработчики сказали, что они всё уже сделали, это фронтендщики тормозят. Поэтому функцию перевода денег с карты на карту мы протестировать через веб-интерфейс не можем.

Зато они выдали нам описание REST API, которое позволяет это сделать, использовать нужно тот же `app-deadline.jar`.

Вот описание API:

- Логин
```http
POST http://localhost:9999/api/auth
Content-Type: application/json

{
  "login": "vasya",
  "password": "qwerty123"
}
```

- Верификация
```http
POST http://localhost:9999/api/auth/verification
Content-Type: application/json

{
  "login": "vasya",
  "code": "599640"
}
```
В ответе, в поле «token» придёт токен аутентификации, который нужно использовать в последующих запросах.

<details>
<summary>Подсказка по REST-assured</summary>

Если вам приходит в ответ следующий JSON:
```json
{
  "status": "ok"
}
```

то вы можете вытащить значение из ответа с помощью REST-assured следующим образом:

```java
      String status = ... // ваш обычный запрос  
      .then()
          .statusCode(200)
      .extract()
          .path("status")
      ;

      // используются matcher'ы Hamcrest
      assertThat(status, equalTo("ok"));
```

Если вам нужно вытащить весь ответ, чтобы потом искать по нему, например, если нужно несколько полей, то:

```java
      Response response = ... // ваш обычный запрос  
      .then()
          .statusCode(200)
      .extract()
          .response()
      ;

      String status = response.path("status");
      // используются matcher'ы Hamcrest
      assertThat(status, equalTo("ok"));
```

</details>

- Просмотр карт
```http
GET http://localhost:9999/api/cards
Content-Type: application/json
Authorization: Bearer {{token}}
```

Где {{token}} — это значение «token» с предыдущего шага. Фигурные скобки писать не нужно.

- Перевод с карты на карту (любую)
```
POST http://localhost:9999/api/transfer
Content-Type: application/json
Authorization: Bearer {{token}}

{
  "from": "5559 0000 0000 0002",
  "to": "5559 0000 0000 0008",
  "amount": 5000
}
```

Внимательно изучите запросы и ответы и, используя любой инструмент, который вам нравится, реализуйте тесты API.

В результате выполнения этой задачи вы должны положить в репозиторий следующие файлы:
* docker-compose.yml*,
* app-deadline.jar,
* schema.sql,
* код ваших автотестов.

P.S. Всё не может быть хорошо, наверняка разработчики где-то допустили ошибки. Не забывайте заводить issue о найденных багах 😈
