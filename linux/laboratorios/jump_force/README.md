# Jump Force — CTF Challenge

## Descripción

Reto de dos contenedores que simula un entorno corporativo con una aplicación web vulnerable y un servidor interno accesible solo desde la red interna de Docker. El objetivo es comprometer ambos contenedores y demostrar un escape de contenedor privilegiado.

## Arquitectura

```
[Atacante] ──HTTP:5000──▶ [Contenedor uno: Apache2 + MySQL]
                                      │
                               Red interna Docker
                                      │
                               [Contenedor dos: SSH:2222]
                               (privileged: true)
                                      │
                              Escape al host
```

| Servicio | Imagen | Puerto externo | Puerto interno |
|---|---|---|---|
| `uno` | `jserrai/tfm_ctf:jump_force_one` | 5000 | 80 |
| `dos` | `jserrai/tfm_ctf:jump_force_two` | — | 2222 (SSH) |

## Vulnerabilidades presentes

| # | Fichero / Servicio | Tipo | Referencia |
|---|---|---|---|
| 1 | `password.php` | SQL Injection | OWASP A03, CWE-89 |
| 2 | `backup.php` | Command Injection | OWASP A03, CWE-78 |
| 3 | `docker-compose.yml` | Privileged Container | CWE-250 |

## Flags

| Flag | Descripción | Valor |
|---|---|---|
| Flag 1 (User) | `/home/pablo/.flag.txt` en contenedor `dos` | `4d8c72671245d9d1b8e03a826db9d5ecead28c8c` |
| Flag 2 (Root) | Escape de contenedor privilegiado | `648d390c021ce7cfde2f95ea3fcd71ec` |
| Flag BD (Bonus) | Tabla `flags` en base de datos `poc` | `003d873449f8e8ff13b72f2061bfbaa4e5a84b82` |

## Levantar el entorno

```bash
cd jump_force/
docker-compose up --build
```

La aplicación web queda expuesta en `http://localhost:5000`.

## Dificultad

Media — requiere encadenar SQL Injection + pivoting interno + container escape.

## Conceptos que trabaja

- SQL Injection (extracción de credenciales vía UNION)
- Command Injection (ejecución de comandos arbitrarios en el servidor)
- Pivoting en red Docker (acceso a servicios internos no expuestos al host)
- Escape de contenedor privilegiado (montaje de disco del host, chroot)
- Principio de mínimo privilegio (CWE-250)
