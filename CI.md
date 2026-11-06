# CI: тесты, сборка и запуск Flaskr

Исходный репозиторий: https://github.com/Teodil/python-test-ci-cd-2

Workflow: [`.github/workflows/ci.yml`](.github/workflows/ci.yml).

## Запуск

- Push в `main`.
- Создание, обновление и повторное открытие pull request в `main`.

## Job `tests`

Матрица Python 3.11 и 3.12 с `fail-fast: false`: ошибка одной версии не отменяет проверку другой. `uv sync --frozen --no-default-groups --group tests` устанавливает текущий проект и тестовые зависимости из `uv.lock`. Запускается весь каталог `tests/`, код возврата pytest не подавляется.

Используется lock-файл проекта: исходные тесты несовместимы с последним pytest 9.2.0 (импортируют удалённый внутренний объект `notset`).

## Job `build-and-run`

`needs: tests` разрешает запуск только после успеха всей матрицы.

На Python 3.12 workflow:

1. Собирает `.whl` и `.tar.gz` командой `python -m build`.
2. Сохраняет дистрибутивы как артефакт `distributions`.
3. Создаёт отдельное виртуальное окружение в каталоге runner.
4. Устанавливает собранный wheel Flask.
5. Устанавливает Flaskr из `examples/tutorial` с `--no-deps`, сохраняя Flask из wheel, затем выполняет `pip check`.
6. Инициализирует базу командой `python -m flask --app flaskr init-db`.
7. Запускает сервер на `127.0.0.1:5000`, проверяет HTTP-ответ и наличие `Flaskr` на странице. При завершении шага останавливает сервер и выводит лог.

В исходном репозитории каталог называется `examples/tutorial`, а не `examples/tutorials`.

GitHub Actions проверяет запуск приложения, но не предоставляет постоянный хостинг: сервер работает только во время job.

## Локальная проверка

- Python 3.11: 488 passed, 2 skipped.
- Python 3.12: 488 passed, 2 skipped.
- Сборка: `flask-3.1.2-py3-none-any.whl` и `flask-3.1.2.tar.gz`.
- Установка wheel и Flaskr в отдельное окружение, `pip check` и инициализация БД успешны.
- HTTP-проверка: 200 OK. Локально использовался порт 5001, поскольку порт 5000 занят системным Control Center macOS. В workflow используется требуемый порт 5000 на Ubuntu runner.
