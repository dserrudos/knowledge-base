# Imagine — Writeup

## Información del reto

| Campo | Valor |
|-------|-------|
| Imagen | `jserrai/tfm_ctf:imagine2` |
| OS | Alpine Linux |
| Servicios | Apache 2.4.62 (puerto 80) + OpenSSH 9.3 (puerto 2222) |
| Dificultad | Media |
| Objetivo principal | Flag 1 — acceso como usuario `jude` |

---

## Fase 1 — Preparación del entorno

Antes de atacar, levanta el contenedor y obtén su IP:

```bash
docker run -d --name imagine jserrai/tfm_ctf:imagine2
docker inspect imagine | grep IPAddress
```

Anota la IP. A partir de aquí se usará como `<IP>` en todos los comandos.

---

## Fase 2 — Reconocimiento de red

El primer paso en cualquier pentest es identificar qué servicios están expuestos.

```bash
nmap -sV -sC -p- <IP>
```

Analiza la salida con estas preguntas en mente:
- ¿Qué puertos están abiertos?
- ¿Qué versiones de software se están ejecutando?
- ¿Alguna versión tiene CVEs conocidos?

> **Concepto:** El reconocimiento pasivo nos da la superficie de ataque sin interactuar con la aplicación. Cada servicio expuesto es un vector potencial. Referencia: OWASP Testing Guide — OTG-INFO-001.

---

## Fase 3 — Enumeración web

Con Apache identificado en el puerto 80, el siguiente paso es descubrir recursos web que no están enlazados visiblemente. Muchas aplicaciones tienen ficheros "ocultos" — scripts, backups, paneles — que solo se encuentran probando nombres comunes.

```bash
ffuf -c \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -u http://<IP>/FUZZ \
  -e .php,.html,.txt \
  -mc 200 -fs 45 -t 50
```

El parámetro `-fs 45` filtra la página por defecto del servidor (45 bytes) para eliminar falsos positivos.

Deberías encontrar varios ficheros. Para cada uno, responde:
- ¿Qué información contiene?
- ¿Revela nombres de usuario, rutas internas o credenciales?
- ¿Hay alguno con un nombre inusual que merezca análisis extra?

> **Concepto:** La enumeración web (directory fuzzing) explora la superficie oculta de una aplicación. Es una técnica fundamental recogida en OWASP A05:2021 — Security Misconfiguration. Los ficheros de backup (`.bak`) o archivos con nombres codificados son señales de alerta.

---

## Fase 4 — Análisis de ficheros encontrados

Examina el contenido de cada fichero encontrado con `curl`:

```bash
curl http://<IP>/<fichero>
```

Presta especial atención a:

1. **Ficheros con nombres en Base64** — el nombre del fichero en sí puede ser una pista. Para decodificar un string en Base64:
   ```bash
   echo "<string>" | base64 -d
   ```

2. **Ficheros que listan usuarios** — son vectores de enumeración de cuentas.

3. **Ficheros con credenciales ofuscadas** — el contenido puede estar también en Base64. Decifra cualquier string sospechoso que encuentres.

> **Concepto:** La ofuscación en Base64 no es cifrado. Es solo una codificación reversible que cualquiera puede decodificar. Almacenar credenciales en ficheros web accesibles públicamente es una vulnerabilidad crítica — OWASP A02:2021 (Cryptographic Failures) y CWE-200 (Exposure of Sensitive Information).

---

## Fase 5 — Acceso inicial vía SSH

Con las credenciales obtenidas en la fase anterior, intenta acceder por SSH al puerto no estándar que encontraste con nmap:

```bash
ssh <usuario>@<IP> -p <puerto>
```

Si el acceso es exitoso, ya tienes una shell como usuario de bajos privilegios.

---

## Fase 6 — Flag 1

Una vez dentro, el objetivo es localizar la flag de usuario. En CTFs las flags suelen estar en el directorio home del usuario comprometido, a veces como ficheros ocultos (empiezan por `.`).

```bash
ls -la ~
cat ~/<fichero_flag>
```

Anota el valor de la flag.

> **Concepto:** El principio de mínimo privilegio dicta que los ficheros sensibles no deben ser accesibles por usuarios no autorizados. En este caso, la flag está en `/home/jude/` y solo es legible por `jude` y `root` — el fichero está correctamente protegido. El fallo estuvo en permitir que un atacante llegase a ser `jude` en primer lugar.

---

## Fase 7 — Post-explotación: análisis de escalada de privilegios

Una vez como `jude`, es buena práctica enumerar posibles vías de escalada a root aunque no sea el objetivo del reto. Esto forma parte de cualquier informe de pentest real.

### 7.1 Inspección de /etc/shadow

```bash
cat /etc/shadow | grep root
```

Observa el campo de contraseña de root (segundo campo, separado por `:`). En sistemas Linux estándar:

| Valor | Significado |
|-------|-------------|
| `$6$...` | Contraseña hasheada con SHA-512 |
| `!` | Cuenta bloqueada — no se puede autenticar |
| `*` | Sin contraseña configurada |

Pregúntate: ¿qué implicaría el valor que ves si `busybox su` estuviera instalado con el bit SUID activo?

### 7.2 Búsqueda de binarios SUID

```bash
find / -perm -4000 -type f 2>/dev/null
```

Identifica qué binarios tienen el bit SUID activado y busca si alguno es explotable.

### 7.3 Comprobación de sudo

```bash
sudo -l
```

### 7.4 Procesos corriendo como root

```bash
ps aux | grep root
```

> **Concepto — CWE-258:** Cuando el campo de contraseña en `/etc/shadow` es `*`, en implementaciones de `busybox su` esto equivale a "sin contraseña": el sistema permitiría `su root` sin autenticación. La forma correcta de deshabilitar una cuenta root es `passwd -l root`, que genera `!`. Esta distinción es crítica en sistemas Alpine Linux y entornos embebidos que usan busybox. En esta imagen concreta, `busybox` no tiene el bit SUID activo, por lo que el vector no es explotable — pero el concepto es válido y documentado.

---

## Resumen de vulnerabilidades

| # | Vulnerabilidad | Impacto | Referencia |
|---|----------------|---------|------------|
| 1 | Enumeración de usuarios en fichero HTML público | Medio | OWASP A05, CWE-200 |
| 2 | Credenciales expuestas en recurso web con nombre ofuscado en Base64 | Crítico | OWASP A02, CWE-256 |
| 3 | Ficheros de backup y configuración accesibles públicamente | Medio | OWASP A05 |
| 4 | Cuenta root sin contraseña en `/etc/shadow` (`*` en vez de `!`) | Alto (teórico) | CWE-258 |

---

## Lecciones aprendidas

- **La oscuridad no es seguridad:** codificar un nombre de fichero en Base64 no lo oculta. Las herramientas de fuzzing prueban decenas de miles de nombres por segundo.
- **Los backups expuestos son un riesgo:** ficheros `.bak` y archivos renombrados con extensión falsa deben estar fuera del webroot o protegidos con autenticación.
- **Gestión de cuentas en Linux:** la diferencia entre `*` y `!` en `/etc/shadow` puede ser la diferencia entre un sistema seguro y uno comprometido. Siempre bloquear cuentas con `passwd -l`.

---

## Limpieza del entorno

```bash
docker stop imagine
docker rm imagine
```
