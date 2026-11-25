# Instalar QR Studio

QR Studio es un generador de códigos QR con cuentas de usuario: interfaz web, servidor en Python (FastAPI) y base
de datos SQLite (un solo archivo). Esta guía explica cómo instalarlo en un servidor **Debian/Ubuntu** (con el
script o a mano), con **Docker**, y en **Windows**.

## Qué hay en este repositorio

| Archivo | Para qué sirve |
| --- | --- |
| `Instalar_QR_Studio_SQLite_GitHub-Token_v1.0.sh` | **El instalador.** Un script con menú que instala, actualiza, respalda y desinstala QR Studio (normal o con Docker). |
| `QR_Studio_vDDMMAAAAHHMM.zip` (dentro de su carpeta) | El **programa**: solo el código de la página, sin datos de nadie. Cada subida es una versión; el instalador usa la más reciente. |
| `INSTALACION.md` | Esta guía. |

> El `.zip` **no** incluye el instalador: es solo el programa. Por eso el script se descarga aparte (opción 1) o
> se instala a mano (opción 2).

---

## Opción 1 · Con el script (recomendada, Debian/Ubuntu)

Hace todo: instala las dependencias, baja la última versión de este repositorio, crea el servicio y lo deja
funcionando. Más tarde sirve también para actualizar, respaldar y desinstalar.

**Requisitos:** Debian 12 (o una distribución con `apt` y `systemd`), acceso `root` o `sudo`, e internet.
El script instala lo que falte (Python 3.10 o superior, `python3-venv`, `git`, `rsync`, `unzip`, `curl`…).

### 1. Descargar el script

```bash
curl -fLO https://raw.githubusercontent.com/llancor/qr_studio/main/Instalar_QR_Studio_SQLite_GitHub-Token_v1.0.sh
```

(o con `wget`, o con «Code → Download ZIP» en la página del repositorio, o `git clone https://github.com/llancor/qr_studio.git`).

> Es buena costumbre leer un script antes de ejecutarlo como administrador:
> `less Instalar_QR_Studio_SQLite_GitHub-Token_v1.0.sh`

### 2. Ejecutarlo

```bash
sudo bash Instalar_QR_Studio_SQLite_GitHub-Token_v1.0.sh
```

Aparece un menú. Para una instalación normal:

1. Elige **1) Instalación normal (Python + servicio en Debian)**.
2. Elige **1. Instalar dependencias** (solo la primera vez en ese servidor).
3. Elige **3. Instalar desde GitHub**. Si este repositorio es **público**, deja el usuario vacío y pulsa Enter;
   si es **privado**, escribe tu usuario y un token con permiso de lectura de contenido (no se muestra ni se guarda).
   Si hay varias versiones, te deja elegir; por defecto baja la más reciente.
4. Al terminar muestra las direcciones para abrirlo, por ejemplo `http://IP-DEL-SERVIDOR:8000/`.

Si el servidor ya tiene el `.zip` o la carpeta del programa, en el paso 3 usa **2. Instalar desde la carpeta
(código o .zip)** y escribe su ruta.

### 3. Crear tu cuenta de administrador

Abre la dirección en el navegador. **La primera cuenta que se crea es la del administrador.** Desde **Ajustes**
podrás crear usuarios, decidir si cualquiera puede registrarse, personalizar el aspecto, poner el logo y la imagen
de la página de acceso, y configurar el correo (SMTP) para recuperar contraseñas.

### Qué deja instalado

| Qué | Dónde |
| --- | --- |
| Programa | `/opt/qr-studio/app` (entorno de Python en `/opt/qr-studio/venv`) |
| Configuración | `/etc/qr-studio/qr-studio.env` |
| Base de datos (usuarios, sesiones, ajustes) | `/var/lib/qr-studio/app.db` |
| Servicio | `qr-studio` (systemd; arranca con el equipo) |
| Respaldos | `/var/backups/qr-studio` |

### Otras cosas del menú

Ver estado, iniciar, detener y reiniciar; ver registros; **cambiar el puerto**; cambiar la dirección de escucha y
las **cookies seguras (HTTPS)**; **actualizar** desde la carpeta o desde GitHub (la base de datos se conserva y, si
la versión nueva no arranca, vuelve sola a la anterior); **respaldo completo** y restaurarlo; **desinstalar**; y
**«Usuarios: crear y restablecer contraseñas»**, para recuperar el acceso si nadie puede entrar.

### Variables opcionales

Se definen con `export` y se pasan al script con `sudo -E` (o se ejecuta directamente como `root`):

```bash
export QR_REPO_BRANCH=main QR_DIR=/srv/qr-studio
sudo -E bash Instalar_QR_Studio_SQLite_GitHub-Token_v1.0.sh
```

| Variable | Qué cambia | Por defecto |
| --- | --- | --- |
| `QR_REPO_URL` | Repositorio del que baja el programa | `https://github.com/llancor/qr_studio.git` |
| `QR_REPO_BRANCH` | Rama | `main` |
| `QR_REPO_CARPETA` | Carpeta del repositorio donde están los `.zip` (vacía = todo el repositorio) | vacía |
| `QR_DIR` | Dónde se instala el programa | `/opt/qr-studio` |
| `QR_DATOS` | Dónde queda la base de datos | `/var/lib/qr-studio` |
| `QR_RESPALDOS` | Dónde se guardan los respaldos | `/var/backups/qr-studio` |

---

## Opción 2 · A tu manera, sin el script (Debian/Ubuntu)

Si prefieres no usar el instalador, esto es lo que hace, paso a paso. Necesitas **Python 3.10 o superior**.

```bash
# 1) Dependencias del sistema
sudo apt-get update
sudo apt-get install -y python3 python3-venv python3-pip unzip curl

# 2) Descargar la última versión del programa (el .zip de este repositorio) y descomprimirla
#    (cambia la ruta por la del .zip más reciente)
curl -fLO "https://github.com/llancor/qr_studio/raw/main/QR_Studio_vDDMMAAAAHHMM/QR_Studio_vDDMMAAAAHHMM.zip"
sudo mkdir -p /opt
sudo unzip QR_Studio_vDDMMAAAAHHMM.zip -d /opt          # crea /opt/qr-studio con app/ y requirements.txt

# 3) Entorno de Python y librerías
cd /opt/qr-studio
sudo python3 -m venv venv
sudo venv/bin/pip install -r requirements.txt
sudo venv/bin/python -m compileall -q app      # opcional: el servicio no puede escribir en esta carpeta

# 4) Probarlo (Ctrl+C para parar). La base de datos se crea sola en la ruta de APP_DB_PATH
sudo mkdir -p /var/lib/qr-studio
APP_DB_PATH=/var/lib/qr-studio/app.db venv/bin/uvicorn --factory app.main:create_app --host 0.0.0.0 --port 8000
```

Abre `http://IP-DEL-SERVIDOR:8000/` y crea la primera cuenta (será la del administrador). Para comprobar que
responde: `curl http://127.0.0.1:8000/healthz`.

### Dejarlo como servicio (arranca con el equipo)

```bash
# Usuario sin privilegios y permisos
sudo useradd --system --home-dir /var/lib/qr-studio --no-create-home --shell /usr/sbin/nologin qrstudio
sudo chown -R qrstudio:qrstudio /var/lib/qr-studio
sudo chmod 750 /var/lib/qr-studio

# Configuración
sudo mkdir -p /etc/qr-studio
sudo tee /etc/qr-studio/qr-studio.env >/dev/null <<'EOF'
APP_HOST=0.0.0.0
APP_PORT=8000
APP_DB_PATH=/var/lib/qr-studio/app.db
APP_COOKIE_SECURE=false
EOF
sudo chgrp qrstudio /etc/qr-studio && sudo chmod 640 /etc/qr-studio/qr-studio.env

# Servicio de systemd
sudo tee /etc/systemd/system/qr-studio.service >/dev/null <<'EOF'
[Unit]
Description=QR Studio (generador de códigos QR)
After=network.target

[Service]
Type=simple
User=qrstudio
Group=qrstudio
WorkingDirectory=/opt/qr-studio
EnvironmentFile=/etc/qr-studio/qr-studio.env
ExecStart=/opt/qr-studio/venv/bin/uvicorn --factory app.main:create_app --host ${APP_HOST} --port ${APP_PORT}
Restart=on-failure
RestartSec=3
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
PrivateDevices=true
ReadWritePaths=/var/lib/qr-studio

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now qr-studio
systemctl status qr-studio
```

**Actualizar a mano:** para el servicio (`sudo systemctl stop qr-studio`), haz una copia de
`/var/lib/qr-studio` (ahí está tu base de datos), descomprime el `.zip` nuevo **encima** de `/opt/qr-studio`
(no toca `/var/lib/qr-studio`), ejecuta otra vez `venv/bin/pip install -r requirements.txt` y arranca
(`sudo systemctl start qr-studio`).

---

## Opción 3 · Con Docker

El script sabe instalar con Docker y se ocupa de todo (Docker, compose, imagen, volumen para la base de datos):

```bash
sudo bash Instalar_QR_Studio_SQLite_GitHub-Token_v1.0.sh
```

Elige **2) Instalación con Docker**, luego **1. Instalar dependencias** (instala Docker y compose) y **2. Instalar con
Docker** (desde carpeta, `.zip` o GitHub). Queda en `/opt/qr-studio-docker` (código en `app/`, configuración en
`.env`) y la base de datos en `/opt/qr-studio-docker/datos/app.db`, fuera de la imagen. Se actualiza y se respalda
desde el mismo menú.

---

## Opción 4 · Windows

Necesitas **Python 3.10 o superior** ([python.org](https://www.python.org/downloads/)). Descomprime el `.zip`
(queda una carpeta `qr-studio`) y, en PowerShell, dentro de esa carpeta:

```powershell
py -m venv venv
venv\Scripts\pip install -r requirements.txt
$env:APP_DB_PATH = "$PWD\datos\app.db"
venv\Scripts\uvicorn --factory app.main:create_app --host 127.0.0.1 --port 8000
```

Abre `http://localhost:8000/`. Esto sirve para probarlo en tu propio equipo; para dejarlo funcionando siempre
en un servidor usa Debian (opciones 1 a 3).

---

## Después de instalar

- **Puerto y acceso desde otros equipos:** si el servidor tiene firewall activo, abre el puerto
  (por ejemplo `sudo ufw allow 8000/tcp`).
- **HTTPS:** QR Studio habla HTTP. Para HTTPS pon delante un proxy inverso (Caddy, nginx…), escucha solo en
  `127.0.0.1` (`APP_HOST=127.0.0.1`) y activa `APP_COOKIE_SECURE=true`. Con cookies seguras y HTTP, el inicio de
  sesión no funciona.
- **QR dinámicos y enlaces del correo:** fija la dirección pública con `APP_PUBLIC_URL=https://qr.midominio.com`
  *antes* de imprimir un QR dinámico: si esa dirección cambia, los ya impresos dejan de funcionar.
- **Recuperar contraseñas por correo:** en **Ajustes → Correo (SMTP)** el formulario ya propone los valores de
  Gmail (`smtp.gmail.com`, puerto 587; con Gmail hace falta una «contraseña de aplicación»).
- **Respaldos:** copia la carpeta de datos completa (`/var/lib/qr-studio`, o `/opt/qr-studio-docker/datos` con
  Docker: la base de datos y los archivos de los usuarios) o usa el respaldo del menú del script.
- **Olvidaste la contraseña del administrador:** menú del script → «Usuarios: crear y restablecer contraseñas», o
  a mano: `cd /opt/qr-studio && sudo -u qrstudio env APP_DB_PATH=/var/lib/qr-studio/app.db venv/bin/python -m app.usuarios restablecer correo@dominio.com`

### Configuración (variables `APP_*`)

Van en `/etc/qr-studio/qr-studio.env` (reinicia con `sudo systemctl restart qr-studio` tras cambiarlas).

| Variable | Qué hace | Por defecto |
| --- | --- | --- |
| `APP_HOST`, `APP_PORT` | Dirección y puerto de escucha | `0.0.0.0`, `8000` |
| `APP_DB_PATH` | Archivo de la base de datos | `/var/lib/qr-studio/app.db` |
| `APP_COOKIE_SECURE` | `true` solo si se sirve por HTTPS | `false` |
| `APP_PUBLIC_URL` | Dirección pública (enlaces de QR dinámicos y del correo) | vacía |
| `APP_REGISTRATION_OPEN` | Valor inicial de «cualquiera puede crear una cuenta» (luego manda Ajustes) | `true` |
| `APP_SESSION_DAYS` | Días que dura una sesión | `14` |
| `APP_MIN_PASSWORD_LENGTH` | Largo mínimo de contraseña | `8` |
| `APP_LOGIN_MAX_ATTEMPTS`, `APP_LOGIN_WINDOW_SECONDS` | Intentos fallidos permitidos y ventana en segundos | `5`, `300` |
| `APP_QR_QUOTA_MB`, `APP_PAGES_QUOTA_MB` | Espacio máximo por usuario (QR guardados y archivos de páginas) | `200` |
| `APP_DYNAMIC_QR`, `APP_PAGES` | Apagar los QR dinámicos o las páginas con `false` | `true` |

## Desinstalar

Desde el menú del script (**Desinstalar**), o a mano:

```bash
sudo systemctl disable --now qr-studio
sudo rm -f /etc/systemd/system/qr-studio.service && sudo systemctl daemon-reload
sudo rm -rf /opt/qr-studio /etc/qr-studio
sudo rm -rf /var/lib/qr-studio && sudo userdel qrstudio      # esto borra la base de datos
```

## Problemas frecuentes

- **«No se pudo consultar GitHub»:** revisa la conexión y que la rama exista. Si el repositorio es privado, el
  token necesita permiso de lectura de contenido.
- **«No hay ningún QR_Studio_v*.zip»:** el repositorio debe tener al menos un `QR_Studio_vDDMMAAAAHHMM.zip` (12
  cifras) en la rama indicada, o la carpeta del programa (`app/main.py`).
- **Entro pero se cierra la sesión / no inicia sesión:** si usas HTTPS pon `APP_COOKIE_SECURE=true`; si usas
  HTTP déjalo en `false`.
- **No abre desde otro equipo:** firewall (`ufw allow 8000/tcp`) o `APP_HOST=127.0.0.1` (solo local).
- **El servicio no arranca:** `journalctl -u qr-studio -n 50 --no-pager`.

