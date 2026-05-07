# Simulacro de Examen — Seguridad y Alta Disponibilidad

> Instrucciones: responde cada pregunta como si fuera el examen real.
> Cubre la sección de respuestas con otra ventana o papel hasta terminar.
> Tiempo recomendado: **50 minutos**. Puntuación total: **10 puntos**.

---

# PARTE I — PREGUNTAS DEL EXAMEN

*(No mires las respuestas hasta terminar todas las preguntas)*

---

### NF1 — Fundamentos de Seguridad *(4 puntos)*

**Pregunta 1** *(0,75 pts)* — Dentro de un sistema informático, ¿a qué nos referimos con **Disponibilidad** de los datos?

**Pregunta 2** *(1 pto)* — Explica el funcionamiento básico de la **criptografía asimétrica**: a) ¿Cuántas claves utiliza y cómo se generan? b) ¿Para qué sirve cada clave? c) ¿Cuál es su principal desventaja frente a la criptografía simétrica?

**Pregunta 3** *(0,75 pts)* — a) Define qué es una **SAN**. b) Explica cuál es su diferencia principal respecto a un NAS y en qué tipo de organización se utiliza.

**Pregunta 4** *(1,5 pts)* — Una empresa de e-commerce recoge datos personales de sus clientes. ¿Qué obligaciones tiene según la **LOPD** respecto a la información que debe facilitar a los clientes al recoger esos datos?

---

### NF2 — Amenazas y Malware *(2 puntos)*

**Pregunta 5** *(1 pto)* — En qué consiste un **ransomware** y qué efectos puede ocasionar a un sistema.

**Pregunta 6** *(1 pto)* — Cita y explica dos métodos de infiltración maliciosa en redes **distintos** a los archivos adjuntos y los dispositivos extraíbles.

---

### NF3 — Seguridad Perimetral *(3 puntos)*

**Pregunta 7** *(0,75 pts)* — a) Define qué es un **proxy**. b) Explica cuál es su funcionalidad principal dentro de una red.

**Pregunta 8** *(0,75 pts)* — ¿Cuál sería el efecto de este comando?
```bash
sudo iptables -A OUTPUT -d 0.0.0.0/0 -p tcp --dport 22 -j REJECT
```

**Pregunta 9** *(0,75 pts)* — ¿Qué diferencia hay entre `-j DROP` y `-j REJECT` en iptables? ¿Cuándo usarías cada uno?

**Pregunta 10** *(0,75 pts)* — Se quiere bloquear el acceso a internet de lunes a viernes de 09:00 a 18:00. ¿Qué comandos añadirías al Squid?

---

### NF4 — Almacenamiento y Alta Disponibilidad *(1 punto)*

**Pregunta 11** *(1 pto)* — a) Explica qué es un **RAID 1**. b) Indica ventajas e inconvenientes en cuanto a tolerancia a fallos y aprovechamiento del espacio.

---
---
---

# PARTE II — CORRECCIÓN DEL PROFESOR

---

## R1 — Disponibilidad *(0,75 pts)*
> La Disponibilidad es la capacidad de un servicio, datos o sistema de ser accesible y utilizable por los usuarios autorizados cuando lo requieran, evitando su pérdida o bloqueo.

**Palabras clave:** "accesible/utilizable" · "usuarios autorizados" · "cuando lo requieran" · "evitar pérdida o bloqueo"
**Trampa:** No mencionar "usuarios autorizados" — la disponibilidad no es para todos, solo para los autorizados.

---

## R2 — Criptografía Asimétrica *(1 pto)*
> a) Usa **dos claves** generadas al mismo tiempo: pública y privada.
> b) **Pública**: cifra el mensaje / establece el canal. **Privada** (solo el propietario): descifra.
> c) Es más **lenta** que la simétrica — claves de 2048 bits requieren mayor carga computacional.

**Palabras clave:** "dos claves" · "pública cifra / privada descifra" · "más lenta"
**Trampa:** Invertir las claves (privada cifra, pública descifra) → pierde el apartado b completo.

---

## R3 — SAN *(0,75 pts)*
> a) SAN (Storage Area Network): red de almacenamiento de alta velocidad independiente a la que los dispositivos se conectan directamente.
> b) Diferencia con NAS: el NAS se conecta a la LAN como carpeta de red; la SAN es una red de almacenamiento propia y exclusiva. Solo viable en **grandes organizaciones** por el alto coste.

**Palabras clave:** "red dedicada/independiente" · "NAS = red local / SAN = red propia" · "grandes organizaciones"
**Trampa:** Describir SAN como "un NAS más grande" — son arquitecturas distintas.

---

## R4 — LOPD *(1,5 pts)*
> Obligaciones al recoger datos (Art. 5): 1) titular del fichero 2) finalidades del tratamiento 3) carácter obligatorio o voluntario 4) derechos del interesado y cómo ejercerlos 5) dirección para ejercerlos.
> El interesado puede ejercer derechos **ARCO**: Acceso, Rectificación, Cancelación, Oposición, y revocar el consentimiento.

**Palabras clave:** al menos 3 de los 5 puntos informativos · "ARCO" o 2 de sus derechos
**Trampa:** Confundir con LSSICE (publicidad por email) → si aparece "Publicidad" o "Publi" en la respuesta, error de concepto.

---

## R5 — Ransomware *(1 pto)*
> Malware que **cifra** los archivos del sistema y exige **rescate económico** para recuperar el acceso. Efectos: pérdida de acceso a datos, paralización de la actividad, pérdida económica, posible filtración de datos (doble extorsión).

**Palabras clave:** "cifra" (no borra) · "rescate económico" · 2 efectos concretos
**Trampa:** Decir que "borra" los datos — el ransomware los cifra (hace inaccesibles), no los elimina.

---

## R6 — Métodos de Infiltración *(1 pto)*
> **Phishing:** suplantación de identidad mediante correos/webs falsas que imitan entidades legítimas para robar credenciales.
> **Ingeniería social:** manipulación psicológica para que el usuario realice acciones que vulneren la seguridad (revelar contraseñas, ejecutar archivos).

**Palabras clave:** Phishing: "suplantación de identidad" · "webs/correos falsos" · "credenciales" | Ingeniería social: "manipulación psicológica" · "confianza"
**Trampa:** Describir phishing como "cualquier correo falso" — debe incluir la suplantación de identidad.

---

## R7 — Proxy *(0,75 pts)*
> a) Servidor **intermediario** entre la red interna e internet que gestiona las peticiones de los usuarios en su nombre.
> b) Controla y filtra el tráfico web: bloquea páginas, registra actividad, mejora rendimiento con caché, oculta IPs internas.

**Palabras clave:** "intermediario" · 2 funciones concretas (filtrado, caché, registro, anonimato)
**Trampa:** Confundir con firewall — el proxy actúa a nivel de aplicación (URLs), el firewall a nivel de red (IPs/puertos).

---

## R8 — Iptables OUTPUT *(0,75 pts)*
> Bloquea todo el tráfico TCP **saliente** del servidor hacia cualquier destino al puerto 22 (SSH). REJECT notifica al origen del rechazo (diferencia con DROP que descarta silenciosamente). Impide que el servidor inicie conexiones SSH hacia otros equipos.

**Palabras clave:** "OUTPUT = saliente" · "puerto 22 = SSH" · "REJECT notifica al origen"
**Trampa:** Leer OUTPUT como tráfico entrante — OUTPUT es el tráfico que **genera el propio servidor**.

---

## R9 — DROP vs REJECT *(0,75 pts)*
> **DROP**: descarta el paquete silenciosamente — el origen no recibe respuesta (timeout). Útil contra atacantes externos (no revela que hay firewall).
> **REJECT**: rechaza y envía mensaje de error al origen. Útil en redes internas para informar al usuario.

**Palabras clave:** DROP: "silencioso/timeout" | REJECT: "notifica/mensaje de error" · criterio de uso para cada uno

---

## R10 — Squid horario laboral *(0,75 pts)*
```squid
acl horario_laboral time MTWHF 09:00-18:00
http_access deny horario_laboral
```
**Palabras clave:** `acl` + `time` + `MTWHF` + `09:00-18:00` + `http_access deny`
**Trampa:** Escribir `http_access block` (no existe) o usar `R` para jueves (el jueves es `H` de tHursday).

---

## R11 — RAID 1 *(1 pto)*
> RAID 1 (espejo/mirroring): replica todos los datos de un disco en un segundo disco de forma sincronizada.
> **Tolerancia a fallos:** muy alta — si falla un disco el sistema sigue con el otro, sin pérdida de datos.
> **Espacio:** inconveniente principal — solo se aprovecha el **50%** (ej: 2×2TB = 2TB útiles).

**Palabras clave:** "espejo/mirroring/replica" · "sigue funcionando si falla uno" · "50%"
**Trampa:** Mencionar "paridad" — la paridad es de RAID 5, no RAID 1. RAID 1 usa espejo.

---

# TABLA DE PUNTUACIÓN

| P | Tema | Pts | Mis pts |
|---|------|-----|---------|
| 1 | Disponibilidad | 0,75 | |
| 2 | Criptografía asimétrica | 1,00 | |
| 3 | SAN | 0,75 | |
| 4 | LOPD | 1,50 | |
| 5 | Ransomware | 1,00 | |
| 6 | Phishing + Ingeniería Social | 1,00 | |
| 7 | Proxy | 0,75 | |
| 8 | Iptables OUTPUT | 0,75 | |
| 9 | DROP vs REJECT | 0,75 | |
| 10 | Squid horario laboral | 0,75 | |
| 11 | RAID 1 | 1,00 | |
| **TOTAL** | | **10,00** | |

**Escala:** 9-10 Excelente · 7-8,9 Bien · 5-6,9 Aprobado justo · <5 Repetir simulacro
