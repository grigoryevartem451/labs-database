# Лабораторная работа 1. Начало работы с MySQL. MySQL Workbench
## Задание 1. Назначение разделов Instance и Performance
Раздел «Instance» («Экземпляр БД»)
Startup/Shutdown - управление запуском и остановкой MySQL-сервера, просмотр текущего статуса, настройка параметров запуска.

Server Logs - просмотр логов сервера: error log, general log, slow query log. Позволяет диагностировать ошибки и анализировать медленные запросы.

Options File - просмотр и редактирование файла конфигурации MySQL (my.ini / my.cnf). Здесь можно менять параметры сервера.

Раздел «Performance» («Производительность»)
Dashboard - визуальная панель с графиками и метриками производительности сервера в реальном времени.

Performance Reports - отчёты по производительности: какие запросы выполняются, какие ожидания преобладают, где узкие места.

Performance Schema Setup - настройка Performance Schema: включение/выключение инструментов сбора статистики, выбор потребителей и инструментов.

## Задание 2. Создание БД simpledb
Я работал используя команды

```
CREATE DATABASE `simpledb`
  CHARACTER SET utf8
  COLLATE utf8_general_ci;

USE `simpledb`;
```

## Задание 3. Создание таблицы users
```
CREATE TABLE `simpledb`.`users` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(45) NOT NULL,
  `email` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE INDEX `email_UNIQUE` (`email` ASC)
);
```
## Задание 4. Добавление записей вручную
```
INSERT INTO `simpledb`.`users` (`name`, `email`) VALUES
('Paul', 'paul@superpochta.ru'),
('Marina', 'marina.ivanova@hotmail.com'),
('Ekaterina', 'ekaterina.petrova@outlook.com');
```
## Задание 5. Изменение таблицы users
Итоговая структура:

```
CREATE TABLE `simpledb`.`users` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(50) NOT NULL,
  `email` VARCHAR(45) NOT NULL,
  `gender` ENUM('M','F') DEFAULT NULL,
  `bday` DATE DEFAULT NULL,
  `postal_code` VARCHAR(10) DEFAULT NULL,
  `rating` FLOAT DEFAULT NULL,
  `created` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP(),
  PRIMARY KEY (`id`),
  UNIQUE INDEX `email_UNIQUE` (`email` ASC)
);
```

### Пояснения
ENUM - тип данных, который допускает только одно значение из заданного списка. В нашем случае gender может быть только 'M' или 'F' (или NULL, если разрешён).

TIMESTAMP DEFAULT CURRENT_TIMESTAMP() - если при вставке не указать значение поля created, MySQL автоматически подставит текущие дату и время. CURRENT_TIMESTAMP() — синоним CURRENT_TIMESTAMP. Это значение по умолчанию, а не автоматическое обновление при изменении строки.

Какие поля могут быть NULL:
gender, bday, postal_code, rating - могут быть NULL, потому что пользователь может не захотеть делиться этой информацией.
name, email - логично оставить NOT NULL, так как это основные идентифицирующие данные.
created - NOT NULL DEFAULT CURRENT_TIMESTAMP().
id - NOT NULL AUTO_INCREMENT, первичный ключ.

### Итоговая таблица
<img width="919" height="207" alt="image" src="https://github.com/user-attachments/assets/3134d178-0e3c-497a-9b2b-23c21b0f1246" />


### Задание 6. Добавление данных двумя способами
Вручную через Result Grid / Form Editor.

SQL-запросы:

```
INSERT INTO `simpledb`.`users`
(`name`, `email`, `postal_code`, `gender`, `bday`, `rating`)
VALUES
('Ekaterina', 'ekaterina.petrova@outlook.com', '145789', 'F', '2000-02-11', 1.123);

INSERT INTO `simpledb`.`users`
(`name`, `email`, `postal_code`, `gender`, `bday`, `rating`)
VALUES
('Paul', 'paul@superpochta.ru', '123789', 'M', '1998-08-12', 1);
```
## Задание 7. Экспорт recordset в SQL-файл

```
INSERT INTO `simpledb`.`users`
(`id`, `name`, `email`, `gender`, `bday`, `postal_code`, `rating`, `created`)
VALUES
(1, 'Paul', 'paul@superpochta.ru', 'M', '1998-08-12', '123789', 1, '2021-02-10 22:28:00'),
(2, 'Marina', 'marina.ivanova@hotmail.com', 'F', '2000-02-11', '124759', 1.123, '2021-02-10 22:32:42'),
(3, 'Ekaterina', 'ekaterina.petrova@outlook.com', 'F', '2000-02-11', '145789', 1.123, '2021-02-10 22:36:44');
```
### Синтаксис: INSERT INTO имя_таблицы (список_столбцов) VALUES (значения)



## Задание 8. Создание таблицы resume и внешнего ключа
```
CREATE TABLE `simpledb`.`resume` (
  `resumeid` INT NOT NULL AUTO_INCREMENT,
  `userid` INT NOT NULL,
  `title` VARCHAR(100) NOT NULL,
  `skills` TEXT DEFAULT NULL,
  `created` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP(),
  PRIMARY KEY (`resumeid`),
  INDEX `fk_resume_users_idx` (`userid` ASC),
  CONSTRAINT `fk_resume_users`
    FOREIGN KEY (`userid`)
    REFERENCES `simpledb`.`users` (`id`)
    ON DELETE CASCADE
    ON UPDATE CASCADE
);
```
### Поведение СУБД
ON DELETE CASCADE - при удалении пользователя из users все его резюме из resume удаляются автоматически.

ON UPDATE CASCADE - при изменении id пользователя в users значение userid в resume автоматически обновится.

## Задание 9. Наполнение resume
```
sql
INSERT INTO `simpledb`.`resume` (`userid`, `title`, `skills`) VALUES
(1, 'Python Developer', 'Python, SQL, Git, Docker'),
(1, 'Data Analyst', 'SQL, Excel, Python, Tableau'),
(2, 'QA Engineer', 'Testing, Selenium, SQL'),
(3, 'Backend Developer', 'Java, Spring, MySQL');
```
Сколько резюме может быть у одного пользователя?
Минимум - 0. Максимум - не ограничен, потому что на userid нет уникального ограничения. Связь «один ко многим»: один пользователь - много резюме.

Попытка добавить резюме с несуществующим userid:

```
INSERT INTO `simpledb`.`resume` (`userid`, `title`, `skills`)
VALUES (999, 'Test Resume', 'Test');
Результат: ошибка внешнего ключа:
```

text
```
ERROR 1452 (23000): Cannot add or update a child row:
a foreign key constraint fails
(`simpledb`.`resume`, CONSTRAINT `fk_resume_users`
FOREIGN KEY (`userid`) REFERENCES `users` (`id`)
ON DELETE CASCADE ON UPDATE CASCADE)
```
Добавить такую запись нельзя, пока в users нет пользователя с id = 999.
<img width="1614" height="103" alt="image" src="https://github.com/user-attachments/assets/bacfd5a7-86d8-4040-8442-653be7f96495" />

### Итоговая таблица
<img width="756" height="203" alt="image" src="https://github.com/user-attachments/assets/c5c9431a-767a-412a-86cd-c8f8551bb633" />



## Задание 10. Удаление и изменение пользователей
Удаление пользователя, у которого есть резюме:

```
DELETE FROM `simpledb`.`users` WHERE `id` = 1;
```
Благодаря ON DELETE CASCADE все резюме с userid = 1 из таблицы resume тоже удалятся.
<img width="1616" height="588" alt="image" src="https://github.com/user-attachments/assets/2c7af331-b20c-434d-858a-4f4bbdef4849" />

Изменение id пользователя:
<img width="1343" height="262" alt="image" src="https://github.com/user-attachments/assets/f3992cb2-6328-4959-aaa1-170b9aec2ac4" />

```
UPDATE `simpledb`.`users` SET `id` = 10 WHERE `id` = 2;
```
Благодаря ON UPDATE CASCADE в таблице resume у связанных записей userid автоматически станет 10.

Если бы каскадов не было, при удалении или изменении родительской записи MySQL выдал бы ошибку внешнего ключа, либо установил бы NULL, если столбец допускает NULL.


