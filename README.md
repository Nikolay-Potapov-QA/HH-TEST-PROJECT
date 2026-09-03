# HH.ru API Testing

Ручное и полуавтоматизированное тестирование открытой части API HH.ru. 

## Об этом проекте

Проект демонстрирует полный цикл QA-работы:
- **Ручное тестирование** реального публичного API (HH.ru)
- **API-тестирование** через Postman с Newman и Allure-отчётами
- **Граничный анализ** и поиск edge-cases
- **Баг-репорты** с документированием расхождений спецификации и реальности

Эндпоинты:
- `GET /areas` и `GET /areas/{id}` — справочник регионов. 
- `GET /dictionaries` — справочник фильтров. 
- `GET /vacancies` — поиск вакансий (закрыт для публичного доступа с апреля 2026; тестируется на моке)

## Как запустить

```bash
# Установить зависимости
npm install -g newman newman-reporter-allure

# Запустить коллекции локально (требуется Postman mock server активным)
newman run ./collections/hh-live.postman_collection.json \
  -e ./environments/hh-api.postman_environment.json \
  --reporters cli,allure

# Генерировать Allure-отчёт
allure generate allure-results -o allure-report
allure open allure-report
```

В CI (GitHub Actions) запускается автоматически при каждом push.

## Тест-кейсы

Всего: **18 тест-кейсов**. Статус: [смотри Allure-отчёт](https://github-username.github.io/hh-api-testing/)

## Найденные баги и наблюдения

- [Integer overflow на INT32_MAX](/bug-reports/BR-001-integer-overflow.md) — статус 502 вместо 400
- [Недокументированное поведение area_id=0](/bug-reports/BR-002-area-id-zero.md) — требует уточнения
- [Остальные наблюдения](/bug-reports/observations.md)

## Структура проекта

- **test-cases/** — markdown-файлы с полным описанием каждого тест-кейса
- **bug-reports/** — задокументированные баги и observations с шагами воспроизведения
- **collections/** — экспортированные Postman-коллекции
- **environment/** — файлы окружений для разных конфигураций (live vs mock)
- **.github/workflows/** — GitHub Actions конфиг для CI/CD

## Примечания

### Ограничения

- `GET /vacancies` тестируется на моке, а не на реальном API, так как:
  - С апреля 2026 публичный доступ закрыт (требуется регистрация dev-приложения и модерация)
  - Регистрация требует верифицированного аккаунта работодателя
  - Мок построен на официальной OpenAPI-схеме HH API и полностью соответствует контракту

- **hh-user-agent** — задокументирован в OpenAPI-схеме, но на практике сервер его игнорирует (не требует, 200 и без него). Это расхождение между документацией и реальностью; в тестах используется для соответствия примерам из доки, но его отсутствие не блокирует запрос.


### Инструменты

- **Newman** — CLI-раннер для Postman-коллекций
- **Allure** — генерация красивых отчётов из результатов прогона
- **GitHub Actions** — автоматизация запуска тестов
- **Postman Mock Server** — облачный мок-сервер для vacancies
