# Лабораторная работа №4

**Тема:** Анализ набора данных с помощью SQL  
**Предметная область примера:** система образовательной платформы  
**СУБД:** SQLite  
**Файл базы:** `lab04_example_bike_rentals_expanded.sqlite`

## Шаг 1. Загрузка базы и первичная проверка

В SandboxSQL загружена SQLite-база. Выполнена проверка числа строк во всех шести таблицах.

**Количество записей.** 

```sql
SELECT 'groups' AS table_name, COUNT(*) AS rows_count FROM groups UNION ALL SELECT 'students', COUNT(*) FROM students UNION ALL SELECT 'teachers', COUNT(*) FROM teachers UNION ALL SELECT 'courses', COUNT(*) FROM courses UNION ALL SELECT 'enrollments', COUNT(*) FROM enrollments UNION ALL SELECT 'results', COUNT(*) FROM results;
```

**Результат:**

![Шаг 1 — количество записей](images/step01_counts.png)

**Вывод:** все шесть таблиц доступны, всего 2 147 записей.

## Шаг 2. Изучение структуры и формулирование аналитических вопросов

База содержит шесть связанных таблиц:

| Таблица | Назначение | Основные поля | Количество записей |
|---|---|---|---:|
| `groups` | Учебные группы | `id`, `group_name`, `study_year` | 12 |
| `students` | Студенты | `id`, `full_name`, `group_id`, `admission_year` | 240 |
| `teachers` | Преподаватели | `id`, `full_name`, `department` | 15 |
| `courses` | Курсы | `id`, `course_name`, `hours`, `teacher_id` | 30 |
| `enrollments` | Записи студентов на курсы | `id`, `student_id`, `course_id`, `enrolled_at`, `status` | 1000 |
| `results` | Итоговые оценки завершённых записей | `id`, `enrollment_id`, `score`, `completed_at` | 850 |

**Связи:** `enrollments.course_id → courses.id`; `courses.teacher_id → teachers.id`; `enrollment.student_id → students.id`; `students.group_id → groups.id`; `results.enrollment_id → enrollments.id`. В каждой таблице `id` — первичный ключ.

**Просмотр данных.** Просмотр первых пяти аренд.

```sql
SELECT * FROM rentals LIMIT 5;
```

**Результат:**

![Шаг 2 — просмотр данных](images/step02_preview.png)

**Структура таблицы.** Просмотр полей и типов данных таблицы `enrollments`.

```sql
PRAGMA table_info(enrollments);
```

**Результат:**

![Шаг 2 — структура таблицы](images/step02_schema.png)

**Первичные и внешние ключи. Связи между таблицами**

В базе данных используются первичные ключи (`PRIMARY KEY`) для идентификации записей и внешние ключи (`FOREIGN KEY`) для связи таблиц.

- `groups.id` — первичный ключ группы.
- `students.id` — первичный ключ студента.
- `teachers.id` — первичный ключ преподавателя.
- `courses.id` — первичный ключ курса.
- `enrollments.id` — первичный ключ прохождения курса.
- `results.id` — первичный ключ записи о колличестве баллов за курс.

**Связи между таблицами:**

- `enrollments.course_id → courses.id` — каждое окончание курса связано с конкретным курсом.
- `enrollment.student_id → students.id` — окончание курса связано с соответствующим студентом.
- `courses.teacher_id → teachers.id` — каждый курс связан с конкретным преподавателем.
- `student.group_id → groups.id` — запись о студенте связана с группой.
- `result.enrollment_id → enrollments.id` — балл связан с окончанием курса.

Таким образом, таблица `enrollments` связывает студентов и курсы, таблица `courses` даёт ввозможность узнать какой преподаватель ведёт курс, таблица `student` позволяет получить информацию об оплате конкретной аренды. Через таблицу `maintenance` можно изучить историю обслуживания велосипедов.

**Сформулированные аналитические вопросы:**

1. Какие студенты имеют от 65 баллов за курс? (`results`)
2. Сколько всего курсов завершено? (`enrollments`)
3. Как соотносятся курсы со студентами и преподавателями? (`courses`, `students`, `teachers`)
4. Студенты какой группы получили максимальный общий балл? (`groups`, `results`, `enrollments`)
5. Какие курсы связаны с наибольшим общим баллом? (`courses`, `results`, `students`)

## Шаг 3. Фильтрация и сортировка

**Фильтрация по баллу.** Какие студенты имеют бал не менее 65?

```sql
SELECT id, enrollment_id, score, completed_at FROM results WHERE score >= 90 ORDER BY score DESC, enrollment_id LIMIT 200
```

**Результат:**

![Шаг 3 — фильтрация по баллу](images/step03_filter.png)

**Сортировка студентов.** Выводим студентов из группы ИВТ-11 по алфавиту.

```sql
SELECT id, full_name, group_id, admission_year FROM students WHERE group_id = '1' ORDER BY full_name LIMIT 10;
```

**Результат:**

![Шаг 3 — сортировка студентов](images/step03_sort.png)

**Вывод:** `WHERE` отбирает нужные строки, `ORDER BY` сортирует, `LIMIT` ограничивает объём вывода.

## Шаг 4. Агрегирование и группировка

**Подсчёт окончаний курсов.** Определяем общее число окончаний курсов.

```sql
SELECT COUNT(*) AS status FROM enrollments WHERE status = 'completed';
```

**Результат:**

![Шаг 4 — Подсчёт окончаний курсов](images/step04_count.png)

**Агрегатные функции.** Рассчитываем минимальный, максимальный, средний и суммарный балл.

```sql
SELECT MIN(score) AS min_score, MAX(score) AS max_score, ROUND(AVG(score),2) AS avg_score, SUM(score) AS sum_scores FROM results;
```

**Результат:**

![Шаг 4 — агрегатные функции](images/step04_aggregates.png)

**GROUP BY и HAVING.** Группируем курсы по статусу и оставляем статусы, встречающиеся более 40 раз.

```sql
SELECT status, COUNT(*) AS enrollments_count FROM enrollments GROUP BY status HAVING COUNT(*) > 40 ORDER BY enrollments_count DESC;
```

**Результат:**

![Шаг 4 — GROUP BY и HAVING](images/step04_having.png)

**Вывод:** в базе 1000 статусов курсов. `HAVING` отбирает группы после агрегирования; условие применяется к `COUNT(*)`.

## Шаг 5. Объединение связанных таблиц

**Студенты и группы.** Используем `INNER JOIN`, чтобы показать первых 10 студентов вместе с группой.

```sql
SELECT r.id AS student_id, c.group_name, r.full_name, r.group_id FROM students r INNER JOIN groups c ON r.group_id=c.id ORDER BY r.id LIMIT 10;
```

**Результат:**

![Шаг 5 — студенты и группы](images/step05_join_students.png)

## Шаг 6. Проверка корректности данных

**Проверка колличества баллов.** Проверяем, нет ли студентов с отрицательным результатом.

```sql
SELECT COUNT(*) AS invalid_score FROM results WHERE score < 0;
```

**Результат:**

![Шаг 6 — проверка колличества баллов](images/step06_invalid.png)

**Проверка связей оценок.** Проверяем, нет ли оценок, относящихся к несуществующим студентам.

```sql
SELECT COUNT(*) AS orphan_score FROM results p LEFT JOIN enrollments r ON p.enrollment_id=r.id WHERE r.id IS NULL;
```

**Результат:**

![Шаг 6 — проверка связей оценок](images/step06_orphans.png)

**Вывод:** оба запроса вернули 0. В рамках проведённых проверок нарушений не обнаружено.
## Шаг 7. Индивидуальное аналитическое исследование

**Вариант:** 4

**Исследовательский вопрос:** Какие студенты имеют самый высокий средний балл среди студентов минимум с тремя итоговыми оценками?

**Используемые таблицы:** `students`, `results`, `enrollments`.

**Метод исследования:** с помощью `INNER JOIN`, `GROUP BY`, `COUNT`, `HAVING COUNT`, `ORDER BY` и `AVG` определяем студентов с тремя оценками, находим среднюю оценку для каждого, находим наибольшие 5 значений.

**Решение.** Группируем оплаченные аренды по категориям велосипедов.

```sql
SELECT s.id, s.full_name, AVG(r.score) AS avg_score, COUNT(r.id) AS grades_count
FROM students s
INNER JOIN enrollments e ON e.student_id = s.id
INNER JOIN results r     ON r.enrollment_id = e.id
GROUP BY s.id, s.full_name
HAVING COUNT(r.id) >= 3
ORDER BY avg_score DESC
LIMIT 5
**Результат:**

![Шаг 7 — решение](images/step07.png)

**Аналитические выводы:**

1. Наибольший средний балл имеют Кузнецов Ольга, Кузнецов Мария, Иванов Виктория, Сидоров Павел, Сидоров Кирилл.
2. Наибольший средний балл — 93.(6) 
## Итог

В работе изучена структура SQLite-базы из шести связанных таблиц, выполнены запросы с `WHERE`, `ORDER BY`, `LIMIT`, агрегатными функциями, `GROUP BY`, `HAVING`, `INNER JOIN`, `LEFT JOIN`; проведены проверки целостности и отдельное аналитическое исследование.