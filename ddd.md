

# ej2:

```sql
CREATE USER ejercicio2 IDENTIFIED BY "a";


CREATE PROFILE ej2_profile LIMIT
    PASSWORD_LIFE_TIME 1
    PASSWORD_REUSE_TIME UNLIMITED
    PASSWORD_REUSE_MAX UNLIMITED
    PASSWORD_VERIFY_FUNCTION ej2_verificar;

CREATE OR REPLACE FUNCTION ej2_verificar(username VARCHAR2, password VARCHAR2, old_password VARCHAR2) RETURN BOOLEAN IS
BEGIN
    IF LENGTH(password) < 10 THEN
        RETURN FALSE;
    ELSIF NOT REGEXP_LIKE(password, '^[A-Z]{6}') THEN
        RETURN FALSE;
    ELSIF NOT REGEXP_LIKE(password, '[0-9]{2}$') THEN
        RETURN FALSE;
    ELSE
        RETURN TRUE;
    END IF;
END;
/


GRANT CREATE USER TO ejercicio2;
GRANT ALTER USER TO ejercicio2;
GRANT DROP USER TO ejercicio2;
GRANT CREATE PROFILE TO ejercicio2;
GRANT ALTER PROFILE TO ejercicio2;
GRANT DROP PROFILE TO ejercicio2;


ALTER USER Jose PROFILE ej2_profile;
```













# ej3:

```sql
CREATE OR REPLACE PROCEDURE Show_User_Privs (p_username IN VARCHARQ)
IS
    -- Privilegios otorgados directamente al usuario
    CURSOR c_direct_privs IS
        SELECT tp.owner AS propietario, tp.table_name AS nombre_procedimiento, tp.privilege AS privilegio, tp.grantor AS otorgado_por
        FROM dba_tab_privs tp
        WHERE tp.grantee = p_username AND tp.privilege IN ('EXECUTE', 'DEBUG') AND tp.table_name IS NOT NULL;

    -- Privilegios otorgados a través de roles
    CURSOR c_role_privs IS
        SELECT tp.owner AS propietario, tp.table_name AS nombre_procedimiento, tp.privilege AS privilegio, rp.granted_role AS otorgado_por
        FROM dba_tab_privs tp JOIN dba_role_privs rp ON tp.grantee = rp.granted_role
        WHERE rp.grantee = p_username AND tp.privilege IN ('EXECUTE', 'DEBUG') AND tp.table_name IS NOT NULL;

BEGIN
    DBMS_OUTPUT.PUT_LINE('Privilegios para el usuario ' || p_username || ':');

    -- Mostrar privilegios otorgados directamente
    FOR x IN c_direct_privs LOOP
        DBMS_OUTPUT.PUT_LINE('Procedimiento: ' || x.nombre_procedimiento || ', Privilegio: ' || x.privilegio || 
                             ', Propietario: ' || x.propietario || ', Otorgado por: ' || x.otorgado_por || ' (Directo)');
    END LOOP;

    -- Mostrar privilegios otorgados a través de roles
    FOR x IN c_role_privs LOOP
        DBMS_OUTPUT.PUT_LINE('Procedimiento: ' || x.nombre_procedimiento || ', Privilegio: ' || x.privilegio || 
                             ', Propietario: ' || x.propietario || ', Otorgado por: ' || x.otorgado_por || ' (Role-Based)');
    END LOOP;
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLCODE || ' - ' || SQLERRM);
END Show_User_Privs;
/
```















# ej5:

En PostgreSQL, el concepto de tablespaces puede ser un poco diferente al de otros sistemas de gestión de bases de datos como Oracle, ya que los tablespaces en PostgreSQL son principalmente lugares donde se almacenan los datos físicamente y no están ligados directamente a los privilegios sobre las tablas. Sin embargo, podemos escribir una consulta que relacione los tablespaces con los permisos de usuario sobre las tablas en esos tablespaces.

Para obtener un listado de los tablespaces para los cuales un usuario específico tiene permisos de lectura (SELECT) o escritura (INSERT, UPDATE, DELETE) en alguna de las tablas alojadas, podemos utilizar las vistas del sistema de PostgreSQL. Vamos a combinar información de varias vistas de sistema para conseguir el resultado deseado.

Aquí está la consulta SQL que puede usar en PostgreSQL para obtener esta información:

```sql
SELECT DISTINCT pg_tablespace.spcname AS tablespace, pg_class.relname AS table_name
FROM pg_tablespace JOIN pg_class ON pg_class.reltablespace = pg_tablespace.oid JOIN pg_namespace  ON pg_namespace.oid = pg_class.relnamespace LEFT JOIN pg_roles ON pg_roles.rolname = 'nombre_usuario' LEFT JOIN information_schema.role_table_grants ON role_table_grants.table_name = pg_class.relname AND role_table_grants.grantee = pg_roles.rolname
WHERE (role_table_grants.privilege_type IN ('SELECT', 'INSERT', 'UPDATE', 'DELETE') OR (pg_class.relkind = 'r' AND pg_class.relowner = pg_roles.oid)) AND pg_namespace.nspname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_tablespace.spcname;
```

### Descripción de la consulta:

1. **FROM Clauses**:
   - `pg_tablespace`: Esta vista contiene información sobre los tablespaces.
   - `pg_class`: Esta vista contiene información sobre las tablas, incluyendo a qué tablespace pertenecen.
   - `pg_namespace`: Esta vista relaciona las tablas con sus esquemas.

2. **JOIN Conditions**:
   - Las tablas se unen con sus respectivos tablespaces y esquemas. Los `LEFT JOIN` se utilizan para incluir los roles y los privilegios correspondientes.

3. **WHERE Clause**:
   - Filtra los privilegios específicos (SELECT, INSERT, UPDATE, DELETE) que tienen relevancia para los permisos de lectura y escritura.
   - Excluye las tablas de los esquemas internos de PostgreSQL (`pg_catalog` y `information_schema`) que son de gestión interna y no suelen ser relevantes para este tipo de consultas.

4. **Privileges Check**:
   - Verifica si el rol actual tiene derechos de SELECT, INSERT, UPDATE, o DELETE sobre las tablas.

Esta consulta necesita ser ajustada con el nombre del usuario específico en la cláusula `pg_roles.rolname = 'nombre_usuario'`. Reemplaza `'nombre_usuario'` con el nombre del usuario que estás investigando.











# ej mongo:

Para completar el ejercicio en MongoDB, primero tendrás que crear los usuarios y configurar las colecciones y permisos como indicas. Suponiendo que tienes MongoDB instalado y estás trabajando desde un entorno con acceso a la línea de comandos de MongoDB (`mongo`), aquí están los pasos y comandos que debes seguir:

### 1. Crear los usuarios:
Primero, conectaré con la base de datos como un usuario administrador para crear los cuatro usuarios con sus respectivos permisos.

```sh
mongo
```

Ahora dentro del shell de MongoDB:

```javascript
use admin

// Crear usuario Ana
db.createUser({
    user: "Ana",
    pwd: "password_ana", // Cambia las contraseñas según las políticas de seguridad de tu entorno
    roles: [
        { role: "read", db: "tu_base_de_datos", collection: "Discos" },
        { role: "read", db: "tu_base_de_datos", collection: "Libros" }
    ]
});

// Crear usuario Eva
db.createUser({
    user: "Eva",
    pwd: "password_eva",
    roles: [
        { role: "readWrite", db: "tu_base_de_datos", collection: "Juegos" },
        { role: "readWrite", db: "tu_base_de_datos", collection: "Discos" },
        { role: "readWrite", db: "tu_base_de_datos", collection: "Libros" }
    ]
});

// Crear usuario Rafa
db.createUser({
    user: "Rafa",
    pwd: "password_rafa",
    roles: [
        { role: "readWrite", db: "tu_base_de_datos", collection: "Libros" },
        { role: "read", db: "tu_base_de_datos", collection: "Discos" },
        { role: "dbAdmin", db: "tu_base_de_datos", collection: "Juegos" }
    ]
});

// Crear usuario Juan
db.createUser({
    user: "Juan",
    pwd: "password_juan",
    roles: [
        { role: "dbAdmin", db: "tu_base_de_datos", collection: "Juegos" }
    ]
});
```

### 2. Crear las colecciones y documentos:
Suponiendo que tu nombre es `tu_nombre` y que ya te conectaste como un usuario con derechos de administración:

```javascript
use tu_base_de_datos

// Crear colecciones y insertar documentos
db.createCollection("Discos")
db.Discos.insertMany([{nombre: "Album1", artista: "Artista1", año: 2020}, {nombre: "Album2", artista: "Artista2", año: 2021}])

db.createCollection("Libros")
db.Libros.insertMany([{titulo: "Libro1", autor: "Autor1", páginas: 300}, {titulo: "Libro2", autor: "Autor2", páginas: 250}])

db.createCollection("Juegos")
db.Juegos.insertMany([{nombre: "Juego1", desarrollador: "Dev1", plataforma: "PC"}, {nombre: "Juego2", desarrollador: "Dev2", plataforma: "Xbox"}])
```

### 3. Verificación del correcto funcionamiento de los permisos:
Para probar y demostrar el correcto funcionamiento de los permisos de cada usuario, deberías iniciar sesión como cada uno de ellos y tratar de realizar las acciones para las cuales tienen y no tienen permisos. Por ejemplo, intenta que Eva añada un documento a `Libros`, o que Rafa modifique un documento en `Libros`, y luego intenta que Ana lea la colección `Juegos`.

```sh
mongo -u Eva -p password_eva --authenticationDatabase tu_base_de_datos
```
Dentro de MongoDB:

```javascript
// Eva intenta añadir un documento en Libros
db.Libros.insertOne({titulo: "Libro3", autor: "Autor3", páginas: 150});

// Eva intenta modificar un documento en Juegos
db.Juegos.updateOne({nombre: "Juego1"}, {$set: {plataforma: "PS5"}});
```

Repite pasos similares con los otros usuarios para asegurar que los permisos están correctamente configurados. Si algún comando no funciona como se espera, revisa los permisos asignados y asegúrate de que están correctamente establecidos según tus requisitos.


pass mongodb: DXtBvuaRtB0QZyzV









# ej1:

### 1. Crear la base de datos y las tablas:

Primero, como un usuario con privilegios de superusuario en PostgreSQL, ejecutaremos los siguientes comandos para establecer la base de datos y las tablas:

```sql
-- Conectar a PostgreSQL con un superusuario, por ejemplo, psql -U postgres
-- Crear la base de datos
CREATE DATABASE Examen;

-- Conectar a la base de datos Examen
\c Examen

-- Crear las tablas Prueba1 y Prueba2
CREATE TABLE Prueba1 (
    id SERIAL PRIMARY KEY,
    dato VARCHAR(255)
);

CREATE TABLE Prueba2 (
    id SERIAL PRIMARY KEY,
    dato VARCHAR(255)
);

-- Insertar dos registros en Prueba1
INSERT INTO Prueba1 (dato) VALUES ('Dato 1'), ('Dato 2');
```

### 2. Crear el usuario con restricciones específicas:

Continuamos con la creación del usuario y la asignación de permisos:

```sql
-- Crear usuario
CREATE USER usuario_examen WITH PASSWORD 'password_temporal' LOGIN;

-- Privilegios de lectura sobre Prueba1 y escritura sobre Prueba2
GRANT SELECT ON Prueba1 TO usuario_examen;
GRANT INSERT ON Prueba2 TO usuario_examen;

-- Permitir que el usuario cree vistas
GRANT CREATE ON DATABASE Examen TO usuario_examen;

-- Permitir que el usuario cree funciones y procedimientos
GRANT CREATE ROUTINE ON DATABASE Examen TO usuario_examen;

-- Ajustar la configuración para que cambie la contraseña en el primer ingreso
ALTER USER usuario_examen PASSWORD EXPIRE;
```

### 3. Restricción de número de consultas por hora:

Esta funcionalidad específica no es directamente soportada por PostgreSQL sin ayuda externa de herramientas adicionales o a través de un disparador (trigger) personalizado que podría ser complejo y no necesariamente efectivo. Una alternativa es gestionar esto a nivel de aplicación o mediante un middleware que controle la tasa de solicitudes.

### 4. Pruebas y validación:

Finalmente, realiza pruebas de conexión con el usuario para asegurarte de que todos los privilegios y restricciones están funcionando como se espera. Conéctate con el usuario y trata de realizar operaciones fuera de las concedidas para confirmar que las restricciones están en vigor:

```sql
-- Conectar usando el usuario nuevo
psql -U usuario_examen -d Examen

-- Intentar seleccionar de Prueba1 (debería ser permitido)
SELECT * FROM Prueba1;

-- Intentar insertar en Prueba1 (debería ser rechazado)
INSERT INTO Prueba1 (dato) VALUES ('Dato no permitido');

-- Intentar insertar en Prueba2 (debería ser permitido)
INSERT INTO Prueba2 (dato) VALUES ('Dato 3');

-- Intentar crear una tabla nueva (debería ser rechazado)
CREATE TABLE Prueba3 (id SERIAL);
```

Con estos pasos, has creado una configuración en PostgreSQL que se aproxima a los requerimientos detallados. Recuerda considerar aspectos de seguridad adicionales como SSL/TLS y monitoreo de actividad de base de datos para asegurar un entorno robusto y seguro.
