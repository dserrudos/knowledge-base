# Jump Force — CTF Challenge

## Descripción

Reto de dos contenedores que simula un entorno corporativo con una aplicación web vulnerable y un servidor interno accesible solo desde la red interna de Docker. El objetivo es comprometer ambos contenedores encadenando SQL Injection, Command Injection y pivoting de red.

## Arquitectura

```
[Atacante]
    │
    │ HTTP :5000
    ▼
[Contenedor uno — jump_force_one]
 Apache2 + MySQL
 /var/www/html/password.php  → SQL Injection
 /var/www/html/backup.php    → Command Injection
    │
    │ SSH :2222 (red interna Docker)
    ▼
[Contenedor dos — jump_force_two]
 SSH servidor, usuario pablo
 privileged: true  ← escape de contenedor posible
```

| Servicio | Imagen | Puerto externo | Puerto interno |
|---|---|---|---|
| `uno` | `jserrai/tfm_ctf:jump_force_one` | 5000 | 80 |
| `dos` | `jserrai/tfm_ctf:jump_force_two` | — | 2222 (SSH) |

## Vulnerabilidades

| # | Fichero / Servicio | Tipo | Referencia |
|---|---|---|---|
| 1 | `password.php` | SQL Injection | OWASP A03:2021, CWE-89 |
| 2 | `backup.php` | Command Injection | OWASP A03:2021, CWE-78 |
| 3 | `docker-compose.yml` | Privileged Container | CWE-250 |

## Flags

| # | Flag | Dónde se obtiene | Valor |
|---|---|---|---|
| 1 | Web / BD | Tabla `flags` en MySQL via SQL Injection (contenedor uno) | `003d873449f8e8ff13b72f2061bfbaa4e5a84b82` |
| 2 | User | `/home/pablo/.flag.txt` via SSH (contenedor dos) | `4d8c72671245d9d1b8e03a826db9d5ecead28c8c` |

> El contenedor `dos` arranca con `privileged: true`. Aunque no existe un fichero de flag de root implementado, esta configuración permite escapar al host montando su disco. El concepto de escape se documenta en `kill_chain.md`.

## Levantar el entorno

```bash
cd jump_force/
docker-compose up --build
```

La aplicación web queda disponible en `http://localhost:5000`.

## Dificultad

Media — requiere encadenar SQL Injection + pivoting de red Docker + SSH con credenciales obtenidas.

## Conceptos que trabaja

- SQL Injection con UNION (extracción de tablas, credenciales y flags)
- Command Injection vía `shell_exec()` sin sanitizar
- Pivoting a servicios internos no expuestos al exterior
- Privileged container y escape al host (demostración conceptual)
- Principio de mínimo privilegio (CWE-250)
