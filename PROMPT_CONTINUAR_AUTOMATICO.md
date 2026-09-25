# Prompt para que Claude Code termine todo solo y deje carpetas listas para revisar

Pégalo en la misma sesión de `claude` donde ddrescue está corriendo.

```
Cambio de instrucciones: a partir de ahora trabaja de forma AUTÓNOMA hasta terminar todo, sin pedirme confirmación. Ya te confirmé los discos: sda es el origen (solo lectura) y el disco de 20 TB es el destino. Estas reglas se mantienen siempre:
- sda nunca se escribe, se monta ni se repara.
- Nunca borres nada. Lo que no sirva se MUEVE a carpetas de descarte.
- Nada de TestDisk interactivo. Usa herramientas no interactivas.

1) Crea un script maestro en /mnt/destino/BACKUP_RECUPERACION/recuperar_todo.sh y lánzalo con nohup en segundo plano. Así el proceso sigue aunque tú te detengas o se cierre la terminal. El script debe:
   - Esperar a que terminen las 3 pasadas de ddrescue. Si alguna fase falla, registrar el error y pasar a la siguiente, sin detenerse.
   - Anotar el inicio y el fin de cada fase en logs/bitacora.txt y el estado actual en logs/ESTADO.txt (fase, porcentaje, archivos encontrados).
   - Ser reanudable: si lo vuelvo a lanzar, debe saltarse las fases que ya terminaron (usa archivos marcador .hecho).

2) Fases del script:
   a) Conecta disco.img con: losetup -r -P. Obtén las particiones con mmls. En cada partición NTFS/FAT (actual o antigua), recupera los archivos borrados CON NOMBRE y carpeta original usando tsk_recover (con el offset de mmls) y ntfsundelete, hacia trabajo/con_nombre/.
   b) Ejecuta PhotoRec en modo NO interactivo con /cmd sobre disco.img, sin particionar (partition_none), en modo paranoid, con solo los formatos de foto y audio (jpg, png, gif, bmp, tif, heic/mov, cr2, nef, arw, dng, mp3, flac, m4a, wma/asf, ogg, wav, aif) y salida a trabajo/photorec/rec. Revisa en photorec /help y en la documentación de CGSecurity la sintaxis exacta de /cmd y los nombres de formatos antes de lanzarlo. Haz una prueba de 1 minuto para confirmar que arranca bien.
   c) Si PhotoRec recuperó menos de 1000 fotos, pasa también foremost hacia trabajo/foremost/.
   d) Verifica cada archivo:
      - Imágenes: jpeginfo -c, y exiftool para dimensiones y EXIF.
      - Audio: ffprobe para ver la duración.
   e) Clasifica y organiza (con MOVER, no copiar) en esta estructura final:

   /mnt/destino/BACKUP_RECUPERACION/REVISAR/
     01_FOTOS_CON_NOMBRE_ORIGINAL/      (conserva el árbol de carpetas original)
     02_FOTOS_CAMARA_CELULAR/AAAA/MM/   (tienen EXIF Make/Model; nombre AAAAMMDD_HHMMSS.ext)
     03_FOTOS_SIN_FECHA/                (válidas, de 800 px o más, sin EXIF de cámara)
     04_MUSICA_CON_NOMBRE_ORIGINAL/
     05_MUSICA_POR_ARTISTA/Artista/Album/NN - Titulo.ext
     06_MUSICA_SIN_ETIQUETAS/           (dura más de 60 s)
     07_PROBABLE_BASURA/
         miniaturas_iconos/             (menos de 800 px o menos de 50 KB)
         imagenes_de_programas/         (png/gif/bmp sin EXIF, típicos de Windows y apps)
         sonidos_de_sistema/            (duran menos de 60 s)
     08_CORRUPTOS/
     09_DUPLICADOS/                     (usa jdupes; deja 1 copia y da preferencia a la que tiene nombre original)

   f) Vista previa rápida: con ImageMagick (montage) genera hojas de contacto JPG de 100 miniaturas cada una, con el nombre del archivo debajo. Hazlo en cada subcarpeta de 01, 02, 03 y 07, y guárdalas en REVISAR/_VISTA_PREVIA/ con la misma estructura. Así puedo decidir rápido qué me sirve.
   g) Genera REVISAR/INVENTARIO.csv con estas columnas: ruta, tipo, tamaño, dimensiones o duración, fecha EXIF, cámara, artista, álbum, título, método de recuperación.
   h) Genera REVISAR/LEEME.txt en español: qué hay en cada carpeta, cuántos archivos y cuánto ocupa cada una, y qué conviene revisar primero.
   i) Haz chown -R 999:999 (usuario ubuntu del Live) y chmod -R u+rwX,go+rX sobre REVISAR para que yo pueda abrir todo.
   j) Al final, separa disco.img y trabajo/ (lo que quede) en BACKUP_RECUPERACION/_NO_TOCAR/. Luego escribe FIN en ESTADO.txt.

3) Instala sin preguntarme lo que falte (imagemagick, jdupes, jpeginfo, ffmpeg, libimage-exiftool-perl, python3-mutagen, sleuthkit, foremost). Usa apt primero. Solo si algo no existe en apt, bájalo de un repositorio GitHub reconocido y anota cuál en la bitácora.

4) Después de lanzar el script, vigílalo: revisa ESTADO.txt y los logs periódicamente. Si falla, corrige el script y relánzalo (es reanudable). Solo al final dame el resumen de LEEME.txt.
```

## Para que no se detenga pidiendo permiso

- Cuando Claude Code pregunte si puede ejecutar un comando, elige la opción
  **"Yes, and don't ask again…"**.
- En **Ajustes → Energía**, desactiva la suspensión y el apagado de pantalla con bloqueo.
- Aunque cierres Claude, el script sigue corriendo. Para ver el avance:
  `cat /mnt/destino/BACKUP_RECUPERACION/logs/ESTADO.txt`
- Si se reinicia la PC, el Live se borra. Vuelve a montar el disco de 20 TB y ejecuta:
  `sudo nohup bash /mnt/destino/BACKUP_RECUPERACION/recuperar_todo.sh &`
