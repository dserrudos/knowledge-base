# Manual Completo de Comandos de Terminal Linux

## SECCIÓN 1: NAVEGACIÓN Y ORIENTACIÓN

### 1.1 Ver la ruta actual (pwd)

**Sintaxis:**
```bash
pwd
```

**Ejemplo:**
```bash
pwd
# Salida: /home/dario/documentos
```

---

### 1.2 Listar contenido de carpetas (ls)

**Sintaxis:**
```bash
ls [opciones] [ruta]
```

**Parámetros principales:**
- `-l`: Lista detallada (permisos, tamaño, fecha)
- `-a`: Muestra archivos ocultos (los que empiezan con .)
- `-h`: Tamaños en formato legible (KB, MB, GB)
- `-R`: Lista recursivamente (incluye subcarpetas)
- `-t`: Ordena por fecha de modificación
- `-S`: Ordena por tamaño

**Ejemplos prácticos:**

**Listar archivos básico:**
```bash
ls
```

**Lista detallada:**
```bash
ls -l
```

**Mostrar archivos ocultos:**
```bash
ls -a
```

**Lista detallada con tamaños legibles:**
```bash
ls -lh
```

**Listar todo incluyendo ocultos, con detalles:**
```bash
ls -lah
```

**Listar ordenado por fecha (más reciente primero):**
```bash
ls -lt
```

**Listar contenido de otra carpeta:**
```bash
ls /home/dario/descargas
```

---

### 1.3 Navegar entre directorios (cd)

**Sintaxis:**
```bash
cd [ruta]
```

**Atajos especiales:**
- `~`: Directorio home del usuario
- `.`: Directorio actual
- `..`: Directorio padre (subir un nivel)
- `-`: Directorio anterior

**Ejemplos prácticos:**

**Ir al directorio home:**
```bash
cd ~
# o simplemente:
cd
```

**Ir a una ruta específica:**
```bash
cd /home/dario/documentos
```

**Subir un nivel:**
```bash
cd ..
```

**Subir dos niveles:**
```bash
cd ../..
```

**Ir al directorio anterior:**
```bash
cd -
```

**Ir a una carpeta relativa:**
```bash
cd fotos/vacaciones
```

---

## SECCIÓN 2: CREACIÓN

### 2.1 Crear carpetas (mkdir)

**Sintaxis:**
```bash
mkdir [opciones] nombre_carpeta
```

**Parámetros principales:**
- `-p`: Crea carpetas padres si no existen
- `-v`: Modo verbose (muestra qué se crea)

**Ejemplos prácticos:**

**Crear una carpeta:**
```bash
mkdir proyectos
```

**Crear múltiples carpetas:**
```bash
mkdir documentos fotos videos
```

**Crear carpeta con espacios (usar comillas):**
```bash
mkdir "mis documentos"
```

**Crear estructura de carpetas anidadas:**
```bash
mkdir -p proyectos/web/css
```

**Crear con confirmación:**
```bash
mkdir -v respaldos
```

---

### 2.2 Crear archivos (touch, echo)

**Sintaxis:**
```bash
touch nombre_archivo
echo "contenido" > nombre_archivo
```

**Ejemplos prácticos:**

**Crear archivo vacío:**
```bash
touch documento.txt
```

**Crear múltiples archivos vacíos:**
```bash
touch archivo1.txt archivo2.txt archivo3.txt
```

**Crear archivo con contenido:**
```bash
echo "Hola Mundo" > saludo.txt
```

**Agregar contenido a un archivo existente:**
```bash
echo "Nueva línea" >> saludo.txt
```

**Crear archivo con varias líneas:**
```bash
cat > notas.txt << EOF
Primera línea
Segunda línea
Tercera línea
EOF
```

---

## SECCIÓN 3: GESTIÓN DE ARCHIVOS Y CARPETAS

### 3.1 Copiar archivos y carpetas (cp)

**Sintaxis:**
```bash
cp [opciones] origen destino
```

**Parámetros principales:**
- `-r` o `-R`: Copia carpetas recursivamente
- `-v`: Modo verbose (muestra qué se copia)
- `-i`: Pregunta antes de sobrescribir
- `-u`: Copia solo si el origen es más nuevo

**Ejemplos prácticos:**

**Copiar un archivo:**
```bash
cp documento.txt documento_respaldo.txt
```

**Copiar a otra carpeta:**
```bash
cp /home/dario/documentos/informe.pdf /home/dario/respaldos/
```

**Copiar carpeta completa:**
```bash
cp -r /home/dario/fotos/vacaciones /home/dario/respaldos/
```

**Copiar múltiples archivos:**
```bash
cp archivo1.txt archivo2.txt archivo3.txt /home/dario/documentos/
```

**Copiar con confirmación:**
```bash
cp -i documento.txt /home/dario/documentos/
```

---

### 3.2 Mover archivos y carpetas (mv)

**Sintaxis:**
```bash
mv [opciones] origen destino
```

**Parámetros principales:**
- `-v`: Modo verbose
- `-i`: Pregunta antes de sobrescribir
- `-n`: No sobrescribe archivos existentes

**Ejemplos prácticos:**

**Mover un archivo:**
```bash
mv informe.pdf /home/dario/documentos/
```

**Mover una carpeta:**
```bash
mv /home/dario/descargas/proyecto /home/dario/trabajo/
```

**Mover múltiples archivos:**
```bash
mv foto1.jpg foto2.jpg foto3.jpg /home/dario/fotos/
```

---

### 3.3 Renombrar archivos y carpetas (mv)

**Sintaxis:**
```bash
mv nombre_actual nombre_nuevo
```

**Ejemplos prácticos:**

**Renombrar archivo:**
```bash
mv informe_viejo.pdf informe_2024.pdf
```

**Renombrar carpeta:**
```bash
mv fotos_vacaciones fotos_verano_2024
```

**Renombrar y mover simultáneamente:**
```bash
mv /home/dario/descargas/documento.txt /home/dario/documentos/informe_final.txt
```

---

### 3.4 Eliminar archivos y carpetas (rm, rmdir)

**⚠️ ADVERTENCIA CRÍTICA**
Los archivos eliminados NO van a la papelera. La eliminación es permanente e irreversible.

**Sintaxis:**
```bash
rm [opciones] archivo
rmdir carpeta_vacía
```

**Parámetros principales:**
- `-r`: Elimina carpetas recursivamente
- `-f`: Fuerza eliminación sin preguntar
- `-i`: Pregunta antes de eliminar
- `-v`: Muestra qué se elimina

**Ejemplos prácticos:**

**Eliminar un archivo:**
```bash
rm archivo.txt
```

**Eliminar con confirmación:**
```bash
rm -i archivo_importante.txt
```

**Eliminar múltiples archivos:**
```bash
rm foto1.jpg foto2.jpg documento.pdf
```

**Eliminar carpeta vacía:**
```bash
rmdir carpeta_vacia
```

**Eliminar carpeta con contenido:**
```bash
rm -r /home/dario/descargas/carpeta_temporal
```

**⚠️ PELIGRO: Eliminar sin preguntar:**
```bash
rm -rf /home/dario/descargas/carpeta_temporal
```

**Eliminar todos los archivos .txt:**
```bash
rm *.txt
```

---

## SECCIÓN 4: VISUALIZACIÓN DE CONTENIDO

### 4.1 Ver contenido completo (cat)

**Sintaxis:**
```bash
cat archivo
```

**Ejemplos prácticos:**

**Ver contenido de un archivo:**
```bash
cat documento.txt
```

**Ver múltiples archivos:**
```bash
cat archivo1.txt archivo2.txt
```

**Ver con números de línea:**
```bash
cat -n documento.txt
```

---

### 4.2 Ver contenido paginado (less, more)

**Sintaxis:**
```bash
less archivo
more archivo
```

**Controles en less:**
- `Espacio`: Siguiente página
- `b`: Página anterior
- `/texto`: Buscar texto
- `q`: Salir

**Ejemplos prácticos:**

**Ver archivo largo:**
```bash
less documento_largo.txt
```

**Ver con more:**
```bash
more informe.txt
```

---

### 4.3 Ver inicio o final de archivos (head, tail)

**Sintaxis:**
```bash
head [opciones] archivo
tail [opciones] archivo
```

**Parámetros principales:**
- `-n X`: Muestra X líneas (por defecto 10)
- `-f`: (solo tail) Modo seguimiento en tiempo real

**Ejemplos prácticos:**

**Ver primeras 10 líneas:**
```bash
head documento.txt
```

**Ver primeras 20 líneas:**
```bash
head -n 20 documento.txt
```

**Ver últimas 10 líneas:**
```bash
tail documento.txt
```

**Ver últimas 30 líneas:**
```bash
tail -n 30 log.txt
```

**Seguir archivo en tiempo real (logs):**
```bash
tail -f /var/log/syslog
```

---

## SECCIÓN 5: BÚSQUEDA

### 5.1 Buscar archivos y carpetas (find)

**Sintaxis:**
```bash
find [ruta] [opciones] [expresión]
```

**Parámetros principales:**
- `-name`: Busca por nombre
- `-iname`: Busca por nombre (ignora mayúsculas)
- `-type f`: Solo archivos
- `-type d`: Solo carpetas
- `-size`: Busca por tamaño

**Ejemplos prácticos:**

**Buscar archivo por nombre:**
```bash
find /home/dario -name "documento.txt"
```

**Buscar ignorando mayúsculas:**
```bash
find /home/dario -iname "INFORME.pdf"
```

**Buscar todos los archivos .txt:**
```bash
find /home/dario/documentos -name "*.txt"
```

**Buscar solo carpetas:**
```bash
find /home/dario -type d -name "fotos"
```

**Buscar archivos modificados en últimas 24 horas:**
```bash
find /home/dario -type f -mtime -1
```

**Buscar archivos mayores a 100MB:**
```bash
find /home/dario -type f -size +100M
```

---

### 5.2 Buscar texto dentro de archivos (grep)

**Sintaxis:**
```bash
grep [opciones] "texto" archivo
```

**Parámetros principales:**
- `-i`: Ignora mayúsculas/minúsculas
- `-r`: Busca recursivamente en carpetas
- `-n`: Muestra número de línea
- `-v`: Muestra líneas que NO coinciden
- `-c`: Cuenta coincidencias

**Ejemplos prácticos:**

**Buscar texto en un archivo:**
```bash
grep "error" log.txt
```

**Buscar ignorando mayúsculas:**
```bash
grep -i "ERROR" log.txt
```

**Buscar con número de línea:**
```bash
grep -n "TODO" codigo.py
```

**Buscar en todos los archivos de una carpeta:**
```bash
grep -r "contraseña" /home/dario/documentos/
```

**Buscar líneas que NO contienen texto:**
```bash
grep -v "comentario" script.sh
```

**Contar coincidencias:**
```bash
grep -c "error" log.txt
```

---

## SECCIÓN 6: TRANSFERENCIA DE ARCHIVOS

### 6.1 Transferir entre Windows y Ubuntu (SCP)

**Sintaxis:**
```bash
scp [opciones] origen destino
```

**Parámetros principales:**
- `-r`: Copia carpetas recursivamente
- `-P`: Especifica puerto SSH (por defecto 22)
- `-v`: Modo verbose

### Desde Windows → Ubuntu (PowerShell)

**Copiar un archivo:**
```bash
scp C:\Users\DARIO\Downloads\archivo.txt dario@192.168.1.200:/home/dario/descargas/
```

**Copiar una carpeta completa:**
```bash
scp -r C:\Users\DARIO\Documents\carpeta dario@192.168.1.200:/home/dario/descargas/
```

### Desde Ubuntu → Windows (Terminal Ubuntu)

**Copiar un archivo:**
```bash
scp /home/dario/descargas/foto_oculta.jpg DARIO@192.168.1.130:/mnt/c/Users/DARIO/Downloads/
```

**Copiar una carpeta completa:**
```bash
scp -r /home/dario/descargas/carpeta DARIO@192.168.1.130:/mnt/c/Users/DARIO/Downloads/
```

**Notas importantes:**
- Te pedirá la contraseña del usuario destino
- SSH debe estar activo en el sistema destino
- Verifica IPs con `ipconfig` (Windows) o `ip a` (Ubuntu)

---

## SECCIÓN 7: PERMISOS Y PROPIEDADES

### 7.1 Cambiar permisos (chmod)

**Sintaxis:**
```bash
chmod [opciones] permisos archivo
```

**Permisos numéricos:**
- `7` = rwx (leer, escribir, ejecutar)
- `6` = rw- (leer, escribir)
- `5` = r-x (leer, ejecutar)
- `4` = r-- (solo leer)

**Estructura:** `chmod XYZ archivo`
- X = permisos del propietario
- Y = permisos del grupo
- Z = permisos de otros

**Ejemplos prácticos:**

**Hacer archivo ejecutable:**
```bash
chmod +x script.sh
```

**Permisos completos para propietario, lectura para otros:**
```bash
chmod 744 archivo.sh
```

**Lectura y escritura para todos:**
```bash
chmod 666 documento.txt
```

**Cambiar permisos recursivamente:**
```bash
chmod -R 755 /home/dario/scripts/
```

---

### 7.2 Cambiar propietario (chown)

**Sintaxis:**
```bash
sudo chown [opciones] usuario:grupo archivo
```

**Ejemplos prácticos:**

**Cambiar propietario:**
```bash
sudo chown dario archivo.txt
```

**Cambiar propietario y grupo:**
```bash
sudo chown dario:dario archivo.txt
```

**Cambiar recursivamente:**
```bash
sudo chown -R dario:dario /home/dario/proyectos/
```

---

## SECCIÓN 8: COMPRESIÓN Y ARCHIVOS

### 8.1 Comprimir y descomprimir con tar

**Sintaxis:**
```bash
tar [opciones] archivo.tar.gz carpeta
```

**Parámetros principales:**
- `-c`: Crear archivo
- `-x`: Extraer archivo
- `-v`: Modo verbose
- `-f`: Especifica nombre de archivo
- `-z`: Comprime con gzip

**Ejemplos prácticos:**

**Crear archivo comprimido:**
```bash
tar -czvf respaldo.tar.gz /home/dario/documentos/
```

**Extraer archivo comprimido:**
```bash
tar -xzvf respaldo.tar.gz
```

**Extraer en carpeta específica:**
```bash
tar -xzvf respaldo.tar.gz -C /home/dario/restaurar/
```

**Ver contenido sin extraer:**
```bash
tar -tzvf respaldo.tar.gz
```

---

### 8.2 Comprimir con zip/unzip

**Sintaxis:**
```bash
zip [opciones] archivo.zip archivos
unzip archivo.zip
```

**Ejemplos prácticos:**

**Comprimir archivo:**
```bash
zip comprimido.zip documento.txt
```

**Comprimir carpeta:**
```bash
zip -r proyecto.zip /home/dario/proyecto/
```

**Descomprimir:**
```bash
unzip archivo.zip
```

**Descomprimir en carpeta específica:**
```bash
unzip archivo.zip -d /home/dario/extraidos/
```

**Ver contenido sin extraer:**
```bash
unzip -l archivo.zip
```

---

## SECCIÓN 9: ESPACIO Y SISTEMA

### 9.1 Ver espacio en disco (df)

**Sintaxis:**
```bash
df [opciones]
```

**Ejemplos prácticos:**

**Ver espacio disponible:**
```bash
df -h
```

**Ver solo sistema de archivos específico:**
```bash
df -h /home
```

---

### 9.2 Ver tamaño de carpetas (du)

**Sintaxis:**
```bash
du [opciones] carpeta
```

**Parámetros principales:**
- `-h`: Formato legible
- `-s`: Resumen total
- `-a`: Incluye archivos

**Ejemplos prácticos:**

**Ver tamaño de carpeta actual:**
```bash
du -sh
```

**Ver tamaño de carpeta específica:**
```bash
du -sh /home/dario/descargas/
```

**Ver tamaño de todas las subcarpetas:**
```bash
du -h --max-depth=1
```

---

## SECCIÓN 10: ENLACES SIMBÓLICOS

### 10.1 Crear enlaces (ln)

**Sintaxis:**
```bash
ln -s archivo_original enlace
```

**Ejemplos prácticos:**

**Crear enlace simbólico:**
```bash
ln -s /home/dario/documentos/proyecto proyecto_link
```

**Crear enlace a carpeta:**
```bash
ln -s /home/dario/descargas descargas_link
```

---

## TABLA DE REFERENCIA RÁPIDA

| Categoría | Comando | Descripción | Ejemplo |
|-----------|---------|-------------|---------|
| **Navegación** | `pwd` | Ver ubicación actual | `pwd` |
| | `ls` | Listar archivos | `ls -lah` |
| | `cd` | Cambiar directorio | `cd /home/dario` |
| **Creación** | `mkdir` | Crear carpeta | `mkdir proyectos` |
| | `touch` | Crear archivo vacío | `touch nota.txt` |
| | `echo` | Crear con contenido | `echo "texto" > file.txt` |
| **Gestión** | `cp` | Copiar | `cp -r origen destino` |
| | `mv` | Mover/Renombrar | `mv viejo.txt nuevo.txt` |
| | `rm` | Eliminar | `rm -rf carpeta` |
| **Visualización** | `cat` | Ver archivo completo | `cat documento.txt` |
| | `less` | Ver paginado | `less archivo.txt` |
| | `head` | Ver inicio | `head -n 20 file.txt` |
| | `tail` | Ver final | `tail -f log.txt` |
| **Búsqueda** | `find` | Buscar archivos | `find . -name "*.txt"` |
| | `grep` | Buscar en contenido | `grep -r "error" logs/` |
| **Transferencia** | `scp` | Copiar por SSH | `scp file user@ip:/path` |
| **Permisos** | `chmod` | Cambiar permisos | `chmod 755 script.sh` |
| | `chown` | Cambiar propietario | `chown user:group file` |
| **Compresión** | `tar` | Comprimir/extraer | `tar -czvf file.tar.gz dir/` |
| | `zip/unzip` | Comprimir zip | `zip -r file.zip dir/` |
| **Sistema** | `df` | Espacio en disco | `df -h` |
| | `du` | Tamaño de carpetas | `du -sh carpeta/` |

---

## CONSEJOS FINALES

✅ Usa **Tab** para autocompletar nombres de archivos y rutas
✅ Usa **Ctrl + C** para cancelar comandos en ejecución
✅ Usa **Ctrl + L** para limpiar la pantalla (o comando `clear`)
✅ Usa **↑** y **↓** para navegar por el historial de comandos
✅ Usa `-i` en comandos destructivos (rm, cp, mv) cuando estés aprendiendo
✅ Prueba comandos con archivos de prueba antes de usarlos con datos importantes
✅ El comando `man comando` muestra el manual completo de cualquier comando
✅ El comando `history` muestra todos los comandos que has ejecutado
