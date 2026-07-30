# 🐧 Gentoo Linux — Desde cero hasta KDE Plasma

<p align="center">
  <img src="https://www.gentoo.org/assets/img/logo/gentoo-signet.svg" alt="Gentoo Logo" width="150"/>
</p>

<p align="center">
  <b>La guía definitiva, técnica, paso a paso y realista</b><br>
  <i>ISO → Terminal → Escritorio KDE Plasma → fastfetch ✅</i>
</p>

---

> ⚠️ **Importante**: Esta guía asume que borrarás TODO el contenido del disco destino. Si haces dual boot o tienes datos importantes, **haz backup antes**. No me hago responsable si formateas el disco equivocado.

---

## 📋 Requisitos previos

Antes de empezar, necesitas:

- [ ] Una computadora con al menos **2 GB de RAM** (4 GB+ recomendado)
- [ ] **10 GB de espacio libre** en disco mínimo (30 GB+ para KDE Plasma)
- [ ] Conexión a internet (cable ethernet recomendado; Wi-Fi posible)
- [ ] Un pendrive **de 1 GB o más**
- [ ] **Paciencia** — Gentoo compila todo desde código fuente

### 📦 Qué vas a obtener al final

- ✅ Gentoo Linux funcional con **OpenRC** o **systemd**
- ✅ **KDE Plasma** como escritorio
- ✅ **SDDM** como gestor de sesión
- ✅ **PipeWire** para audio
- ✅ **Drivers gráficos** funcionales
- ✅ **Wi-Fi, Bluetooth y red** operativos
- ✅ **fastfetch** mostrando información del sistema

---

# 1️⃣ Descargar la ISO

## 1.1 Elegir la imagen correcta

Gentoo ofrece dos variantes principales:

| Variante | Cuándo usarla |
|----------|---------------|
| **Gentoo Linux** (OpenRC) | Recomendada para aprendizaje, estilo tradicional |
| **Gentoo Linux con systemd** | Si prefieres systemd o lo necesitas para algún software |

> 💡 **Consejo**: Si no estás seguro, elige **OpenRC**. Es la experiencia Gentoo clásica y más fácil de depurar.

## 1.2 Descargar

Ve al [sitio oficial de descargas](https://www.gentoo.org/downloads/) o usa directamente:

```bash
# OpenRC
wget https://distfiles.gentoo.org/releases/amd64/autobuilds/current-install-amd64-minimal/install-amd64-minimal-*.iso

# systemd
wget https://distfiles.gentoo.org/releases/amd64/autobuilds/current-install-amd64-minimal/install-amd64-minimal-*.iso
```

> 🔥 **Error común**: Usar un mirror lento. Si la descarga es muy lenta, usa un mirror cercano desde https://www.gentoo.org/downloads/mirrors/

## 1.3 Verificar la ISO (opcional pero recomendado)

```bash
# Descargar el archivo de firmas y checksums
wget https://distfiles.gentoo.org/releases/amd64/autobuilds/current-install-amd64-minimal/install-amd64-minimal-{firma}.DIGESTS.asc

# Verificar SHA512
sha512sum --check install-amd64-minimal-*.DIGESTS.asc 2>/dev/null | grep OK
```

Si ves `OK` en la salida, la ISO está íntegra.

> 💡 **Consejo**: En Linux ya tienes `sha512sum`. En Windows puedes usar `certutil -hashfile archivo.iso SHA512`.

---

# 2️⃣ Crear USB booteable

## 2.1 Identificar el dispositivo

```bash
lsblk
```

Busca tu pendrive. Normalmente es `sdb` o `sdc`. **Confirmalo dos veces**, porque el siguiente paso lo borra todo.

## 2.2 Escribir la ISO

```bash
# ⚠️ ¡CUIDADO! Reemplaza /dev/sdX con tu dispositivo real
sudo dd if=install-amd64-minimal-*.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

> ⚠️ **Importante**: Usa `of=/dev/sdX` (el dispositivo completo), **no** `of=/dev/sdX1` (la partición).

### 🧪 Verificación

```bash
# Después de escribir, verifica que se lee bien
sudo dd if=/dev/sdX bs=4M count=1 | sha256sum
# Debe coincidir con el hash esperado de la ISO
```

> 🔥 **Error común** — El USB no bootea:
> - Causa: No se usó `conv=fsync` o el USB está en formato de fábrica extraño
> - Solución: Vuelve a escribir la ISO con `conv=fsync` o prueba con `balenaEtcher`

### 🧭 Ruta alternativa: Ventoy

Si prefieres mantener varios ISOs en un solo USB:

```bash
# Descarga Ventoy desde https://github.com/ventoy/Ventoy/releases
# Instálalo en el USB y luego solo copia la ISO a la partición de datos

# O desde terminal
wget https://github.com/ventoy/Ventoy/releases/download/v1.0.99/ventoy-1.0.99-linux.tar.gz
tar -xzf ventoy-1.0.99-linux.tar.gz
cd ventoy-1.0.99
sudo sh Ventoy2Disk.sh -i /dev/sdX
# Luego copia la ISO a la partición que aparece
```

---

# 3️⃣ Arrancar la ISO

## 3.1 Bootear el USB

Reinicia la computadora y entra al menú de arranque:

| Fabricante | Tecla típica |
|------------|--------------|
| Dell | F12 |
| HP | F9 |
| Lenovo | F12 o Novo button |
| ASUS | F8 o ESC |
| Acer | F12 |
| MSI | F11 |
| Mac | Option (Alt) |

> 💡 **Consejo**: Si no sabes, prueba F12, ESC, F2, F10, Del. O busca en Google "[tu modelo] boot menu key".

## 3.2 En el menú de GRUB

Verás algo como:

```
Gentoo Linux [...]
Gentoo Linux [...nofb...]
Gentoo Linux [...memtest...]
Boot from HDD
```

Selecciona la primera opción y presiona Enter. Si no ves video, prueba con la opción `nonfb` o `nofb`.

> 💡 **Consejo**: Si estás en una laptop NVIDIA Optimus, puede que necesites añadir `nomodeset` al final de la línea del kernel. Presiona `e` en GRUB para editar y agrega `nomodeset` al final de la línea que empieza con `linux`.

## 3.3 Detectar firmware: UEFI vs BIOS

Cuando estés en el terminal de la ISO, verifica:

```bash
ls /sys/firmware/efi
```

- Si el directorio **existe** y tiene contenido → estás en **modo UEFI** 🟢
- Si el directorio **no existe** o está vacío → estás en **modo BIOS/legado** 🟡

> ⚠️ **Importante**: Esta detección determina cómo particionarás e instalarás el bootloader. **No la saltes**.

---

# 4️⃣ Conectarse a internet

## 4.1 Verificar conexión actual

```bash
ip a
ping -c 3 google.com
```

Si tienes internet, **sáltate este paso**. Si no, sigue leyendo.

## 4.2 Conexión por cable (DHCP)

```bash
# El servicio dhcpcd debería iniciar automáticamente
# Si no, inícialo manualmente:
dhcpcd
```

### 🧪 Verificación

```bash
ping -c 3 google.com
```

## 4.3 Conexión Wi-Fi

```bash
# Ver dispositivos inalámbricos
iw dev

# Si no ves nada, carga el módulo correcto (ejemplo para Intel):
modprobe iwlwifi
# Para otras marcas: iwlmvm, ath9k, ath10k, rtl8723be, etc.

# Escanea redes
iw dev wlan0 scan | grep SSID

# Conectar con wpa_supplicant
wpa_passphrase "MiRed" "MiClave" > /etc/wpa_supplicant.conf
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf

# Obtener IP
dhcpcd wlan0
```

> 🔥 **Error común** — No hay internet:
> - **Causa**: El módulo del Wi-Fi no se cargó
> - **Solución**: `lspci -k | grep -A3 Network` para ver el chipset y cargar el módulo correcto
> - **Causa**: wpa_supplicant no logra autenticar
> - **Solución**: Verifica SSID y contraseña. Para WPA2-Enterprise necesitas configuración extra en `wpa_supplicant.conf`
> - **Causa**: DHCP no responde
> - **Solución**: Prueba IP estática: `ip addr add 192.168.1.100/24 dev wlan0 && ip route add default via 192.168.1.1`

### 🧭 Ruta alternativa: nmtui (si está disponible)

```bash
nmtui
```

Interfaz textual simple que te guía.

---

# 5️⃣ Sincronización de hora

## 5.1 Ver hora actual

```bash
date
```

## 5.2 Sincronizar

```bash
# Con OpenRC
ntpd -q -g

# Si ntpd no funciona o no está disponible:
# Usa la herramienta de fecha de la ISO
```

> ⚠️ **Importante**: Una hora incorrecta puede causar errores al descargar archivos (certificados TLS caducados o no válidos).

### 🧪 Verificación

```bash
date
# Debe mostrar la hora local correcta
```

> 🔥 **Error común** — La hora está mal aunque sincronices:
> - **Causa**: La zona horaria del hardware (RTC) está en UTC/local y no coincide
> - **Solución**: `hwclock --systohc` para escribir la hora actual al hardware

---

# 6️⃣ Particionado del disco

## 6.1 Identificar el disco

```bash
lsblk
fdisk -l
```

Identifica tu disco:

| Tipo | Nombre típico |
|------|---------------|
| SATA/HDD | `/dev/sda` |
| NVMe | `/dev/nvme0n1` |
| SD/MMC | `/dev/mmcblk0` |

> ⚠️ **Importante**: Los discos NVMe tienen particiones nombradas como `nvme0n1p1`, `nvme0n1p2`. Los SATA como `sda1`, `sda2`.

## 6.2 Elegir esquema de particiones

### 🧭 Ruta recomendada: UEFI + GPT (moderno)

| Partición | Tamaño | Tipo | Sistema de archivos | Punto de montaje |
|-----------|--------|------|---------------------|-----------------|
| 1 | 512M - 1G | EFI System | FAT32 | `/boot/efi` (o `/boot`) |
| 2 | 4G - 16G | Linux swap | swap | — |
| 3 | Resto | Linux filesystem | ext4/btrfs | `/` |

### 🧭 Ruta alternativa: BIOS + MBR (legado)

| Partición | Tamaño | Tipo | Sistema de archivos | Punto de montaje |
|-----------|--------|------|---------------------|-----------------|
| 1 | 4G - 16G | Linux swap | swap | — |
| 2 | Resto | Linux filesystem | ext4/btrfs | `/` |

### 🧭 Ruta avanzada: Varias particiones

| Partición | Tamaño | Tipo | Sistema de archivos | Punto de montaje |
|-----------|--------|------|---------------------|-----------------|
| 1 | 512M - 1G | EFI System (UEFI) | FAT32 | `/boot/efi` |
| 2 | 1G | Linux filesystem | ext4 | `/boot` |
| 3 | 4G - 16G | Linux swap | swap | — |
| 4 | 50G - 100G | Linux filesystem | ext4/btrfs | `/` |
| 5 | Resto | Linux filesystem | ext4/btrfs | `/home` |

> 💡 **Consejo**: Separar `/home` en su propia partición facilita reinstalar Gentoo sin perder datos personales.

> 💡 **Consejo SSD vs HDD**: En SSD usa ext4 con `discard` o btrfs. En HDD añade `noatime` para reducir escrituras. Para NVMe, btrfs con opciones de compresión (`compress=zstd`) es excelente.

## 6.3 Crear particiones con `fdisk`

### Para UEFI + GPT:

```bash
fdisk /dev/sda  # Reemplaza con tu disco

# Dentro de fdisk:
g           # Crear tabla GPT
n           # Nueva partición (EFI)
[Enter]     # Número por defecto (1)
[Enter]     # Primer sector por defecto
+512M       # Tamaño
t           # Cambiar tipo
1           # Tipo EFI System

n           # Nueva partición (swap)
[Enter]
[Enter]
+8G         # Tamaño (ajusta según tu RAM)
t
2
19          # Tipo Linux swap

n           # Nueva partición (raíz)
[Enter]
[Enter]
[Enter]     # Usa el resto del disco

w           # Guardar y salir
```

### Para BIOS + MBR:

```bash
fdisk /dev/sda

o           # Crear tabla MBR (DOS)
n           # Partición primaria
p           # Primaria
1
[Enter]
+8G         # Swap
t
82          # Linux swap

n
p
2
[Enter]
[Enter]     # Resto del disco

w           # Guardar
```

> 🔥 **Error común** — No aparece el disco:
> - **Causa**: El disco está en RAID/ahci en modo incorrecto desde la BIOS
> - **Solución**: Entra a la BIOS/UEFI y cambia SATA Mode de RAID/Intel RST a **AHCI**
> - **Causa**: El módulo del controlador no está cargado
> - **Solución**: `modprobe ahci` o el módulo específico de tu chipset

---

# 7️⃣ Formateo

## 7.1 Formatear las particiones

```bash
# Partición EFI (solo en UEFI)
mkfs.fat -F 32 /dev/sda1

# Partición /boot separada (opcional)
mkfs.ext4 /dev/sda2   # O el número que corresponda

# Swap
mkswap /dev/sda3   # Ajusta el número

# Raíz (/) — varias opciones:

# Opción A: ext4 (recomendada para HDD, compatible universal)
mkfs.ext4 /dev/sda4

# Opción B: btrfs (recomendada para SSD/NVMe, con snapshots)
mkfs.btrfs /dev/sda4

# Opción C: XFS (buena para archivos grandes)
mkfs.xfs /dev/sda4

# /home separada (si la creaste)
mkfs.ext4 /dev/sda5
```

> 💡 **Consejo**: Si usas btrfs, crea subvolúmenes:
> ```bash
> mkfs.btrfs /dev/sda4
> mount /dev/sda4 /mnt/gentoo
> btrfs subvolume create /mnt/gentoo/@
> btrfs subvolume create /mnt/gentoo/@home
> btrfs subvolume create /mnt/gentoo/@var
> umount /mnt/gentoo
> ```

---

# 8️⃣ Montaje

## 8.1 Montar las particiones

### Esquema UEFI con `/boot` dedicado:

```bash
# Raíz
mount /dev/sda4 /mnt/gentoo

# Boot (opcional)
mkdir -p /mnt/gentoo/boot
mount /dev/sda2 /mnt/gentoo/boot

# EFI
mkdir -p /mnt/gentoo/boot/efi
mount /dev/sda1 /mnt/gentoo/boot/efi

# Home (opcional)
mkdir -p /mnt/gentoo/home
mount /dev/sda5 /mnt/gentoo/home

# Swap
swapon /dev/sda3
```

### Esquema BIOS sin `/boot` dedicado:

```bash
mount /dev/sda2 /mnt/gentoo
mkdir -p /mnt/gentoo/home
mount /dev/sda3 /mnt/gentoo/home  # si existe /home
swapon /dev/sda1
```

### Para btrfs con subvolúmenes:

```bash
mount -o compress=zstd /dev/sda4 /mnt/gentoo
mkdir -p /mnt/gentoo/{@,@home,@var}
# Desmonta y remonta cada subvolumen
umount /mnt/gentoo
mount -o subvol=@,compress=zstd /dev/sda4 /mnt/gentoo
mkdir -p /mnt/gentoo/{home,var,boot,efi}
mount -o subvol=@home,compress=zstd /dev/sda4 /mnt/gentoo/home
mount -o subvol=@var,compress=zstd /dev/sda4 /mnt/gentoo/var
mount /dev/sda1 /mnt/gentoo/boot/efi  # EFI
```

### 🧪 Verificación

```bash
lsblk
mount | grep /mnt/gentoo
```

Asegúrate de que todas las particiones están montadas donde corresponde.

> 🔥 **Error común** — No se monta `/boot/efi`:
> - **Causa**: No existe el directorio
> - **Solución**: `mkdir -p /mnt/gentoo/boot/efi`
> - **Causa**: La partición no está formateada como FAT32
> - **Solución**: `mkfs.fat -F 32 /dev/sda1`

---

# 9️⃣ Descargar y extraer Stage 3

## 9.1 Elegir el Stage 3

```bash
cd /mnt/gentoo
```

| Archivo | Para qué |
|---------|----------|
| `stage3-amd64-*.tar.xz` | OpenRC (recomendado) |
| `stage3-amd64-systemd-*.tar.xz` | Systemd |
| `stage3-amd64-desktop-*.tar.xz` | OpenRC + perfiles desktop (ahorra compilación) |
| `stage3-amd64-desktop-systemd-*.tar.xz` | Systemd + perfiles desktop |

> 💡 **Consejo**: Para una instalación con KDE Plasma, el stage `desktop` o incluso `plasma` te ahorrará mucho tiempo de compilación.

## 9.2 Descargar

```bash
# Opción A: Desde la ISO (puede estar cacheada)
links https://www.gentoo.org/downloads/
# O usa el mirror más cercano manualmente:

# Opción B: Con wget directo
wget https://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64-desktop-openrc/stage3-amd64-desktop-openrc-*.tar.xz

# Opción C: Con curl
curl -LO https://distfiles.gentoo.org/releases/amd64/autobuilds/current-stage3-amd64-desktop-openrc/stage3-amd64-desktop-openrc-*.tar.xz
```

> 🔥 **Error común** — `mirrorselect` falla:
> - **Causa**: La herramienta no está disponible o falla la resolución DNS
> - **Solución**: Usa `wget` directamente con una URL completa de un mirror conocido, e.g. `https://mirror.rackspace.com/gentoo/...`

## 9.3 Extraer

```bash
tar xpvf stage3-amd64-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

Explicación de las opciones:

| Opción | Qué hace |
|--------|----------|
| `x` | Extraer |
| `p` | Preservar permisos |
| `v` | Verboso (puedes omitirlo) |
| `f` | Archivo |
| `--xattrs-include='*.*'` | Preservar atributos extendidos |
| `--numeric-owner` | Preservar dueños numéricos |

### 🧪 Verificación

```bash
ls -la /mnt/gentoo/
# Deberías ver: bin, boot, dev, etc, home, lib, media, mnt, opt, proc, root, run, sbin, sys, tmp, usr, var
```

---

# 🔟 Entrar al chroot

## 10.1 Copiar información de DNS

```bash
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

> 💡 **Consejo**: Esto permite que dentro del chroot tengas acceso a internet.

## 10.2 Montar sistemas de archivos virtuales

```bash
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run
```

> Para systemd, necesitas un paso extra:
> ```bash
> test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
> mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
> chmod 1777 /dev/shm
> ```

## 10.3 Entrar al chroot

```bash
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

Si tu prompt cambia a `(chroot) #`, estás dentro.

> 🔥 **Error común** — `chroot` falla:
> - **Causa**: El archivo `/bin/bash` no existe en el stage (muy raro)
> - **Solución**: Verifica que extrajiste bien el stage. `file /mnt/gentoo/bin/bash`
> - **Causa**: Error de arquitectura (ej. stage i386 en sistema x86_64)
> - **Solución**: Asegúrate de usar stage3-amd64 para sistemas de 64 bits

---

# 1️⃣1️⃣ Configurar Portage

## 11.1 Sincronizar el repositorio

```bash
emerge --sync
```

> 💡 **Consejo**: Este paso puede tardar bastante en la primera ejecución. Si es muy lento, considera usar `emerge-webrsync` como alternativa:

```bash
emerge-webrsync
```

## 11.2 Elegir perfil

```bash
# Listar perfiles disponibles
eselect profile list

# Para KDE Plasma con OpenRC (elige el número correspondiente):
# Busca algo como: default/linux/amd64/23.0/desktop/plasma
eselect profile set 7  # Reemplaza 7 con el número correcto
```

> 💡 **Consejo**: Los perfiles `desktop/plasma` ya vienen optimizados con los USE flags necesarios para KDE. Esto te ahorrará mucha configuración manual.

## 11.3 Configurar `make.conf`

Este es **el archivo más importante** de tu instalación. Controla cómo se compila TODO.

```bash
nano /etc/portage/make.conf
```

### Ejemplo base para KDE Plasma:

```makefile
# /etc/portage/make.conf

COMMON_FLAGS="-march=native -O2 -pipe"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"

MAKEOPTS="-j$(nproc --all)"
EMERGE_DEFAULT_OPTS="--ask --verbose --tree"

# USE flags generales
USE="X dbus elogind kde plasma qt5 qt6 wayland glamor 
     pulseaudio bluetooth wifi crypt udev opengl vulkan 
     vaapi vdpau ffmpeg gstreamer networkmanager 
     jpeg png webp iconv icu bash-completion 
     cups colord policykit systemd elogind"

# CPU_FLAGS_X86: detecta con `cpuid2cpuflags`
# CPU_FLAGS_X86="mmx mmxext sse sse2 ssse3 sse4_1 sse4_2 avx avx2 aes"

VIDEO_CARDS="amdgpu radeonsi"   # Para AMD
# VIDEO_CARDS="intel iris"      # Para Intel
# VIDEO_CARDS="nvidia"          # Para NVIDIA

ACCEPT_LICENSE="* -@EULA"

GRUB_PLATFORMS="efi-64"
```

> ⚠️ **Importante**: `-march=native` optimiza para **tu CPU exacta**. No copies este make.conf entre máquinas diferentes.

### Diferencias clave según hardware:

| Situación | Ajuste en `make.conf` |
|-----------|----------------------|
| **Poca RAM** (< 4GB) | `MAKEOPTS="-j1"`, `PORTAGE_TMPDIR="/var/tmp/portage"` y considera añadir más swap |
| **SSD/NVMe** | Añade `PORTAGE_TMPDIR="/tmp"` (en RAM si tienes suficiente) |
| **HDD + poca RAM** | `MAKEOPTS="-j1"` — compilar en paralelo satura el HDD |
| **CPU vieja** | No uses `-march=native`, usa `-march=x86-64-v2` o `-march=core2` |

> 💡 **Consejo RAM**: Si tienes poca RAM, configura `PORTAGE_NICENESS=15` en `make.conf` para que Portage no acapare el sistema.

## 11.4 Configurar directorios de Portage

```bash
mkdir -p /etc/portage/package.use
mkdir -p /etc/portage/package.accept_keywords
mkdir -p /etc/portage/package.mask
mkdir -p /etc/portage/package.unmask
```

## 11.5 Configurar `package.use`

### Para KDE Plasma con PipeWire y Wayland:

```bash
nano /etc/portage/package.use/kde
```

```
# /etc/portage/package.use/kde

kde-plasma/plasma-meta -accessibility -grub
kde-frameworks/* -webkit
media-libs/mesa vulkan
media-video/pipewire sound-server
x11-wm/kwin wayland
```

## 11.6 Configurar `repos.conf`

```bash
mkdir -p /etc/portage/repos.conf
cp /usr/share/portage/config/repos.conf /etc/portage/repos.conf/gentoo.conf
```

### 🧪 Verificación

```bash
emerge --info | grep -E "^(CHOST|CFLAGS|MAKEOPTS|PORTAGE)"
```

Todo debe mostrar tus valores configurados.

> 🔥 **Error común** — `emerge` falla:
> - **Causa**: USE flags incompatibles o faltantes
> - **Solución**: `emerge --info | grep USE` para ver qué está activo. Revisa `package.use` para flags específicos
> - **Causa**: Dependencias circulares
> - **Solución**: `emerge --backtrack=30 nombre` para aumentar el backtracking
> - **Causa**: Paquete bloqueado
> - **Solución**: Revisa `emerge --pretend --verbose` para ver qué bloquea qué

---

# 1️⃣2️⃣ Configurar sistema base

## 12.1 Zona horaria

```bash
# Listar zonas
ls /usr/share/zoneinfo/

# Elegir (ejemplo para México CDMX)
ln -sf /usr/share/zoneinfo/America/Mexico_City /etc/localtime

# Para España
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime

# Para Argentina
ln -sf /usr/share/zoneinfo/America/Argentina/Buenos_Aires /etc/localtime

# Para Chile
ln -sf /usr/share/zoneinfo/America/Santiago /etc/localtime
```

## 12.2 Localización

```bash
nano /etc/locale.gen
```

Busca y descomenta (quita el `#`):

```
es_MX.UTF-8 UTF-8   # Para México
es_ES.UTF-8 UTF-8   # Para España
es_AR.UTF-8 UTF-8   # Para Argentina
es_CL.UTF-8 UTF-8   # Para Chile
en_US.UTF-8 UTF-8   # Siempre útil tener inglés de respaldo
```

Luego:

```bash
locale-gen
```

Verifica:

```bash
eselect locale list
eselect locale set N   # N = número con es_MX.UTF-8 o el que elegiste
```

## 12.3 Hostname

```bash
echo "gentoo-pc" > /etc/hostname

# También edita /etc/hosts
nano /etc/hosts
```

Agrega:

```
127.0.0.1    localhost
::1          localhost
127.0.1.1    gentoo-pc.localdomain gentoo-pc
```

## 12.4 Configurar `/etc/fstab`

Este archivo le dice al sistema qué montar al arrancar. **Si está mal, el sistema no bootea**.

```bash
nano /etc/fstab
```

### Para UEFI + GPT:

```
# /etc/fstab

# Partición EFI
/dev/sda1  /boot/efi  vfat  defaults  0  2

# Raíz (/)
/dev/sda4  /          ext4  noatime   0  1

# /home (si existe)
/dev/sda5  /home      ext4  noatime   0  2

# Swap
/dev/sda3  none       swap  sw        0  0
```

### Para BIOS + MBR:

```
/dev/sda2  /          ext4  noatime   0  1
/dev/sda1  none       swap  sw        0  0
```

### Para btrfs con subvolúmenes:

```
/dev/sda4  /          btrfs subvol=@,compress=zstd,noatime  0  1
/dev/sda4  /home      btrfs subvol=@home,compress=zstd,noatime  0  2
/dev/sda1  /boot/efi  vfat  defaults  0  2
/dev/sda3  none       swap  sw        0  0
```

> 💡 **Consejo**: En lugar de `/dev/sdX`, puedes usar UUIDs que son más robustos (no cambian si agregas discos):
> ```bash
> blkid /dev/sda4   # Copia el UUID
> ```
> ```
> UUID=tu-uuid-aquí  /  ext4  noatime  0  1
> ```

> 🔥 **Error común** — `fstab` está mal:
> - **Síntoma**: El sistema bootea y te deja en un "initramfs emergency shell"
> - **Solución**: Verifica que cada línea tenga **6 campos** separados por espacios. El orden es: dispositivo, punto_montaje, tipo_fs, opciones, dump, pass
> - **dump**: normalmente 0
> - **pass**: 1 para raíz, 2 para el resto, 0 para swap y EFI

### 🧪 Verificación

```bash
mount -a   # Monta todo lo del fstab sin reiniciar
# Si no hay errores, está bien
```

---

# 1️⃣3️⃣ Compilar el kernel

Este es **el paso más importante y donde más se rompen las cosas**.

## 13.1 Elegir método

| Método | Para quién |
|--------|------------|
| **`dist-kernel`** (recomendado) | Usuarios que quieren algo que funcione sin complicaciones |
| **`genkernel`** | Usuarios que quieren personalización pero sin hacerlo todo manual |
| **Manual** | Usuarios avanzados que saben exactamente qué necesitan |

> 💡 **Consejo**: Si es tu primera vez con Gentoo, usa **`dist-kernel`**. Es un kernel precompilado con soporte para la mayoría del hardware. Puedes cambiarte a manual después.

## 🧭 Ruta A (recomendada): `dist-kernel`

```bash
emerge sys-kernel/installkernel-gentoo
emerge sys-kernel/gentoo-kernel-bin
```

> ¿Por qué? `gentoo-kernel-bin` es un binario precompilado con configuraciones genéricas. Incluye módulos para casi todo el hardware moderno.

Después de instalar:

```bash
# Verifica que se instaló en /boot
ls /boot/vmlinuz-*
ls /boot/initrd-*
```

### Para crear initramfs automático (necesario para sistema de archivos encriptados o root en btrfs):

Los kernels `dist-kernel` ya generan initramfs automáticamente si usas `installkernel-gentoo`.

## 🧭 Ruta B: `genkernel`

```bash
emerge sys-kernel/gentoo-sources
eselect kernel list
eselect kernel set 1

emerge sys-kernel/genkernel
genkernel all
```

Esto compila el kernel con detección automática de hardware y genera initramfs.

> 💡 **Consejo**: `genkernel` es bueno cuando quieres un kernel compilado para tu CPU sin hacer la configuración manual, pero es más lento que dist-kernel.

## 🧭 Ruta C: Kernel manual (avanzado)

```bash
emerge sys-kernel/gentoo-sources
eselect kernel list
eselect kernel set 1

cd /usr/src/linux

# Opción: Configuración basada en el hardware actual
make localyesconfig

# O: Configuración desde cero (menú interactivo)
make menuconfig

# Compilar
make -j$(nproc)

# Instalar módulos
make modules_install

# Instalar kernel
make install
```

### Configuración mínima recomendada para KDE Plasma:

Si usas `menuconfig`, asegúrate de habilitar:

```
Device Drivers → Graphics support
  → <*> Direct Rendering Manager (DRM)
  → <*> DRM driver for Intel / AMD GPU / NVIDIA (nouveau o nvidia)

Device Drivers → Network device support
  → Ethernet driver support (tu chip)
  → Wireless LAN (tu chip)

File systems → <*> ext4 / btrfs / XFS
File systems → Native language support → UTF-8

Processor type and features → [*] Symmetric multi-processing support

Device Drivers → Sound card support → <*> ALSA
```

> ⚠️ **Importante**: Si omites tu controlador de red o de video, el sistema arrancará pero no tendrás internet ni imagen gráfica.

### Generar initramfs (si es necesario):

```bash
emerge sys-kernel/dracut
dracut --kver $(make kernelrelease)
```

## 13.2 Firmware y microcode

### Firmware general (imprescindible)

```bash
emerge sys-kernel/linux-firmware
```

> 💡 **Consejo**: `linux-firmware` incluye firmware para Wi-Fi, GPU, Bluetooth, etc. Es **obligatorio** para casi cualquier hardware moderno, especialmente laptops.

### Microcode Intel

```bash
# Para OpenRC
emerge sys-firmware/intel-microcode
# Y luego en el bootloader se agrega "ucode_initramfs"
```

### Microcode AMD

```bash
# Generalmente viene incluido en linux-firmware, pero verifica:
emerge sys-firmware/amd-microcode
```

---

# 1️⃣4️⃣ Instalar bootloader

## 14.1 Elegir bootloader

| Bootloader | Cuándo usarlo |
|------------|---------------|
| **GRUB** | La opción más compatible. Sirve para UEFI y BIOS |
| **efibootmgr** | Solo UEFI, más simple, menos configuración |
| **systemd-boot** | Solo UEFI + systemd |

## 🧭 Ruta recomendada: GRUB

### Instalar GRUB

```bash
emerge sys-boot/grub
```

### Configurar para UEFI

```bash
# Instalar GRUB en el ESP (EFI System Partition)
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Gentoo

# Generar configuración
grub-mkconfig -o /boot/grub/grub.cfg
```

### Configurar para BIOS

```bash
# Instalar GRUB en el MBR
grub-install /dev/sda

# Generar configuración
grub-mkconfig -o /boot/grub/grub.cfg
```

> 💡 **Consejo**: Si haces **dual boot**, `grub-mkconfig` a veces detecta Windows automáticamente. Si no lo hace, puedes agregarlo manualmente editando `/etc/grub.d/40_custom`.

### 🧪 Verificación

```bash
grub-install --version
ls /boot/grub/grub.cfg
```

> 🔥 **Error común** — GRUB no detecta Gentoo:
> - **Causa**: `grub-mkconfig` no encuentra kernels en `/boot`
> - **Solución**: Verifica `ls /boot/vmlinuz-*` y `ls /boot/initrd-*`. Si no hay, no se instaló el kernel correctamente
> - **Causa**: La variable `GRUB_CMDLINE_LINUX` en `/etc/default/grub` está mal
> - **Solución**: Revisa y regenera con `grub-mkconfig -o /boot/grub/grub.cfg`

> 🔥 **Error común** — GRUB no arranca:
> - **Síntoma**: Pantalla negra o mensaje "No bootable device"
> - **Causa UEFI**: GRUB no se instaló correctamente en la partición EFI
> - **Solución**: Vuelve a correr `grub-install` con el `--efi-directory` correcto
> - **Causa BIOS**: GRUB no se instaló en el MBR
> - **Solución**: `grub-install /dev/sda` (sin número)
> - **Causa firmware**: La BIOS/UEFI no está booteando desde el dispositivo correcto
> - **Solución**: Revisa el orden de arranque en la BIOS

## 🧭 Ruta alternativa: efibootmgr

```bash
emerge sys-boot/efibootmgr
efibootmgr --create --disk /dev/sda --part 1 --label "Gentoo" --loader /vmlinuz-6.x.x-gentoo -u "root=/dev/sda4 ro quiet"
```

---

# 1️⃣5️⃣ Configurar red y servicios

## 15.1 OpenRC — Configurar red

```bash
emerge --ask net-misc/dhcpcd
rc-update add dhcpcd default

# Para NetworkManager (recomendado para escritorio)
emerge --ask net-misc/networkmanager
rc-update add NetworkManager default
```

> 💡 **Consejo**: Si usas KDE Plasma, instala NetworkManager. KDE lo integra perfectamente desde las preferencias del sistema.

## 15.2 OpenRC — Configurar Wi-Fi (si no usas NetworkManager)

```bash
emerge --ask net-wireless/wpa_supplicant
rc-update add wpa_supplicant default
```

## 15.3 OpenRC — Servicios adicionales

```bash
# Agregar servicios esenciales
rc-update add elogind boot   # Para sesiones de usuario
rc-update add dbus default   # Para comunicación entre aplicaciones

# Sincronización de hora
emerge --ask net-misc/ntp
rc-update add ntpd default
```

## 15.4 systemd — Configurar red

```bash
# NetworkManager
systemctl enable NetworkManager

# systemd-networkd (alternativa ligera)
systemctl enable systemd-networkd
systemctl enable systemd-resolved
```

## 15.5 systemd — Servicios adicionales

```bash
systemctl enable elogind
systemctl enable systemd-timesyncd  # Sincronización de hora
```

---

# 1️⃣6️⃣ Crear usuario y permisos

## 16.1 Contraseña de root

```bash
passwd
```

Elige una contraseña **segura**.

> ⚠️ **Importante**: No pierdas esta contraseña. Sin root no puedes arreglar el sistema.

## 16.2 Crear usuario normal

```bash
useradd -m -G users,wheel,audio,video,usb,portage,plugdev -s /bin/bash cristopher
# Reemplaza "cristopher" por tu nombre de usuario
```

Explicación de grupos:

| Grupo | Para qué |
|-------|----------|
| `wheel` | Poder usar `sudo` |
| `audio` | Acceso a dispositivos de audio |
| `video` | Aceleración gráfica |
| `usb` | Acceso a dispositivos USB |
| `portage` | Poder compilar paquetes |
| `plugdev` | Montar dispositivos extraíbles |
| `cdrom` | Acceso a CD/DVD (si aplica) |

## 16.3 Establecer contraseña del usuario

```bash
passwd cristopher
```

## 16.4 Configurar sudo

```bash
emerge --ask app-admin/sudo
```

```bash
EDITOR=nano visudo
```

Descomenta la línea (quita el `#`):

```
%wheel ALL=(ALL:ALL) ALL
```

O agrega para que no pida contraseña (útil en escritorio personal):

```
%wheel ALL=(ALL:ALL) NOPASSWD: ALL
```

### 🧪 Verificación

```bash
su - cristopher
sudo whoami   # Debe decir "root"
```

> 🔥 **Error común** — El usuario no puede usar sudo:
> - **Causa**: El usuario no está en el grupo `wheel`
> - **Solución**: `usermod -aG wheel cristopher` y vuelve a iniciar sesión
> - **Causa**: `sudo` no está instalado
> - **Solución**: `emerge app-admin/sudo`

---

# 1️⃣7️⃣ Instalar entorno gráfico

## 17.1 Xorg vs Wayland

| Opción | Cuándo usarla |
|--------|---------------|
| **Wayland** | Predeterminado en KDE Plasma moderno. Más seguro, más moderno |
| **Xorg** | Si necesitas compatibilidad con aplicaciones viejas o compartir pantalla |

> 💡 **Consejo**: KDE Plasma 6 usa Wayland por defecto. Quédate con Wayland. Si algo falla, puedes cambiar a Xorg fácilmente.

## 17.2 Instalar Xorg (opcional, para compatibilidad)

```bash
emerge --ask x11-base/xorg-drivers x11-base/xorg-server
```

## 17.3 Instalar Mesa (gráficos 3D)

```bash
emerge --ask media-libs/mesa
```

---

# 1️⃣8️⃣ Instalar KDE Plasma

## 18.1 Instalar Plasma meta

Este comando instala **todo KDE Plasma**:

```bash
emerge --ask kde-plasma/plasma-meta
```

> ⚠️ **Importante**: Esto compila **muchas** cosas. En una máquina moderna toma 2-4 horas. En una vieja, puede tomar 8-12 horas o más. **Ten paciencia**.

> 💡 **Consejo**: Si quieres una instalación más ligera y agregar solo lo que necesitas:
> ```bash
> emerge --ask kde-plasma/plasma-desktop kde-apps/kate kde-apps/dolphin kde-apps/konsole
> ```
>
> `plasma-meta` instala todo el ecosistema KDE. `plasma-desktop` solo el escritorio base.

## 18.2 Instalar SDDM (gestor de sesión)

```bash
emerge --ask kde-misc/sddm-kcm
```

SDDM se instala automáticamente como dependencia de KDE Plasma, pero verifica:

```bash
emerge --ask x11-misc/sddm
```

### Habilitar SDDM

**OpenRC:**
```bash
rc-update add sddm default
```

**systemd:**
```bash
systemctl enable sddm
```

---

# 1️⃣9️⃣ Drivers de video, audio y red

## 19.1 Drivers de GPU

### 🟢 Intel (integradas modernas)

```bash
# En make.conf
VIDEO_CARDS="intel iris"
```

```bash
emerge --ask x11-drivers/xf86-video-intel
```

O para GPUs muy modernas (Alder Lake+):

```bash
VIDEO_CARDS="intel"
```

### 🔴 AMD (Radeon)

```bash
# En make.conf
VIDEO_CARDS="amdgpu radeonsi"
```

```bash
emerge --ask x11-drivers/xf86-video-amdgpu
```

### 🔵 NVIDIA

**Opción 1: Drivers privativos (nvidia-drivers)** — recomendado para gaming y CUDA

```bash
# En make.conf
VIDEO_CARDS="nvidia"
```

```bash
emerge --ask x11-drivers/nvidia-drivers
```

```bash
# Agregar nvidia al kernel cmdline
echo 'GRUB_CMDLINE_LINUX="nvidia-drm.modeset=1"' >> /etc/default/grub
grub-mkconfig -o /boot/grub/grub.cfg
```

**Opción 2: Nouveau (open source)** — para compatibilidad básica

```bash
# En make.conf
VIDEO_CARDS="nouveau"
```

```bash
emerge --ask x11-drivers/xf86-video-nouveau
```

### 🧪 Verificación de GPU

```bash
glxinfo | grep "OpenGL renderer"
# o
vulkaninfo | grep "deviceName"
```

> 🔥 **Error común** — El kernel arranca pero no muestra video:
> - **Causa**: Faltan los controladores DRM en el kernel
> - **Solución**: Revisa `make menuconfig` → Device Drivers → Graphics support → DRM → tu GPU
> - **Causa**: `nomodeset` en el kernel cmdline bloquea los modos nativos
> - **Solución**: Quita `nomodeset` de `GRUB_CMDLINE_LINUX`
> - **Causa**: NVIDIA sin `nvidia-drm.modeset=1`
> - **Solución**: Agrega el parámetro al kernel y regenera GRUB

> 🔥 **Error común** — KDE Plasma abre en negro:
> - **Causa**: Wayland no está funcionando con tu GPU
> - **Solución 1**: En la pantalla de SDDM, selecciona "Plasma (X11)" en lugar de "Plasma (Wayland)"
> - **Solución 2**: Cambia a Xorg: `echo "QT_QPA_PLATFORM=xcb" >> /etc/environment`
> - **Causa**: Faltan firmware para la GPU AMD
> - **Solución**: `emerge sys-kernel/linux-firmware` y reinicia
> - **Causa**: NVIDIA + Wayland — NVIDIA soporta Wayland desde la serie 500+, pero necesita configuración extra
> - **Solución**: Asegúrate de tener `nvidia-drm.modeset=1` en los parámetros del kernel

> 🔥 **Error común** — SDDM no inicia:
> - **Causa**: `/etc/conf.d/sddm` no está configurado correctamente
> - **Solución**: Revisa los logs `journalctl -u sddm` o `/var/log/sddm.log`
> - **Causa**: dbus o elogind no están corriendo
> - **Solución OpenRC**: `rc-update add elogind boot && rc-update add dbus default`
> - **Solución systemd**: `systemctl enable elogind && systemctl enable dbus`

## 19.2 Audio con PipeWire

```bash
# Emerge PipeWire con soporte para PulseAudio y ALSA
emerge --ask media-video/pipewire media-libs/libpulse
```

```bash
# Para OpenRC
rc-update add pipewire default
rc-update add wireplumber default
```

```bash
# Para systemd
systemctl --user enable --now pipewire
systemctl --user enable --now pipewire-pulse
systemctl --user enable --now wireplumber
```

### 🧪 Verificación

```bash
pactl info
# Debe mostrar "Server Name: PulseAudio (on PipeWire ...)"
```

> 🔥 **Error común** — El audio no sale:
> - **Causa**: PipeWire no está activo o ALSA usa el dispositivo equivocado
> - **Solución 1**: `pactl list sinks` para ver las salidas disponibles
> - **Solución 2**: Instala `media-sound/pavucontrol` y selecciona la salida correcta
> - **Causa**: El módulo de sonido del kernel no se cargó
> - **Solución**: `lspci -k | grep -A3 Audio`, luego `modprobe snd_hda_intel` (o snd_hda_intel/snd_soc_skl según corresponda)
> - **Causa**: Volumen al mínimo o mute
> - **Solución**: `amixer sset Master unmute && amixer sset Master 80%`
> - **Causa**: PipeWire no reemplazó a PulseAudio
> - **Solución**: Asegúrate de tener `media-libs/libpulse` instalado y `media-video/pipewire` con USE flag `sound-server`

## 19.3 Wi-Fi y Bluetooth

### NetworkManager (Wi-Fi)

```bash
# Ya debería estar instalado con KDE Plasma
# Verifica:
emerge --ask net-misc/networkmanager
```

Si usas OpenRC:
```bash
rc-update add NetworkManager default
```

### Bluetooth

```bash
emerge --ask net-wireless/bluez

# OpenRC
rc-update add bluetooth default

# systemd
systemctl enable bluetooth
```

Integración con KDE:

```bash
emerge --ask kde-plasma/bluedevil
```

---

# 2️⃣0️⃣ Últimos pasos antes del reinicio

## 20.1 Limpiar sistema

```bash
emerge --depclean
```

Esto elimina dependencias huérfanas.

## 20.2 Revisar configuraciones importantes

```bash
# Verificar que el kernel está instalado
ls /boot/vmlinuz-*
ls /boot/initrd-*   # Si usas initramfs

# Verificar fstab
cat /etc/fstab

# Verificar red
cat /etc/hostname
cat /etc/hosts

# Verificar servicios
rc-update show   # OpenRC
# O
systemctl list-unit-files | grep enabled   # systemd
```

## 20.3 Revisar lista de configuraciones pendientes

```bash
emerge --info
etc-update   # O dispatch-conf
```

Si hay archivos de configuración que actualizar, hazlo ahora.

## 20.4 Salir del chroot y reiniciar

```bash
exit   # Salir del chroot

# Desmontar todo
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -R /mnt/gentoo

# Reiniciar
reboot
```

> ⚠️ **Importante**: Retira el USB booteable cuando la máquina se apague, o entra a la BIOS para arrancar desde el disco.

---

# 2️⃣1️⃣ Primer arranque

Cuando la máquina reinicie, deberías ver:

1. GRUB con la entrada de Gentoo
2. El kernel cargándose
3. Los servicios iniciándose
4. SDDM pidiendo usuario y contraseña
5. Inicio de sesión → KDE Plasma

## 21.1 Si ves KDE Plasma

🎉 ¡Felicidades! Ya tienes Gentoo con KDE Plasma funcionando.

## 21.2 Si algo falla

Si te quedas en una terminal (modo rescate, o sin SDDM), revisa la sección de **Problemas comunes**.

## 21.3 Configuración post-instalación

Una vez en KDE Plasma:

1. Conéctate a Wi-Fi desde el panel de NetworkManager
2. Abre Discover (tienda de software) o usa konsola para instalar más paquetes
3. Personaliza el escritorio

---

# 2️⃣2️⃣ Ejecutar fastfetch

## 22.1 Instalar fastfetch

```bash
sudo emerge --ask app-misc/fastfetch
```

## 22.2 Ejecutarlo

```bash
fastfetch
```

Deberías ver algo como:

```
                    ▪▄▄▄▄▄▄▄▄▄▪   cristopher@gentoo-pc
                  ▄▀▀           ▀▀  OS: Gentoo Linux 2.15
              ▄▀                   ▀▄  Host: ThinkPad X1 Carbon Gen 10
            ▄▀        ▄▄▄▄▄▄        ▀▄  Kernel: 6.6.x-gentoo
           ▀       ▄▀▀   ▀▀▀▀▀▀▀▀▀▄    Uptime: 15 mins
          ▀       ▀                   ▀  Packages: 1340 (emerge)
          ▀                           ▀  Shell: bash 5.2
           ▄       ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄   Resolution: 1920x1080
            ▀▄   ▄▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀   DE: Plasma 6.1.x
              ▀▀▀▀                    WM: KWin (Wayland)
                                      Terminal: konsole
                                      CPU: Intel i7-1365U (12) @ 4.8 GHz
                                      GPU: Intel Iris Xe Graphics
                                      Memory: 15648MiB / 32178MiB
```

> 💡 **Consejo**: Puedes personalizar la salida de fastfetch editando `~/.config/fastfetch/config.jsonc`.

---

# 🛠️ Problemas comunes y soluciones

## 📌 El USB no bootea

**Causa probable**: La ISO no se escribió correctamente, o el firmware no arranca desde USB.

**Cómo detectarlo**: La computadora ignora el USB y arranca desde el disco duro.

**Cómo arreglarlo**:

```bash
# Vuelve a escribir la ISO con dd
sudo dd if=install-amd64-minimal-*.iso of=/dev/sdX bs=4M status=progress conv=fsync
```

- Desactiva **Secure Boot** en la BIOS/UEFI
- Cambia el modo SATA de RAID a AHCI
- Prueba otro puerto USB (preferiblemente USB 2.0)

**Verificar**: El menú de GRUB aparece al bootear el USB.

---

## 📌 No aparece el disco en la instalación

**Causa probable**: El controlador SATA/NVMe no está en modo AHCI, o falta el módulo del kernel.

**Cómo detectarlo**: `lsblk` muestra solo el USB pero no el disco interno.

**Cómo arreglarlo**:

```bash
# Ver controladores disponibles
lspci -k | grep -i "SATA\|NVMe\|Storage"

# Cargar módulos comunes
modprobe ahci
modprobe nvme
modprobe i2c_piix4
```

Si el disco aparece después de cargar módulos, asegúrate de configurarlos en el kernel.

Si sigue sin aparecer, revisa la BIOS:

- SATA Mode → AHCI (no RAID, no Intel RST)
- Secure Boot → Disabled

**Verificar**: `lsblk` muestra el disco interno.

---

## 📌 No hay internet

**Causa**: dhcpcd no se inició, Wi-Fi sin driver, o DNS no resuelve.

**Cómo detectarlo**: `ping -c 3 google.com` falla.

**Cómo arreglarlo**:

```bash
# Ver interfaces
ip a

# Si no hay interfaces:
lspci -k | grep -i network
# Carga el módulo correcto

# Verificar DHCP
dhcpcd

# Si Wi-Fi, escanea
iw dev wlan0 scan | grep SSID

# Probar DNS
ping -c 3 8.8.8.8   # Si esto funciona pero google.com no → problema de DNS
```

El problema de DNS se arregla configurando `/etc/resolv.conf`:

```
nameserver 8.8.8.8
nameserver 1.1.1.1
```

**Verificar**: `ping -c 3 google.com` responde.

---

## 📌 La hora está mal

**Causa**: El RTC del hardware está en hora local y el sistema espera UTC, o viceversa.

**Cómo detectarlo**: `date` muestra una hora incorrecta.

**Cómo arreglarlo**:

```bash
# Ajustar hora
ntpd -q -g

# Sincronizar hardware clock
hwclock --systohc
```

Si el problema persiste, verifica la zona horaria:

```bash
ls -la /etc/localtime
```

**Verificar**: `date` muestra la hora correcta.

---

## 📌 emerge --sync falla

**Causa**: Mirror lento, DNS no resuelve, o repos.conf mal configurado.

**Cómo detectarlo**: El comando regresa con error de conexión.

**Cómo arreglarlo**:

```bash
# Probar con webrsync
emerge-webrsync

# O cambiar de mirror
mirrorselect -i -o >> /etc/portage/make.conf
```

**Verificar**: `emerge --sync` se completa sin errores.

---

## 📌 emerge falla

**Causa**: Dependencias circulares, USE flags en conflicto, o paquetes bloqueados.

**Cómo detectarlo**: `emerge` muestra mensajes de error con bloques o dependencias.

**Cómo arreglarlo**:

```bash
# Ver el error completo
emerge --pretend --verbose nombre 2>&1 | less

# Aumentar backtracking
emerge --backtrack=30 nombre

# Si hay paquetes bloqueados:
# Revisar package.mask
nano /etc/portage/package.mask/bloqueados

# Ver USE flags activos
emerge --info | grep USE
```

**Verificar**: `emerge nombre` se completa.

---

## 📌 make.conf quedó mal

**Causa**: `-march=native` no es compatible con la CPU, o USE flags están mal escritos.

**Cómo detectarlo**: `emerge --info` muestra valores que no esperabas, o la compilación falla con errores de instrucción ilegal.

**Cómo arreglarlo**:

```bash
# Probar con -march=x86-64-v2
# Verificar CPU flags
cat /proc/cpuinfo | grep flags

# Si emerge --info no tiene lo que esperas
nano /etc/portage/make.conf
```

**Verificar**: `emerge --info | grep CFLAGS` muestra lo que configuraste.

---

## 📌 fstab está mal

**Causa**: Errores de sintaxis, particiones incorrectas, o UUIDs cambiados.

**Cómo detectarlo**: El arranque se detiene en un shell de emergencia con "Failed to mount ...".

**Cómo arreglarlo**:

En el shell de emergencia:

```bash
# Montar el sistema en modo lectura-escritura
mount -o remount,rw /

# Verificar fstab
cat /etc/fstab

# Ver UUIDs reales
blkid

# Editar fstab
nano /etc/fstab

# Probar el montaje
mount -a
```

**Errores comunes en fstab**:

- ❌ Faltan campos (deben ser 6 por línea)
- ❌ El punto de montaje está mal escrito
- ❌ La partición no existe
- ❌ Typo en el tipo de sistema de archivos

**Verificar**: `mount -a` no produce errores.

---

## 📌 El kernel no compila

**Causa**: Faltan dependencias, configuración incorrecta, o espacio en disco insuficiente.

**Cómo detectarlo**: `make` falla en el proceso de compilación.

**Cómo arreglarlo**:

```bash
# Verificar espacio en disco
df -h

# Espacio en /tmp
df -h /tmp

# Dependencias mínimas
emerge sys-devel/gcc sys-devel/binutils sys-libs/glibc

# Limpiar caché de compilación
rm -rf /var/tmp/portage/*
```

**Verificar**: `make -j$(nproc)` se completa.

---

## 📌 El kernel arranca pero no muestra video

**Causa**: DRM no configurado, firmware faltante, o parámetros incorrectos del kernel.

**Cómo detectarlo**: La computadora arranca (se ve actividad en el teclado/leds) pero la pantalla está negra.

**Cómo arreglarlo**:

```bash
# En GRUB, presiona 'e' y agrega:
nomodeset   # Desactiva KMS, usa framebuffer genérico
# Si ves video con nomodeset, el problema es DRM

# Para Intel:
i915.modeset=1

# Para AMD:
amdgpu.modeset=1

# Para NVIDIA:
nvidia-drm.modeset=1
```

**Verificar**: La pantalla muestra el arranque correctamente.

---

## 📌 No arranca después de instalar GRUB

**Causa**: GRUB no se instaló en el dispositivo correcto, o la entrada de boot en UEFI no se creó.

**Cómo detectarlo**: Pantalla "No bootable device" o "Reboot and Select proper Boot device".

**Cómo arreglarlo**:

Bootea desde la ISO, monta todo, entra al chroot:

```bash
mount /dev/sda4 /mnt/gentoo
mount /dev/sda1 /mnt/gentoo/boot/efi
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
chroot /mnt/gentoo /bin/bash

# Reinstalar GRUB
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Gentoo
grub-mkconfig -o /boot/grub/grub.cfg

# Para BIOS
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

**Verificar**: El sistema arranca desde el disco.

---

## 📌 No carga el entorno gráfico

**Causa**: SDDM no está habilitado, servidor X/Wayland no instalado, o controladores incorrectos.

**Cómo detectarlo**: Arranca en terminal (TTY) en lugar de SDDM.

**Cómo arreglarlo**:

```bash
# Verificar que SDDM está habilitado
rc-update show | grep sddm   # OpenRC
systemctl status sddm         # systemd

# Verificar que los servicios necesarios están activos
rc-update add dbus default
rc-update add elogind boot

# Instalar si falta
emerge --ask x11-misc/sddm
```

Para iniciar SDDM manualmente desde terminal:

```bash
sudo /etc/init.d/sddm start   # OpenRC
sudo systemctl start sddm     # systemd
```

**Verificar**: SDDM aparece pidiendo usuario y contraseña.

---

## 📌 Wayland no funciona

**Causa**: Driver de GPU sin soporte Wayland, o KWin no puede iniciar en Wayland.

**Cómo detectarlo**: La sesión de Wayland no aparece en SDDM, o al seleccionarla se cierra y vuelve a SDDM.

**Cómo arreglarlo**:

```bash
# Probar con X11
# En SDDM, selecciona "Plasma (X11)" en lugar de "Plasma (Wayland)"

# Para AMD/Intel, asegúrate de tener Mesa con soporte Wayland
emerge --ask media-libs/mesa

# Para NVIDIA, necesitas nvidia-drivers >= 470 y:
# En /etc/environment:
echo "QT_QPA_PLATFORM=wayland" >> /etc/environment
echo "GDK_BACKEND=wayland" >> /etc/environment
echo "CLUTTER_BACKEND=wayland" >> /etc/environment
```

**Verificar**: `echo $XDG_SESSION_TYPE` muestra `wayland`.

---

## 📌 X11 falla

**Causa**: `/etc/X11/xorg.conf` mal configurado, o `/dev/dri` no tiene permisos.

**Cómo detectarlo**: Al iniciar X11, pantalla negra o mensaje de error.

**Cómo arreglarlo**:

```bash
# Verificar permisos
ls -la /dev/dri/
# Asegúrate de que tu usuario está en el grupo video

# Forzar detección automática
rm /etc/X11/xorg.conf
startx

# Ver logs
cat /var/log/Xorg.0.log | grep EE
```

**Verificar**: `startx` inicia un entorno gráfico básico.

---

## 📌 El audio no sale

**Causa**: PipeWire no está activo, o la salida de audio incorrecta.

**Cómo detectarlo**: Los parlantes no emiten sonido, o el icono de audio muestra "Dummy Output".

**Cómo arreglarlo**:

```bash
# Ver sinks disponibles
pactl list sinks short

# Verificar que PipeWire está corriendo
ps aux | grep pipewire

# Cargar módulos ALSA
modprobe snd_hda_intel
modprobe snd_hda_codec_hdmi

# Guardar módulos para que se carguen al inicio
echo "snd_hda_intel" >> /etc/modules-load.d/sound.conf
echo "snd_hda_codec_hdmi" >> /etc/modules-load.d/sound.conf
```

**Para OpenRC**: `rc-update add pipewire default && rc-update add wireplumber default`
**Para systemd**: `systemctl --user enable --now pipewire pipewire-pulse wireplumber`

**Verificar**:
```bash
speaker-test -t sine -f 440 -l 1
# Deberías escuchar un tono
```

---

## 📌 Wi-Fi no conecta

**Causa**: NetworkManager no está activo, firmware faltante, o wpa_supplicant mal configurado.

**Cómo detectarlo**: El applet de Wi-Fi de KDE no muestra redes.

**Cómo arreglarlo**:

```bash
# Habilitar NetworkManager
rc-update add NetworkManager default   # OpenRC
systemctl enable NetworkManager        # systemd

# Verificar firmware
dmesg | grep firmware
# Si ves "firmware: failed to load ...", instala linux-firmware
emerge sys-kernel/linux-firmware

# Verificar que wpa_supplicant está presente
which wpa_supplicant
```

**Verificar**: `nmcli dev wifi list` muestra redes disponibles.

---

## 📌 El touchpad no responde

**Causa**: Falta el driver synaptics/libinput, o el módulo del kernel.

**Cómo detectarlo**: El touchpad no mueve el cursor en el escritorio.

**Cómo arreglarlo**:

```bash
# Instalar libinput (recomendado para Plasma Wayland)
emerge --ask x11-drivers/xf86-input-libinput

# Alternativa: synaptics (legado, X11)
emerge --ask x11-drivers/xf86-input-synaptics

# En KDE Plasma → Configuración del sistema → Mouse y touchpad
# Asegúrate de que no está desactivado
```

**Para laptops con tecla Fn para desactivar touchpad**, asegúrate de que no está apagado por hardware.

**Verificar**: El cursor se mueve con el touchpad.

---

## 📌 fastfetch no aparece o da error

**Causa**: No está instalado o falla al detectar cierta información.

**Cómo detectarlo**: `fastfetch` no se encuentra o muestra errores de permisos.

**Cómo arreglarlo**:

```bash
# Instalar
sudo emerge --ask app-misc/fastfetch

# Si falla al leer información del sistema:
fastfetch --help   # Ver opciones
fastfetch -c none  # Sin configuración personalizada

# Si es problema de permisos para leer /sys:
sudo fastfetch     # Para probar
```

**Verificar**: `fastfetch` muestra la información del sistema.

---

## 📌 Permisos mal configurados

**Causa**: El usuario no está en los grupos correctos.

**Cómo detectarlo**: No puedes montar dispositivos, usar audio, o acceder a ciertos periféricos.

**Cómo arreglarlo**:

```bash
# Ver grupos del usuario
groups cristopher

# Agregar a grupos faltantes
usermod -aG audio,video,usb,plugdev,wheel,portage cristopher
```

> ⚠️ **Importante**: Después de cambiar grupos, debes cerrar sesión y volver a entrar para que los cambios surtan efecto.

**Verificar**: `groups` muestra todos los grupos deseados.

---

## 📌 Dual boot con Windows no aparece en GRUB

**Causa**: `os-prober` no está habilitado o no detecta Windows.

**Cómo detectarlo**: `grub-mkconfig` no muestra "Found Windows..." en su salida.

**Cómo arreglarlo**:

```bash
# Instalar os-prober
emerge --ask sys-boot/os-prober

# En /etc/default/grub, agrega:
GRUB_DISABLE_OS_PROBER=false

# Regenerar GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

**Si Windows sigue sin aparecer**, arranca Windows y desde una terminal administrativa:

```cmd
bcdedit /set {bootmgr} path \EFI\Microsoft\Boot\bootmgfw.efi
```

**Verificar**: GRUB muestra Windows al arrancar.

---

## 📌 Se olvidó la contraseña de root

**Causa**: 🙃

**Cómo arreglarlo**:

Bootea desde la ISO de Gentoo, monta el sistema:

```bash
mount /dev/sda4 /mnt/gentoo
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
chroot /mnt/gentoo /bin/bash

# Cambiar contraseña
passwd
```

**Verificar**: Puedes iniciar sesión como root con la nueva contraseña.

---

## 📌 HDMI externo no funciona

**Causa**: El módulo de audio HDMI no se cargó, o el kernel no tiene soporte.

**Cómo detectarlo**: Al conectar un monitor HDMI, no se detecta.

**Cómo arreglarlo**:

```bash
# Verificar si el kernel cargó el módulo
lsmod | grep snd_hda_codec_hdmi

# Cargar manualmente
modprobe snd_hda_codec_hdmi

# Agregar a módulos persistentes
echo "snd_hda_codec_hdmi" >> /etc/modules-load.d/hdmi.conf
```

**Para NVIDIA**:
```bash
# Asegúrate de que nvidia-drm.modeset=1 está en los parámetros del kernel
```

**Verificar**: El monitor externo aparece en `xrandr` o en Preferencias del Sistema → Pantallas.

---

## 📌 Compilación muy lenta

**Causa**: `MAKEOPTS` muy alto para la RAM disponible, disco lento, o CPU muy vieja.

**Cómo detectarlo**: `emerge` tarda horas enteras en paquetes pequeños.

**Cómo arreglarlo**:

```bash
# Para poca RAM (<4GB): reduce paralelismo
MAKEOPTS="-j1"

# Para HDD: reduce paralelismo a -j2 máximo
MAKEOPTS="-j2"

# Para SSD/NVMe: sube el paralelismo
MAKEOPTS="-j$(nproc)"

# Usa compilación en tmpfs si tienes suficiente RAM
PORTAGE_TMPDIR="/tmp"   # Y asegúrate de que tmpfs está montado en /tmp
PORTAGE_NICENESS=15     # Baja la prioridad de compilación
```

**Opciones de ahorro**:

```bash
# Usa distcc para distribuir compilación en red
emerge --ask sys-devel/distcc

# Usa ccache para acelerar recompilaciones
emerge --ask dev-util/ccache
# En make.conf: FEATURES="ccache"
```

**Verificar**: La compilación de paquetes grandes toma menos tiempo que antes.

---

## 📌 Portage usa demasiado espacio en disco

**Causa**: Caché de compilación binarios, fuentes descargadas, o paquetes compilados.

**Cómo arreglarlo**:

```bash
# Limpiar paquetes fuente descargados
eclean-dist

# Limpiar binarios compilados
eclean-pkg

# Limpiar caché de compilación
rm -rf /var/tmp/portage/*

# Ver espacio usado
du -sh /var/cache/distfiles/
du -sh /var/tmp/portage/
```

**Verificar**: `df -h` muestra más espacio libre.

---

## 📌 No se puede instalar steam/paquetes con licencia restringida

**Causa**: `ACCEPT_LICENSE` no permite licencias como `steam` o `@EULA`.

**Cómo arreglarlo**:

```bash
# En make.conf:
ACCEPT_LICENSE="* -@EULA"

# O específicamente para Steam:
nano /etc/portage/package.license
```

Agrega:

```
games-util/steam-launcher steam
games-util/game-device-udev-rules game-device-udev-rules
```

**Verificar**: `emerge games-util/steam-launcher` se completa.

---

## 📌 La laptop no hiberna/suspende

**Causa**: Falta `pm-utils` o `elogind` no está funcionando correctamente.

**Cómo arreglarlo**:

```bash
# Para OpenRC
emerge --ask sys-power/pm-utils
rc-update add elogind boot

# Probar suspensión
sudo pm-suspend
```

**Para systemd**:
```bash
systemctl suspend
```

**Verificar**: `sudo pm-suspend` suspende y reactiva la computadora correctamente.

---

## 📌 Actualización del kernel (después de instalación inicial)

**Causa**: Necesitas actualizar el kernel a una versión más nueva.

**Cómo hacerlo**:

```bash
# Si usas dist-kernel
sudo emerge --update sys-kernel/gentoo-kernel-bin
sudo emerge --update sys-kernel/linux-firmware
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Si usas fuentes manuales
sudo emerge --update sys-kernel/gentoo-sources
sudo eselect kernel set N  # N = nueva versión
cd /usr/src/linux
sudo make oldconfig
sudo make -j$(nproc)
sudo make modules_install
sudo make install
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

**Verificar**: `uname -r` muestra la nueva versión del kernel después de reiniciar.

---

# 📖 Glosario

| Término | Significado |
|---------|-------------|
| **Portage** | Sistema de gestión de paquetes de Gentoo |
| **emerge** | Comando para instalar/actualizar paquetes |
| **ebuild** | Script que define cómo compilar e instalar un paquete |
| **USE flags** | Opciones que activan/desactivan funcionalidades en los paquetes |
| **Stage 3** | Archivo comprimido con un sistema base precompilado |
| **chroot** | Cambiar la raíz del sistema a otro directorio |
| **make.conf** | Archivo principal de configuración de Portage |
| **fstab** | Tabla de sistemas de archivos, define qué montar al arrancar |
| **GRUB** | Grand Unified Bootloader — gestor de arranque |
| **genkernel** | Herramienta que compila un kernel genérico automáticamente |
| **initramfs** | Sistema de archivos temporal usado durante el arranque |
| **EAPI** | Versión de la especificación de ebuilds |
| **OpenRC** | Sistema de inicio de servicios basado en scripts |
| **systemd** | Sistema de inicio moderno con parallelización y gestión de servicios |
| **SDDM** | Simple Desktop Display Manager — gestor de sesión |
| **PipeWire** | Servidor multimedia para audio y video |
| **WirePlumber** | Gestor de sesión para PipeWire |
| **DRM/KMS** | Direct Rendering Manager / Kernel Mode Setting — controladores de video |
| **ESP** | EFI System Partition — partición necesaria para UEFI |
| **GPT** | GUID Partition Table — estándar moderno de particionado |
| **MBR** | Master Boot Record — estándar legacy de particionado |
| **NVMe** | Non-Volatile Memory Express — protocolo para SSDs ultrarrápidos |
| **distcc** | Herramienta para distribuir compilaciones entre múltiples computadoras |
| **ccache** | Caché de compilación para acelerar recompilaciones |

---

# 🧰 Apéndice: Comandos útiles

## Información del sistema

```bash
# Información de hardware
lspci -k          # Dispositivos PCI y módulos del kernel
lsusb             # Dispositivos USB
lscpu             # Información de la CPU
free -h           # Memoria RAM
df -h             # Espacio en disco
uname -a          # Información del kernel

# Información de video
glxinfo | grep -E "OpenGL|renderer"
vulkaninfo | grep deviceName

# Módulos cargados
lsmod

# Logs del sistema
dmesg | less
journalctl -xe    # systemd
```

## Gestión de Portage

```bash
# Buscar paquetes
emerge --search nombre
eix nombre

# Ver dependencias
emerge --pretend --verbose nombre

# Actualizar sistema
emerge --update --deep --newuse @world

# Limpiar
emerge --depclean
eclean-dist
eclean-pkg

# Ver qué hay instalado
equery list "*"
```

## Gestión de servicios

**OpenRC:**
```bash
rc-update show                # Servicios en cada nivel
rc-service nombre start/stop  # Gestionar un servicio
rc-status                     # Estado de servicios
```

**systemd:**
```bash
systemctl status nombre       # Estado de un servicio
systemctl start/stop nombre
systemctl enable/disable nombre
journalctl -u nombre          # Logs del servicio
```

## Kernel

```bash
# Saber qué kernel está activo
uname -r
eselect kernel list

# Reconstruir módulos externos (como NVIDIA)
emerge --ask @module-rebuild
```

## Red

```bash
# Ver interfaces
ip a

# Escanear Wi-Fi
nmcli dev wifi list

# Probar conectividad
ping -c 3 google.com

# Ver tabla de rutas
ip route

# Ver resolución DNS
cat /etc/resolv.conf
```

## Mantenimiento

```bash
# Actualizar todo el sistema
sudo emerge --update --deep --newuse @world

# Limpiar dependencias huérfanas
sudo emerge --depclean

# Limpiar caché de paquetes fuente
sudo eclean-dist

# Ver qué hay que recompilar (por cambio de USE flags)
sudo emerge --update --deep --newuse @world --pretend

# Reconstruir módulos del kernel
sudo emerge @module-rebuild

# Configuración pendiente
etc-update

# Ver paquetes con configuraciones nuevas
dispatch-conf
```

---

# ✅ Qué revisar si algo falla

## El sistema no arranca

```
1. ¿EL USB bootea?       → Revisa BIOS, Secure Boot, escribe ISO de nuevo
2. ¿GRUB aparece?        → Reinstala GRUB desde chroot
3. ¿El kernel carga?     → Revisa que vmlinuz esté en /boot
4. ¿initramfs falla?     → Reconstruye initramfs con dracut o genkernel
5. ¿FSTAB está mal?      → Revisa /etc/fstab, verifica UUIDs
6. ¿SDDM aparece?        → Habilita sddm, dbus, elogind
7. ¿KDE Plasma sale?     → Revisa drivers de GPU, Wayland vs X11
```

## No hay internet

```
1. Cable conectado       → dhcpcd
2. Wi-Fi disponible      → NetworkManager, firmware, wpa_supplicant
3. DNS resuelve           → /etc/resolv.conf (8.8.8.8, 1.1.1.1)
4. Módulo del kernel     → lspci -k, modprobe iwlwifi/ath9k/etc
```

## Audio no funciona

```
1. PipeWire activo       → rc-update/systemctl --user enable
2. Módulo ALSA cargado   → modprobe snd_hda_intel
3. Volumen no mute       → amixer, pavucontrol
4. Salida correcta       → pactl list sinks
```

## Video no funciona

```
1. Mesa instalado         → emerge media-libs/mesa
2. DRM en kernel          → make menuconfig → Graphics support
3. Firmware GPU           → emerge linux-firmware
4. Grupo video            → usermod -aG video usuario
5. Parámetros kernel      → nomodeset, i915.modeset=1, nvidia-drm.modeset=1
```

---

# 🏁 Cierre final

Has llegado al final de la guía. Si todo salió bien, en este momento tienes:

- 🐧 **Gentoo Linux** funcionando — compilado específicamente para tu CPU
- 🖥️ **KDE Plasma** como escritorio — moderno, bonito y personalizable
- 🔊 **PipeWire** — audio que funciona en todas partes
- 🌐 **NetworkManager** — Wi-Fi, ethernet y Bluetooth desde el panel
- 🎮 **Drivers gráficos** — aceleración 3D, video y juegos
- 🚀 **fastfetch** — mostrando con orgullo tu sistema

## 📝 Notas finales

1. **Gentoo es un sistema vivo**. Los paquetes se actualizan constantemente. Corre `emerge --sync && emerge --update --deep --newuse @world` de vez en cuando para mantenerlo actualizado.

2. **La documentación oficial de Gentoo** (https://wiki.gentoo.org) es tu mejor amiga. Si algo no funciona, busca allí primero.

3. **No tengas miedo de romper cosas**. Así se aprende. Y si rompes algo, ya sabes cómo entrar al chroot desde la ISO para arreglarlo.

4. **Únete a la comunidad**: Los canales de Gentoo en Reddit, los foros oficiales y el canal de IRC (#gentoo en Libera.Chat) son excelentes lugares para aprender y ayudar.

5. **Personaliza tu sistema**. Ahora que tienes la base funcionando, explora. Cambia de WM, prueba otros kernels, juega con los USE flags. Gentoo es tuyo para moldearlo como quieras.

---

<p align="center">
  <b>Hecho con 💪 y mucha paciencia</b><br>
  <i>Una instalación de Gentoo no es solo un sistema operativo. Es una declaración de principios.</i>
</p>

---

## 🧪 Créditos y recursos

- [Wiki oficial de Gentoo](https://wiki.gentoo.org)
- [Foros de Gentoo](https://forums.gentoo.org)
- [Handbook de Gentoo (amd64)](https://wiki.gentoo.org/wiki/Handbook:AMD64)
- [Gentoo en Reddit](https://reddit.com/r/Gentoo)
- [#gentoo en Libera.Chat](irc://irc.libera.chat/#gentoo)

---

<p align="center">
  <img src="https://www.gentoo.org/assets/img/logo/gentoo-signet.svg" alt="Gentoo" width="64"/>
  <br>
  <i>"If it compiles, it ships."</i>
  <br>
  <b>¡Bienvenido a Gentoo! 🐧✨</b>
</p>
