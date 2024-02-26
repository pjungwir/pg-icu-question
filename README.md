Trying to answer Manoj's question from Feb 2024: Why can/must you give `--locale` to initdb when you also give `--locale-provider=icu` and `--icu-locale=something`.

# Must give it in 15?

Laurenz says you **must** give `--locale` too (for pg 15):

https://stackoverflow.com/questions/74851888/can-i-create-database-with-icu-collation-in-postges-in-2022

But it's not true to say, "you have to specify a C library locale as well". The questioner's error was giving an ICU value to the `--locale` option,
instead of either a libc value to the `--locale` option or an ICU value to the `--icu-locale` option.

But Laurenz is right that the libc locale still matters (a little) even when using an ICU cluster.

Peter elaborates:

https://peter.eisentraut.org/blog/2022/09/26/icu-features-in-postgresql-15

Seems like it's mostly about extensions and the core need is very small.

# Still needed in 16?

Not shown in these docs:

https://www.postgresql.org/docs/current/locale.html#LOCALE-PROVIDERS

> Here is an example to initialize a database cluster using the ICU provider:
>
> > initdb --locale-provider=icu --icu-locale=en

But this sounds like things didn't change in 16:

https://www.postgresql.org/docs/current/app-initdb.html

> Alternatively, initdb can use the ICU library to provide locale services by specifying --locale-provider=icu. The server must be built with ICU support. To choose the specific ICU locale ID to apply, use the option --icu-locale. Note that for implementation reasons and to support legacy code, initdb will still select and initialize libc locale settings when the ICU locale provider is used.

This won't change in 17 either according to [the devel docs](https://www.postgresql.org/docs/devel/app-initdb.html).

# More

Joe Conway on ICU:

https://www.youtube.com/watch?v=0E6O-V8Jato

Btw he shows how to use a custom ICU collation that seems better than citext.

# Tests

## setup

```
vagrant up
```

## pg15

```
# Omitting `--locale` fails (and uses the default libc locale).
/usr/lib/postgresql/15/bin/initdb --locale-provider=icu --icu-locale=und-x-icu -E utf8 /var/lib/postgresql/15/onlyicu
# start it with the shown command, then run psql. . . .

# settings:
postgres=# \l+ postgres
                                                                             List of databases
   Name   |  Owner   | Encoding | Collate |  Ctype  | ICU Locale | Locale Provider | Access privileges |  Size   | Tablespace |                Description                 
----------+----------+----------+---------+---------+------------+-----------------+-------------------+---------+------------+--------------------------------------------
 postgres | postgres | UTF8     | C.UTF-8 | C.UTF-8 | und-x-icu  | icu             |                   | 7453 kB | pg_default | default administrative connection database
(1 row)


# Giving an ICU name for `--locale` fails (which seems expected):
/usr/lib/postgresql/15/bin/initdb --locale-provider=icu --icu-locale=und-x-icu --locale=und-x-icu -E utf8 /var/lib/postgresql/15/both

# Giving a libc locale name for `--locale` works:
/usr/lib/postgresql/15/bin/initdb --locale-provider=icu --icu-locale=und-x-icu --locale=en_IN.UTF8 -E utf8 /var/lib/postgresql/15/both
# start it with the shown command, then run psql. . . .

# settings:
postgres=# \l+ postgres
                                                                                List of databases
   Name   |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider | Access privileges |  Size   | Tablespace |                Description
----------+----------+----------+------------+------------+------------+-----------------+-------------------+---------+------------+--------------------------------------------
 postgres | postgres | UTF8     | en_IN.UTF8 | en_IN.UTF8 | und-x-icu  | icu             |                   | 7453 kB | pg_default | default administrative connection database
(1 row)
```

## pg16

```
# Omitting `--locale` fails (and uses the default libc locale).
/usr/lib/postgresql/16/bin/initdb --locale-provider=icu --icu-locale=und-x-icu -E utf8 /var/lib/postgresql/16/onlyicu
# start it with the shown command, then run psql. . . .

# settings:
postgres=# \l+ postgres
                                                                                   List of databases
   Name   |  Owner   | Encoding | Locale Provider | Collate |  Ctype  | ICU Locale | ICU Rules | Access privileges |  Size   | Tablespace |                Description
----------+----------+----------+-----------------+---------+---------+------------+-----------+-------------------+---------+------------+--------------------------------------------
 postgres | postgres | UTF8     | icu             | C.UTF-8 | C.UTF-8 | und-x-icu  |           |                   | 7492 kB | pg_default | default administrative connection database
(1 row)

# Giving an ICU name for `--locale` fails (which seems expected):
/usr/lib/postgresql/16/bin/initdb --locale-provider=icu --icu-locale=und-x-icu --locale=und-x-icu -E utf8 /var/lib/postgresql/16/both

# Giving a libc locale name for `--locale` works:

# start it with the shown command, then run psql. . . .
/usr/lib/postgresql/16/bin/initdb --locale-provider=icu --icu-locale=und-x-icu --locale=en_IN.UTF8 -E utf8 /var/lib/postgresql/16/both
# settings:

postgres=# \l+ postgres
                                                                                      List of databases
   Name   |  Owner   | Encoding | Locale Provider |  Collate   |   Ctype    | ICU Locale | ICU Rules | Access privileges |  Size   | Tablespace |                Description
----------+----------+----------+-----------------+------------+------------+------------+-----------+-------------------+---------+------------+--------------------------------------------
 postgres | postgres | UTF8     | icu             | en_IN.UTF8 | en_IN.UTF8 | und-x-icu  |           |                   | 7492 kB | pg_default | default administrative connection database
(1 row)
```
