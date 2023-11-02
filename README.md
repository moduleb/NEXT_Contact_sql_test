### NEXT_Contact_sql_test

# Тестовое задание по БД.
## Предметная область
В компании есть несколько проектов (`PROJECTS`), которые требуют определенных навыков (`SKILLS`) для поддержки. Информация о необходимых навыках для каждого проекта хранится в таблице `REL_PROJECT_SKILL`. Сотрудники (`USERS`), занятые поддержкой проектов, должны обладать определенными навыками (`SKILLS`). Данные о навыках, которыми владеет каждый сотрудник, содержатся в таблице `REL_USER_SKILL`. В таблицах связей присутствуют внешние ключи. Логическая схема базы данных представлена ниже.

<img src="https://github.com/ModuleB/NEXT_Contact_sql_test/blob/main/image.png" height="300">

- `P` - primary key
- `F` - foreign key

### 1. Получение уникального списка имен сотрудников
```sql
SELECT DISTINCT "name" FROM users
```

### 2. Изменение имени всех сотрудников с именем "Саша" на "Александр"
```sql
UPDATE users
SET name = 'Александр'
WHERE name = 'Саша'
```

### 3. Вывод имен сотрудников, владеющих более чем 1 навыком
```sql
SELECT users.name
FROM rel_user_skill
JOIN users ON rel_user_skill.fid_user = users.id  
GROUP BY users.name
HAVING COUNT(users.name) > 1;
```

### 4. Получение списка сотрудников, способных поддерживать определенный проект
```sql
WITH ProjectSkillCheck AS (
    SELECT u.name as user_name, 
           p.name as project_name,
           rus.fid_user,
           rps.fid_project,
           rps.fid_skill,
           CASE 
               WHEN (
                     SELECT COUNT(rps2.fid_skill)
                     FROM rel_project_skill rps2
                     WHERE rps2.fid_project = rps.fid_project
                   ) = COUNT(rus.fid_skill)
                     OVER (PARTITION BY rus.fid_user, rps.fid_project)
               THEN 'ok' 
               ELSE 'no' 
           END AS result
    FROM rel_user_skill rus
    JOIN rel_project_skill rps ON rus.fid_skill = rps.fid_skill
    JOIN projects p ON rps.fid_project = p.id
    JOIN users u ON rus.fid_user = u.id
)
SELECT DISTINCT user_name,
STRING_AGG(DISTINCT project_name, ', ') AS projects
FROM ProjectSkillCheck psc
WHERE result = 'ok'
GROUP BY user_name;
```

