# Recuperar fotos y música de un disco con Windows reinstalado (Ubuntu Live)

Guía para usar desde Ubuntu Desktop Live. Las fotos y la música se recuperan con
**PhotoRec** (paquete `testdisk`) y se guardan en el disco de 20 TB.

> ⚠️ Regla de oro: **nunca escribas nada en el disco que quieres recuperar.**
> No lo montes, no instales nada ahí y no guardes lo recuperado en él.

## 1. Usar Claude Code en esta máquina (opcional)

```bash
curl -fsSL https://claude.ai/install.sh | bash
exec bash        # recarga el PATH
claude           # inicia sesión y trabaja desde la terminal
```

En un Live todo vive en RAM: al reiniciar hay que reinstalarlo.

## 2. Instalar las herramientas

```bash
sudo add-apt-repository -y universe
sudo apt update
sudo apt install -y testdisk gddrescue
```

## 3. Identificar los discos

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MODEL,MOUNTPOINT
```

- **Disco origen** (el de las fotos borradas): p. ej. `/dev/sdb`
- **Disco destino** de 20 TB: p. ej. `/dev/sdc1` (≈18,2 TiB en `lsblk`)

Comprueba dos veces cuál es cuál antes de seguir.

## 4. Montar el disco de 20 TB y crear la carpeta de destino

Si Ubuntu ya lo montó solo (aparece en `/media/ubuntu/...`), usa esa ruta. Si no:

```bash
sudo mkdir -p /mnt/destino
sudo mount /dev/sdc1 /mnt/destino
sudo mkdir -p /mnt/destino/recuperado
df -h /mnt/destino
```

## 5. (Recomendado) Clonar primero el disco origen

Si el disco es viejo o hace ruidos, haz una imagen y trabaja sobre ella:

```bash
sudo ddrescue -n /dev/sdb /mnt/destino/disco.img /mnt/destino/disco.map
sudo ddrescue -r3 /dev/sdb /mnt/destino/disco.img /mnt/destino/disco.map
```

Luego, en el paso 6, usa `/mnt/destino/disco.img` en lugar de `/dev/sdb`.

## 6. Recuperar con PhotoRec

```bash
sudo photorec /log /d /mnt/destino/recuperado/rec /dev/sdb
```

En el menú:

1. Elige el disco → **Proceed**.
2. Elige **No partition [Whole disk]** (al reinstalar Windows la tabla de
   particiones cambió, así se escanea todo).
3. **File Opt** → pulsa `s` para desmarcar todo y marca solo lo que quieres:
   `jpg`, `png`, `gif`, `tif`, `bmp`, `heic`, `raw`/`cr2`/`nef` (cámaras),
   `mp3`, `wma`, `m4a`/`mov` (AAC), `flac`, `ogg`, `wav`. Pulsa `b` para guardar.
4. Tipo de sistema de archivos: **Other** (NTFS/FAT).
5. Elige **Whole** (todo el disco, no solo el espacio libre).
6. Confirma el destino con `C`.

Puede tardar muchas horas. Se crean carpetas `rec.1`, `rec.2`, …

## 7. Ordenar lo recuperado

PhotoRec pierde los nombres originales (`f123456.jpg`). Para agrupar por tipo
y quitar miniaturas pequeñas:

```bash
cd /mnt/destino/recuperado
sudo mkdir -p fotos musica
sudo find . -path ./fotos -prune -o -path ./musica -prune -o -type f \
  \( -iname '*.jpg' -o -iname '*.png' -o -iname '*.heic' -o -iname '*.cr2' -o -iname '*.nef' \) \
  -size +100k -exec mv -n -t fotos {} +
sudo find . -path ./fotos -prune -o -path ./musica -prune -o -type f \
  \( -iname '*.mp3' -o -iname '*.flac' -o -iname '*.m4a' -o -iname '*.wma' -o -iname '*.ogg' -o -iname '*.wav' \) \
  -exec mv -n -t musica {} +
```

Opcional: renombrar fotos por fecha EXIF y música por etiquetas:

```bash
sudo apt install -y libimage-exiftool-perl
sudo exiftool -r '-FileName<DateTimeOriginal' -d '%Y/%m/%Y%m%d_%H%M%S%%-c.%%e' fotos
```
