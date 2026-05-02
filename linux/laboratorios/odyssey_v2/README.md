# Informe de Laboratorio: Odyssey v2

## Especificaciones Tecnicas

*   **Plataforma:** CTF Training-B (TFM3)
*   **Dificultad:** Media
*   **Categoria:** Web / Esteganografia / Linux
*   **Enfoque:** Enumeracion de directorios ocultos, analisis de imagenes y acceso SSH.

## Resumen del Proyecto

Este repositorio documenta el proceso de compromiso de la maquina Odyssey v2. El reto gira en torno a la enumeracion recursiva de rutas y archivos ocultos en un servidor nginx, el descifrado de datos codificados y la extraccion de credenciales ocultas mediante esteganografia en imagenes JPEG.

## Objetivos de Aprendizaje

1.  Enumeracion de directorios y archivos ocultos (prefijo `.`) en servidores web con listado desactivado.
2.  Identificacion y descarte de red herrings durante la enumeracion.
3.  Decodificacion de datos ofuscados en Base64 y formato hexadecimal (xxd).
4.  Extraccion de datos ocultos en imagenes JPEG mediante esteganografia (steghide).
5.  Acceso SSH con credenciales obtenidas y enumeracion de rutas ocultas en el sistema de ficheros.

## Vulnerabilidades Documentadas

| # | Vulnerabilidad | Referencia |
|---|----------------|------------|
| 1 | `phpinfo.php` expuesto en produccion | CWE-552 |
| 2 | nginx sin restriccion de acceso directo a ficheros ocultos | OWASP A05 |
| 3 | Credenciales codificadas en Base64/hex (no cifradas) | CWE-261 |
| 4 | Credenciales SSH ocultas mediante esteganografia en imagenes | CWE-311 |
| 5 | Acceso SSH directo como root habilitado | CWE-250 |

## Documentacion de la Solucion

*   [SOLUCION.md](./SOLUCION.md) — Writeup paso a paso con comandos y explicaciones
*   [kill_chain.md](./kill_chain.md) — Ruta de compromiso segun el modelo Cyber Kill Chain

## Aviso Legal y Etica

Los archivos necesarios para el despliegue de la infraestructura (Dockerfiles, scripts de configuracion) no se incluyen en este repositorio para respetar la propiedad intelectual de los creadores originales. Este contenido tiene una finalidad exclusivamente didactica.
