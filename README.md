<div align="center">

<a href="https://nodejs.org/en" target="_blank"><img src="https://user-images.githubusercontent.com/3438503/260495760-ca5d8c24-b382-43d9-8fbe-d905b4b208f0.png" width="260" alt="Logo de Node.js" /></a>

# Clase 7 de NodeJS

### Kopius · The Academy · Cohort 05 · M3

**Persistencia real: bases de datos relacionales, SQL, PostgreSQL, Sequelize y DTO**

![Node.js](https://img.shields.io/badge/Node.js-26.x_LTS-5FA04E?style=for-the-badge&logo=nodedotjs&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-ES_Modules-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16.15-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Sequelize](https://img.shields.io/badge/Sequelize-6-52B0E7?style=for-the-badge&logo=sequelize&logoColor=white)

</div>

---

## CONTENIDO

1. [Bases de datos relacionales](#1-bases-de-datos-relacionales-fundamentos)
   - [Qué es una base de datos relacional](#11-qué-es-una-base-de-datos-relacional)
   - [Tabla, fila y columna](#12-tabla-fila-y-columna)
   - [Claves y relaciones](#13-claves-y-relaciones)
   - [Restricciones](#14-restricciones)
   - [Integridad y normalización](#15-integridad-y-normalización)
   - [ACID y transacciones](#16-acid-y-transacciones)
   - [Por qué PostgreSQL](#17-por-qué-postgresql)
2. [SQL: resumen de instrucciones](#2-sql-resumen-de-instrucciones)
   - [Los cinco grupos](#21-los-cinco-grupos)
   - [DDL](#22-ddl)
   - [DML](#23-dml)
   - [SELECT](#24-select)
   - [JOIN](#25-join)
   - [Agregaciones](#26-agregaciones)
   - [Índices](#27-índices)
   - [Transacciones](#28-transacciones)
   - [Trampas clásicas](#29-trampas-clásicas)
3. [PostgreSQL en Docker y TablePlus](#3-postgresql-en-docker-y-tableplus)
   - [Por qué Docker](#31-por-qué-docker)
   - [Instalar la imagen](#32-instalar-la-imagen)
   - [El archivo compose](#33-el-archivo-compose)
   - [Levantar la base](#34-levantar-la-base)
   - [Conectar TablePlus](#35-conectar-tableplus)
   - [Crear la tabla users](#36-crear-la-tabla-users)
   - [Problemas frecuentes](#37-problemas-frecuentes)
4. [ORM: Sequelize](#4-orm-sequelize)
   - [Qué es un ORM y para qué sirve](#41-qué-es-un-orm-y-para-qué-sirve)
   - [Instalación](#42-instalación)
   - [Variables de entorno](#43-variables-de-entorno)
   - [Archivo de conexión](#44-archivo-de-conexión)
   - [Arranque con fail fast](#45-arranque-con-fail-fast)
5. [Modelo User con Sequelize](#5-modelo-user-con-sequelize)
   - [Del SQL al modelo](#51-del-sql-al-modelo)
   - [El modelo](#52-el-modelo)
   - [Operaciones básicas](#53-operaciones-básicas)
   - [Modificar el esquema: agregar un campo](#54-modificar-el-esquema-agregar-un-campo)
   - [Validación de modelo vs DTO](#55-validación-de-modelo-vs-dto)
   - [Errores de Sequelize](#56-errores-de-sequelize)
6. [DTO: fundamentos](#6-dto-fundamentos)
   - [Qué es un DTO](#61-qué-es-un-dto)
   - [Por qué existe](#62-por-qué-existe)
   - [DTO de entrada con Yup](#63-dto-de-entrada-con-yup)
   - [Middleware de validación](#64-middleware-de-validación)
   - [DTO de salida](#65-dto-de-salida)
   - [DTO, modelo y entidad](#66-dto-modelo-y-entidad)
7. [Tarea 013](#7-tarea-013-patrón-clásico)
   - [Objetivo](#71-objetivo)
   - [Estructura](#72-estructura)
   - [Código completo](#73-código-completo)
   - [Pruebas con curl](#74-pruebas-con-curl)
   - [Criterios de aceptación](#75-criterios-de-aceptación)
   - [Desafío: password](#76-desafío-agregar-password-a-los-usuarios)
   - [Errores frecuentes](#77-errores-frecuentes)
8. [Referencias](#8-referencias)

---

> [!NOTE]
> Esta clase arma una API REST de usuarios donde el estado vive en PostgreSQL. Cada pieza tiene una razón de cambio: la ruta decide el path, el middleware valida, el controller traduce HTTP y el modelo habla con la base.

## 1. Bases de datos relacionales: fundamentos

### 1.1 Qué es una base de datos relacional

Una **base de datos relacional** es un sistema que guarda datos en tablas y **hace cumplir** las reglas que los relacionan. No es un depósito de bytes: es un motor que impone tipos, restricciones y transacciones, y que responde consultas sin que tengas que traer todo a la memoria del proceso.

El modelo se llama _relacional_ porque los datos viven en **relaciones** (tablas) y los vínculos entre ellos se expresan con **claves**, no con punteros ni con objetos anidados. Una fila no "contiene" a otra: la referencia por su clave.

Eso da cuatro garantías que un archivo de texto o una variable no pueden dar:

| Garantía         | Quién la hace cumplir                            | Con qué                                             |
| ---------------- | ------------------------------------------------ | --------------------------------------------------- |
| **Persistencia** | El motor escribe en disco de forma transaccional | Los datos sobreviven al proceso                     |
| **Integridad**   | El motor rechaza lo inválido                     | Tipos, `NOT NULL`, `UNIQUE`, `FOREIGN KEY`, `CHECK` |
| **Concurrencia** | El motor coordina accesos simultáneos            | Transacciones, aislamiento, bloqueos                |
| **Consultas**    | El motor busca por índices                       | `SELECT`, `WHERE`, `JOIN`, `ORDER BY`               |

La consecuencia práctica: las reglas dejan de depender de que "el código se acuerde". Si dos clientes intentan registrar el mismo email al mismo tiempo, la base rechaza el segundo aunque el controller tenga un bug. Esa es la diferencia entre guardar datos y **administrarlos**.

> [!IMPORTANT]
> Persistir no es lo mismo que guardar un archivo. La base de datos es la que **hace cumplir** las reglas: unicidad, tipos, referencias entre tablas y atomicidad.

### 1.2 Tabla, fila y columna

El modelo relacional organiza todo en tablas, y una tabla es una grilla con tres propiedades:

| Concepto           | Nombre formal    | Ejemplo                               |
| ------------------ | ---------------- | ------------------------------------- |
| Estructura         | tabla / relación | `users`                               |
| Registro           | fila / tupla     | `1 · John Doe · john.doe@example.com` |
| Atributo           | columna / campo  | `email`                               |
| Tipo               | dominio          | `VARCHAR(255)`                        |
| Valor de una celda | dato             | `"john.doe@example.com"`              |
| Conjunto de tablas | esquema          | `public`                              |

La tabla `users` de esta clase:

```text
┌────┬───────────┬─────────────────────────┬─────────────────────┐
│ id │ name      │ email                   │ created_at          │
├────┼───────────┼─────────────────────────┼─────────────────────┤
│  1 │ John Doe  │ john.doe@example.com    │ 2026-11-12 10:00:00 │
│  2 │ Jane Smith│ jane.smith@example.com  │ 2026-11-12 10:01:00 │
└────┴───────────┴─────────────────────────┴─────────────────────┘
```

Cada columna declara un **tipo**, y el motor rechaza lo que no encaja. Si `id` es entero, `'abc'` no entra. Esa es la primera diferencia real con JavaScript: la base de datos no es permisiva.

### 1.3 Claves y relaciones

Toda tabla necesita una forma de identificar una fila sin ambigüedad.

| Clave                | Qué es                                                             | Ejemplo                    |
| -------------------- | ------------------------------------------------------------------ | -------------------------- |
| **Primary key (PK)** | Identifica una fila dentro de su tabla. Única, no nula, inmutable. | `users.id`                 |
| **Foreign key (FK)** | Columna que referencia la PK de otra tabla.                        | `posts.user_id → users.id` |
| **Unique key**       | Valor único, pero puede cambiar.                                   | `users.email`              |

Las relaciones se leen desde la FK, no desde "una propiedad del objeto":

| Relación | Cómo se modela              | Ejemplo                                |
| -------- | --------------------------- | -------------------------------------- |
| 1 : 1    | FK con `UNIQUE` en un lado  | `users` ↔ `profiles.user_id UNIQUE`    |
| 1 : N    | FK en el lado "muchos"      | `users` 1 → N `posts.user_id`          |
| N : M    | Tabla intermedia con dos FK | `students` ↔ `enrollments` ↔ `courses` |

```sql
CREATE TABLE posts (
  id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(160) NOT NULL
);
```

`ON DELETE CASCADE` significa: si borrás el usuario, sus posts se borran con él. Sin eso, el motor **impide** borrar el usuario y te avisa que hay filas que lo referencian. Esa negativa es una función, no un obstáculo.

> [!NOTE]
> Una FK no es una columna decorativa: es una restricción que el motor valida en cada `INSERT` y en cada `DELETE`. Si insertás un `user_id` que no existe, la operación falla.

### 1.4 Restricciones

Las restricciones son reglas que viven en la base y se cumplen siempre, sin importar qué código las toque.

| Restricción   | Garantiza                           | Ejemplo                      |
| ------------- | ----------------------------------- | ---------------------------- |
| `NOT NULL`    | La columna no acepta `NULL`         | `name VARCHAR(120) NOT NULL` |
| `UNIQUE`      | No hay dos filas con el mismo valor | `email VARCHAR(255) UNIQUE`  |
| `PRIMARY KEY` | `UNIQUE` + `NOT NULL`               | `id INTEGER PRIMARY KEY`     |
| `DEFAULT`     | Valor cuando no se envía            | `created_at DEFAULT now()`   |
| `CHECK`       | Condición booleana                  | `CHECK (length(name) >= 2)`  |
| `FOREIGN KEY` | El valor existe en la otra tabla    | `REFERENCES users(id)`       |

```sql
CREATE TABLE users (
  id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name VARCHAR(120) NOT NULL CHECK (length(trim(name)) >= 2),
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

> [!WARNING]
> `NULL` no es `0` ni `""`. `NULL` significa "no hay valor". Por eso se compara con `IS NULL` y `IS NOT NULL`, nunca con `= NULL`: `email = NULL` no devuelve nada, ni siquiera cuando el email es nulo.

### 1.5 Integridad y normalización

La **integridad** es que los datos no se contradigan entre sí. Una parte la pone el motor (tipos, FK, `UNIQUE`) y otra la ponemos nosotros al diseñar.

La **normalización** es el conjunto de reglas para no duplicar información. En la práctica, para este curso, alcanza con tres ideas:

| Forma | Pregunta que responde                                              | Ejemplo de violación                                                                         |
| ----- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| 1NF   | ¿Cada celda tiene un solo valor atómico?                           | `emails = "a@x.com, b@x.com"`                                                                |
| 2NF   | ¿Cada columna depende de la PK completa?                           | En `enrollments(student_id, course_id, student_name)` el nombre depende solo de `student_id` |
| 3NF   | ¿Cada columna depende solo de la PK y no de otra columna no clave? | `users(city, zip_code)` donde el zip determina la ciudad                                     |

No normalices por deporte. Normalizá cuando un dato repetido pueda quedar desincronizado:

```text
Mal:  posts(id, title, author_name, author_email)
Bien: posts(id, title, user_id) + users(id, name, email)
```

Si mañana Jane cambia su email, en el primer diseño hay que actualizar todos sus posts. En el segundo, una sola fila.

### 1.6 ACID y transacciones

Una **transacción** agrupa varias operaciones: o se aplican todas, o ninguna.

| Propiedad        | Significado                   | Ejemplo                                                 |
| ---------------- | ----------------------------- | ------------------------------------------------------- |
| **A**tomicidad   | Todo o nada                   | Transferir plata: no puede debitarse sin acreditarse    |
| **C**onsistencia | Se respetan las reglas        | No queda una FK rota                                    |
| **I**solation    | Las transacciones no se pisan | Dos compras simultáneas no venden el mismo último stock |
| **D**urabilidad  | Lo confirmado sobrevive       | Si responde `COMMIT`, quedó en disco                    |

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

Si algo falla entre las dos sentencias, `ROLLBACK` deja la base como estaba. Ese "o todo o nada" es lo que distingue una base de datos de un archivo de texto.

### 1.7 Por qué PostgreSQL

| Motor           | Cuándo conviene                                                                                         |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| **PostgreSQL**  | Relacional serio, tipos ricos, JSON, `ILIKE`, transacciones robustas, gratis y estándar de la industria |
| MySQL / MariaDB | Muy popular en hosting compartido, ecosistema enorme                                                    |
| SQLite          | Un archivo, cero servidor. Ideal para tests y prototipos locales                                        |
| MongoDB         | Documental, no relacional. Otro modelo mental, otro conjunto de problemas                               |

Usamos PostgreSQL porque es el que vas a encontrar en la mayoría de los equipos backend, porque el SQL que aprendas acá es estándar, y porque Sequelize lo soporta con `pg`.

> [!TIP]
> SQLite es una gran opción para tests automatizados: mismo dialecto casi, sin contenedor. Pero no reemplaza probar contra el motor real antes de entregar.

<sub>[⬆ Volver al índice](#contenido)</sub>

---

## 2. SQL: resumen de instrucciones

### 2.1 Los cinco grupos

| Grupo   | Instrucciones                         | Para qué              |
| ------- | ------------------------------------- | --------------------- |
| **DDL** | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Cambiar la estructura |
| **DML** | `INSERT`, `UPDATE`, `DELETE`          | Cambiar los datos     |
| **DQL** | `SELECT`                              | Consultar             |
| **TCL** | `BEGIN`, `COMMIT`, `ROLLBACK`         | Agrupar operaciones   |
| **DCL** | `GRANT`, `REVOKE`                     | Permisos              |

SQL no distingue mayúsculas en las palabras clave (`select` y `SELECT` son lo mismo). Por convención se escriben en mayúsculas para separarlas de los identificadores.

### 2.2 DDL

```sql
-- Crear
CREATE TABLE users (
  id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name VARCHAR(120) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Agregar una columna
ALTER TABLE users ADD COLUMN is_active BOOLEAN NOT NULL DEFAULT true;

-- Cambiar el tipo
ALTER TABLE users ALTER COLUMN name TYPE VARCHAR(160);

-- Agregar una restricción
ALTER TABLE users ADD CONSTRAINT users_email_unique UNIQUE (email);

-- Borrar la estructura y los datos
DROP TABLE users;

-- Vaciar los datos y reiniciar el contador de identidad
TRUNCATE TABLE users RESTART IDENTITY;
```

| Instrucción            | Borra datos       | Se puede deshacer con `ROLLBACK` | Reinicia el id         |
| ---------------------- | ----------------- | -------------------------------- | ---------------------- |
| `DELETE FROM users`    | Sí, fila por fila | Sí                               | No                     |
| `TRUNCATE TABLE users` | Sí, rápido        | Sí (en PostgreSQL)               | Con `RESTART IDENTITY` |
| `DROP TABLE users`     | Sí, y la tabla    | Sí                               | N/A                    |

> [!WARNING]
> `DROP TABLE` y `TRUNCATE` no son cosas que se prueban "a ver qué pasa" en la base de un compañero. Son irreversibles fuera de una transacción.

### 2.3 DML

```sql
-- Insertar una fila
INSERT INTO users (name, email) VALUES ('Ada Lovelace', 'ada@example.com');

-- Insertar varias
INSERT INTO users (name, email) VALUES
  ('John Doe', 'john.doe@example.com'),
  ('Jane Smith', 'jane.smith@example.com');

-- Devolver la fila insertada
INSERT INTO users (name, email)
VALUES ('Alan Turing', 'alan@example.com')
RETURNING id, name, email;

-- Actualizar
UPDATE users SET name = 'Ada L.' WHERE id = 3;

-- Borrar
DELETE FROM users WHERE id = 3;
```

`RETURNING` es una joya de PostgreSQL: te devuelve lo que acabás de escribir sin hacer un segundo `SELECT`. Sequelize lo usa internamente en cada `create`.

> [!WARNING]
> `UPDATE users SET name = 'X';` sin `WHERE` actualiza **todas** las filas. `DELETE FROM users;` sin `WHERE` borra **todas** las filas. Antes de ejecutar, leé la sentencia completa.

### 2.4 SELECT

```sql
-- Todo
SELECT * FROM users;

-- Columnas específicas
SELECT id, name FROM users;

-- Filtrar
SELECT id, name FROM users WHERE name = 'Jane Smith';

-- Comparaciones
SELECT * FROM users WHERE id > 10;
SELECT * FROM users WHERE id BETWEEN 1 AND 20;
SELECT * FROM users WHERE name IN ('Ada', 'Alan');

-- Patrones: LIKE distingue mayúsculas, ILIKE no (solo PostgreSQL)
SELECT * FROM users WHERE name ILIKE '%ada%';

-- Nulos
SELECT * FROM users WHERE email IS NULL;

-- Orden y límite
SELECT * FROM users ORDER BY name ASC, id DESC LIMIT 10 OFFSET 20;

-- Sin duplicados
SELECT DISTINCT name FROM users;

-- Alias
SELECT name AS full_name FROM users;
```

| Comodín | Significado                      |
| ------- | -------------------------------- |
| `%`     | Cualquier cantidad de caracteres |
| `_`     | Exactamente un carácter          |

`'%ada%'` encuentra `Ada`, `AdaLovelace` y `Lady Ada`. `'ada%'` solo las que empiezan con `ada`.

### 2.5 JOIN

Un `JOIN` combina filas de dos tablas usando una condición.

```sql
-- Solo los usuarios que tienen posts
SELECT users.name, posts.title
FROM users
INNER JOIN posts ON posts.user_id = users.id;

-- Todos los usuarios, tengan o no posts
SELECT users.name, posts.title
FROM users
LEFT JOIN posts ON posts.user_id = users.id;

-- Todos los posts, tengan o no usuario
SELECT users.name, posts.title
FROM posts
LEFT JOIN users ON users.id = posts.user_id;
```

| JOIN              | Devuelve                                                    |
| ----------------- | ----------------------------------------------------------- |
| `INNER JOIN`      | Solo las filas que coinciden en ambas tablas                |
| `LEFT JOIN`       | Todas las de la izquierda; `NULL` donde no hay coincidencia |
| `RIGHT JOIN`      | Todas las de la derecha                                     |
| `FULL OUTER JOIN` | Todas las de ambas                                          |

> [!NOTE]
> Si un usuario no tiene posts, `INNER JOIN` lo hace desaparecer del resultado. Eso sorprende y genera bugs. Si la pregunta es "todos los usuarios y sus posts", es `LEFT JOIN`.

### 2.6 Agregaciones

```sql
SELECT COUNT(*) AS total FROM users;

SELECT COUNT(*) AS total, COUNT(email) AS con_email FROM users;

SELECT user_id, COUNT(*) AS total_posts
FROM posts
GROUP BY user_id
HAVING COUNT(*) > 2
ORDER BY total_posts DESC;
```

| Función          | Devuelve                            |
| ---------------- | ----------------------------------- |
| `COUNT(*)`       | Cantidad de filas                   |
| `COUNT(columna)` | Cantidad de filas con valor no nulo |
| `SUM(columna)`   | Suma                                |
| `AVG(columna)`   | Promedio                            |
| `MIN` / `MAX`    | Mínimo / máximo                     |

`WHERE` filtra filas **antes** de agrupar. `HAVING` filtra grupos **después** de agrupar.

### 2.7 Índices

Un índice acelera búsquedas a costa de escrituras más lentas y espacio en disco.

```sql
CREATE INDEX users_name_idx ON users (name);
CREATE UNIQUE INDEX users_email_idx ON users (email);
CREATE INDEX users_email_lower_idx ON users (lower(email));
```

| Regla                                         | Detalle                                         |
| --------------------------------------------- | ----------------------------------------------- |
| `PRIMARY KEY` y `UNIQUE`                      | Ya crean su índice                              |
| Indexá lo que filtrás                         | Columnas usadas en `WHERE`, `JOIN` y `ORDER BY` |
| No indexes todo                               | Cada índice se actualiza en cada `INSERT`       |
| El índice tiene que coincidir con la consulta | `lower(email)` no usa el índice de `email`      |

`EXPLAIN ANALYZE` es la herramienta para saber si el motor usa un índice o recorre la tabla entera. Se escribe delante de cualquier `SELECT`: no devuelve filas, devuelve el **plan de ejecución** con los tiempos reales de cada paso.

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE lower(email) = 'ada@example.com';
```

La primera línea de la salida dice el veredicto: `Index Scan using users_email_lower_idx` significa que usó el índice; `Seq Scan on users` significa que leyó la tabla completa. No hace falta optimizar en esta clase, pero cuando una consulta tarda, esta es la primera instrucción que se corre.

### 2.8 Transacciones

```sql
BEGIN;

UPDATE users SET name = 'Ada Lovelace' WHERE id = 1;
INSERT INTO posts (user_id, title) VALUES (1, 'Mi primer post');

COMMIT;    -- confirma todo
-- ROLLBACK;  -- descarta todo
```

En Node, Sequelize expone lo mismo con `sequelize.transaction()`.

### 2.9 Trampas clásicas

| Trampa                               | Qué pasa                             | Cómo se evita                          |
| ------------------------------------ | ------------------------------------ | -------------------------------------- |
| `UPDATE` sin `WHERE`                 | Actualiza todas las filas            | Escribir el `WHERE` antes que el `SET` |
| `DELETE` sin `WHERE`                 | Borra toda la tabla                  | Idem, o usar `TRUNCATE` a propósito    |
| `= NULL`                             | Nunca devuelve filas                 | `IS NULL`                              |
| `LIKE` con mayúsculas                | `%ada%` no encuentra `Ada`           | `ILIKE` en PostgreSQL                  |
| Guardar fechas como texto            | Comparaciones y zonas horarias rotas | `TIMESTAMPTZ`                          |
| Guardar listas en una columna        | No se puede indexar ni filtrar bien  | Otra tabla                             |
| Confiar en el orden sin `ORDER BY`   | El motor devuelve cualquier orden    | `ORDER BY` explícito                   |
| Comparar `COUNT(*)` con `COUNT(col)` | Diferentes por los `NULL`            | Saber cuál querés                      |

<sub>[⬆ Volver al índice](#contenido)</sub>

---

## 3. PostgreSQL en Docker y TablePlus

### 3.1 Por qué Docker

Una base de datos no es un archivo que se abre: es un proceso que escucha en un puerto, con usuarios, permisos, configuración propia y un directorio de datos que no se puede mover a mano. Instalarlo dentro del sistema operativo obliga a registrar un servicio, crear usuarios del sistema, pelear con el `PATH` y limpiar restos cuando algo sale mal.

Docker separa ese proceso de tu máquina en tres piezas: la **imagen** trae el motor y su configuración inicial, el **contenedor** es el proceso aislado que corre, y el **volumen** es donde vive el dato, fuera del contenedor. Esa separación es lo que permite borrar y recrear el motor sin perder la base.

| Problema real                                          | Cómo lo resuelve Docker                                    |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| La versión depende de lo que instaló cada uno          | Queda fijada en el `docker-compose.yaml`: `postgres:16.15` |
| Reinstalar deja restos (servicios, usuarios, carpetas) | `docker compose down -v` y no queda nada                   |
| Estado y motor están mezclados                         | El dato vive en el volumen; el contenedor es descartable   |
| Levantar el entorno son ocho pasos de memoria          | Es un archivo versionado: `docker compose up -d`           |
| Probar otra versión implica desinstalar                | Cambiás una línea y recreás el contenedor                  |

Lo que Docker **no** hace: no reemplaza a PostgreSQL ni a SQL, no hace que la API sea segura y no convierte esto en producción. Es una forma reproducible de tener el motor, nada más.

> [!IMPORTANT]
> Todo lo que sigue usa **`docker compose`**. No hay `docker run` en esta clase: la infraestructura se declara en un archivo, no se escribe a mano en la terminal cada vez.

### 3.2 Instalar la imagen

```bash
docker pull postgres:16.15
```

```text
16.15: Pulling from library/postgres
...
Digest: sha256:f9a7b9ebf6f56201f949db745e47bc423c0f0f8294bafe991cc10bf74b029657
Status: Downloaded newer image for postgres:16.15
docker.io/library/postgres:16.15
```

`docker compose up` la descarga si falta, pero correr el `pull` primero separa dos problemas distintos: "no tengo la imagen" y "el compose está mal".

Usamos **16.15** a propósito: es una versión estable y el path del volumen (`/var/lib/postgresql/data`) es el que corresponde a esa versión. Fijar la versión en el archivo evita que un `pull` te cambie el motor sin avisar.

### 3.3 El archivo compose

En la raíz de la directorio del proyecto, un archivo `docker-compose.yaml`:

```yaml
services:
  db:
    image: postgres:16.15
    container_name: nodepgdb
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    ports:
      - "5432:5432"
    volumes:
      - postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d ${DB_NAME}"]
      interval: 5s
      timeout: 5s
      retries: 10

volumes:
  postgres:
```

| Línea                               | Qué hace                                                            |
| ----------------------------------- | ------------------------------------------------------------------- |
| `image: postgres:16.15`             | Fija la versión del motor. No "la última"                           |
| `container_name: nodepgdb`          | Nombre fijo del contenedor: lo ves en Docker Desktop y en TablePlus |
| `POSTGRES_PASSWORD: ${DB_PASSWORD}` | Toma la contraseña del `.env`, no la escribe en el repo             |
| `POSTGRES_DB: ${DB_NAME}`           | Crea la base `nodepgdb` al inicializar                              |
| `"5432:5432"`                       | Publica el puerto del contenedor en tu máquina                      |
| `postgres:/var/lib/postgresql/data` | El dato sobrevive a `docker compose down`                           |
| `healthcheck`                       | `pg_isready` cada 5 segundos; habilita `up -d --wait`               |

Tres detalles que importan:

1. **`POSTGRES_USER` no está declarado.** Si no se define, la imagen crea el usuario `postgres` por defecto. Por eso el `.env` usa `DB_USER=postgres`.
2. **`${DB_PASSWORD}` y `${DB_NAME}` no son valores fijos.** Docker Compose los reemplaza con las variables del `.env` que esté en la misma carpeta que el compose. Si el `.env` no existe, el reemplazo queda vacío, el contenedor falla y lo vas a ver en los logs.
3. **El volumen se llama `postgres` en el archivo**, pero Docker lo prefija con el nombre de la carpeta: `{projectName}_postgres`. Ese volumen es lo que hace que `docker compose down` no borre los datos.

```ini
# .env.example  → este SÍ se commitea, sin secretos
PORT=3000
CORS_ORIGIN=http://localhost:5173
DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=nodepgdb
DB_USER=postgres
DB_PASSWORD=change_me
DB_LOGGING=false
```

### 3.4 Levantar la base

```bash
docker compose up -d --wait
```

```text
 Container nodepgdb Creating
 Container nodepgdb Created
 Container nodepgdb Starting
 Container nodepgdb Started
 Container nodepgdb Waiting
 Container nodepgdb Healthy
```

`--wait` no devuelve el control hasta que el healthcheck pasa. Cuando el comando termina, la base ya está escuchando en `127.0.0.1:5432` y podés conectarte.

Para apagarla cuando termines: `docker compose down`. Los datos quedan en el volumen y vuelven la próxima vez que la levantes.

### 3.5 Conectar TablePlus

TablePlus es un cliente gráfico: una ventana cómoda para hablar con el mismo motor. No reemplaza el SQL, lo hace visible.

1. Abrí TablePlus → **Create a new connection** → **PostgreSQL**.
2. Completá:

| Campo    | Valor                                            |
| -------- | ------------------------------------------------ |
| Name     | `Node Users (local)`                             |
| Host     | `127.0.0.1`                                      |
| Port     | `5432`                                           |
| User     | `postgres`                                       |
| Password | `app_secret` (el mismo `DB_PASSWORD` del `.env`) |
| Database | `nodepgdb` (el mismo `DB_NAME` del `.env`)       |
| SSL      | desactivado (es local)                           |

3. **Test** → **Connect**.

> [!NOTE]
> El usuario es `postgres` porque no declaramos `POSTGRES_USER` en el compose: la imagen crea ese usuario por defecto y es superusuario. Para una clase alcanza. En un proyecto real crearías un usuario de aplicación con permisos mínimos y conectarías con ese, no con el superusuario.

> [!NOTE]
> Usá `127.0.0.1`, no `localhost`. En algunos sistemas `localhost` resuelve primero a IPv6 (`::1`) y el contenedor publica en IPv4. Si la conexión falla con `ECONNREFUSED` y el contenedor está arriba, probá con `127.0.0.1`.

### 3.6 Crear la tabla users

Con la conexión abierta, `⌘ + ;` abre el editor SQL. Pegá y ejecutá:

```sql
CREATE TABLE users (
  id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name VARCHAR(120) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

Sembrá dos filas para tener datos con los que probar:

```sql
INSERT INTO users (name, email) VALUES
  ('John Doe', 'john.doe@example.com'),
  ('Jane Smith', 'jane.smith@example.com');

SELECT * FROM users ORDER BY id;
```

> [!TIP]
> Si no usás TablePlus, cualquier cliente de PostgreSQL sirve (DBeaver, pgAdmin, DataGrip): los datos de conexión son los mismos de la sección anterior.

| Decisión                                  | Por qué                                                                   |
| ----------------------------------------- | ------------------------------------------------------------------------- |
| `GENERATED BY DEFAULT AS IDENTITY`        | Es el estándar SQL moderno; `SERIAL` es el atajo viejo de PostgreSQL      |
| `BY DEFAULT` y no `ALWAYS`                | Te permite insertar un `id` explícito si alguna vez lo necesitás          |
| `VARCHAR(120)`                            | Un límite explícito documenta la intención y evita basura                 |
| `UNIQUE` en `email`                       | La regla vive en la base, no en un `if` del controller                    |
| `TIMESTAMPTZ`                             | Guarda zona horaria. `TIMESTAMP` no, y después se compara mal             |
| `created_at` / `updated_at` en snake_case | Es la convención de SQL y lo que Sequelize espera con `underscored: true` |

> [!IMPORTANT]
> Los nombres de las columnas tienen que coincidir con lo que el modelo declare. Si la tabla dice `created_at` y el modelo espera `createdAt`, vas a ver `column "createdAt" does not exist`. Se arregla con `underscored: true` en el modelo, que es lo que vamos a configurar.

### 3.7 Problemas frecuentes

| Síntoma                                                             | Causa probable                                          | Solución                                                                                           |
| ------------------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `port is already allocated`                                         | Ya hay un PostgreSQL local en `5432`                    | `lsof -i :5432` y detenerlo, o cambiar a `"5433:5432"` en el compose y `DB_PORT=5433` en el `.env` |
| `The "DB_PASSWORD" variable is not set` (warning de compose)        | No existe el `.env` en la misma carpeta que el compose  | Crear el `.env` con las variables                                                                  |
| `Database is uninitialized and superuser password is not specified` | El compose interpoló un password vacío                  | Revisar que `DB_PASSWORD` esté en el `.env` y volver a levantar                                    |
| `password authentication failed for user "postgres"`                | El volumen se inicializó con otro password              | El password queda grabado en el volumen: `docker compose down -v` y volver a levantar              |
| `database "nodepgdb" does not exist`                                | `DB_NAME` distinto del que se usó al crear el volumen   | Revisar `DB_NAME` o `down -v` para reinicializar                                                   |
| `ECONNREFUSED 127.0.0.1:5432`                                       | El contenedor no está corriendo                         | Levantar la base con `docker compose up -d --wait`                                                 |
| `no configuration file provided`                                    | Estás en otra carpeta                                   | Ejecutá los comandos desde el directorio del proyecto                                              |
| TablePlus conecta pero no ve la tabla                               | Otra base seleccionada                                  | Verificar el campo **Database**: `nodepgdb`                                                        |
| `docker: command not found`                                         | Docker Desktop no está instalado o no está en el `PATH` | Instalar Docker Desktop y abrirlo                                                                  |

<sub>[⬆ Volver al índice](#contenido)</sub>

---

## 4. ORM: Sequelize

### 4.1 Qué es un ORM y para qué sirve

Un **ORM** (Object-Relational Mapper) es una capa que traduce entre los objetos de tu lenguaje y las filas de una tabla. Su responsabilidad es el **mapeo**: sabe que `User` vive en `users`, que `user.email` es la columna `email`, y arma el SQL con los nombres y tipos correctos.

```text
User.findOne({ where: { id: 1 } })   ⇄   SELECT * FROM users WHERE id = 1 LIMIT 1;
```

No decide reglas de negocio ni reemplaza el SQL: traduce y ejecuta.

#### Por qué usarlo en vez de hablar con el driver

El driver (`pg`) te da `client.query("SELECT ...")` y nada más. Con eso podés hacer todo, pero **vos** te encargás de cada detalle:

| Problema            | Con el driver a mano                                      | Con un ORM                                                  |
| ------------------- | --------------------------------------------------------- | ----------------------------------------------------------- |
| **Inyección SQL**   | Si concatenás strings, `' OR 1=1 --` entra en la consulta | Usa consultas parametrizadas: el valor viaja aparte del SQL |
| Nombres de columnas | `created_at` escrito a mano en cada consulta              | Declarado una vez en el modelo                              |
| Resultado           | Filas planas `{ name: ... }`                              | Instancias con métodos (`user.update()`)                    |
| Relaciones          | `JOIN` escrito y mapeado a mano                           | `hasMany`, `belongsTo`                                      |
| Validación          | La escribís en cada `INSERT`                              | Reglas declaradas en el modelo                              |
| Cambiar de motor    | Rehacés las consultas                                     | Cambiás el dialecto                                         |

La **inyección SQL** es la razón número uno. Mirá la diferencia:

```javascript
// ❌ Concatenación: el email entra en la consulta como código
const sql = `SELECT * FROM users WHERE email = '${email}'`;
// Si email = "' OR '1'='1", la consulta devuelve todos los usuarios

// ✅ Parametrizado: el valor no se interpreta como SQL
const { rows } = await pool.query("SELECT * FROM users WHERE email = $1", [
  email,
]);
```

Un ORM hace esto por vos en cada `findOne`, `create` y `update`. No te vuelve inmune (una consulta cruda mal armada sigue siendo tu responsabilidad), pero elimina la clase de error más común y más grave.

#### Qué es Sequelize

**Sequelize** es un ORM para Node.js. Define modelos con `class X extends Model` + `Model.init`, soporta PostgreSQL, MySQL, MariaDB, SQLite y SQL Server, maneja relaciones, transacciones, validaciones y trae un CLI de migraciones. Es la opción más usada junto a Express y la que vas a encontrar en proyectos que ya tienen años.

#### No es el único

| Herramienta   | Para qué se usa                                                         |
| ------------- | ----------------------------------------------------------------------- |
| **Sequelize** | El clásico con Express. Modelos con clases, SQL, migraciones            |
| **Prisma**    | Schema declarativo propio (`schema.prisma`) y cliente generado y tipado |
| **TypeORM**   | Decoradores y entidades; muy usado con TypeScript y NestJS              |
| **Mongoose**  | MongoDB (NoSQL). Modela documentos, no tablas                           |
| **Drizzle**   | Query builder + ORM tipado, más cerca del SQL                           |

Mongoose es el caso interesante: es un **ODM** (Object-Document Mapper) para una base NoSQL. La idea de mapear objetos a registros es la misma; lo que cambia es que el registro es un documento y no una fila.

En esta clase usamos **Sequelize** porque el SQL que ya sabés se traduce de forma directa y porque su API de modelos se parece a la que vas a ver en el resto del curso. Si mañana cambiás a Prisma o TypeORM, el concepto de ORM es el mismo.

| Situación                               | ORM                 | SQL a mano         |
| --------------------------------------- | ------------------- | ------------------ |
| CRUD simple                             | ✅ Ideal            | Trabajo repetitivo |
| Reportes con `GROUP BY`, `HAVING`, CTEs | Forzado             | ✅ Ideal           |
| Migraciones de esquema                  | Con `sequelize-cli` | ✅ Control total   |
| Prototipo rápido                        | ✅                  | Más código         |

> [!IMPORTANT]
> Un ORM no reemplaza saber SQL. Si no sabés qué es un `JOIN`, no vas a poder depurar el `JOIN` que el ORM generó. Por eso primero SQL y después Sequelize.

### 4.2 Instalación

```bash
npm install express cors yup dotenv sequelize pg pg-hstore
```

| Paquete            | Rol                                                                  |
| ------------------ | -------------------------------------------------------------------- |
| `sequelize`        | El ORM                                                               |
| `pg`               | Driver de PostgreSQL (el que habla el protocolo)                     |
| `pg-hstore`        | Serialización de columnas `hstore` (Sequelize lo pide para Postgres) |
| `dotenv`           | Carga `.env` en `process.env`                                        |
| `yup`              | Validación del DTO                                                   |
| `express` / `cors` | La API que ya conocés                                                |

> [!NOTE]
> `sequelize` 6.x es la línea estable. La 7 sigue en alpha, así que para clase usamos la 6: la API que vas a encontrar en la mayoría de los proyectos y en la documentación oficial.

Y en `package.json`:

```json
{
  "type": "module",
  "scripts": {
    "dev": "node --watch src/server.js",
    "start": "node src/server.js"
  }
}
```

Node 26 trae `--watch`: no necesitás `nodemon`.

### 4.3 Variables de entorno

Las credenciales nunca van en el código. Van en `.env`, que **no se commitea**.

```ini
# .env
PORT=3000
CORS_ORIGIN=http://localhost:5173

DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=nodepgdb
DB_USER=postgres
DB_PASSWORD=app_secret
DB_LOGGING=false
```

```ini
# .env.example  → este SÍ se commitea, sin secretos
PORT=3000
CORS_ORIGIN=http://localhost:5173
DB_HOST=127.0.0.1
DB_PORT=5432
DB_NAME=nodepgdb
DB_USER=postgres
DB_PASSWORD=change_me
DB_LOGGING=false
```

```gitignore
node_modules/
.env
```

> [!WARNING]
> Si commiteaste un `.env` con una contraseña real, cambiar el `.gitignore` no alcanza: la contraseña quedó en el historial. Rotá la credencial.

### 4.4 Archivo de conexión

Un solo lugar crea la conexión. Todo lo demás lo importa.

```javascript
// src/config/database.js
import "dotenv/config";
import { Sequelize } from "sequelize";

const required = (name) => {
  const value = process.env[name];

  if (!value) {
    throw new Error(`Missing required environment variable: ${name}`);
  }

  return value;
};

export const sequelize = new Sequelize({
  dialect: "postgres",
  host: process.env.DB_HOST ?? "127.0.0.1",
  port: Number(process.env.DB_PORT ?? 5432),
  database: required("DB_NAME"),
  username: required("DB_USER"),
  password: required("DB_PASSWORD"),
  logging: process.env.DB_LOGGING === "true" ? console.log : false,
  define: {
    underscored: true,
    timestamps: true,
  },
  pool: {
    max: 10,
    min: 0,
    acquire: 30_000,
    idle: 10_000,
  },
});
```

| Opción                | Por qué                                              |
| --------------------- | ---------------------------------------------------- |
| `dialect: "postgres"` | Usa `pg` internamente                                |
| `required(...)`       | Falla al arrancar, no en la primera request          |
| `logging`             | En `true`, imprime el SQL generado. Enseña muchísimo |
| `define.underscored`  | `createdAt` → `created_at` automáticamente           |
| `define.timestamps`   | Sequelize maneja `created_at` y `updated_at` solo    |
| `pool`                | Reutiliza conexiones en vez de abrir una por request |

> [!NOTE]
> `define` aplica valores por defecto a **todos** los modelos. Así no repetís `underscored: true` en cada `init`.

### 4.5 Arranque con fail fast

**Fail fast** significa: si falta un requisito para funcionar, el proceso muere **al arrancar**, no cuando llega la primera request.

En esta API el requisito es uno: la base tiene que responder. Crear el objeto `Sequelize` no prueba nada, porque la conexión es perezosa: se abre recién cuando alguien consulta. Por eso hay que probarla explícitamente.

```javascript
// src/config/database.js (continuación)
export async function connectDatabase() {
  try {
    await sequelize.authenticate();
    console.log(`Database connection established → ${process.env.DB_NAME}`);
  } catch (error) {
    console.error("Unable to connect to the database");
    throw error;
  }
}
```

`authenticate()` abre una conexión y ejecuta un `SELECT 1`. Si el contenedor está apagado, la contraseña no coincide o la base no existe, el error aparece ahí. En `server.js` ese error termina en:

```javascript
main().catch((error) => {
  console.error("Failed to start server:", error.message);
  process.exit(1);
});
```

`process.exit(1)` es la parte importante: el proceso termina con código de error. Docker, systemd o el panel que uses lo ven como un servicio caído, no como uno "sano que falla en cada request".

|                     | Con fail fast                    | Sin fail fast                                    |
| ------------------- | -------------------------------- | ------------------------------------------------ |
| Cuándo lo descubrís | Al arrancar                      | Con la primera request del usuario               |
| Qué ve el usuario   | Nada: el servicio no está arriba | Un `500` con stack trace                         |
| Qué ves vos         | `ECONNREFUSED 127.0.0.1:5432`    | El mismo error, escondido entre logs de requests |
| Estado del proceso  | Muerto, sin aceptar tráfico      | Vivo, aceptando y fallando                       |

| Alternativa                  | Cuándo                                                                |
| ---------------------------- | --------------------------------------------------------------------- |
| `authenticate()` al arrancar | Siempre que la API no sirva sin base                                  |
| Sin `authenticate()`         | Tests unitarios que mockean todo, o procesos que toleran modo offline |

> [!TIP]
> Un `console.log` con el nombre de la base te ahorra el clásico "estoy conectado a la base equivocada". Ponelo.

<sub>[⬆ Volver al índice](#contenido)</sub>

---

## 5. Modelo User con Sequelize

### 5.1 Del SQL al modelo

El modelo es la representación en JavaScript de una tabla. No es un DTO y no es una entidad de dominio: es la puerta de entrada al motor.

| SQL                        | Sequelize                                                            |
| -------------------------- | -------------------------------------------------------------------- |
| `CREATE TABLE users (...)` | `User.init({...}, { tableName: "users" })`                           |
| `id INTEGER ... IDENTITY`  | `{ type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true }` |
| `VARCHAR(120) NOT NULL`    | `{ type: DataTypes.STRING(120), allowNull: false }`                  |
| `UNIQUE`                   | `{ unique: true }`                                                   |
| `DEFAULT now()`            | `{ defaultValue: DataTypes.NOW }`                                    |
| `created_at TIMESTAMPTZ`   | `timestamps: true` + `underscored: true`                             |

### 5.2 El modelo

```javascript
// src/models/user.model.js
import { DataTypes, Model } from "sequelize";
import { sequelize } from "../config/database.js";

export class User extends Model {}

User.init(
  {
    id: {
      type: DataTypes.INTEGER,
      primaryKey: true,
      autoIncrement: true,
    },
    name: {
      type: DataTypes.STRING(120),
      allowNull: false,
      validate: {
        notEmpty: { msg: "Name cannot be empty" },
        len: {
          args: [2, 120],
          msg: "Name must be between 2 and 120 characters",
        },
      },
    },
    email: {
      type: DataTypes.STRING(255),
      allowNull: false,
      unique: true,
      validate: {
        isEmail: { msg: "Email must be a valid email address" },
      },
    },
  },
  {
    sequelize,
    modelName: "User",
    tableName: "users",
    underscored: true,
    timestamps: true,
  },
);
```

| Decisión                   | Por qué                                                    |
| -------------------------- | ---------------------------------------------------------- |
| `class User extends Model` | Permite métodos propios y `instanceof`                     |
| `modelName: "User"`        | Nombre en el código (`User.findOne`)                       |
| `tableName: "users"`       | Nombre real en la base. Sin esto, Sequelize pluraliza solo |
| `underscored: true`        | `createdAt` ↔ `created_at`                                 |
| `timestamps: true`         | Sequelize completa y actualiza las fechas                  |
| `unique` en `email`        | Sequelize valida antes de llegar al `UNIQUE` de la tabla   |

> [!IMPORTANT]
> `modelName` y `tableName` no son lo mismo. Si `tableName` no coincide con la tabla real, la primera query falla con `relation "..." does not exist`. Declaralo siempre de forma explícita: no dependas de la pluralización automática.

> [!NOTE]
> Hay dos estilos para definir un modelo: `sequelize.define("User", {...})` o `class extends Model` + `Model.init`. Usamos el segundo porque se lleva mejor con clases, `instanceof` y `#private` fields, y porque es el que vas a ver en código moderno.

### 5.3 Operaciones básicas

| Operación  | Sequelize                                  | SQL equivalente                                    |
| ---------- | ------------------------------------------ | -------------------------------------------------- |
| Listar     | `User.findAll({ order: [["id", "ASC"]] })` | `SELECT * FROM users ORDER BY id ASC`              |
| Por id     | `User.findByPk(id)`                        | `SELECT * FROM users WHERE id = $1 LIMIT 1`        |
| Por email  | `User.findOne({ where: { email } })`       | `SELECT * FROM users WHERE email = $1 LIMIT 1`     |
| Crear      | `User.create({ name, email })`             | `INSERT INTO users (...) VALUES (...) RETURNING *` |
| Actualizar | `user.update({ name })`                    | `UPDATE users SET name = $1 WHERE id = $2`         |
| Borrar     | `user.destroy()`                           | `DELETE FROM users WHERE id = $1`                  |
| Contar     | `User.count()`                             | `SELECT COUNT(*) FROM users`                       |

Consultas con operadores:

```javascript
import { Op } from "sequelize";

// Buscar en name o email, sin distinguir mayúsculas (PostgreSQL)
const users = await User.findAll({
  where: {
    [Op.or]: [
      { name: { [Op.iLike]: `%${keywords}%` } },
      { email: { [Op.iLike]: `%${keywords}%` } },
    ],
  },
  order: [["id", "ASC"]],
});
```

| Operador               | SQL                |
| ---------------------- | ------------------ |
| `Op.eq`                | `=`                |
| `Op.ne`                | `<>`               |
| `Op.gt` / `Op.gte`     | `>` / `>=`         |
| `Op.lt` / `Op.lte`     | `<` / `<=`         |
| `Op.like` / `Op.iLike` | `LIKE` / `ILIKE`   |
| `Op.in`                | `IN (...)`         |
| `Op.is`                | `IS` (para `NULL`) |
| `Op.and` / `Op.or`     | `AND` / `OR`       |

> [!TIP]
> Poné `logging: console.log` un rato y mirá el SQL que genera cada consulta. Es la forma más rápida de entender el ORM, y la más rápida de detectar un `SELECT *` innecesario.

### 5.4 Modificar el esquema: agregar un campo

El modelo y la tabla son dos cosas distintas: el modelo dice qué espera tu código, la tabla dice qué existe. **Si agregás un campo al modelo, la tabla no se entera sola.**

Ejemplo concreto: sumar `password_hash` a `users`.

**1. La tabla primero.** El `ALTER TABLE` en TablePlus:

```sql
ALTER TABLE users ADD COLUMN password_hash VARCHAR(255);
```

Si la tabla ya tiene filas, agregar `NOT NULL` de una vez falla: esas filas no tienen valor. Hay dos caminos:

```sql
-- Opción A: completar las filas existentes y después exigir NOT NULL
UPDATE users SET password_hash = 'cambiame' WHERE password_hash IS NULL;
ALTER TABLE users ALTER COLUMN password_hash SET NOT NULL;

-- Opción B: vaciar la tabla en desarrollo y agregar el campo completo
TRUNCATE users RESTART IDENTITY;
ALTER TABLE users ADD COLUMN password_hash VARCHAR(255) NOT NULL;
```

**2. El modelo después.** El mismo campo, con su tipo y su validación:

```javascript
password_hash: {
  type: DataTypes.STRING(255),
  allowNull: false,
},
```

**3. Y el DTO de salida.** `toUserResponse` no lo incluye: lo que no está en el DTO, no sale.

Si te saltás el paso 1, la primera query falla con `column "password_hash" does not exist`. Si te saltás el paso 2, la columna existe pero Sequelize no la lee.

#### ¿Y `sync()`?

`sync()` crea las tablas que faltan a partir de los modelos. No es una migración: no versiona nada ni te avisa qué va a cambiar.

| Método                      | Qué hace                                                                 | Riesgo                                |
| --------------------------- | ------------------------------------------------------------------------ | ------------------------------------- |
| `sync()`                    | Crea las tablas que faltan. **No** agrega columnas a una tabla existente | Bajo                                  |
| `sync({ alter: true })`     | Compara modelo y tabla, y aplica `ALTER`                                 | **Puede borrar o renombrar columnas** |
| `sync({ force: true })`     | `DROP` + `CREATE`                                                        | Borra todo                            |
| Migración (`sequelize-cli`) | Cambios versionados, en orden y reversibles                              | El camino profesional                 |

```javascript
// SOLO en una base de desarrollo descartable
await sequelize.sync({ force: true });
```

La regla práctica:

| Base                                  | Qué usar                                              |
| ------------------------------------- | ----------------------------------------------------- |
| Desarrollo, sin datos que te importen | `sync({ alter: true })` y revisar qué hizo            |
| Cualquier base con datos              | `ALTER TABLE` a mano, como arriba                     |
| Proyecto en serio                     | Migraciones: un archivo por cambio, con `up` y `down` |

> [!WARNING]
> `sync({ alter: true })` en una base con datos es una de las formas más rápidas de perder información, y no siempre genera el `ALTER` que esperás. Con datos reales, la columna se agrega con SQL o con una migración.

### 5.5 Validación de modelo vs DTO

Hay dos validaciones y no son la misma:

|              | Validación del DTO   | Validación del modelo               |
| ------------ | -------------------- | ----------------------------------- |
| Dónde vive   | `src/dtos/`          | `src/models/`                       |
| Qué protege  | La frontera HTTP     | La integridad de la fila            |
| Cuándo corre | Antes del controller | En cada `create` / `update`         |
| Qué devuelve | `400` con `details`  | `SequelizeValidationError`          |
| Cubre        | Requests de la API   | Cualquier camino, incluso un script |

No es redundancia: son cinturones distintos. El DTO dice "esto no es un usuario válido para mi API". El modelo dice "esto no es una fila válida para mi tabla".

> [!IMPORTANT]
> El DTO no reemplaza las restricciones de la base. Si mañana otro servicio escribe en `users`, la validación de tu DTO no se ejecuta. El `UNIQUE` y los `NOT NULL` sí.

### 5.6 Errores de Sequelize

| Error                                | Cuándo aparece                     | Status sugerido | Código                 |
| ------------------------------------ | ---------------------------------- | --------------- | ---------------------- |
| `SequelizeUniqueConstraintError`     | Violaste un `UNIQUE`               | `409`           | `EMAIL_TAKEN`          |
| `SequelizeValidationError`           | Falló una validación del modelo    | `400`           | `VALIDATION_ERROR`     |
| `SequelizeConnectionRefusedError`    | La base no está arriba             | `503`           | `DATABASE_UNAVAILABLE` |
| `SequelizeConnectionError`           | Credenciales o base inexistente    | `503`           | `DATABASE_UNAVAILABLE` |
| `SequelizeForeignKeyConstraintError` | Insertaste una FK que no existe    | `409`           | `CONSTRAINT_VIOLATION` |
| `SequelizeDatabaseError`             | SQL inválido o columna inexistente | `500`           | `DATABASE_ERROR`       |

```javascript
import { UniqueConstraintError, ValidationError } from "sequelize";

// UniqueConstraintError → 409, no 500.
// El usuario hizo algo que la base prohíbe, no se rompió el servidor.
```

<sub>[⬆ Volver al índice](#contenido)</sub>

---

## 6. DTO: fundamentos

### 6.1 Qué es un DTO

**DTO** significa **Data Transfer Object**: un objeto que define los datos que cruzan una frontera. Ni más ni menos.

```text
HTTP body  →  createUserDto  →  Controller  →  Model  →  tabla users
   (crudo)      (contrato)        (traduce)    (persiste)
```

| Es                                   | No es                         |
| ------------------------------------ | ----------------------------- |
| Un contrato de entrada o salida      | El modelo de Sequelize        |
| El lugar donde se valida y normaliza | Una entidad de dominio        |
| Una allowlist de campos              | Un `{...req.body}` disfrazado |

### 6.2 Por qué existe

Sin DTO, el controller hace esto:

```javascript
// ❌ Todo el body va a la base
const user = await User.create(req.body);
```

Consecuencias:

1. Un cliente manda `{ "id": 999, "role": "admin" }` y decide cosas que no le corresponden.
2. El email entra con mayúsculas y espacios: `"  ADA@Example.COM "`.
3. El error de validación lo tira la base, con un mensaje pensado para un DBA.
4. Si mañana la tabla cambia de nombre una columna, el contrato HTTP se rompe solo.

Con DTO:

```javascript
// ✅ Solo lo que el contrato permite, ya normalizado
const user = await User.create(req.dto);
```

| Beneficio               | Ejemplo concreto                                           |
| ----------------------- | ---------------------------------------------------------- |
| **Allowlist**           | `noUnknown(true)` rechaza `role`, `id`, `isAdmin`          |
| **Normalización**       | `trim()` y `lowercase()` antes de guardar                  |
| **Validación temprana** | `400` con `details` antes de tocar la base                 |
| **Desacople**           | Cambiás la tabla sin cambiar el JSON que recibe el cliente |
| **Seguridad**           | Lo que no está declarado, no pasa                          |

> [!IMPORTANT]
> El DTO no "filtra" datos por comodidad: define el contrato. Si el campo no está en el DTO, el cliente no puede escribirlo. No hay `delete req.body.role` que sobreviva al próximo refactor.

### 6.3 DTO de entrada con Yup

Un DTO **no necesita ser una clase**. El contrato es el schema: dice qué campos cruzan la frontera, cuáles son obligatorios y cómo se normalizan. Una clase con `static from()` y campos privados es ceremonia: agrega líneas y no agrega garantías.

```javascript
// src/dtos/user.dto.js
import * as yup from "yup";

// Los campos se escriben una sola vez y se reusan en los dos contratos.
const name = yup.string().trim().min(2).max(120);
const email = yup.string().trim().lowercase().email().max(255);

// Crear: los dos campos son obligatorios.
export const createUserDto = yup
  .object({
    name: name.required(),
    email: email.required(),
  })
  .noUnknown(true, "Unknown fields are not allowed");

// Actualizar: los dos opcionales, pero al menos uno tiene que venir.
export const updateUserDto = yup
  .object({ name, email })
  .noUnknown(true, "Unknown fields are not allowed")
  .test(
    "at-least-one",
    "At least one field is required",
    (value) => Object.keys(value ?? {}).length > 0,
  );

// Salida: lo que la API expone de un usuario.
export function toUserResponse(user) {
  return {
    id: user.id,
    name: user.name,
    email: user.email,
  };
}
```

| Línea                        | Qué hace                                                         |
| ---------------------------- | ---------------------------------------------------------------- |
| `const name = ...`           | Una sola definición del campo, reusada en crear y actualizar     |
| `.required()`                | Solo en `createUserDto`: al crear, el nombre no es opcional      |
| `.noUnknown(true, ...)`      | Rechaza las claves que el contrato no declara                    |
| `.test("at-least-one", ...)` | Un `PATCH {}` no es una actualización válida                     |
| `toUserResponse`             | Allowlist de salida: si mañana agregás una columna, no sale sola |

> [!NOTE]
> **¿Por qué no una clase?** Porque hoy el DTO no tiene comportamiento: valida y normaliza. Una clase se justifica cuando el DTO necesita lógica propia (por ejemplo, aceptar `fullName` y partirlo en `name` y `lastName`) o cuando querés métodos como `toEntity()`. Hasta entonces, el contrato es el schema y alcanza.

> [!NOTE]
> `stripUnknown: false` es deliberado: preferimos **rechazar** una clave desconocida antes que descartarla en silencio. Un `201` con un campo ignorado es peor que un `400` que dice qué sobra. Si preferís la estrategia permisiva, cambiá a `stripUnknown: true` y quitá `noUnknown`.

### 6.4 Middleware de validación

El middleware recibe **el contrato** (el schema) y deja el resultado en `req.dto`. No valida "el body": aplica el contrato de ese endpoint.

```javascript
// src/middlewares/validate-body.middleware.js
import * as yup from "yup";

export const validateBody = (schema) => async (req, res, next) => {
  try {
    req.dto = await schema.validate(req.body ?? {}, {
      abortEarly: false,
      stripUnknown: false,
    });
    next();
  } catch (error) {
    if (!(error instanceof yup.ValidationError)) {
      return next(error);
    }

    res.status(400).json({
      error: {
        code: "VALIDATION_ERROR",
        message: "Invalid request body",
        details: error.errors,
      },
    });
  }
};
```

| Aspecto                 | Schema suelto junto a las rutas        | Contrato en `src/dtos/`                                      |
| ----------------------- | -------------------------------------- | ------------------------------------------------------------ |
| Nombre                  | `createUserSchema`                     | `createUserDto`                                              |
| Ubicación               | Mezclado con rutas y configuración     | Un archivo por dominio: `user.dto.js`                        |
| Resultado en la request | `req.body` reemplazado por el validado | `req.dto`, con nombre propio                                 |
| Operaciones             | Un schema por operación                | Un contrato por operación: `createUserDto` y `updateUserDto` |

> [!NOTE]
> El controller nunca ve `req.body`: ve `req.dto`, y sabe que pasó por la frontera.

### 6.5 DTO de salida

Lo que devolvés también es contrato. `res.json(user)` con un modelo de Sequelize expone todo lo que la tabla tenga, hoy y en el futuro.

```javascript
// ❌ Si mañana agregás password_hash, se filtra solo
res.json({ data: user });

// ✅ Explícito, y el día que agregues una columna, no sale sola
res.json({ data: toUserResponse(user) });
```

| Contrato         | Dirección | Qué controla           |
| ---------------- | --------- | ---------------------- |
| `createUserDto`  | Entrada   | Qué se puede crear     |
| `updateUserDto`  | Entrada   | Qué se puede modificar |
| `toUserResponse` | Salida    | Qué se puede ver       |

### 6.6 DTO, modelo y entidad

| Pieza              | Conoce HTTP    | Conoce la base | Conoce reglas de negocio | Ejemplo                    |
| ------------------ | -------------- | -------------- | ------------------------ | -------------------------- |
| DTO                | Sí (body/JSON) | No             | Solo de formato          | `createUserDto`            |
| Modelo (ORM)       | No             | Sí             | Validaciones de fila     | `User`                     |
| Entidad de dominio | No             | No             | Sí                       | `User` con `changeEmail()` |
| Row / registro     | No             | Sí (cruda)     | No                       | `{ id, name, email }`      |

Esta clase usa DTO + Modelo. La entidad de dominio aparece cuando las reglas dejan de ser "validar campos" y pasan a ser "un usuario no puede cambiar de email sin confirmarlo". Ahí el modelo de Sequelize deja de ser el lugar de las reglas.

> [!IMPORTANT]
> Un mismo nombre (`User`) puede existir en dos capas con dos formas distintas. No es duplicación: es la frontera. Si te obliga a copiar campos a mano, es señal de que el diseño todavía no está claro.

<sub>[⬆ Volver al índice](#contenido)</sub>

---

## 7. Tarea 013: patrón clásico

### 7.1 Objetivo

Implementar una API de usuarios con:

- **Persistencia real** en PostgreSQL 16.15 (Docker).
- **Sequelize** como ORM, con un modelo por tabla.
- **DTO** para entrada y salida.
- **Patrón clásico de Express**: `Route → Middleware → Controller → Model`.

```text
HTTP → Route → Middleware → Controller → Model → PostgreSQL
        (¿qué path?)  (¿es válido?)  (traduce HTTP)  (habla con la base)
```

> [!IMPORTANT]
> Si `npm run dev` arranca, respondés un `POST` y después reiniciás el proceso, el usuario tiene que seguir ahí. Si no, la tarea no está terminada.

### 7.2 Estructura

```text
tarea-013/
├── .env
├── .env.example
├── .gitignore
├── docker-compose.yaml
├── package.json
├── README.md
└── src/
    ├── app.js
    ├── server.js
    ├── config/
    │   └── database.js
    ├── models/
    │   └── user.model.js
    ├── dtos/
    │   └── user.dto.js
    ├── errors/
    │   └── app-error.js
    ├── middlewares/
    │   ├── validate-body.middleware.js
    │   ├── validate-id.middleware.js
    │   ├── not-found.middleware.js
    │   └── error.middleware.js
    ├── controllers/
    │   └── user.controller.js
    └── routes/
        └── user.routes.js
```

| Carpeta       | Responsabilidad             | No debería conocer  |
| ------------- | --------------------------- | ------------------- |
| `routes`      | Path, método, middlewares   | Reglas de negocio   |
| `middlewares` | Validar, normalizar, cortar | Cómo se guarda      |
| `controllers` | Traducir HTTP ↔ datos       | SQL crudo           |
| `models`      | Tabla y relaciones          | `req` / `res`       |
| `dtos`        | Contrato de entrada/salida  | Express y Sequelize |
| `config`      | Conexión                    | Rutas               |
| `app.js`      | Armar la app                | Arrancar el proceso |
| `server.js`   | Conectar, escuchar, cerrar  | Rutas concretas     |

> [!NOTE]
> Una vez que las reglas de negocio dejen de entrar en un controller, el paso siguiente es la arquitectura por capas: `Service` y `Repository`, con las dependencias armadas en un único lugar. No son enemigas de este patrón: son etapas.

### 7.3 Código completo

#### `package.json`

```json
{
  "name": "tarea-013",
  "version": "1.0.0",
  "type": "module",
  "private": true,
  "scripts": {
    "dev": "node --watch src/server.js",
    "start": "node src/server.js"
  },
  "dependencies": {
    "cors": "^2.8.6",
    "dotenv": "^18.0.10",
    "express": "^5.3.0",
    "pg": "^8.23.1",
    "pg-hstore": "^2.3.4",
    "sequelize": "^6.37.8",
    "yup": "^1.7.1"
  }
}
```

#### `src/config/database.js`

```javascript
import "dotenv/config";
import { Sequelize } from "sequelize";

const required = (name) => {
  const value = process.env[name];

  if (!value) {
    throw new Error(`Missing required environment variable: ${name}`);
  }

  return value;
};

export const sequelize = new Sequelize({
  dialect: "postgres",
  host: process.env.DB_HOST ?? "127.0.0.1",
  port: Number(process.env.DB_PORT ?? 5432),
  database: required("DB_NAME"),
  username: required("DB_USER"),
  password: required("DB_PASSWORD"),
  logging: process.env.DB_LOGGING === "true" ? console.log : false,
  define: {
    underscored: true,
    timestamps: true,
  },
  pool: {
    max: 10,
    min: 0,
    acquire: 30_000,
    idle: 10_000,
  },
});

export async function connectDatabase() {
  try {
    await sequelize.authenticate();
    console.log(`Database connection established → ${process.env.DB_NAME}`);
  } catch (error) {
    console.error("Unable to connect to the database");
    throw error;
  }
}
```

#### `src/models/user.model.js`

```javascript
import { DataTypes, Model } from "sequelize";
import { sequelize } from "../config/database.js";

export class User extends Model {}

User.init(
  {
    id: {
      type: DataTypes.INTEGER,
      primaryKey: true,
      autoIncrement: true,
    },
    name: {
      type: DataTypes.STRING(120),
      allowNull: false,
      validate: {
        notEmpty: { msg: "Name cannot be empty" },
        len: {
          args: [2, 120],
          msg: "Name must be between 2 and 120 characters",
        },
      },
    },
    email: {
      type: DataTypes.STRING(255),
      allowNull: false,
      unique: true,
      validate: {
        isEmail: { msg: "Email must be a valid email address" },
      },
    },
  },
  {
    sequelize,
    modelName: "User",
    tableName: "users",
    underscored: true,
    timestamps: true,
  },
);
```

#### `src/dtos/user.dto.js`

```javascript
import * as yup from "yup";

// Los campos se escriben una sola vez y se reusan en los dos contratos.
const name = yup.string().trim().min(2).max(120);
const email = yup.string().trim().lowercase().email().max(255);

// Crear: los dos campos son obligatorios.
export const createUserDto = yup
  .object({
    name: name.required(),
    email: email.required(),
  })
  .noUnknown(true, "Unknown fields are not allowed");

// Actualizar: los dos opcionales, pero al menos uno tiene que venir.
export const updateUserDto = yup
  .object({ name, email })
  .noUnknown(true, "Unknown fields are not allowed")
  .test(
    "at-least-one",
    "At least one field is required",
    (value) => Object.keys(value ?? {}).length > 0,
  );

// Salida: lo que la API expone de un usuario.
export function toUserResponse(user) {
  return {
    id: user.id,
    name: user.name,
    email: user.email,
  };
}
```

#### `src/errors/app-error.js`

```javascript
export class AppError extends Error {
  constructor(
    message,
    { status = 500, code = "INTERNAL_ERROR", details } = {},
  ) {
    super(message);

    this.name = "AppError";
    this.status = status;
    this.code = code;
    this.details = details;
  }
}
```

#### `src/middlewares/validate-body.middleware.js`

```javascript
import * as yup from "yup";

export const validateBody = (schema) => async (req, res, next) => {
  try {
    req.dto = await schema.validate(req.body ?? {}, {
      abortEarly: false,
      stripUnknown: false,
    });
    next();
  } catch (error) {
    if (!(error instanceof yup.ValidationError)) {
      return next(error);
    }

    res.status(400).json({
      error: {
        code: "VALIDATION_ERROR",
        message: "Invalid request body",
        details: error.errors,
      },
    });
  }
};
```

#### `src/middlewares/validate-id.middleware.js`

```javascript
export const validateUserId = (req, res, next) => {
  const id = Number(req.params.id);

  if (!Number.isInteger(id) || id <= 0) {
    return res.status(400).json({
      error: {
        code: "INVALID_ID",
        message: "User id must be a positive integer",
      },
    });
  }

  req.userId = id;
  next();
};
```

#### `src/middlewares/not-found.middleware.js`

```javascript
export const notFoundHandler = (req, res) => {
  res.status(404).json({
    error: { code: "ROUTE_NOT_FOUND", message: "Route not found" },
  });
};
```

#### `src/middlewares/error.middleware.js`

```javascript
import { UniqueConstraintError, ValidationError } from "sequelize";

export const errorHandler = (error, req, res, next) => {
  if (res.headersSent) {
    return next(error);
  }

  // JSON mal formado: lo lanza express.json()
  if (error.type === "entity.parse.failed") {
    return res.status(400).json({
      error: {
        code: "INVALID_JSON",
        message: "Request body is not valid JSON",
      },
    });
  }

  // La base rechazó un UNIQUE (email repetido)
  if (error instanceof UniqueConstraintError) {
    return res.status(409).json({
      error: {
        code: "EMAIL_TAKEN",
        message: "Email is already registered",
      },
    });
  }

  // La base rechazó una validación del modelo
  if (error instanceof ValidationError) {
    return res.status(400).json({
      error: {
        code: "VALIDATION_ERROR",
        message: "Invalid user data",
        details: error.errors.map((item) => item.message),
      },
    });
  }

  const status = error.status ?? 500;

  if (status >= 500) {
    console.error(error);
  }

  res.status(status).json({
    error: {
      code: error.code ?? "INTERNAL_ERROR",
      message: status >= 500 ? "Unexpected server error" : error.message,
      ...(error.details ? { details: error.details } : {}),
    },
  });
};
```

#### `src/controllers/user.controller.js`

```javascript
import { Op } from "sequelize";
import { User } from "../models/user.model.js";
import { toUserResponse } from "../dtos/user.dto.js";
import { AppError } from "../errors/app-error.js";

const userNotFound = () =>
  new AppError("User not found", { status: 404, code: "USER_NOT_FOUND" });

export const list = async (req, res) => {
  const users = await User.findAll({ order: [["id", "ASC"]] });

  res.json({ data: users.map(toUserResponse) });
};

export const search = async (req, res) => {
  const keywords = String(req.query.keywords ?? "").trim();

  if (!keywords) {
    return res.json({ data: [] });
  }

  const pattern = `%${keywords}%`;

  const users = await User.findAll({
    where: {
      [Op.or]: [
        { name: { [Op.iLike]: pattern } },
        { email: { [Op.iLike]: pattern } },
      ],
    },
    order: [["id", "ASC"]],
  });

  res.json({ data: users.map(toUserResponse) });
};

export const getById = async (req, res) => {
  const user = await User.findByPk(req.userId);

  if (!user) {
    throw userNotFound();
  }

  res.json({ data: toUserResponse(user) });
};

export const create = async (req, res) => {
  const user = await User.create(req.dto);

  res
    .status(201)
    .location(`/api/users/${user.id}`)
    .json({ data: toUserResponse(user) });
};

export const update = async (req, res) => {
  const user = await User.findByPk(req.userId);

  if (!user) {
    throw userNotFound();
  }

  await user.update(req.dto);

  res.json({ data: toUserResponse(user) });
};

export const remove = async (req, res) => {
  const user = await User.findByPk(req.userId);

  if (!user) {
    throw userNotFound();
  }

  await user.destroy();

  res.status(204).send();
};
```

> [!NOTE]
> Express 5 reenvía solo al `errorHandler` el rechazo de un handler `async`. Por eso `create` no tiene `try/catch`: un `UniqueConstraintError` llega al middleware central. En Express 4 tenías que envolver cada handler con un `asyncHandler` o usar `.catch(next)`.
>
> Si el `req.dto` te parece magia, no lo es: el middleware lo dejó ahí. Es el mismo `req.body` transformado y validado, con un nombre que obliga a pensar en la frontera.

#### `src/routes/user.routes.js`

```javascript
import { Router } from "express";
import * as controller from "../controllers/user.controller.js";
import { validateBody } from "../middlewares/validate-body.middleware.js";
import { validateUserId } from "../middlewares/validate-id.middleware.js";
import { createUserDto, updateUserDto } from "../dtos/user.dto.js";

export const userRouter = Router();

// /search va ANTES de /:id, o Express lo interpreta como un id
userRouter.get("/search", controller.search);
userRouter.get("/", controller.list);
userRouter.get("/:id", validateUserId, controller.getById);
userRouter.post("/", validateBody(createUserDto), controller.create);
userRouter.patch(
  "/:id",
  validateUserId,
  validateBody(updateUserDto),
  controller.update,
);
userRouter.delete("/:id", validateUserId, controller.remove);
```

> [!WARNING]
> El orden de las rutas es significativo. Si declarás `/:id` antes de `/search`, un `GET /api/users/search` entra por `/:id` con `id = "search"` y responde `400 INVALID_ID`. Express no elige la más específica: usa la primera que coincide.

#### `src/app.js`

```javascript
import express from "express";
import cors from "cors";
import { userRouter } from "./routes/user.routes.js";
import { notFoundHandler } from "./middlewares/not-found.middleware.js";
import { errorHandler } from "./middlewares/error.middleware.js";

export function createApp() {
  const app = express();

  app.disable("x-powered-by");

  const corsOrigins = (process.env.CORS_ORIGIN ?? "http://localhost:5173")
    .split(",")
    .map((origin) => origin.trim())
    .filter(Boolean);

  app.use(cors({ origin: corsOrigins }));
  app.use(express.json({ limit: "100kb" }));

  app.get("/api/health", async (req, res) => {
    res.json({ status: "ok" });
  });

  app.use("/api/users", userRouter);

  app.use(notFoundHandler);
  app.use(errorHandler);

  return app;
}
```

| Orden | Middleware        | Por qué en ese lugar                                      |
| ----- | ----------------- | --------------------------------------------------------- |
| 1     | `cors()`          | Tiene que responder el `OPTIONS` antes que cualquier ruta |
| 2     | `express.json()`  | Sin esto, `req.body` es `undefined`                       |
| 3     | Rutas             | Ya hay body parseado                                      |
| 4     | `notFoundHandler` | Después de todas las rutas                                |
| 5     | `errorHandler`    | Último, con cuatro argumentos                             |

#### `src/server.js`

```javascript
import "dotenv/config";
import { createApp } from "./app.js";
import { connectDatabase, sequelize } from "./config/database.js";

const port = Number(process.env.PORT ?? 3000);

const main = async () => {
  await connectDatabase();

  const app = createApp();
  const server = app.listen(port, () => {
    console.log(`Server is running on http://localhost:${port}`);
  });

  const shutdown = (signal) => {
    console.log(`\n${signal} received: shutting down`);

    server.close(async () => {
      await sequelize.close();
      process.exit(0);
    });
  };

  process.on("SIGINT", () => shutdown("SIGINT"));
  process.on("SIGTERM", () => shutdown("SIGTERM"));
};

main().catch((error) => {
  console.error("Failed to start server:", error.message);
  process.exit(1);
});
```

> [!TIP]
> Cerrar el pool con `sequelize.close()` es lo que evita que `Ctrl + C` deje conexiones colgadas en la base. En desarrollo no lo notás; en tests y en producción sí.

### 7.4 Pruebas con curl

Debajo de cada comando está **la salida real**, capturada contra PostgreSQL 16.15. Podés pegar el bloque completo de una vez: las líneas que empiezan con `#` son comentarios y no se ejecutan.

```bash
# 0. Opcional: desde TablePlus, para que los ids coincidan con los de abajo
# TRUNCATE users RESTART IDENTITY;
# INSERT INTO users (name, email) VALUES ('John Doe','john.doe@example.com'), ('Jane Smith','jane.smith@example.com');

# 1. Health
curl -s -w '\n%{http_code}\n' http://localhost:3000/api/health
# {"status":"ok"}
# 200

# 2. Listar
curl -s -w '\n%{http_code}\n' http://localhost:3000/api/users
# {"data":[{"id":1,"name":"John Doe","email":"john.doe@example.com"},{"id":2,"name":"Jane Smith","email":"jane.smith@example.com"}]}
# 200

# 3. Buscar sin distinguir mayúsculas
curl -s -w '\n%{http_code}\n' "http://localhost:3000/api/users/search?keywords=JANE"
# {"data":[{"id":2,"name":"Jane Smith","email":"jane.smith@example.com"}]}
# 200

# 4. Buscar sin keywords
curl -s -w '\n%{http_code}\n' "http://localhost:3000/api/users/search"
# {"data":[]}
# 200

# 5. Por id existente
curl -s -w '\n%{http_code}\n' http://localhost:3000/api/users/1
# {"data":{"id":1,"name":"John Doe","email":"john.doe@example.com"}}
# 200

# 6. Por id inexistente
curl -s -w '\n%{http_code}\n' http://localhost:3000/api/users/999
# {"error":{"code":"USER_NOT_FOUND","message":"User not found"}}
# 404

# 7. Id no numérico
curl -s -w '\n%{http_code}\n' http://localhost:3000/api/users/abc
# {"error":{"code":"INVALID_ID","message":"User id must be a positive integer"}}
# 400

# 8. Crear válido (con espacios y mayúsculas a propósito)
curl -s -w '\n%{http_code}\n' -X POST http://localhost:3000/api/users \
  -H 'Content-Type: application/json' \
  -d '{"name":"  Ada Lovelace ","email":" ADA@Example.com "}'
# {"data":{"id":3,"name":"Ada Lovelace","email":"ada@example.com"}}
# 201

# 9. Crear con email repetido
curl -s -w '\n%{http_code}\n' -X POST http://localhost:3000/api/users \
  -H 'Content-Type: application/json' \
  -d '{"name":"Ada","email":"ada@example.com"}'
# {"error":{"code":"EMAIL_TAKEN","message":"Email is already registered"}}
# 409

# 10. Crear inválido (devuelve todos los errores juntos)
curl -s -w '\n%{http_code}\n' -X POST http://localhost:3000/api/users \
  -H 'Content-Type: application/json' \
  -d '{"email":"no-es-email"}'
# {"error":{"code":"VALIDATION_ERROR","message":"Invalid request body","details":["name is a required field","email must be a valid email"]}}
# 400

# 11. Crear con una clave que el contrato no declara
curl -s -w '\n%{http_code}\n' -X POST http://localhost:3000/api/users \
  -H 'Content-Type: application/json' \
  -d '{"name":"Ada","email":"ada2@example.com","role":"admin"}'
# {"error":{"code":"VALIDATION_ERROR","message":"Invalid request body","details":["Unknown fields are not allowed"]}}
# 400

# 12. PATCH parcial
curl -s -w '\n%{http_code}\n' -X PATCH http://localhost:3000/api/users/1 \
  -H 'Content-Type: application/json' \
  -d '{"name":"John D."}'
# {"data":{"id":1,"name":"John D.","email":"john.doe@example.com"}}
# 200

# 13. PATCH con body vacío
curl -s -w '\n%{http_code}\n' -X PATCH http://localhost:3000/api/users/1 \
  -H 'Content-Type: application/json' \
  -d '{}'
# {"error":{"code":"VALIDATION_ERROR","message":"Invalid request body","details":["At least one field is required"]}}
# 400

# 14. PATCH con id inexistente
curl -s -w '\n%{http_code}\n' -X PATCH http://localhost:3000/api/users/999 \
  -H 'Content-Type: application/json' \
  -d '{"name":"Nadie"}'
# {"error":{"code":"USER_NOT_FOUND","message":"User not found"}}
# 404

# 15. DELETE
curl -s -w '\n%{http_code}\n' -X DELETE http://localhost:3000/api/users/3
#
# 204

# 16. JSON mal formado
curl -s -w '\n%{http_code}\n' -X POST http://localhost:3000/api/users \
  -H 'Content-Type: application/json' \
  -d '{"name":"Ada",}'
# {"error":{"code":"INVALID_JSON","message":"Request body is not valid JSON"}}
# 400

# 17. Ruta inexistente
curl -s -w '\n%{http_code}\n' http://localhost:3000/api/nope
# {"error":{"code":"ROUTE_NOT_FOUND","message":"Route not found"}}
# 404

# 18. Estado final (Ada fue borrada en el paso 15)
curl -s -w '\n%{http_code}\n' http://localhost:3000/api/users
# {"data":[{"id":1,"name":"John D.","email":"john.doe@example.com"},{"id":2,"name":"Jane Smith","email":"jane.smith@example.com"}]}
# 200
```

> [!NOTE]
> Los `id` de los ejemplos salen de una tabla recién sembrada. Si ya corriste la batería antes, reiniciá la tabla con los dos comandos del paso `0` para que coincidan. Los cuerpos y los códigos de estado no cambian.

> [!TIP]
> Guardá estas pruebas en un `api.http` y usalo con la extensión **REST Client** de VS Code. Te queda una colección viva que corre con un click.

### 7.5 Criterios de aceptación

- [ ] `npm install` instala `express`, `cors`, `yup`, `dotenv`, `sequelize`, `pg` y `pg-hstore`.
- [ ] `.env.example` está en el repo y `.env` está en `.gitignore`.
- [ ] La base corre en el contenedor definido en `docker-compose.yaml`, con el volumen declarado.
- [ ] La tabla `users` existe con `id`, `name`, `email`, `created_at` y `updated_at`.
- [ ] `npm run dev` conecta a la base y arranca en el puerto `3000`.
- [ ] Si la base no está arriba, el proceso falla al arrancar con un mensaje claro.
- [ ] `GET /api/health` responde `200` con `{ "status": "ok" }`.
- [ ] `GET /api/users` responde `200` con `{ data: [...] }` ordenado por `id`.
- [ ] `GET /api/users/search?keywords=` busca en `name` y `email` sin distinguir mayúsculas.
- [ ] `GET /api/users/search` declarado antes de `/:id` (un `GET /api/users/search` no devuelve `INVALID_ID`).
- [ ] `GET /api/users/:id` responde `200` si existe y `404` con `code: "USER_NOT_FOUND"` si no.
- [ ] `GET /api/users/abc` responde `400` con `code: "INVALID_ID"`.
- [ ] `POST /api/users` responde `201`, incluye header `Location` y devuelve el recurso.
- [ ] `POST` con email repetido responde `409` con `code: "EMAIL_TAKEN"`.
- [ ] `POST` inválido responde `400` con `details` y todos los errores juntos.
- [ ] `POST` con una clave desconocida responde `400` (no la ignora en silencio).
- [ ] `PATCH /api/users/:id` actualiza solo los campos enviados y responde `200`.
- [ ] `PATCH` con body vacío responde `400`.
- [ ] `DELETE /api/users/:id` responde `204` sin body.
- [ ] Los datos sobreviven a reiniciar el proceso de Node.
- [ ] Los datos persisten cuando se recrea el contenedor.
- [ ] Ninguna respuesta devuelve columnas internas ni el modelo crudo.
- [ ] Un JSON mal formado responde `400` con `code: "INVALID_JSON"`.
- [ ] Los errores inesperados responden `500` con `{ error: { code, message } }`.
- [ ] Ningún archivo de `src/` tiene credenciales hardcodeadas.
- [ ] El README del proyecto explica los pasos para levantarlo desde cero.
- [ ] `Ctrl + C` cierra el servidor y el pool sin dejar el proceso colgado.

### 7.6 Desafío: agregar password a los usuarios

**Objetivo:** que un usuario tenga contraseña, que se guarde de forma segura y que se pueda verificar en un login. La contraseña **no se guarda**: se guarda su hash.

**1. La columna en la base**

En TablePlus, sobre una tabla vacía o con datos descartables:

```sql
ALTER TABLE users ADD COLUMN password_hash VARCHAR(255) NOT NULL;
```

Si la tabla ya tiene filas, ese `NOT NULL` va a fallar: las filas existentes no tienen valor. Las dos opciones están en [5.4](#54-modificar-el-esquema-agregar-un-campo).

| Decisión                                | Por qué                                                                         |
| --------------------------------------- | ------------------------------------------------------------------------------- |
| Se llama `password_hash`, no `password` | El nombre tiene que decir qué contiene. `password` invita a guardar texto plano |
| `VARCHAR(255)`                          | Un hash bcrypt ocupa 60 caracteres; 255 deja aire                               |
| `NOT NULL`                              | Un usuario sin contraseña no puede loguearse                                    |

**2. El modelo**

```javascript
password_hash: {
  type: DataTypes.STRING(255),
  allowNull: false,
},
```

**3. El DTO de entrada**

El cliente manda `password`, nunca `password_hash`; `noUnknown(true)` ya rechaza el segundo.

```javascript
const password = yup.string().trim().min(8).max(72);

export const createUserDto = yup
  .object({
    name: name.required(),
    email: email.required(),
    password: password.required(),
  })
  .noUnknown(true, "Unknown fields are not allowed");

export const updateUserDto = yup
  .object({ name, email, password })
  .noUnknown(true, "Unknown fields are not allowed")
  .test(
    "at-least-one",
    "At least one field is required",
    (value) => Object.keys(value ?? {}).length > 0,
  );
```

> [!IMPORTANT]
> El máximo de 72 no es capricho: bcrypt **trunca** la contraseña a 72 bytes. Si aceptás más, dos contraseñas distintas que empiezan igual se consideran la misma.

**4. Hashear antes de guardar**

```bash
npm install bcrypt
```

```javascript
import bcrypt from "bcrypt";

const SALT_ROUNDS = 12;

export const create = async (req, res) => {
  const { password, ...data } = req.dto;

  const user = await User.create({
    ...data,
    password_hash: await bcrypt.hash(password, SALT_ROUNDS),
  });

  res
    .status(201)
    .location(`/api/users/${user.id}`)
    .json({ data: toUserResponse(user) });
};
```

`bcrypt.hash` genera un **salt** aleatorio por contraseña y lo guarda dentro del hash. Dos usuarios con la misma contraseña terminan con hashes distintos: eso invalida las tablas precalculadas (rainbow tables).

| Regla                       | Detalle                                                            |
| --------------------------- | ------------------------------------------------------------------ |
| Nunca guardar `password`    | Si la base se filtra, las contraseñas se filtran                   |
| Nunca usar `md5` ni `sha1`  | Son rápidos: se rompen por fuerza bruta en segundos                |
| No inventar el algoritmo    | `bcrypt`, `argon2` o `scrypt`. Son los tres aceptados              |
| `SALT_ROUNDS` entre 10 y 12 | Más rondas = más seguro y más lento. Con 12, un hash tarda ~250 ms |

**5. El login**

```javascript
// dtos/auth.dto.js
export const loginDto = yup
  .object({ email: email.required(), password: yup.string().trim().required() })
  .noUnknown(true, "Unknown fields are not allowed");
```

```javascript
// POST /api/auth/login  { email, password }
export const login = async (req, res) => {
  const { email: inputEmail, password } = req.dto;
  const user = await User.findOne({ where: { email: inputEmail } });

  if (!user || !(await bcrypt.compare(password, user.password_hash))) {
    throw new AppError("Invalid credentials", {
      status: 401,
      code: "INVALID_CREDENTIALS",
    });
  }

  res.json({ data: toUserResponse(user) });
};
```

| Regla                                                        | Por qué                                                                 |
| ------------------------------------------------------------ | ----------------------------------------------------------------------- |
| Un solo mensaje para email inexistente y password incorrecta | Si el mensaje cambia, el atacante descubre qué emails están registrados |
| `bcrypt.compare` y no `===`                                  | El hash lleva salt: se compara con la función                           |
| `401`, no `404`                                              | Un `404` también revela que el email no existe                          |
| Nunca devolver `password_hash`                               | Ni en el login, ni en el listado, ni en un error                        |

**6. Lo que no puede pasar**

- [ ] `GET /api/users` no devuelve `password_hash` en ninguna fila.
- [ ] `POST /api/users` no devuelve `password_hash`.
- [ ] Ningún `console.log` imprime la contraseña ni el hash.
- [ ] Si el body trae `password_hash`, responde `400` (lo bloquea `noUnknown`).
- [ ] En la base, la columna `password_hash` no contiene texto legible.
- [ ] Un login con email inexistente y uno con password incorrecta responden **exactamente lo mismo**.
- [ ] `PATCH /api/users/:id` con `password` nuevo vuelve a hashear; no guarda el valor recibido.

> [!TIP]
> Verificación rápida: `SELECT email, password_hash FROM users;` tiene que mostrar algo como `$2b$12$...`. Si ves la contraseña, el paso 4 no se ejecutó.

### 7.7 Errores frecuentes

| Síntoma                                                 | Causa probable                                                       | Solución                                                  |
| ------------------------------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------- |
| `column "createdAt" does not exist`                     | Falta `underscored: true` en `define` o en el modelo                 | Agregarlo; la tabla usa `created_at`                      |
| `relation "users" does not exist`                       | La tabla no se creó o `tableName` está mal                           | Ejecutar el DDL y revisar `tableName`                     |
| `relation "Users" does not exist`                       | Sequelize pluralizó y capitalizó solo                                | Declarar `tableName: "users"`                             |
| `ECONNREFUSED 127.0.0.1:5432`                           | El contenedor no está corriendo                                      | Levantar la base con `docker compose up -d --wait`        |
| `password authentication failed`                        | `.env` distinto a las variables del contenedor                       | Alinear credenciales o recrear el contenedor              |
| `SequelizeConnectionError: database ... does not exist` | `DB_NAME` no coincide con `POSTGRES_DB`                              | Corregir `.env` o crear la base                           |
| Los datos desaparecen al recrear el contenedor          | El volumen del compose no apunta al directorio de datos de la imagen | Revisar que `volumes` monte en `/var/lib/postgresql/data` |
| `GET /api/users/search` devuelve `INVALID_ID`           | `/:id` declarada antes de `/search`                                  | Reordenar las rutas                                       |
| `req.body` es `undefined`                               | Falta `express.json()`                                               | Registrarlo antes de las rutas                            |
| `req.dto` es `undefined`                                | Falta `validateBody(schema)` en la ruta                              | Agregar el middleware                                     |
| `Cannot use import statement outside a module`          | Falta `"type": "module"`                                             | Agregarlo y reiniciar                                     |
| `ERR_HTTP_HEADERS_SENT`                                 | El handler responde y además llama a `next()`                        | `return` al responder                                     |
| El `errorHandler` nunca corre                           | Tiene tres argumentos                                                | Declararlo con cuatro `(error, req, res, next)`           |
| `201` con el campo desconocido ignorado                 | `stripUnknown: true` sin `noUnknown`                                 | Usar `noUnknown(true)` y `stripUnknown: false`            |
| `SequelizeValidationError` como `500`                   | No se mapeó el error en el middleware                                | Mapearlo a `400`                                          |
| `409` no aparece y explota con `500`                    | No se captura `UniqueConstraintError`                                | Agregar el `instanceof` en `errorHandler`                 |
| `ECONNRESET` al apagar                                  | No se cerró el pool                                                  | `sequelize.close()` en el shutdown                        |
| `--watch` reinicia en loop                              | El proceso escribe archivos dentro del proyecto                      | Excluir logs/temporales o no escribir en el repo          |
| Todo responde `404`                                     | El `notFoundHandler` se registró antes de las rutas                  | Moverlo al final                                          |
| CORS falla solo en el navegador                         | Origen no incluido en `CORS_ORIGIN`                                  | Agregarlo separado por coma                               |

<sub>[⬆ Volver al índice](#contenido)</sub>

---

## 8. Referencias

- 📘 [PostgreSQL 16 — Documentation](https://www.postgresql.org/docs/16/index.html)
- 📘 [PostgreSQL — Data Types](https://www.postgresql.org/docs/16/datatype.html)
- 📘 [SQL Tutorial — W3Schools](https://www.w3schools.com/sql/)
- 📘 [Docker Official Image — postgres](https://hub.docker.com/_/postgres)
- 📘 [Docker — Get started](https://docs.docker.com/get-started/)
- 📘 [TablePlus — Documentation](https://docs.tableplus.com/)
- 📘 [Sequelize 6 — Getting Started](https://sequelize.org/docs/v6/getting-started/)
- 📘 [Sequelize 6 — Model Basics](https://sequelize.org/docs/v6/core-concepts/model-basics/)
- 📘 [Sequelize 6 — Validations & Constraints](https://sequelize.org/docs/v6/core-concepts/validations-and-constraints/)
- 📘 [Express 5 — Error Handling](https://expressjs.com/en/guide/error-handling.html)
- 📘 [Yup — README](https://github.com/jquense/yup)
- 📘 [Martin Fowler — Data Transfer Object](https://martinfowler.com/eaaCatalog/dataTransferObject.html)
- 📘 [The Twelve-Factor App — Config](https://12factor.net/config)

## Mentor

**Ariel Duarte** · Kopius · The Academy

<sub>[⬆ Volver al índice](#contenido)</sub>
