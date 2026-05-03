# Kill Chain — Imagine

## Resumen ejecutivo

| Fase | Técnica | Vector | Resultado |
|------|---------|--------|-----------|
| Reconocimiento | Port scanning | nmap -sV -sC | Apache 80, SSH 2222 identificados |
| Enumeración web | Directory fuzzing | ffuf + wordlist | Recursos sensibles descubiertos |
| Análisis de información | Decodificación Base64 | Nombre de fichero + contenido | Credenciales SSH obtenidas |
| Acceso inicial | Autenticación SSH | Usuario + contraseña filtrada | Shell como `jude` |
| Flag 1 | Lectura de fichero | cat .flag.txt | Flag de usuario obtenida |
| Post-explotación | Análisis de shadow | /etc/shadow | Misconfiguration identificada (teórico) |

---

## Detalle de la cadena

### 1. Reconocimiento de red

**Objetivo:** Mapear la superficie de ataque expuesta.

```bash
nmap -sV -sC -p- <IP>
```

**Hallazgos:**
- Puerto 80 → Apache HTTP Server 2.4.62 (Alpine Linux)
- Puerto 2222 → OpenSSH 9.3

**Por qué importa:** Dos superficies de ataque independientes. La aplicación web es el vector de entrada para obtener credenciales; SSH es el vector de acceso una vez obtenidas.

---

### 2. Enumeración web

**Objetivo:** Descubrir recursos web no enlazados que puedan contener información sensible.

```bash
ffuf -c \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -u http://<IP>/FUZZ \
  -e .php,.html,.txt \
  -mc 200 -fs 45 -t 50
```

**Hallazgos clave:**

| Recurso | Tipo de información |
|---------|---------------------|
| `creditcard.html` | Nombres de usuario del sistema |
| Fichero con nombre en Base64 | Credenciales de acceso SSH (codificadas) |
| `xcart.tgz` | Pistas adicionales sobre credenciales |
| `index.php.bak` | Backup de código fuente expuesto |

**Por qué importa:** Un atacante sin conocimiento previo puede recuperar nombres de usuario válidos y credenciales simplemente explorando la aplicación web. No se requiere ninguna vulnerabilidad de software — solo malas prácticas de despliegue.

---

### 3. Extracción de credenciales

**Objetivo:** Descifrar la información encontrada para obtener credenciales válidas.

**Paso 1 — Decodificar el nombre del fichero:**
```bash
echo "<nombre_en_base64>" | base64 -d
```
El nombre decodificado revela el propósito del fichero.

**Paso 2 — Decodificar el contenido del fichero:**
El contenido también está en Base64. Aplicar el mismo proceso revela la contraseña en texto claro.

**Por qué importa:** Base64 es una codificación, no un cifrado. No ofrece ninguna protección real. Cualquier atacante con acceso al fichero puede recuperar el valor original en milisegundos.

---

### 4. Acceso inicial — Shell como usuario

**Objetivo:** Usar las credenciales obtenidas para autenticarse en el servicio SSH.

```bash
ssh <usuario>@<IP> -p 2222
```

**Resultado:** Shell interactiva como usuario de bajos privilegios.

---

### 5. Flag 1 — Objetivo completado

```bash
ls -la ~
cat ~/<fichero_flag>
```

**Resultado:** Flag de usuario obtenida en el directorio home del usuario comprometido.

---

### 6. Post-explotación — Análisis de escalada (teórico)

**Objetivo:** Evaluar si existe una vía para escalar a root.

```bash
# Inspección de shadow
cat /etc/shadow | grep root

# Binarios SUID
find / -perm -4000 -type f 2>/dev/null

# Directorios escribibles
find / -writable -not -path '*/proc/*' -not -path '*/sys/*' 2>/dev/null
```

**Hallazgo:** La entrada de root en `/etc/shadow` contiene `*` en el campo de contraseña. En Alpine Linux con `busybox su` instalado con bit SUID, esto permitiría `su root` sin contraseña (CWE-258). En esta imagen, el vector no es explotable porque `busybox` carece del bit SUID, pero el hallazgo debe documentarse en un informe real como vulnerabilidad de configuración.

---

## Diagrama de ataque

```
[Atacante - Kali Linux]
        │
        ▼  nmap → descubre Apache:80 y SSH:2222
        │
        ▼  ffuf → enumera recursos web
        │
        ├──► creditcard.html ──► extrae nombres de usuario válidos
        │
        ├──► fichero (nombre en Base64) ──► decodifica nombre → accede al recurso
        │                                  decodifica contenido → obtiene contraseña
        │
        ▼  SSH :2222 con credenciales obtenidas
        │
[Shell como jude]
        │
        ▼  cat ~/.flag.txt ──► FLAG 1 obtenida ✓
        │
        ▼  cat /etc/shadow → root:* → misconfiguration documentada
        │
[Análisis post-explotación completado]
```

---

## Referencias técnicas

| Referencia | Descripción |
|-----------|-------------|
| CWE-200 | Exposure of Sensitive Information to an Unauthorized Actor |
| CWE-256 | Plaintext Storage of a Password |
| CWE-258 | Empty Password in Configuration File |
| OWASP A02:2021 | Cryptographic Failures |
| OWASP A05:2021 | Security Misconfiguration |
| OWASP Testing Guide OTG-INFO-001 | Information Gathering |
