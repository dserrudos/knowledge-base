# Guía Pareto — Seguridad y Alta Disponibilidad

> **Lógica de esta guía:** Lo que salió en el examen de ejemplo **no repetirá exactamente**.
> El profesor preguntará el tema adyacente o la variante lógica.
> Cada bloque muestra qué salió → qué estudiar para el examen real.

---

## NF1 — Fundamentos de Seguridad

### Tema 1 — CIDAN: Pilares de Seguridad

> **Ejemplo preguntó:** Confidencialidad
> **Examen real:** Cualquiera de los otros 4 pilares

Memoriza la definición de los 5. El patrón de respuesta es siempre el mismo.

| Pilar | Definición |
|-------|-----------|
| **Confidencialidad** | La información solo puede ser accedida o modificada por personas o sistemas autorizados. Protege la privacidad de los datos. |
| **Integridad** | Permite comprobar que la información no ha sido alterada ni manipulada. Asegura que los datos no se han falseado. |
| **Disponibilidad** | La información debe ser accesible y utilizable por los usuarios autorizados en el momento que la necesiten. Evita pérdida o bloqueo. |
| **Autenticación** | Verifica que un documento o usuario es quien dice ser. Se aplica mediante usuario y contraseña u otros métodos de verificación de identidad. |
| **No Repudio** | Prueba la participación de las partes en una comunicación. El emisor no puede negar el envío (no repudio en origen) y el receptor no puede negar haberlo recibido (no repudio en destino). |

---

### Tema 2 — Criptografía

> **Ejemplo preguntó:** Criptografía simétrica
> **Examen real:** Criptografía asimétrica

**Criptografía Simétrica** *(repaso rápido)*
- Una única clave secreta compartida entre emisor y receptor.
- Pasos: seleccionar archivo → introducir clave → obtener archivo cifrado. Se descifra con la misma clave.
- Ventaja: rápida. Desventaja: hay que compartir la clave de forma segura.

**Criptografía Asimétrica** *(lo que preguntará)*
- Usa **dos claves** generadas al mismo tiempo: una **pública** y una **privada**.
- La clave **pública** sirve para cifrar el mensaje o establecer el canal de comunicación.
- La clave **privada** (solo la conoce el propietario) sirve para descifrar.
- Las claves alcanzan 2048 bits, lo que las hace muy robustas.
- Desventaja: más lenta que la criptografía simétrica.

---

### Tema 3 — Almacenamiento en Red

> **Ejemplo preguntó:** NAS
> **Examen real:** DAS o SAN (o comparativa)

| Tipo | Definición | Uso típico |
|------|-----------|-----------|
| **DAS** (Direct Attached Storage) | Disco conectado físicamente y directamente al equipo. Solución tradicional, simple y económica. | PC doméstico, disco duro externo |
| **NAS** (Network Attached Storage) | Dispositivo de almacenamiento conectado a la red local. Permite acceso centralizado desde varios equipos mediante protocolos NFS, FTP, SMB. | Empresas pequeñas/medianas, copias de seguridad en red |
| **SAN** (Storage Area Network) | Red de almacenamiento de alta velocidad independiente. Los dispositivos se conectan directamente a esta red. Solo viable en grandes organizaciones por su coste de infraestructura. | Grandes corporaciones, centros de datos |

---

### Tema 4 — Legislación

> **Ejemplo preguntó:** LSSICE (publicidad por email)
> **Examen real:** LOPD (protección de datos personales)

**LSSICE — publicidad por email** *(repaso rápido — 3 requisitos)*
1. Identificar claramente quién envía el mensaje.
2. Incluir la palabra **"Publicidad"** o **"Publi"** al inicio del mensaje.
3. Facilitar la revocación del consentimiento de forma sencilla y gratuita.

**LOPD — Ley Orgánica de Protección de Datos** *(lo que preguntará)*
- Convierte la protección de datos personales en un **derecho fundamental** (art. 18 Constitución).
- Obliga a las empresas a cumplirla bajo supervisión de la **AGPD** (Agencia Española de Protección de Datos).
- Información obligatoria que debe facilitarse al recabar datos (Art. 5):
  1. El titular del fichero
  2. Las finalidades del tratamiento
  3. El carácter obligatorio o no de las respuestas
  4. Los derechos del interesado y cómo ejercerlos
  5. La dirección para ejercitar esos derechos
- **Derechos del interesado (ARCO):** Acceso, Rectificación, Cancelación y Oposición. También puede revocar el consentimiento en cualquier momento.

---

## NF2 — Amenazas y Malware

### Tema 5 — Tipos de Malware

> **Ejemplo preguntó:** Virus
> **Examen real:** Cualquier otro tipo de malware

| Malware | Definición y efectos |
|---------|---------------------|
| **Virus** | Programa que se replica y propaga entre sistemas. Efectos: robo de información, puertas traseras (backdoor), cifrado de datos y petición de rescate. |
| **Gusano** *(Worm)* | Se replica a sí mismo el máximo de veces posible para propagarse. Se propaga por correo, redes P2P, mensajería y dispositivos extraíbles. No necesita archivo anfitrión. |
| **Troyano** *(Trojan)* | Aparenta ser software legítimo pero crea una **puerta trasera** que permite la administración remota al atacante. |
| **Ransomware** | Cifra los archivos del equipo y pide un **rescate económico** para recuperar la información. |
| **Spyware** | Roba información del equipo infectado. Registra hábitos de navegación y datos del usuario sin su conocimiento. |
| **Botnet** | Conjunto de equipos infectados controlados de forma remota y automática. Se usan para rastrear información confidencial o cometer actos delictivos a gran escala. |

---

### Tema 6 — Métodos de Infiltración

> **Ejemplo preguntó:** Archivos maliciosos + dispositivos extraíbles
> **Examen real:** Otros métodos (phishing, ingeniería social, vulnerabilidades)

| Método | Descripción |
|--------|------------|
| **Archivos maliciosos** | Adjuntos en spam, aplicaciones web, descargas P2P, cracks y keygens. *(ya salió)* |
| **Dispositivos extraíbles** | Gusanos que se copian a USB y se ejecutan automáticamente al conectar el dispositivo. *(ya salió)* |
| **Phishing** | Suplantación de identidad mediante correos o webs falsas para robar credenciales bancarias o de acceso. |
| **Ingeniería social** | Manipulación psicológica del usuario para que realice acciones que vulneren la seguridad (revelar contraseñas, abrir archivos, etc.). |
| **Explotación de vulnerabilidades** | Aprovecha fallos en sistemas operativos, firmware o aplicaciones para acceder sin autorización. |
| **Cookies maliciosas** | Monitorizan y registran actividades del usuario para capturar credenciales o vender datos de navegación. |

---

## NF3 — Seguridad Perimetral

### Tema 7 — Cortafuegos / Firewall

> **Ejemplo preguntó:** Definición + funcionalidad
> **Examen real:** Misma estructura o aplicación práctica

**Definición lista para escribir:**
> Un cortafuegos o firewall es una aplicación o dispositivo de seguridad que controla el tráfico de red, permitiendo las comunicaciones autorizadas y bloqueando las no autorizadas. Su función principal es proteger la red o el equipo frente a accesos no permitidos, filtrando las conexiones de entrada y salida según reglas configuradas.

---

### Tema 8 — Iptables

> **Ejemplo preguntó:** Interpretar una regla DROP
> **Examen real:** Interpretar otra regla diferente (puede ser ACCEPT, OUTPUT, distinto puerto)

**Anatomía de un comando iptables:**

```
sudo iptables  -A INPUT  -s 192.168.1.0/24  -p tcp  --dport 110  -j DROP
               [cadena]  [origen]            [prot]  [puerto dst] [acción]
```

| Flag | Significado |
|------|------------|
| `-A INPUT` | Añadir regla al tráfico **entrante** |
| `-A OUTPUT` | Añadir regla al tráfico **saliente** |
| `-A FORWARD` | Tráfico que pasa por el equipo (enrutado) |
| `-s <IP/red>` | Origen (source) |
| `-d <IP/red>` | Destino (destination) |
| `-p tcp / udp` | Protocolo |
| `--dport <nº>` | Puerto destino |
| `--sport <nº>` | Puerto origen |
| `-j DROP` | Descartar el paquete silenciosamente |
| `-j REJECT` | Rechazar y notificar al origen |
| `-j ACCEPT` | Permitir el paquete |
| `-i eth0` | Interfaz de entrada |

**Puertos habituales a reconocer:**

| Puerto | Protocolo |
|--------|----------|
| 22 | SSH |
| 25 | SMTP (correo saliente) |
| 80 | HTTP |
| 110 | POP3 (correo entrante) |
| 143 | IMAP |
| 443 | HTTPS |
| 3389 | RDP (escritorio remoto) |

---

### Tema 9 — Proxy y caché

> **Ejemplo preguntó:** Qué es la caché del proxy y para qué sirve
> **Examen real:** Misma pregunta con distinta formulación o pregunta sobre qué es un proxy en general

**Caché del proxy:**
> La memoria caché de un proxy almacena copias locales de recursos web ya solicitados. Cuando otro usuario pide el mismo recurso, el proxy lo sirve desde la caché sin contactar al servidor original. Sirve para mejorar el rendimiento (menos tiempo de carga) y reducir el consumo de ancho de banda.

---

### Tema 10 — Squid: configuración por horario

> **Ejemplo preguntó:** Bloquear el fin de semana
> **Examen real:** Bloquear en otro horario o bloquear un dominio/sitio concreto

**Códigos de días:**

| Código | Día |
|--------|-----|
| M | Lunes |
| T | Martes |
| W | Miércoles |
| H | Jueves |
| F | Viernes |
| A | **Sábado** |
| S | **Domingo** |

**Variante probable** (bloquear horario laboral):
```squid
acl horario_laboral time MTWHF 09:00-18:00
http_access deny horario_laboral
```

**Bloquear un dominio:**
```squid
acl redes_sociales dstdomain .facebook.com .instagram.com
http_access deny redes_sociales
```

---

## NF4 — Almacenamiento y Alta Disponibilidad

### Tema 11 — RAID

> **Ejemplo preguntó:** RAID 5
> **Examen real:** RAID 0, RAID 1 o RAID 10 (los más probables)

| RAID | Alias | Mínimo de discos | Tolerancia a fallos | Rendimiento | Espacio útil |
|------|-------|-----------------|--------------------|-----------  |-------------|
| **RAID 0** | Striping | 2 | **Ninguna** (si falla 1 disco → pérdida total) | Muy alto | 100% (suma total) |
| **RAID 1** | Espejo / Mirroring | 2 | 1 disco puede fallar | Lectura/escritura simultánea en ambos discos | 50% (mitad para copia) |
| **RAID 5** | Paridad distribuida | 3 | 1 disco puede fallar | Alto (distribución de datos) | N-1 discos útiles |
| **RAID 10** | RAID 1+0 | 4 | Hasta 2 discos (uno por subgrupo) | Alto (RAID 0 sobre espejos) | 50% |

---

## Resumen: Lo que salió vs. Lo que estudias

| Ejemplo | Lo que salió | Lo que estudias para el real |
|---------|-------------|------------------------------|
| NF1 | Confidencialidad | **Integridad, Disponibilidad, Autenticación, No Repudio** |
| NF1 | Criptografía simétrica | **Criptografía asimétrica** |
| NF1 | NAS | **DAS y SAN** |
| NF1 | LSSICE publicidad | **LOPD** |
| NF2 | Virus | **Gusano, Troyano, Ransomware, Spyware, Botnet** |
| NF2 | Archivos + USB | **Phishing, Ingeniería social, Vulnerabilidades** |
| NF3 | Firewall definición | Misma estructura (dominar la definición) |
| NF3 | Iptables DROP | **Otras reglas: ACCEPT, OUTPUT, distinto puerto** |
| NF3 | Caché proxy | Misma estructura (dominar la definición) |
| NF3 | Squid fin de semana | **Horario laboral o bloqueo por dominio** |
| NF4 | RAID 5 | **RAID 0, RAID 1, RAID 10** |

---

*Fuentes: examen de ejemplo del profesor, Resumen Contenido NF1.pdf, Presentación MP11 UF2.pdf, nf4 material.pdf*
