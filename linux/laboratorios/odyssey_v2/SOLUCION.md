# Writeup — Odyssey v2

**Dificultad:** Media  
**Servicios:** HTTP (nginx), SSH  
**Objetivos:**
- Flag 1 (User): archivo oculto dentro del directorio `/admin/`
- Flag 2 (Root): archivo oculto en una ruta anidada bajo `/root/`

**Habilidades practicadas:** enumeración de directorios, archivos ocultos, esteganografía, SSH

---

## 1. Preparación del entorno

Construimos la imagen a partir del Dockerfile local (que descarga la imagen del reto desde Docker Hub) y arrancamos el contenedor en segundo plano.

```bash
cd /home/kali/Documents/CTF-Docker/odyssey_v2/
sudo docker build -t odyssey_v2 .
sudo docker run --rm -d --name odyssey odyssey_v2
```

Obtenemos la IP del contenedor:

```bash
sudo docker inspect odyssey | grep IPAddress
# Resultado habitual en Docker bridge: 172.17.0.2
```

A partir de aquí usaremos `<IP>` para referirnos a esa IP.

---

## 2. Reconocimiento — Escaneo de puertos

Escaneamos todos los puertos con detección de versiones y scripts por defecto de nmap.

```bash
nmap -sV -sC -p- <IP>
```

**Resultado:**

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 22/tcp | SSH | OpenSSH 7.6p1 Ubuntu |
| 80/tcp | HTTP | nginx 1.14.0 |

**Hallazgos clave:**
- El servidor web devuelve **403 Forbidden** en la raíz `/` — el contenido está bloqueado pero el servidor está activo.
- SSH disponible para acceso con credenciales (a descubrir más adelante).

> **Concepto:** Un 403 en la raíz no significa que el servidor esté vacío. Significa que el listado del directorio está desactivado (`autoindex off` en nginx). Puede haber subdirectorios y archivos accesibles si conocemos sus nombres. El siguiente paso siempre es **enumeración de rutas**. (OWASP A05 — Security Misconfiguration)

---

## 3. Enumeración web — Primera pasada

Usamos **gobuster** para descubrir directorios y archivos probando palabras de un diccionario.

```bash
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x php,txt,html
```

**Resultado:**

| Ruta | Código | Observación |
|------|--------|-------------|
| `/admin/` | 301 | Directorio objetivo |
| `/notes/` | 301 | Puede contener pistas |
| `/images/` | 301 | Imágenes — revisar después |
| `/0/` … `/15/` | 301 | 16 directorios numéricos, sospechosos |
| `/phpinfo.php` | 200 | Información del servidor expuesta |

> **Concepto:** `phpinfo.php` expuesto en producción es una vulnerabilidad de **Information Disclosure** (CWE-552). Revela la versión de PHP, rutas del servidor, variables de entorno y configuración de seguridad, información muy útil para un atacante.

De phpinfo.php extraemos datos relevantes:
- El proceso web corre como `www-data`
- `disable_functions` está vacío → PHP puede ejecutar funciones del sistema
- `open_basedir` está vacío → PHP puede leer cualquier archivo del servidor

---

## 4. Investigar /notes/ — La pista clave

```bash
gobuster dir -u http://<IP>/notes/ -w /usr/share/wordlists/dirb/common.txt -x txt
# Resultado: note.txt (200)

curl http://<IP>/notes/note.txt
# Resultado: 1 3 11
```

La nota contiene tres números: **1, 3, 11**. Estos son índices que apuntan a tres de los dieciséis directorios numéricos.

---

## 5. Directorios numéricos — Datos codificados

Todos los directorios del `/0/` al `/15/` contienen un archivo `junk.txt`. La mayoría dice `no soy clave` (señuelos / red herrings). Solo los indicados en la nota tienen datos reales:

```bash
curl http://<IP>/1/junk.txt   # MTIzNF9zZWM=
curl http://<IP>/3/junk.txt   # aG9vcmEh
curl http://<IP>/11/junk.txt  # 00000000: 6361 6c69 666f 726e 6961
```

Decodificamos cada formato:

```bash
echo "MTIzNF9zZWM=" | base64 -d      # → 1234_sec
echo "aG9vcmEh" | base64 -d          # → hoora!
echo "6361 6c69 666f 726e 6961" | xxd -r -p  # → california
```

**Resultado:**

| Directorio | Encoding | Valor |
|------------|----------|-------|
| `/1/junk.txt` | Base64 | `1234_sec` |
| `/3/junk.txt` | Base64 | `hoora!` |
| `/11/junk.txt` | Hex dump (xxd) | `california` |

> **Concepto:** Codificar datos en Base64 o hexadecimal **no es cifrado** — es solo ofuscación reversible en segundos con herramientas estándar. No protege credenciales. (CWE-261 — Weak Encoding for Password)

---

## 6. Flag 1 — Archivo oculto en /admin/

`/admin/` devuelve 403 al intentar listar el directorio, pero podemos buscar archivos concretos. Los archivos ocultos de Unix empiezan por `.` y no aparecen en wordlists estándar — hay que buscarlos específicamente.

```bash
# Creamos un wordlist de archivos ocultos comunes
printf ".flag\n.flag.txt\n.secret\n.htpasswd\n.env\n.hidden\n.passwd\n.key\n" > /tmp/hidden_files.txt

gobuster dir -u http://<IP>/admin/ -w /tmp/hidden_files.txt
# Resultado: .flag.txt (200, 41 bytes)

curl http://<IP>/admin/.flag.txt
```

**Flag 1 obtenida.**

> **Concepto:** nginx con `autoindex off` bloquea el **listado** del directorio, pero no el acceso **directo** a archivos si conoces su nombre. Los archivos con prefijo `.` (ocultos en Unix) no aparecen en wordlists genéricas — hay que usar listas específicas o generarlas. Este es el núcleo del reto: enumeración de rutas ocultas. (OWASP A05)

---

## 7. Esteganografía — Credenciales ocultas en imágenes

En `/images/` encontramos 16 imágenes JPEG numeradas del 0 al 15. Siguiendo el mismo patrón de la nota (directorios 1, 3 y 11), analizamos esas tres imágenes con **steghide**, una herramienta que oculta datos dentro de archivos JPEG.

Cada imagen usa como contraseña el valor que encontramos en su directorio numérico correspondiente:

```bash
steghide extract -sf 1.jpg  -p "1234_sec"   # extrae: s1
steghide extract -sf 3.jpg  -p "hoora!"     # extrae: s2
steghide extract -sf 11.jpg -p "california" # extrae: s3

cat s1  # user: root
cat s2  # pass: !3QwX?j4
cat s3  # flag: /root/.hide/.last
```

Las tres imágenes contienen las credenciales SSH de root divididas en tres partes.

> **Concepto:** La **esteganografía** oculta la existencia de un mensaje dentro de un archivo aparentemente inocente (aquí, una imagen JPEG). A diferencia del cifrado, que protege el contenido pero revela que hay un secreto, la esteganografía busca que nadie sospeche que hay información oculta. Steghide modifica los bits menos significativos (LSB) de los píxeles para incrustar los datos. (CWE-311)

---

## 8. Flag 2 — Acceso SSH como root

Con las credenciales obtenidas nos conectamos por SSH directamente como `root`:

```bash
ssh root@<IP>
# Contraseña: !3QwX?j4

# Verificar acceso
whoami   # root

# La pista de s3 indicaba la ruta — enumeramos
find /root/.hide -type f
# /root/.hide/.last/.flag.txt

cat /root/.hide/.last/.flag.txt
```

**Flag 2 obtenida.**

> **Concepto:** La flag está en una ruta triplemente oculta: directorio `.hide` → subdirectorio `.last` → archivo `.flag.txt`. Los tres segmentos empiezan por `.` y son invisibles con un `ls` normal. Se necesita `ls -la` o `find` para descubrirlos. Este es exactamente el concepto de **enumeración recursiva de archivos ocultos** que ilustra el reto. (OWASP A05)

---

## Resumen de vulnerabilidades

| # | Vulnerabilidad | Impacto | Referencia |
|---|----------------|---------|------------|
| 1 | `phpinfo.php` expuesto | Information Disclosure — revela configuración interna | CWE-552 |
| 2 | nginx con `autoindex off` pero sin restricción de acceso directo a ficheros | Acceso a archivos ocultos conociendo su nombre | OWASP A05 |
| 3 | Credenciales codificadas en Base64/hex (no cifradas) | Ofuscación trivialmente reversible | CWE-261 |
| 4 | Credenciales SSH ocultas mediante esteganografía | Acceso root si se descubren las imágenes clave | CWE-311 |
| 5 | SSH accesible con usuario root directamente | Sin capa intermedia de usuario limitado | CWE-250 |

---

## Cyber Kill Chain

Ver [kill_chain.md](kill_chain.md)

---

## Comandos de limpieza

```bash
sudo docker stop odyssey
```
