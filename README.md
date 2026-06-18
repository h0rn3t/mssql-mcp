# mssql-mcp

[MCP-сервер (Model Context Protocol)](https://modelcontextprotocol.io/), який надає AI-асистентам структурований, оптимізований для читання доступ до баз даних Microsoft SQL Server. Він надає можливості дослідження схеми, профілювання даних, аналізу зв'язків, пояснення запитів та безпечних операцій читання/запису через стандартизований MCP-інтерфейс.

## Можливості

- **23 MCP-інструменти** для пошуку схеми, опису таблиць, профілювання даних, перевірки зв'язків/залежностей, переліку об'єктів, інспекції DDL, пояснення запитів, тестування з'єднання та багаторівневого доступу до запису
- **Багаторівнева модель доступу** через `MSSQL_ACCESS_LEVEL`: `READONLY` (за замовчуванням), `DML-RW` (додає insert/update/delete), `DDL-RW` (додає create/drop table/index)
- **SQL-безпечний дизайн** із цитуванням ідентифікаторів, перевіркою складених імен та примусовим виконанням запитів лише для читання
- **Захист запитів лише для читання**, який відхиляє мутуючі інструкції (`INSERT`, `UPDATE`, `DELETE`, `MERGE`, `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `EXEC` тощо) в інструменті `read_data`
- **Підтвердження мутацій** у режимі попереднього перегляду, який показує рядки, що будуть змінені, перед виконанням запису, коли ввімкнено `MSSQL_REQUIRE_CONFIRMATION`
- **План виконання** через `SHOWPLAN_XML` для розуміння продуктивності запитів
- **Тестування з'єднання**, яке перевіряє доступність і повідомляє про затримку
- **Перелік середовищ**, що показує поточні налаштування сервера, бази даних та рівня доступу

## Встановлення

### Варіант 1. Завантаження готового релізу (рекомендовано)

1. Перейдіть на сторінку [Releases](https://github.com/h0rn3t/mssql-mcp/releases) у репозиторії проєкту.
2. Оберіть реліз, що відповідає вашій операційній системі та архітектурі:
   - `mssql-mcp_darwin_amd64.tar.gz` — macOS (Intel)
   - `mssql-mcp_darwin_arm64.tar.gz` — macOS (Apple Silicon)
   - `mssql-mcp_linux_amd64.tar.gz` — Linux (x86_64)
   - `mssql-mcp_linux_arm64.tar.gz` — Linux (ARM64)
   - `mssql-mcp_windows_amd64.zip` — Windows (x86_64)
3. Завантажте архів і розпакуйте його у зручну директорію, наприклад `~/bin` або `/usr/local/bin`.

   **macOS / Linux:**
   ```bash
   mkdir -p ~/bin
   cd ~/bin
   # Замініть URL на актуальний з релізу
   curl -L -o mssql-mcp.tar.gz https://github.com/h0rn3t/mssql-mcp/releases/latest/download/mssql-mcp_darwin_arm64.tar.gz
   tar -xzf mssql-mcp.tar.gz
   rm mssql-mcp.tar.gz
   chmod +x mssql-mcp
   ```

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\bin"
   cd "$env:USERPROFILE\bin"
   Invoke-WebRequest -Uri "https://github.com/h0rn3t/mssql-mcp/releases/latest/download/mssql-mcp_windows_amd64.zip" -OutFile "mssql-mcp.zip"
   Expand-Archive -Path "mssql-mcp.zip" -DestinationPath "." -Force
   Remove-Item "mssql-mcp.zip"
   ```

4. Перевірте, що бінарник працює:
   ```bash
   ./mssql-mcp --version
   # або
   ./mssql-mcp --help
   ```

5. (Опціонально) Додайте директорію з бінарником до `PATH`, щоб запускати його з будь-якого місця:
   ```bash
   # macOS / Linux (додайте у ~/.zshrc або ~/.bashrc)
   export PATH="$HOME/bin:$PATH"
   source ~/.zshrc
   ```
   ```powershell
   # Windows (PowerShell)
   [Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:USERPROFILE\bin", "User")
   ```

### Варіант 2. Збірка з вихідного коду

1. Переконайтеся, що встановлено [Go](https://go.dev/dl/) версії 1.22 або вище:
   ```bash
   go version
   ```

2. Клонуйте репозиторій:
   ```bash
   git clone https://github.com/h0rn3t/mssql-mcp.git
   cd mssql-mcp
   ```

3. Зберіть бінарник:
   ```bash
   # За допомогою Go
   go build -o mssql-mcp ./cmd/mssql-mcp

   # Або за допомогою Taskfile (якщо встановлено task)
   task build
   ```

4. (Опціонально) Перемістіть бінарник у директорію з `PATH`:
   ```bash
   mv mssql-mcp /usr/local/bin/
   ```

5. Перевірте збірку:
   ```bash
   mssql-mcp --help
   ```

### Варіант 3. Запуск через `go run`

Для швидкого тестування без збірки бінарника:
```bash
git clone https://github.com/h0rn3t/mssql-mcp.git
cd mssql-mcp
go run ./cmd/mssql-mcp
```

## Використання

### Змінні середовища

| Змінна | Обов'язкова | За замовчуванням | Опис |
|----------|----------|---------|-------------|
| `MSSQL_SERVER` | Так | - | Ім'я хоста або IP SQL Server |
| `MSSQL_DATABASE` | Так | - | Назва бази даних |
| `MSSQL_USERNAME` | Так | - | Ім'я користувача для входу |
| `MSSQL_PASSWORD` | Так | - | Пароль для входу |
| `MSSQL_PORT` | Ні | `1433` | Порт SQL Server |
| `MSSQL_ACCESS_LEVEL` | Ні | `READONLY` | `READONLY`, `DML-RW` або `DDL-RW` |
| `MSSQL_ENCRYPT` | Ні | `true` | Увімкнути шифровані з'єднання з SQL Server |
| `MSSQL_TRUST_SERVER_CERTIFICATE` | Ні | `false` | Довіряти самопідписаним сертифікатам |
| `MSSQL_CONNECTION_TIMEOUT` | Ні | `30` | Тайм-аут з'єднання в секундах |
| `MSSQL_QUERY_TIMEOUT` | Ні | `120` | Тайм-аут запиту в секундах |
| `MSSQL_MAX_ROWS_DEFAULT` | Ні | `1000` | Ліміт рядків за замовчуванням для запитів |
| `MSSQL_REQUIRE_CONFIRMATION` | Ні | `true` | Вимагати прапорця підтвердження для записів |
| `MSSQL_TRANSPORT` | Ні | `stdio` | MCP-транспорт: `stdio` або `sse` |
| `MSSQL_HTTP_ADDR` | Ні | `:8080` | Адреса HTTP-прослуховування, коли `MSSQL_TRANSPORT=sse` |
| `MSSQL_SSE_PATH` | Ні | `/sse` | Шлях SSE-ендпоінту, коли `MSSQL_TRANSPORT=sse` |

### Запуск як MCP-сервер

За замовчуванням сервер взаємодіє через stdio. Налаштуйте ваш MCP-клієнт для його запуску:

```json
{
  "mcpServers": {
    "mssql": {
      "command": "/path/to/bin/mssql-mcp",
      "env": {
        "MSSQL_SERVER": "localhost",
        "MSSQL_DATABASE": "YourDatabase",
        "MSSQL_USERNAME": "sa",
        "MSSQL_PASSWORD": "YourPassword",
        "MSSQL_ENCRYPT": "true",
        "MSSQL_TRUST_SERVER_CERTIFICATE": "true",
        "MSSQL_ACCESS_LEVEL": "READONLY"
      }
    }
  }
}
```

Щоб обслуговувати MCP через SSE, встановіть `MSSQL_TRANSPORT=sse` і запустіть сервер як HTTP-процес:

```bash
MSSQL_TRANSPORT=sse \
MSSQL_HTTP_ADDR=:8080 \
MSSQL_SSE_PATH=/sse \
/path/to/bin/mssql-mcp
```

Потім налаштуйте MCP-клієнт із підтримкою SSE для підключення до:

```text
http://localhost:8080/sse
```

### Рівні доступу

- **`READONLY`** (за замовчуванням) — дослідження схеми, перелік об'єктів, читання даних, профілювання, перевірка зв'язків, інспекція DDL, пояснення запитів, тестування з'єднання. 17 інструментів.
- **`DML-RW`** — усі інструменти лише для читання плюс `insert_data`, `update_data`, `delete_data`. 20 інструментів.
- **`DDL-RW`** — усі DML-інструменти плюс `create_table`, `create_index`, `drop_table`. 23 інструменти.

Мутації (`update_data`, `delete_data`, `drop_table`) вимагають прапорця `"confirm": true`, коли ввімкнено `MSSQL_REQUIRE_CONFIRMATION` (за замовчуванням). Без підтвердження сервер повертає попередній перегляд рядків, що будуть змінені.

## MCP-інструменти

### Лише для читання (READONLY)

| Інструмент | Опис |
|------|-------------|
| `search_schema` | Пошук таблиць і стовпців за шаблоном назви з пагінацією |
| `describe_table` | Отримання стовпців, первинних ключів, зовнішніх ключів та індексів таблиці |
| `list_table` | Перелік таблиць, опціонально відфільтрованих за схемою та назвою |
| `list_databases` | Перелік усіх баз даних на сервері |
| `list_environments` | Показ поточних налаштувань з'єднання та рівня доступу |
| `profile_table` | Кількість рядків, null-значень, унікальних значень, min/max для кожного стовпця, з опціональними зразками даних |
| `inspect_relationships` | Перелік зовнішніх ключів, що виходять з таблиці та входять у неї |
| `inspect_dependencies` | Пошук об'єктів (представлень, процедур, функцій), які залежать від таблиці |
| `explain_query` | Отримання XML-плану виконання для запиту лише для читання |
| `read_data` | Виконання запиту SELECT лише для читання з обмеженням рядків |
| `test_connection` | Перевірка доступності сервера та повернення затримки й інформації про версію сервера |
| `validate_environment_config` | Перевірка коректності налаштування всіх змінних середовища |
| `list_schemas` | Перелік схем бази даних та їхніх власників |
| `list_views` | Перелік представлень та їхніх визначень |
| `list_triggers` | Перелік тригерів таблиці, подій, часу спрацювання та визначень |
| `show_create_table` | Генерування інструкції CREATE TABLE для існуючої таблиці |
| `table_size` | Звіт про оцінку кількості рядків та розмір таблиці/індексу в КБ |

### DML (DML-RW)

| Інструмент | Опис |
|------|-------------|
| `insert_data` | Вставлення одного або кількох рядків у таблицю |
| `update_data` | Оновлення рядків, що відповідають умові WHERE (з опціональним попереднім переглядом) |
| `delete_data` | Видалення рядків, що відповідають умові WHERE (з опціональним попереднім переглядом) |

### DDL (DDL-RW)

| Інструмент | Опис |
|------|-------------|
| `create_table` | Створення таблиці з визначеннями стовпців, первинними ключами та стовпцями identity |
| `create_index` | Створення стандартного або унікального індексу на вказаних стовпцях |
| `drop_table` | Видалення таблиці (з опціональним попереднім переглядом/підтвердженням) |

## Ліцензія

Ліцензія MIT. Див. [LICENSE](LICENSE) для деталей.
