# Informe de Laboratorio: Imagine

## Especificaciones Técnicas
- **Plataforma:** CTF Training (TFM)
- **Dificultad:** Media
- **Categoría:** Boot2Root / Linux
- **Enfoque:** Enumeración web, análisis de credenciales expuestas y acceso SSH.

## Resumen del Proyecto
Este repositorio documenta el proceso de intrusión y compromiso de la máquina Imagine. El análisis se centra en la identificación de recursos web mal protegidos que exponen credenciales de acceso, y en la inspección de configuraciones inseguras del sistema una vez obtenido el acceso inicial.

## Objetivos de Aprendizaje
1. Enumeración de servicios con nmap.
2. Fuzzing de directorios y ficheros web con ffuf.
3. Identificación y decodificación de información sensible expuesta (Base64).
4. Análisis de `/etc/shadow` y gestión de cuentas en Alpine Linux / busybox.

## Documentación de la Solución
El informe detallado con los comandos ejecutados y la metodología seguida se encuentra en:

- [SOLUCION.md](./SOLUCION.md) — Writeup paso a paso con comandos y explicaciones
- [kill_chain.md](./kill_chain.md) — Visión panorámica de la cadena de ataque (Cyber Kill Chain)

## Aviso Legal y Ética
Los archivos necesarios para el despliegue de la infraestructura (Dockerfiles, scripts de configuración) no se incluyen en este repositorio para respetar la propiedad intelectual de los creadores originales. Este contenido tiene una finalidad exclusivamente didáctica.
