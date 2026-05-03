# Imagine

Laboratorio CTF de enumeración web y análisis de credenciales en Alpine Linux.

## Descripción

El objetivo es comprometer un servidor web Apache mal configurado para obtener acceso SSH como usuario sin privilegios. El reto pone en práctica la enumeración metódica de recursos web, el análisis de información sensible expuesta y el uso de credenciales encontradas para acceder al sistema.

## Servicios expuestos

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 80 | Apache HTTP | 2.4.62 |
| 2222 | OpenSSH | 9.3 |

## Objetivo

- **Flag 1 (User):** Obtener acceso SSH como usuario de bajos privilegios y leer su flag.

## Arrancar el laboratorio

```bash
docker run -d --name imagine jserrai/tfm_ctf:imagine2
docker inspect imagine | grep IPAddress
```

Para detener y limpiar:

```bash
docker stop imagine && docker rm imagine
```

## Habilidades practicadas

- Reconocimiento de red con nmap
- Enumeración de directorios y ficheros web (fuzzing)
- Identificación de información sensible en recursos públicos
- Decodificación Base64
- Acceso SSH con credenciales encontradas
- Análisis post-explotación: revisión de `/etc/shadow`, binarios SUID y permisos

## Conceptos de seguridad cubiertos

| Referencia | Concepto |
|-----------|----------|
| OWASP A02:2021 | Credenciales expuestas / codificación ≠ cifrado |
| OWASP A05:2021 | Ficheros sensibles accesibles sin autenticación |
| CWE-200 | Exposición de información a actores no autorizados |
| CWE-258 | Campo de contraseña vacío en configuración del sistema |

## Documentación

| Fichero | Contenido |
|---------|-----------|
| [`SOLUCION.md`](SOLUCION.md) | Walkthrough paso a paso con explicaciones de cada fase |
| [`kill_chain.md`](kill_chain.md) | Diagrama y resumen estructurado de la cadena de ataque |
