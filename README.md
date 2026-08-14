##  Быстрый старт

### 1. Создание пользователя в Keycloak

1. Откройте **http://localhost:8080**
2. Войдите:
   - **Username**: `admin`
   - **Password**: `admin`
3. В левом верхнем углу выберите realm: **reports-realm**
4. Перейдите в **Users** → **Add new user**
5. Заполните:
   - **Username**: `user_123`
   - **Email**: `user123@example.com`
   - **First name**: `Иван`
   - **Last name**: `Иванов`
   - **User enabled**:
6. Нажмите **Save**
7. Перейдите во вкладку **Credentials** → **Set password**
   - **Password**: `password123`
   - **Password confirmation**: `password123`
   -  **Выключите** переключатель **Temporary**
8. Нажмите **Save**

### 2. Запуск ETL в Airflow

1. Откройте **http://localhost:8081**
2. Войдите:
   - **Username**: `admin`
   - **Password**: `admin`
3. Найдите DAG `etl_prosthesis_reports_mart`
4. Включите его (тумблер слева → ON)
5. Нажмите **Trigger DAG** (▶️ в правом верхнем углу)
6. Подождите ~30 секунд, пока обе задачи станут зелёными ✅

### 3. Проверка данных в ClickHouse (опционально)

 bash
docker compose exec clickhouse clickhouse-client \
  --user=default --password=secret \
  --query="SELECT * FROM bionicpro.prosthesis_reports_mart LIMIT 5"
 

### 4. Открытие приложения

1. Откройте **http://localhost:3000**
2. Нажмите **Login** (перенаправление на Keycloak)
3. Войдите:
   - **Username**: `user_123`
   - **Password**: `password123`
4. Нажмите **"Получить отчёт"** — увидите таблицу с телеметрией
5. Нажмите **"Скачать отчёт (JSON)"** — скачается файл `report_user_123.json`
**ВНИМАНИЕ: МОЖЕТ БЫТЬ ОШИБКА ИЗ-ЗА БРАУЗЕРА. ПОСЛЕ СОЗДАНИЕ ПОЛЬЗОВАТЕЛЯ ПОЧИТСТИТЕ КЭШ!**

### Ошибка Fernet key в Airflow
 bash
# Сгенерировать новый ключ
python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
 
Добавить в `docker-compose.yml` в сервис `airflow`:
 yaml
- AIRFLOW__CORE__FERNET_KEY=<ваш_ключ>
 

### Ошибка аутентификации в ClickHouse
Пользователь `default` подключается **без пароля** через терминал:
 bash
docker compose exec clickhouse clickhouse-client --user=default --query="SELECT 1"
 
Через веб-интерфейс (http://localhost:8123/play) пароль тоже должен быть **пустым**.

### "No reports found" в UI
1. Убедитесь, что DAG в Airflow выполнен успешно (зелёные квадраты)
2. Проверьте, что username в Keycloak совпадает с `user_id` в ClickHouse (`user_123`, а не UUID)
3. Запустите DAG вручную: Airflow UI → Trigger DAG

### CORS ошибки
Убедитесь, что в `report-service/app/main.py` указан правильный origin:
 python
allow_origins=["http://localhost:3000"]
 

