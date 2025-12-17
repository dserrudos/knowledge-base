# Steghide - Guía Rápida de Esteganografía

Ocultar y extraer información secreta en imágenes JPEG/BMP y audio WAV.

---

## ⚙️ Instalación

```bash
sudo apt install steghide
```

---

## 🔐 Ocultar Mensaje

```bash
# Crear mensaje
echo "Mensaje secreto" > secreto.txt

# Ocultar en imagen (pedirá contraseña)
steghide embed -cf foto_original.jpg -ef secreto.txt -sf foto_oculta.jpg

# Sin contraseña (presiona Enter)
steghide embed -cf foto.jpg -ef secreto.txt -sf foto_oculta.jpg -p ""
```

---

## 🔓 Extraer Mensaje

```bash
# Extraer (pedirá contraseña)
steghide extract -sf foto_oculta.jpg

# Leer mensaje extraído
cat secreto.txt
```

---

## ℹ️ Ver Información

```bash
# Ver si imagen contiene datos ocultos
steghide info foto_oculta.jpg
```

---

## 📝 Flags Principales

| Flag | Descripción | Ejemplo |
|------|-------------|---------|
| `-cf` | Cover file (archivo original) | `-cf foto.jpg` |
| `-ef` | Embed file (archivo a ocultar) | `-ef secreto.txt` |
| `-sf` | Stego file (archivo con datos ocultos) | `-sf foto_oculta.jpg` |
| `-p` | Passphrase (contraseña) | `-p "MiClave123"` |
| `-xf` | Extract file (nombre al extraer) | `-xf mensaje.txt` |
| `-f` | Force (forzar sobrescritura) | `-f` |

---

## 🎯 Casos de Uso Rápidos

**Ocultar PDF en imagen:**
```bash
steghide embed -cf paisaje.jpg -ef documento.pdf -sf paisaje_seguro.jpg
```

**Ocultar múltiples archivos (comprimir primero):**
```bash
tar -czf archivos.tar.gz *.txt
steghide embed -cf foto.jpg -ef archivos.tar.gz -sf foto_oculta.jpg
```

**Ocultar en audio:**
```bash
steghide embed -cf cancion.wav -ef secreto.txt -sf cancion_oculta.wav
```

---

## ⚠️ Notas Importantes

✅ **Formatos soportados:** JPEG, BMP, WAV, AU  
❌ **No soporta:** PNG directamente  
🔒 **Seguridad:** Usar contraseñas fuertes (min. 12 caracteres)  
👁️ **Invisible:** La imagen se ve idéntica a simple vista  
📦 **Capacidad:** Depende del tamaño de la imagen (usar imágenes grandes)

---

## 🔧 Troubleshooting

**Error "could not embed":** Archivo muy grande → usar imagen más grande  
**Error "could not extract":** Contraseña incorrecta o sin datos ocultos  
**Formato no soportado:** Convertir PNG a JPG: `convert imagen.png imagen.jpg`

---

## 📚 Recursos

- Manual completo: `man steghide`
- Sitio oficial: http://steghide.sourceforge.net/
