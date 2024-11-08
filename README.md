# Домашнее задание к занятию "Резервное копирование баз данных" - `Александра Бужор`

---

### Задание 1

Задание 1. Резервное копирование

Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования.

Необходимо описать, какие варианты резервного копирования подходят в случаях:

1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.

1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.

1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.

Приведите ответ в свободной форме.

### Решение:

Разберем кейсы финансовой компании:

1.1. Если нужно восстанавливать данные за предыдущий день и, соответственно более ранний период, то подойдёт полное резервное копирование, выполненное, скажем, в конце рабочего дня, или в конце суток. Это значит, что каждый день будет создаваться полная копия базы данных, которую можно использовать для восстановления на случай сбоя.

1.2. Для восстановления данных за час до поломки хорошо подходят инкрементные или дифференциальные резервные копии. Инкрементные копии фиксируют изменения с момента последнего копирования (полного или инкрементного), а дифференциальные - все изменения с момента последнего полного копирования. Это позволит более гибко восстанавливать данные, минимизируя потери и не создавая высокую нагрузку на БД, как было бы при создании полной копии.

1.3.* Да, такой кейс возможен и называется высокодоступной (High Availability, HA) конфигурацией. В этом случае используется репликация данных между основной и резервной базами данных в реальном времени. При сбое основной базы происходит автоматическое переключение на резервную, что минимизирует простои в работе. Но есть минусы - это дороже по ресурсам, из-за необходимости поддерживать вторую активную копию базы данных, но зато это обеспечивает максимальную надежность.

---

### Задание 2

PostgreSQL

2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).

2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?

Приведите ответ в свободной форме.

### Решение:

### 2.1 Резервное копирование и восстановление данных с pg_dump / pgrestore

Копирование
```bash
pg_dump -U username -W -F t database_name > backup_file.tar

-U username указывает пользователя базы данных.
-W требует ввода пароля.
-F t говорит, что формат выходного файла должен быть tar.
database_name это имя базы данных, которую ты хочешь сохранить.
backup_file.tar это имя файла резервной копии.
```

Восстановление
```bash
pg_restore -U username -W -d database_name backup_file.tar

-U username и -W аналогично указывают пользователя и требуют пароль.
-d database_name указывает имя базы данных, куда будет восстановлена копия.
backup_file.tar это файл резервной копии.
```

### 2.1* Автоматизация процесса

Да, автоматизировать процесс резервного копирования и восстановления возможно. Как пример, вот несколько способов, которыми можно это сделать:

- **Cron Jobs**: Можно настроить задачу в cron, чтобы автоматически выполнять pg_dump в запланированное время. Пример записи в crontab для создания резервной копии каждый день в 3 утра:

```bash
0 3 * * * pg_dump -U username -W -F t database_name > /backup_dir/backup_file_$(date +\%d-\%m-\%Y).tar
```

- **Скрипты**: Можно написать скрипт на bash или Python, который будет выполнять резервное копирование и/или восстановление, и запускать его по расписанию через cron или другой планировщик задач.

---

### Задание 3 *

MySQL

3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL.

3.1.* В каких случаях использование реплики будет давать преимущество по сравнению с обычным резервным копированием?

Приведите ответ в свободной форме.

### Решение:

### 3.1 Инкрементное резервное копирование в MySQL

Для MySQL ситуация с инкрементным резервным копированием отличается от PostgreSQL, так как MySQL не имеет встроенного инструмента, аналогичного pg_dump, для выполнения инкрементных резервных копий напрямую без использования сторонних решений. 
Рассмотрим следующие продукты MySQL:
- MySQL Enterprise Edition
- MySQL Community Edition

#### MySQL Enterprise Edition

MySQL Enterprise Backup (часть MySQL Enterprise Edition) поддерживает инкрементное резервное копирование.

Пример команды для создания инкрементного резервного копирования:

```bash
mysqlbackup --user=admin --password=admin_password --backup-dir=/path/to/full/backup/dir --backup-image=/path/to/incremental/backup_image_file_ --with-timestamp backup-to-image

--user и --password указывают реквизиты пользователя для доступа к базе данных.
--backup-dir указывает на директорию, где хранится последнее полное резервное копирование.
--backup-image указывает путь и имя файла образа для инкрементного резервного копирования.
--with-timestamp добавляет метку времени к имени директории резервного копирования, что удобно для идентификации и организации резервных копий.
backup-to-image говорит инструменту создать резервное копирование в виде образа (файла)
```

Для восстановления из инкрементного резервного копирования, сначала требуется восстановить БД  из последнего полного резервного копирования, а затем применить все инкрементные копии до нужного момента времени.

#### MySQL Community Edition

Для MySQL Community Edition один из подходов к инкрементному резервному копированию - это использование бинарных логов MySQL, для восстановления базы данных до определенного момента времени. Но этот метод требует, чтобы бинарное логирование было включено в инстансе MySQL.

Чтобы включить бинарные логи, необходимо добавить следующие строки в конфигурационный файл my.cnf:
```ini
[mysqld]
log_bin=mysql-bin
```

После включения и настройки бинарного логирования, MySQL будет записывать все изменения в базе данных, позволяя выполнить точное восстановление до любого момента времени, используя эти логи.

Восстановление с использованием бинарных логов включает два шага:
- Восстановление из последней полной резервной копии.
- Применение бинарных логов с момента создания этой резервной копии до нужного момента времени.

Пример команды для применения бинарных логов начиная с определённого файла:
```bash
mysqlbinlog mysql-bin.000001 mysql-bin.000002 | mysql -u root -p
```

Это позволит "проиграть" изменения, произошедшие после создания последней полной копии.

### 3.1* Использование реплики

Использование реплики предлагает несколько преимуществ по сравнению с обычным резервным копированием:
- Высокая доступность: Как и в рассматриваемом ранее случае с PostgreSQL Реплика может автоматически заменить основную базу данных в случае её сбоя, минимизируя простои.
- Распределение нагрузки: Запросы на чтение могут быть перенаправлены на реплику, уменьшая нагрузку на основной сервер.
- Безопасность: Репликация позволяет проводить резервное копирование с реплики, не влияя на производительность основной базы данных.
- Точка восстановления: Реплики могут использоваться для восстановления данных на определенный момент времени, если они настроены на хранение бинарных логов.

---
