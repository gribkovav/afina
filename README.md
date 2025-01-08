# Afina
Exercise in creating a relational database ready to be embedded into .NET process (SQLite like)

I once thought about the fact that on the one hand having a database in most even small projects is more often justified than not - on the other hand using a DBMS leads to unnecessary dependencies and infrastructure deployment (yes, it's fast with docker nowadays), which in principle you could do without.

Among other things, I missed the classical course on DBMS construction, as I was studying a speciality more related to hardware (solid state physics, semiconductors, chip design and simulations and all this stuff). It would be an interesting exercise, I thought, to make my own database. They say that there is no better way to understand something than to try to explain it to someone else. In the case of programming, there's no better way to understand something than to try to write it. :)

After that, I opened my sketchbook and sketched out some uncomplicated requirements:
1. Ability to build into the .NET process and the engine itself written in C# 
2. Availability of providers under EF and Dapper
3. Implementation of the basic pgsql syntax - the subset will definitely not be complete, but the point is to be able to perform classic CRUD operations in their entirety
4. ACID - transactionality
5. Optional ability to listen to the port and have binary compatibility on the interface with tools that are ready to interact with postgresql

Here and later in other md-files I'm going to log my thoughts and progress on the subject.

## High level design
Given the requirements I decided to go from the top level this time, gradually going down in description. So we are talking about the application layer using EF and Dapper, there will certainly be its own code with providers that will eventually operate on System.Data interfaces such as IDbConnection, IDbCommand, IDbTransaction and others. This will be the layer following the application layer - let's call it the interface layer. After that we should organise the work with queries directly and here we can't get away from several things that we can think about at once - firstly, a lexer and a parser for building a syntax tree, secondly, this syntax tree will need to be traversed with certain purposes - that's why it's better to make an abstract visitor on the basis of which to make a scheduler. The query cache and the data cache should be around the same place. And the level below should be used to organise work directly with disks.

## Plan (ongoing)
* Getting hands on "Database internals" by Alex Petrov
* Create solution and upload it here
* Defining supported column types
* Defining physical structure of files

##  Базовые типы данных для Afina
### Строковые типы (Text):

* VARCHAR(n): Строка переменной длины с ограничением n (например, VARCHAR(255)).
* CHAR(n): Строка фиксированной длины. Удобно для полей, где длина всегда фиксирована (например, коды страны).
* TEXT: Строка переменной длины без ограничения. Подходит для хранения больших текстов.
### Целочисленные типы (Integer):

* INT: Соответствует int в C# (-2,147,483,648 до 2,147,483,647).
* BIGINT: Соответствует long в C# (-9,223,372,036,854,775,808 до 9,223,372,036,854,775,807).
* (Опционально) SMALLINT: Для экономии пространства (-32,768 до 32,767).

### Десятичные числа (Decimal):

* DECIMAL(p, s): Число с фиксированной точностью и масштабом (p — количество цифр, s — число знаков после запятой).
* Соответствует decimal в C#.
* Используется для финансовых расчётов или любых других случаев, где важна точность.

### Двоичные данные (Binary):

* VARBINARY(n): Бинарные данные переменной длины, полезно для хранения небольших блобов (например, хэшей).
* BLOB (Binary Large Object): Для хранения больших двоичных данных (например, изображений, файлов).

### Дополнительные типы:

* BOOLEAN: Логическое значение (true/false). Соответствует bool в C#.
* FLOAT / REAL: Число с плавающей точкой.  Соответствует float (32 бита) или double (64 бита).  Используется, если не требуется высокая точность, например, для научных расчётов.
* DATE: Дата (yyyy-MM-dd).
* DATETIME: Дата и время (yyyy-MM-dd HH:mm:ss).

### Итоговый набор типов для Afina
| **Тип**         | **Описание**                           | **Соответствие C#**       |
|------------------|----------------------------------------|---------------------------|
| `VARCHAR(n)`     | Строка переменной длины               | `string`                 |
| `CHAR(n)`        | Строка фиксированной длины            | `string` (с padding)     |
| `TEXT`           | Большая строка (переменной длины)     | `string`                 |
| `INT`            | Целое число (32 бита)                | `int`                    |
| `BIGINT`         | Целое число (64 бита)                | `long`                   |
| `SMALLINT`       | Маленькое целое число (16 бит)       | `short` (опционально)    |
| `DECIMAL(p,s)`   | Десятичное число                     | `decimal`                |
| `VARBINARY(n)`   | Двоичные данные переменной длины с ограничением n | `byte[]` |
| `VARBINARY(max)` | Двоичные данные переменной длины без ограничения | `byte[]` |
| `BOOLEAN`        | Логическое значение                  | `bool`                   |
| `FLOAT`          | Число с плавающей точкой (32 бита)   | `float`                  |
| `DOUBLE`         | Число с плавающей точкой (64 бита)   | `double`                 |
| `DATE`           | Дата                                 | `DateTime`               |
| `DATETIME`       | Дата и время                         | `DateTime`               |
| `DATETIMEOFFSET` | Дата, время и часовой пояс           | `DateTimeOffset`         |


