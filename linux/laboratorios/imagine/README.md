# Imagine

Laboratorio CTF de escalada de privilegios en Alpine Linux.

## Servicios

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 80 | Apache HTTP | 2.4.62 |
| 2222 | OpenSSH | 9.3 |

## Objetivos

- **Flag 1 (User):** Acceder como usuario sin privilegios vía SSH
- **Flag 2 (Root):** Escalar a root mediante misconfiguration en `/etc/shadow`

## Arrancar el laboratorio

```bash
docker run -d --name imagine jserrai/tfm_ctf:imagine2
docker inspect imagine | grep IPAddress
```

## Habilidades practicadas

- Enumeración web (ffuf, curl)
- Decodificación Base64
- Reconocimiento de usuarios en aplicaciones web
- Análisis de `/etc/shadow` y gestión de contraseñas en Linux
- Escalada de privilegios local (privilege escalation)

## Referencias

| Referencia | Descripción |
|-----------|-------------|
| CWE-258 | Empty Password in Configuration File |
| OWASP A02:2021 | Cryptographic Failures |
| OWASP A05:2021 | Security Misconfiguration |

## Writeup

Ver [`SOLUCION.md`](SOLUCION.md) para la solución completa y [`kill_chain.md`](kill_chain.md) para el diagrama de ataque.
