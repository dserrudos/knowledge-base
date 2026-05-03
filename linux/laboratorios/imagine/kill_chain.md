# Cyber Kill Chain — Imagine

Modelo **Lockheed Martin Cyber Kill Chain** aplicado a la ruta de compromiso de este reto.  
El objetivo es documentar cada fase como si fuera un ataque real, no solo una secuencia de comandos.

---

## Las 7 fases

| Fase | Descripción general | Acción en este reto | Herramienta |
|------|--------------------|--------------------|-------------|
| **1. Reconocimiento** | El atacante recopila información sobre el objetivo | Escaneo de puertos; identificación de Apache y SSH en puerto no estándar | `nmap -sV -sC -p-` |
| **2. Armamento** | El atacante prepara la técnica de ataque | Identificación del fichero con credenciales y decodificación Base64 | `ffuf`, `curl`, `base64` |
| **3. Entrega** | El atacante envía el ataque al objetivo | No aplica — las credenciales ya están expuestas en el servidor web | — |
| **4. Explotación** | Se aprovecha la vulnerabilidad | Uso de las credenciales obtenidas para autenticarse por SSH | `ssh` |
| **5. Instalación** | El atacante establece persistencia | No aplica en este reto (CTF de lectura, no de persistencia) | — |
| **6. C2 (Command & Control)** | El atacante controla el sistema comprometido | Shell interactiva como `jude` vía SSH | `ssh jude@<IP> -p 2222` |
| **7. Acción sobre objetivos** | El atacante logra su objetivo final | Lectura de la flag en el directorio home del usuario comprometido | `cat ~/.flag.txt` |

---

## Diagrama de la cadena

```
[nmap] → puerto 80 (Apache) y puerto 2222 (SSH)
    ↓
[ffuf] → descubre ficheros web con información sensible
    ↓
[curl + base64 -d] → decodifica nombre y contenido del fichero
    ↓
credenciales SSH en texto claro
    ↓
[ssh jude@<IP> -p 2222]
    ↓
[cat ~/.flag.txt]
    ↓
FLAG 1 ✓
    ↓
[cat /etc/shadow] → root:* → misconfiguration documentada (CWE-258)
```

---

## Lecciones aprendidas

1. **La oscuridad no es seguridad.** Codificar el nombre de un fichero en Base64 no lo oculta. Las herramientas de fuzzing prueban cientos de miles de rutas por minuto; un nombre inusual no detiene al atacante, solo lo ralentiza segundos.

2. **Los ficheros web son públicos por definición.** Cualquier fichero dentro del webroot es accesible por cualquier persona en internet. Las credenciales, backups o código fuente nunca deben estar en el webroot sin autenticación.

3. **La enumeración encadena los pasos.** Todo el compromiso se basó en enumeración metódica: nmap → ffuf → decodificación → SSH → flag. No fue necesaria ninguna vulnerabilidad de software, solo malas prácticas de despliegue.

4. **`*` y `!` no son lo mismo en `/etc/shadow`.** En sistemas que usan busybox (Alpine Linux, entornos embebidos), el hash `*` puede equivaler a "sin contraseña". El estándar correcto para deshabilitar una cuenta es `passwd -l`, que genera `!`. Esta distinción es crítica en entornos de producción con Linux embebido.
