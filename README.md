# SGSII — Sistema de Gestión de Salas e Insumos Informáticos

MVP web monolítico para docentes y administradores: reservas, disponibilidad, inventario, préstamos, incidencias y reportes.

## Arquitectura

Es un único proyecto PHP; no hay un frontend y un backend separados. PHP genera las páginas, aplica las reglas y accede mediante PDO a uno de estos motores:

- **MySQL/MariaDB:** opción recomendada para XAMPP.
- **SQLite:** opción liviana para desarrollo y pruebas.

No se necesitan Node.js, npm ni Composer.

## Instalación automática en XAMPP

### Requisitos

1. Instalar XAMPP en Windows. La ubicación habitual es `C:\xampp`.
2. Mantener este proyecto en cualquier carpeta de trabajo; no es necesario copiarlo manualmente a `htdocs`.
3. Tener PowerShell disponible.

No es necesario crear la base en phpMyAdmin, configurar tablas, iniciar servicios manualmente ni inventar credenciales administrativas.

### Único comando de instalación

Abrir PowerShell dentro de la carpeta original del proyecto y ejecutar:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\install_xampp.ps1
```

El instalador hace una sola pregunta:

```text
Ruta donde está instalado XAMPP [C:\xampp]:
```

Si XAMPP está en `C:\xampp`, sólo hay que presionar Enter. También se puede proporcionar la ruta directamente y evitar la pregunta:

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\install_xampp.ps1 -XamppPath "D:\xampp"
```

### Qué realiza automáticamente

1. Localiza `php.exe`, `htdocs`, Apache y MySQL dentro de XAMPP.
2. Comprueba que `pdo_mysql` esté habilitado.
3. Copia o actualiza el proyecto en `C:\xampp\htdocs\sgsii`.
4. Copia sólo código, documentación y esquemas; nunca copia SQLite local ni `config.local.php`.
5. Inicia MySQL si está detenido.
6. Crea la base `sgsii` con la configuración estándar de un XAMPP nuevo: `127.0.0.1:3306`, usuario `root` y contraseña vacía.
7. Crea o completa las seis tablas MySQL con InnoDB y `utf8mb4`, sin borrar datos existentes.
8. Genera `config.local.php` para que la aplicación use MySQL.
9. Si no existe un administrador, crea `admin` con una contraseña aleatoria segura y la muestra una sola vez en la terminal.
10. Conserva el administrador y su contraseña en ejecuciones posteriores.
11. Verifica la conexión, las tablas y el administrador.
12. Ejecuta las 40 pruebas automatizadas cuando las extensiones SQLite de XAMPP están disponibles.
13. Inicia Apache si está detenido y comprueba la página real por HTTP.

Al finalizar debe aparecer:

```text
INSTALACIÓN/ACTUALIZACIÓN COMPLETADA
Abra: http://localhost/sgsii/
```

Guardar las credenciales iniciales mostradas y cambiar la contraseña después del primer ingreso.

## Actualizar el diseño o el código

Se utiliza el mismo instalador para una instalación nueva y para una actualización.

1. Editar los archivos originales. El aspecto visual está principalmente en:

   ```text
   public/assets/css/styles.css
   public/assets/js/app.js
   public/index.php
   ```

2. Volver a ejecutar:

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\scripts\install_xampp.ps1
   ```

3. Indicar la misma ruta de XAMPP.
4. Recargar el navegador con `Ctrl+F5`.

El instalador vuelve a copiar CSS, JavaScript, PHP, documentación y esquemas. No reemplaza `config.local.php`, no reinicia la base, no borra registros y no cambia la contraseña del administrador existente.

Para cambios normales de PHP, CSS o JavaScript **no hace falta reiniciar Apache ni MySQL**. Los archivos se leen nuevamente en cada petición; `Ctrl+F5` evita que el navegador conserve CSS o JavaScript anterior. Sólo se necesita reiniciar Apache cuando se modifica `php.ini` o la configuración del propio servidor. Si Apache o MySQL estaban detenidos, el instalador intenta iniciarlos.

## Configuración avanzada de MySQL

El modo automático presupone un XAMPP nuevo con `root` sin contraseña. Si MySQL fue personalizado, se pueden definir estas variables antes de ejecutar `install_mysql.php`:

```powershell
$env:SGSII_DB_HOST="127.0.0.1"
$env:SGSII_DB_PORT="3306"
$env:SGSII_DB_NAME="sgsii"
$env:SGSII_DB_USER="usuario_mysql"
$env:SGSII_DB_PASSWORD="contraseña_mysql"
C:\xampp\php\php.exe scripts\install_mysql.php
```

Una vez creado `config.local.php`, las actualizaciones automáticas reutilizan esa configuración.

## Ejecución alternativa con SQLite

Si no existe `config.local.php`, la aplicación usa SQLite por defecto:

```text
php scripts/init_db.php
php scripts/create_admin.php admin "Nombre" "Apellido" "UnaClaveSegura123!"
php -S 127.0.0.1:8000 -t public
```

Abrir <http://127.0.0.1:8000>. Se requiere PHP 8.1+ con `pdo_sqlite` y `sqlite3`.

## Crear administradores adicionales

`create_admin.php` utiliza automáticamente MySQL o SQLite según `config.local.php`:

```text
C:\xampp\php\php.exe scripts\create_admin.php usuario "Nombre" "Apellido" "UnaClaveSegura123!"
```

## Pruebas y verificación

Suite funcional aislada:

```text
php tests/run.php
```

Verificación de la base configurada:

```text
php scripts/check_install.php
```

## Solución de problemas en XAMPP

- **No se encuentra XAMPP:** introducir la carpeta que contiene `xampp-control.exe`, `php`, `apache`, `mysql` y `htdocs`.
- **`could not find driver`:** habilitar `extension=pdo_mysql` en `xampp\php\php.ini`.
- **MySQL no inicia en 3306:** abrir XAMPP Control Panel y revisar si otro MySQL ocupa el puerto.
- **Apache no inicia en 80:** revisar IIS u otro servidor que use el puerto 80.
- **La instalación existente no se reconoce:** confirmar que `htdocs\sgsii\public\index.php` exista.
- **Un cambio visual no aparece:** usar `Ctrl+F5`; no es necesario borrar la base ni reinstalar XAMPP.
- **Se modificó `php.ini`:** reiniciar Apache desde XAMPP Control Panel.

## Estructura principal

```text
index.php                  Redirección simple para XAMPP
public/index.php           Entrada web y vistas
public/assets/             CSS y JavaScript
src/App.php                Reglas de negocio
src/Database.php           Configuración y conexión PDO
src/Security.php           Sesiones, CSRF y escape
database/schema.mysql.sql  Esquema MySQL/MariaDB
database/schema.sql        Esquema SQLite
scripts/install_xampp.ps1  Copia, actualización e instalación
scripts/install_mysql.php  Instalación MySQL idempotente
scripts/check_install.php  Verificación de la instalación
tests/                     Suite automatizada
```

## Seguridad

Las consultas usan PDO preparado. Las contraseñas se guardan con hash, las acciones mutables requieren CSRF y los permisos se validan en el servidor. `config.local.php` contiene las credenciales locales y está excluido de Git y de la copia desde el proyecto original.

XAMPP debe utilizarse en un entorno local; este instalador no configura un servidor público de producción.

Más información: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md), [`docs/SECURITY.md`](docs/SECURITY.md) y [`docs/REQUIREMENTS.md`](docs/REQUIREMENTS.md).
