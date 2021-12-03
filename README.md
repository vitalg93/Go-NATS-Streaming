# Тестовое задание
В сервисе реализовано:
- Подключение и подписка на канал в Nats-streaming
- Сохранение полученных данных в Postgres
- Хранение и выдача данных по id из кеша (с помощью http-сервера)
- В случае падения сервиса Кеш восстанваливается из Postgres
- Сделан простейший интерфейс отображения полученных данных, для
их запроса по id используя Bootstrap и JavaScript
![Alt-текст](https://raw.githubusercontent.com/vitalg93/hello-world/main/web-interface.jpg "Интерфейс сервиса")
Схема базы данных приведена ниже:
![Alt-текст](https://raw.githubusercontent.com/vitalg93/hello-world/main/db_scheme.JPG "Интерфейс сервиса")

## Начало работы

### Требования
Установите компилятор `Go` (если еще не установлен): https://golang.org/doc/tutorial/getting-started
Необходим доступ к рабочему Nats server: https://docs.nats.io/running-a-nats-service/introduction/installation и доступ к Postgres https://www.postgresql.org/download/

### Установка
Склонируйте репозиторий и запустите сервер, который будет доступен по URL http://localhost:3333 в вашем браузере, установите настройки подключения в файле `/cmd/config/config.go`

```bash
$ go run cmd/main.go
```
### Взаимодействие с сервером
- При запуске сервер загружает конфигурацию из `/cmd/config/config.toml` файла. Конфигурация содержит настройки доступа к БД, Nats-streaming и настройки кеша: размер буфера (по умолчанию - 10 элементов) и имя приложения (для работы с кешем нужно уникальное имя, если запущено несколько копий этого приложения)
- Далее сервер подключается к Nats-streaming. Для тестирования - работает Publisher, который отправляет 1 сообщение через Nats
- Полученные сообщения парсятся, сохраняются в кеш (в память) и в БД. Кеш дублируется в БД (список `Order id`) для его восстановления в случае падения сервиса
- Далее запускается http-сервер, который выдает `Order` по `id` доступный по адресу `http://localhost:3333` (главная страница). Пользователь вводит идентификатор `Order` в единственное поле для ввода на html-форме и нажимает 'Search'. С помощью JS осуществляется редирект на `http://localhost:3333/orders/{id}`, где отображаются данные о заказе.

### Завершение работы с сервером
- Для завершения работы нажмите `Ctrl+C` в его консоли (graceful shutdown). Это необходимо для корректного завершения работы: очистится кеш из БД, закроются подключения к Nats.