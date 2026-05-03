# Solución — Imagine

**Dificultad:** Media  
**Servicios:** HTTP, SSH  
**Objetivos:**
- Flag 1 (User): `/home/jude/.flag.txt`

---

## 1. Preparación del entorno

```bash
docker run -d --name imagine jserrai/tfm_ctf:imagine2
docker inspect imagine | grep IPAddress
```

A partir de aquí usaremos `<IP>` para referirnos a la IP del contenedor.

---

## 2. Reconocimiento — Escaneo de puertos

```bash
nmap -sV -sC -p- <IP>
```

**Resultado esperado:**

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 80/tcp | HTTP | Apache 2.4.62 |
| 2222/tcp | SSH | OpenSSH 9.3 |

**Hallazgos clave:**
- Servidor web Apache accesible en el puerto 80
- SSH disponible en un puerto no estándar (2222)

> **Concepto:** El uso de un puerto no estándar para SSH (2222 en lugar de 22) es seguridad por oscuridad — no impide el acceso pero puede retrasar un escaneo rápido. Nmap con `-p-` escanea todos los puertos y lo encuentra igualmente.

---

## 3. Enumeración web

```bash
ffuf -c \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -u http://<IP>/FUZZ \
  -e .php,.html,.txt \
  -mc 200 -fs 45 -t 50
```

**Resultado esperado:** Varios ficheros `.html` con contenido relevante.

Para ver el contenido de cada fichero encontrado:

```bash
curl http://<IP>/<fichero>
```

**Hallazgo crítico:**  
Uno de los ficheros tiene el **nombre codificado en Base64**. Para decodificarlo:

```bash
echo "<nombre_sin_extension>" | base64 -d
```

El contenido de ese fichero también está en Base64. Aplica el mismo proceso para obtener las credenciales SSH.

> **Concepto:** La codificación Base64 no es cifrado — es completamente reversible. Guardar credenciales en un fichero web accesible públicamente es una vulnerabilidad crítica aunque el nombre del fichero esté "ofuscado". (OWASP A02:2021 — Cryptographic Failures)

---

## 4. Enumeración de usuarios

Uno de los ficheros encontrados lista **nombres de usuarios del sistema**. Anótalos — serán candidatos para el acceso SSH.

> **Concepto:** La exposición de nombres de usuario permite al atacante hacer ataques dirigidos en lugar de fuerza bruta ciega. (CWE-200 — Exposure of Sensitive Information)

---

## 5. Acceso SSH

Con las credenciales obtenidas en el paso 3:

```bash
ssh <usuario>@<IP> -p 2222
```

**Verificar acceso:**

```bash
whoami
id
```

---

## 6. Flag 1

```bash
ls -la ~
cat ~/.flag.txt
```

**Flag 1 obtenida.** Guárdala.

---

## 7. Post-explotación — Análisis de escalada de privilegios

Una vez dentro, enumera posibles vías de escalada. Es parte esencial de cualquier análisis de seguridad.

### Inspección de /etc/shadow

```bash
cat /etc/shadow | grep root
```

**Hallazgo:** Observa el campo de contraseña de root (segundo campo). Compara estos valores:

| Valor | Significado en Linux |
|-------|----------------------|
| `$6$...` | Contraseña hasheada (SHA-512) |
| `!` | Cuenta bloqueada correctamente |
| `*` | Sin contraseña — vulnerable en sistemas busybox |

> **Concepto — CWE-258:** En Alpine Linux con busybox, el hash `*` en `/etc/shadow` significa que la cuenta no tiene contraseña configurada. Si `busybox` tuviera el bit SUID activo, cualquier usuario podría ejecutar `su root` sin autenticación. La forma correcta de deshabilitar root es `passwd -l root`, que genera `!`. En esta imagen el bit SUID no está activo, pero el hallazgo debe documentarse en un informe real.

### Binarios SUID

```bash
find / -perm -4000 -type f 2>/dev/null
```

Investiga para qué sirve cada binario encontrado y si tiene exploits conocidos.

---

## Resumen de vulnerabilidades

| # | Vulnerabilidad | Referencia |
|---|----------------|------------|
| 1 | Enumeración de usuarios en fichero HTML público | CWE-200, OWASP A05 |
| 2 | Credenciales expuestas en recurso web con nombre en Base64 | CWE-256, OWASP A02 |
| 3 | Ficheros de backup accesibles públicamente (`index.php.bak`) | OWASP A05 |
| 4 | Cuenta root sin contraseña en `/etc/shadow` (`*` en lugar de `!`) | CWE-258 |

---

## Cyber Kill Chain

Ver [kill_chain.md](./kill_chain.md)

---

## Limpieza

```bash
docker stop imagine && docker rm imagine
```
