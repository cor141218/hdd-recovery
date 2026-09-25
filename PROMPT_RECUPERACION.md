# Prompt completo para Claude Code: recuperación de fotos y música

Copia todo el bloque y pégalo dentro de `claude` en la terminal de Ubuntu Live.

```
Eres un especialista en recuperación de datos. Estoy en Ubuntu Desktop Live. Tengo un disco duro donde reinstalaron Windows y se borraron fotos y música que quiero recuperar. También tengo un disco de 20 TB para guardar todo. Quiero el mejor resultado posible, con los archivos ordenados, verificados y sin duplicados. Explícame todo en español.

REGLAS DE SEGURIDAD (obligatorias):
- El disco de origen es SOLO LECTURA. Nunca lo montes en escritura, nunca lo formatees, nunca ejecutes chkdsk, fsck ni ntfsfix sobre él, y nunca guardes nada en él. Márcalo como solo lectura con: sudo blockdev --setro /dev/sdX
- Pídeme confirmación antes de cualquier comando que monte, formatee, particione o escriba en un disco.
- Todo lo que generes va al disco de 20 TB, dentro de BACKUP_RECUPERACION/.
- Antes de instalar algo desde GitHub, muéstrame el repositorio, cuántas estrellas tiene, la fecha de su última actualización y su licencia, y espera mi aprobación. Da prioridad a los paquetes oficiales de apt. Nunca ejecutes scripts desconocidos con sudo sin preguntarme.
- Lleva un registro de todo lo que hagas en BACKUP_RECUPERACION/logs/bitacora.txt.

FASE 1 – Herramientas
Activa el repositorio universe y luego instala con apt:
testdisk (incluye PhotoRec), gddrescue, smartmontools, sleuthkit, foremost, scalpel, ntfs-3g, libimage-exiftool-perl, jdupes, jpeginfo, ffmpeg, python3-mutagen, pv
Si alguna herramienta no está en apt, búscala en GitHub (siguiendo las reglas de seguridad de arriba) y compílala o instálala en ~/tools.
Al final, verifica que cada herramienta funciona mostrando su versión.

FASE 2 – Diagnóstico
1. Ejecuta: lsblk -o NAME,SIZE,FSTYPE,LABEL,MODEL,SERIAL,MOUNTPOINT
   Identifica el disco de origen y el de 20 TB, y espera a que yo te lo confirme.
2. Ejecuta smartctl -a sobre el disco de origen. Interpreta el resultado: sectores reasignados o pendientes, horas de uso, estado general. Dime si el disco está sano o en riesgo.
3. Monta el disco de 20 TB (sin formatearlo) y crea esta estructura:
   BACKUP_RECUPERACION/{imagen,particiones,recuperado_con_nombres,photorec,foremost,fotos,musica,dudosos,duplicados,logs}
   Muestra el espacio libre con df -h.

FASE 3 – Clonar el disco (trabajar sobre la copia, nunca sobre el original)
1. Clona con ddrescue en tres pasadas:
   - primero rápido: -n
   - después reintentos: -r3
   - ambas usando el mapa: BACKUP_RECUPERACION/imagen/disco.map
   La imagen se guarda en BACKUP_RECUPERACION/imagen/disco.img. Muéstrame el progreso y cuánto espacio quedó sin leer.
2. A partir de aquí, trabaja SOLO sobre disco.img. Conéctala en solo lectura con: sudo losetup -r -P --find --show disco.img

FASE 4 – Recuperar con nombres originales (primero lo mejor)
1. Ejecuta TestDisk (Analyse + Deeper Search) sobre la imagen para buscar particiones NTFS/FAT anteriores al reinstalado de Windows. Guíame por el menú, porque es interactivo, y explícame qué elegir en cada pantalla.
2. Por cada partición encontrada (actual o antigua):
   - Lista los archivos borrados con los nombres y carpetas originales usando sleuthkit (mmls, fls -rd) y ntfsundelete --scan.
   - Recupera las fotos y la música con tsk_recover y ntfsundelete a BACKUP_RECUPERACION/recuperado_con_nombres/, conservando las carpetas.
   - También puedes montar la partición en solo lectura (-o ro) y copiar lo que siga visible.

FASE 5 – PhotoRec (carving de todo el disco)
Dame el comando exacto para ejecutarlo en otra terminal:
  sudo photorec /log /d BACKUP_RECUPERACION/photorec/rec disco.img
Explícame qué elegir en cada pantalla:
- Whole disk
- File Opt: desmarca todo con "s" y marca solo jpg, png, gif, bmp, tif, heic, cr2, nef, arw, dng, mp3, flac, m4a/mov, wma (asf), ogg, wav, aif; guarda con "b"
- Paranoid: Yes
- Other
- Whole
Mientras corre, revisa el avance periódicamente y dime cuántos archivos van.

FASE 6 – Segunda pasada opcional
Si PhotoRec recuperó pocos archivos, o si ves muchos corruptos, ejecuta foremost solo con jpg, png y mp3 hacia BACKUP_RECUPERACION/foremost/ para comparar resultados.

FASE 7 – Dejar todo impecable
1. Verificación:
   - Fotos: comprueba con jpeginfo -c y exiftool.
   - Audio: comprueba con ffprobe.
   - Los archivos corruptos o incompletos van a dudosos/ (no los borres).
2. Filtrado: las imágenes de menos de 50 KB o de menos de 300x300 px (miniaturas o iconos de Windows) van a dudosos/miniaturas/.
3. Duplicados: busca con jdupes entre todas las carpetas. Deja una sola copia y da preferencia a la de recuperado_con_nombres/. Mueve las copias sobrantes a duplicados/ (no las borres).
4. Organizar fotos:
   - Con exiftool, organízalas en fotos/AAAA/MM/AAAAMMDD_HHMMSS.ext usando DateTimeOriginal.
   - Las que no tengan fecha van a fotos/sin_fecha/.
5. Organizar música:
   - Lee las etiquetas con mutagen o exiftool.
   - Organízala en musica/Artista/Álbum/NN - Título.ext.
   - La música sin etiquetas va a musica/sin_etiquetas/.
6. Arregla los permisos para que mi usuario pueda abrir todo: chown -R y chmod.

FASE 8 – Informe final
Crea BACKUP_RECUPERACION/INFORME.txt con:
- Estado del disco de origen.
- Cuántos archivos se recuperaron por tipo y por método (con nombre / PhotoRec / foremost).
- Cuántos resultaron válidos, dudosos y duplicados.
- Espacio total usado.
- Qué más se podría intentar.
Muéstrame el resumen al terminar.

Ve paso a paso. Al terminar cada fase, dime qué encontraste y qué sigue.
```

## Si Claude Code se detiene a mitad

Pega este prompt para que retome donde se quedó:

```
Lee BACKUP_RECUPERACION/logs/bitacora.txt y continúa desde la fase en la que nos quedamos, respetando las mismas reglas de seguridad.
```
