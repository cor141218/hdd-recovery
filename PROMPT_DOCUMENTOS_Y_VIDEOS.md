# Prompt: recuperar documentos + reparar los 116 vídeos _ftyp.mov

Conecta la laptop a la corriente y el disco de 20 TB a un puerto USB 3.0 (el azul). Luego pega esto en `claude`:

```
Nueva tarea (con las mismas reglas de siempre: el Seagate de origen no se toca, no borres nada y trabaja de forma autónoma hasta terminar):
Además de las fotos y la música, necesito recuperar DOCUMENTOS perdidos. Hasta ahora PhotoRec solo buscó fotos y audio.

0. ANTES DE TODO, evita que la laptop se suspenda, se duerma o hiberne hasta que termines. Luego compruébalo:
   a) Bloquea la suspensión y la hibernación a nivel de sistema:
      sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target suspend-then-hibernate.target
   b) Lanza con setsid nohup un inhibidor que ignore también el cierre de la tapa y las teclas de suspender. Así no hace falta reiniciar logind, que cerraría la sesión:
      sudo systemd-inhibit --what=sleep:idle:handle-lid-switch:handle-suspend-key:handle-hibernate-key --who=recuperacion --why="Recuperando datos" --mode=block sleep infinity
   c) Desactiva el ahorro de energía de GNOME. Hazlo como mi usuario, sin sudo:
      gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-ac-type 'nothing'
      gsettings set org.gnome.settings-daemon.plugins.power sleep-inactive-battery-type 'nothing'
      gsettings set org.gnome.settings-daemon.plugins.power idle-dim false
      gsettings set org.gnome.settings-daemon.plugins.power power-button-action 'nothing'
      gsettings set org.gnome.desktop.session idle-delay 0
      gsettings set org.gnome.desktop.screensaver lock-enabled false
   d) Evita que el disco USB de 20 TB se apague solo:
      - Pon /sys/module/usbcore/parameters/autosuspend en -1.
      - Pon power/control en "on" para el dispositivo USB del disco.
      - Intenta hdparm -S 0 sobre ese disco; si no lo soporta, ignora el error.
   e) Comprueba con upower -i que la laptop está conectada a la corriente. Si está con batería, avísame y espera a que la conecte.
   f) Verifica con systemd-inhibit --list y systemctl status sleep.target que todo quedó aplicado, y anótalo en la bitácora.
   g) Al final de todo, después de apagar el disco de 20 TB, deshaz los cambios:
      - systemctl unmask de los mismos targets.
      - Termina el proceso del inhibidor.

1. Monta solo el disco de 20 TB (número de serie 1PG20RRX) en /mnt/destino. Conecta _NO_TOCAR/imagen/disco.img en solo lectura con losetup -r -P. Si falta untrunc en ~/tools, vuelve a compilarlo desde el mismo commit.

2. Documentos con nombre original:
   - Revisa primero trabajo/con_nombre y lo que recuperaron tsk_recover y ntfsundelete.
   - Si ahí solo se guardaron fotos y audio, vuelve a pasar tsk_recover y ntfsundelete por las mismas particiones, esta vez con documentos: doc, docx, xls, xlsx, ppt, pptx, pdf, odt, ods, rtf, txt, csv, pub, one, msg, pst, zip y rar.
   - Deja los resultados en trabajo/con_nombre_docs/, conservando las carpetas.

3. PhotoRec para documentos: lanza una segunda pasada no interactiva con /cmd sobre disco.img, hacia trabajo/photorec_docs/rec, en modo paranoid. Activa solo las familias de documentos: doc (Word, Excel y PowerPoint antiguos), zip (docx, xlsx, pptx, odt), pdf, rtf, pst y dbx.
   - No actives txt, porque genera cientos de miles de archivos basura.
   - Comprueba en la lista de fileopt de PhotoRec los nombres exactos de las familias antes de lanzarlo.
   - Hazlo con el mismo esquema de script reanudable, nohup y ESTADO.txt que antes.

4. Clasificación de documentos en REVISAR/12_DOCUMENTOS/:
   - Primero valida cada archivo:
     - PDF: pdfinfo.
     - Office moderno (docx, xlsx, pptx): unzip -t y que tenga [Content_Types].xml.
     - Office antiguo (doc, xls, ppt): que file lo reconozca como Composite Document.
     - Los que no pasen la validación van a 08_CORRUPTOS/documentos/.
   - Separa en subcarpetas: Word/, Excel/, PowerPoint/, PDF/, OpenOffice/ y Otros/.
   - Mueve a 07_PROBABLE_BASURA/documentos_de_sistema/ lo que sea de Windows o de programas: licencias, EULA, manuales de instalación, plantillas de Office o archivos de Program Files. Para decidirlo, usa el autor y la empresa de los metadatos (Microsoft Corporation, Adobe...), el título y el texto (pdftotext o docx2txt).
   - Renombra los documentos recuperados por carving con exiftool, usando fecha, título o autor: AAAA-MM-DD_Titulo.ext.
   - Guarda en 12_DOCUMENTOS/_texto/ un .txt con las primeras líneas de cada documento, y genera una miniatura de la primera página de cada PDF (pdftoppm) en _VISTA_PREVIA/12_DOCUMENTOS/. Así puedo buscar y revisar rápido.
   - Busca duplicados con jdupes. Si hay copias, conserva la que tiene nombre original.
   - Añade los documentos a INVENTARIO.csv (título, autor, fecha, páginas) y a PARA_COPIAR/DOCUMENTOS/ con enlaces duros.

5. Videos: repara con untrunc los 116 *_ftyp.mov de 08_CORRUPTOS/errores.
   - Usa como referencia un video sano del mismo modelo. Identifícalo por el códec y la resolución de su cabecera ftyp o stsd; si hace falta, prueba con las 4 referencias.
   - Los que ffprobe valide y duren más de 1 s van a 10_VIDEOS/reparados/, con miniatura y enlace en PARA_COPIAR/VIDEOS/.
   - Los que no se puedan reparar se quedan donde están.

6. Al terminar:
   - Actualiza el LEEME con cuántos documentos hay por tipo y cuántos videos se repararon.
   - Desmonta todo con losetup -d, sync y udisksctl power-off.
   - Deshaz los cambios de energía del paso 0g.
   - Dame el resumen.
```
