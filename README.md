# Calc-LMS-Orchestrator

**Calc-LMS-Orchestrator** — это распределённый вычислитель арифметических выражений. Система принимает выражение, разбивает его на отдельные задачи, и вычисляет результат асинхронно с помощью агента, работающего в нескольких горутинах. Веб-интерфейс позволяет пользователю отправлять выражения и наблюдать за их состоянием.  
**Важно:** Калькулятор работает только с однозначными числами — выражения с двузначными и более числами блокируются.

tg: @prud_official

## Функциональность

- **API Оркестратора:**
  - `POST /api/v1/calculate` — отправка арифметического выражения (например, `"2+2*2"`) для вычисления.
  - `GET /api/v1/expressions` — получение списка всех выражений с их статусами и результатами.
  - `GET /api/v1/expressions/{id}` — получение подробной информации по конкретному выражению.
  - `GET /internal/task` — выдача задачи агенту для вычисления.
  - `POST /internal/task` — приём результата вычисленной задачи.

- **Агент (Worker):**  
  Агент в цикле запрашивает задачи с эндпоинта `/internal/task`, выполняет операцию (с имитацией задержки, зависящей от переменных окружения) и отправляет результат обратно.

- **Веб-интерфейс:**  
  Простая HTML-страница для отправки выражения и просмотра списка выражений с их статусами.

- **Тесты:**  
  Проект покрыт модульными тестами для проверки логики оркестратора (разбор выражения, генерация задач и т.д.).

## Структура проекта

```
calc-LMS-orchestrator/
├── cmd/
│   ├── calc/           # Оркестратор (сервер)
│   │   └── main.go
│   └── agent/          # Агент (демон)
│       └── main.go
├── go.mod
├── internal/
│   ├── api/v1/
│   │   └── api.go       # API-эндпоинты для оркестратора и агентов
│   ├── orchestrator/
│   │   ├── orchestrator.go         # Логика разбора выражений и генерации задач
│   │   └── orchestrator_test.go    # Тесты для оркестратора
│   └── server/
│       └── server.go    # HTTP-сервер с роутингом и отдачей статики (веб-интерфейс)
└── web/
    ├── index.html       # Веб-интерфейс
    ├── main.js          # Логика веб-интерфейса
    └── style.css        # Стили веб-интерфейса
```

## Установка

1. **Клонирование репозитория:**

   ```bash
   git clone https://github.com/ImNotDarKing/calc-LMS-2.10
   cd calc-LMS-2.10
   ```

2. **Обновление зависимостей:**

   ```bash
   go mod tidy
   ```

## Запуск системы

### Тестирование

Для запуска тестов оркестратора выполните из корневой директории:

```bash
go test ./internal/orchestrator/...
```

### Запуск оркестратора (сервера)

Запустите сервер из корневой директории, используя команду:

```bash
go run cmd/calc/main.go
```

Сервер будет запущен на порту **8080** и отдавать как API, так и веб-интерфейс.

### Запуск агента (демона)

В отдельном терминале перейдите в директорию агента и запустите его. Для задания количества горутин (например, 3) используйте переменную окружения `COMPUTING_POWER`:

- **Linux/macOS:**

  ```bash
  cd cmd/agent
  COMPUTING_POWER=3 go run main.go
  ```

- **Windows (PowerShell):**

  ```powershell
  cd cmd/agent
  COMPUTING_POWER=3 go run main.go
  ```

### Доступ к веб-интерфейсу

После запуска оркестратора откройте браузер и перейдите по адресу:

```
http://localhost:8080
```

Веб-интерфейс позволяет:
- Ввести арифметическое выражение.
- Отправить его на сервер.
- Просмотреть список всех выражений с их статусами и результатами (обновляется каждые 5 секунд).

## Примеры использования с помощью curl

### 1. Отправка выражения на вычисление

```bash
curl --location 'http://localhost:8080/api/v1/calculate' `
--header 'Content-Type: application/json' `
--data '{"expression": "2+2*2"}'
```

_Ожидаемый ответ (при успешном принятии выражения):_

```json
{"id": 1}
```

### 2. Получение списка выражений

```bash
curl --location 'http://localhost:8080/api/v1/expressions'
```

_Пример ответа:_

```json
{
    "expressions": [
        {"id": 1, "status": "completed", "result": 6},
        {"id": 2, "status": "pending", "result": NaN}
    ]
}
```

### 3. Получение информации по конкретному выражению

```bash
curl --location 'http://localhost:8080/api/v1/expressions/1'
```

_Пример ответа:_

```json
{
    "expression": {
        "id": 1,
        "expression": "2+2*2",
        "status": "completed",
        "result": 6,
        "taskIDs": [1, 2],
        "rootTaskID": 2
    }
}
```

## Схема работы системы

```
flowchart TD
    A[Пользователь] -->|Отправка запроса &#40;curl или веб-интерфейс&#41;| B[Оркестратор (сервер)]
    B -->|Генерация задач| C[Хранилище выражений и задач]
    C -->|Запрос задачи| D[Агент (worker)]
    D -->|Выполнение задачи| E[Вычисление операции]
    E -->|Отправка результата| B
    B -->|Обновление статуса выражения| C
```

## Дополнительная информация

- **Поддержка чисел:**  
  Калькулятор поддерживает только однозначные числа. Если во вводимом выражении встречается двузначное число (например, "12"), система возвращает ошибку.

- **Логирование:**  
  В консоли оркестратора и агента выводятся сообщения о ходе выполнения задач, что позволяет отследить работу системы.

- **Конфигурация:**  
  Значения времени выполнения операций задаются через переменные окружения (например, `TIME_ADDITION_MS` и т.д.) — в данном примере установлены фиксированные значения.
