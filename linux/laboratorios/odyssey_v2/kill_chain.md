# Cyber Kill Chain — Odyssey v2

Modelo **Lockheed Martin Cyber Kill Chain** aplicado a la ruta de compromiso de este reto.  
El objetivo es documentar cada fase como si fuera un ataque real, no solo una secuencia de comandos.

---

## Las 7 fases

| Fase | Descripción general | Acción en este reto | Herramienta |
|------|--------------------|--------------------|-------------|
| **1. Reconocimiento** | El atacante recopila información sobre el objetivo | Escaneo de puertos y servicios; identificación de nginx, SSH y rutas web | `nmap -sV -sC -p-` |
| **2. Armamento** | El atacante prepara el arma o técnica de ataque | No se crea malware — el "arma" es el conocimiento de rutas y wordlists personalizadas | Wordlist de archivos ocultos, `steghide` |
| **3. Entrega** | El atacante envía el ataque al objetivo | Peticiones HTTP a rutas específicas; análisis de imágenes descargadas del servidor | `gobuster`, `curl`, `steghide` |
| **4. Explotación** | Se aprovecha la vulnerabilidad | Acceso directo a `/admin/.flag.txt` (misconfiguration nginx) + extracción de credenciales ocultas en imágenes JPEG | `curl`, `steghide extract` |
| **5. Instalación** | El atacante establece persistencia | No aplica en este reto (CTF de lectura, no de persistencia) | — |
| **6. C2 (Command & Control)** | El atacante controla el sistema comprometido | Conexión SSH interactiva como `root` al contenedor | `ssh root@<IP>` |
| **7. Acción sobre objetivos** | El atacante logra su objetivo final | Lectura de ambas flags: `/admin/.flag.txt` y `/root/.hide/.last/.flag.txt` | `cat`, `find` |

---

## Diagrama de la cadena

```
[nmap] → puertos 22, 80
    ↓
[gobuster] → /notes/, /admin/, /images/, /0-15/
    ↓
[note.txt] → clave: "1 3 11"
    ↓
[/1/ /3/ /11/ junk.txt] → base64/hex → 1234_sec | hoora! | california
    ↓                                                        ↓
[gobuster hidden] → /admin/.flag.txt              [steghide 1.jpg 3.jpg 11.jpg]
    ↓                                                        ↓
  FLAG 1                                        user:root / pass:!3QwX?j4
                                                             ↓
                                               [ssh root@<IP>] → /root/.hide/.last/.flag.txt
                                                             ↓
                                                           FLAG 2
```

---

## Lecciones aprendidas

1. **Un 403 no es un muro.** Bloquea el listado del directorio, pero si conoces el nombre exacto de un archivo (incluyendo los ocultos con `.`), puedes acceder directamente. Siempre enumerar archivos ocultos en directorios con 403.

2. **Los red herrings son parte del reto.** 13 de los 16 directorios decían "no soy clave" — la nota `1 3 11` era imprescindible para no perder tiempo. En pentesting real, saber filtrar el ruido es tan importante como encontrar los datos.

3. **Esteganografía como vector de exfiltración/ocultación.** Las credenciales estaban repartidas en tres imágenes distintas, cada una con una contraseña diferente. Sin los valores de los `junk.txt`, steghide no habría funcionado — el reto encadena los pasos.

4. **Base64 y hex no son cifrado.** Cualquier herramienta estándar los revierte en segundos. Nunca usar encodings como mecanismo de seguridad.

5. **Siempre revisar imágenes en un CTF.** Cuando un servidor web expone imágenes sin un propósito funcional claro, es una señal de que pueden contener datos ocultos (esteganografía, metadatos EXIF, archivos adjuntos).
