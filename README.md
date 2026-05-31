<div align="center">

# MyLinux

### *Arch Linux + Hyprland — Mi entorno personal desde cero*

[![Arch Linux](https://img.shields.io/badge/Arch_Linux-1793D1?style=for-the-badge&logo=arch-linux&logoColor=white)](https://archlinux.org)
[![Hyprland](https://img.shields.io/badge/Hyprland-58E1FF?style=for-the-badge&logo=wayland&logoColor=black)](https://hyprland.org)
[![Wayland](https://img.shields.io/badge/Wayland-FFBC00?style=for-the-badge&logo=wayland&logoColor=black)](https://wayland.freedesktop.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

</div>

---

## Introducción

**MyLinux** es un repositorio personal dedicado a documentar mi proceso de aprendizaje, exploración y personalización de **Arch Linux** con **Hyprland** como gestor de ventanas principal.

El objetivo es centralizar configuraciones, notas, procedimientos y herramientas utilizadas durante la construcción de un entorno Linux moderno, ligero y altamente personalizable. Además de servir como referencia personal, este repositorio también puede ayudar a otros usuarios interesados en construir su propio entorno basado en Arch Linux.

### Áreas principales

| Área | Descripción |
|------|-------------|
| Instalación | Instalación y mantenimiento de Arch Linux |
| Rendimiento | Optimización del sistema |
| Dotfiles | Gestión y organización de configuraciones |
| Scripts | Automatización de tareas |
| Personalización | Visual y funcional con Hyprland |
| Documentación | Problemas, soluciones y buenas prácticas |

---

## Índice

- [ Instalación de Arch Linux ](#-instalación-de-arch-linux)
 - [Sistema único (disco vacío)](#-sistema-único-disco-vacío)
 - [Dual Boot con Windows](#-dual-boot-con-windows)
- [Configuración inicial](#-configuración-inicial)
- [Gestor de sesión — SDDM](#-gestor-de-sesión--sddm)
- [Hyprland](#-hyprland)
- [Herramientas esenciales](#-herramientas-esenciales)
- [Instalación de paquetes](#-instalación-de-paquetes)
- [Gestión de dotfiles](#-gestión-de-dotfiles)
- [Scripts personalizados](#-scripts-personalizados)
- [Documentación oficial](#-documentación-oficial)

---

## Instalación de Arch Linux

La filosofía de este repositorio parte de una **instalación mínima**, construyendo el sistema paso a paso para comprender cada componente.

### Antes de comenzar

- Descargar la imagen ISO oficial: https://archlinux.org/download/
- Crear un USB booteable con herramientas como `dd`, Rufus o Ventoy
- Verificar si el sistema usa **BIOS Legacy** o **UEFI** (recomendado)
- Confirmar conexión a Internet

---

### Sistema único (disco vacío)

> Ideal cuando instalas Arch en un disco limpio, sin otro sistema operativo.

#### Particionado sugerido para UEFI

| Partición | Tamaño sugerido | Sistema de archivos | Punto de montaje |
|-----------|----------------|---------------------|-----------------|
| EFI | 512 MB | FAT32 | `/boot` |
| Root | 40 GB o más | ext4 / btrfs | `/` |
| Home | Resto del disco | ext4 / btrfs | `/home` |
| Swap | Opcional (≥ RAM) | swap | — |

```bash
# Herramientas de particionado disponibles:
fdisk /dev/sdX
cfdisk /dev/sdX
gdisk /dev/sdX # Recomendado para GPT/UEFI
```

#### Instalación del sistema base

```bash
# Montar las particiones e instalar el sistema base
pacstrap -K /mnt base linux linux-firmware

# Generar fstab
genfstab -U /mnt >> /mnt/etc/fstab

# Entrar al nuevo sistema
arch-chroot /mnt
```

---

### Dual Boot con Windows

> Si ya tienes Windows instalado y quieres agregar Arch Linux como segundo sistema operativo.

#### Consideraciones importantes

- **Realiza una copia de seguridad de tus datos** antes de modificar particiones.
- Desactiva **Bitlocker** en Windows si está habilitado (puede impedir el arranque).
- Desactiva el **Inicio Rápido** de Windows:
 `Panel de control → Opciones de energía → Elegir el comportamiento de los botones → Desactivar inicio rápido`
- Desactiva **Secure Boot** en la BIOS/UEFI si hay conflictos con GRUB.
- Windows ya habrá creado una partición EFI — **no la formatees**, la compartirás.

#### Paso 1 — Liberar espacio desde Windows

Desde Windows, reduce el volumen de tu partición principal para crear espacio para Arch:

```
Administración de discos → Clic derecho en C: → Reducir volumen
```

Deja al menos **40 GB** sin asignar para Arch Linux.

#### Paso 2 — Arrancar desde el USB de Arch

Reinicia con el USB de instalación de Arch Linux.

#### Paso 3 — Identificar particiones existentes

```bash
lsblk -f
fdisk -l
```

Identifica:
- La partición EFI de Windows (FAT32, ~100–512 MB) — **no la toques**
- El espacio sin asignar donde irá Arch

#### Paso 4 — Crear particiones para Arch

Usa el espacio libre para crear las particiones de Arch. **NO crees una nueva EFI** si ya existe una de Windows, solo crea root y home:

```bash
cfdisk /dev/sdX # Selecciona el espacio libre y crea particiones
```

Esquema resultante aproximado:

| Partición | Sistema | Descripción |
|-----------|---------|-------------|
| `/dev/sdX1` | Windows | EFI compartida (FAT32) |
| `/dev/sdX2` | Windows | Partición del sistema (NTFS) |
| `/dev/sdX3` | Arch | Root `/` (ext4/btrfs) |
| `/dev/sdX4` | Arch | Home `/home` (ext4/btrfs) |

#### Paso 5 — Formatear y montar particiones de Arch

```bash
# Solo formatea las particiones de Arch, nunca la EFI de Windows
mkfs.ext4 /dev/sdX3 # Root
mkfs.ext4 /dev/sdX4 # Home

# Montar
mount /dev/sdX3 /mnt
mkdir -p /mnt/home /mnt/boot
mount /dev/sdX4 /mnt/home
mount /dev/sdX1 /mnt/boot # La EFI de Windows (sin formatear)
```

#### Paso 6 — Instalar el sistema base

```bash
pacstrap -K /mnt base linux linux-firmware
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```

#### Paso 7 — Instalar GRUB con soporte dual boot

```bash
pacman -S grub efibootmgr os-prober

# Instalar GRUB
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

# Habilitar detección de otros sistemas (Windows)
echo "GRUB_DISABLE_OS_PROBER=false" >> /etc/default/grub

# Generar configuración (detectará Windows automáticamente)
grub-mkconfig -o /boot/grub/grub.cfg
```

> Si Windows no aparece en el menú de GRUB al primer arranque, reinicia una vez desde Arch y ejecuta `sudo grub-mkconfig -o /boot/grub/grub.cfg` nuevamente.

---

## Configuración inicial

Dentro de `arch-chroot /mnt` o ya en el sistema instalado:

### Zona horaria

```bash
ln -sf /usr/share/zoneinfo/America/Bogota /etc/localtime
hwclock --systohc
```

### Idioma

```bash
# Editar /etc/locale.gen y descomentar la línea deseada:
es_ES.UTF-8 UTF-8

# Generar locales
locale-gen

# Crear /etc/locale.conf
echo "LANG=es_ES.UTF-8" > /etc/locale.conf
```

### Nombre del equipo

```bash
echo "mi-linux" > /etc/hostname
```

### Usuario principal

```bash
useradd -m -G wheel -s /bin/bash usuario
passwd usuario

pacman -S sudo
visudo # Descomentar: %wheel ALL=(ALL:ALL) ALL
```

### Gestor de arranque (sistema único)

```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Gestor de sesión — SDDM

**SDDM** (Simple Desktop Display Manager) es el gestor de inicio de sesión recomendado para Hyprland. Permite seleccionar la sesión (Hyprland u otras) y gestionar el login gráfico.

### Instalación

```bash
sudo pacman -S sddm
```

### Activar al inicio

```bash
sudo systemctl enable sddm
sudo systemctl start sddm
```

### Tema recomendado (opcional)

SDDM soporta temas. Uno popular compatible con Wayland es `sddm-theme-sugar-candy` disponible en el AUR:

```bash
yay -S sddm-theme-sugar-candy
```

Configurar el tema en `/etc/sddm.conf`:

```ini
[Theme]
Current=sugar-candy
```

### Configuración básica de SDDM

```bash
sudo nano /etc/sddm.conf
```

Ejemplo de configuración:

```ini
[Autologin]
# Opcional: inicio de sesión automático
User=tu_usuario
Session=hyprland

[General]
HaltCommand=/usr/bin/systemctl poweroff
RebootCommand=/usr/bin/systemctl reboot
```

> SDDM es quien lanzará Hyprland. Asegúrate de que el archivo de sesión `/usr/share/wayland-sessions/hyprland.desktop` exista después de instalar Hyprland.

---

## Hyprland

### ¿Por qué Hyprland?

Hyprland es un gestor de ventanas en mosaico (*tiling WM*) basado en **Wayland**, elegido por:

- Excelente rendimiento
- Compatibilidad nativa con Wayland
- Animaciones modernas y suaves
- Configuración flexible y expresiva
- Comunidad activa y en crecimiento

### Instalación

```bash
sudo pacman -S hyprland
```

El objetivo es construir un entorno productivo, ligero y visualmente agradable alrededor de Hyprland.

---

## Herramientas esenciales

### Barra de estado — Waybar

Muestra información del sistema en tiempo real: espacios de trabajo, CPU, memoria, red, audio, batería, fecha y hora.

```bash
sudo pacman -S waybar
```

### Lanzadores de aplicaciones

```bash
# Wofi (nativo Wayland)
sudo pacman -S wofi

# Rofi (puede requerir versión AUR para Wayland)
yay -S rofi-wayland
```

### Notificaciones — Mako

```bash
sudo pacman -S mako
```

### Terminales

```bash
sudo pacman -S kitty # Recomendada — GPU accelerated
sudo pacman -S alacritty # Alternativa rápida
```

### Gestores de archivos

```bash
sudo pacman -S thunar # Ligero (GTK)
sudo pacman -S dolphin # Completo (KDE)
```

### Fondo de pantalla — Hyprpaper

```bash
sudo pacman -S hyprpaper
```

### Bloqueo de pantalla — Hyprlock

```bash
sudo pacman -S hyprlock
```

### Capturas de pantalla

```bash
sudo pacman -S grim slurp
```

Uso básico:

```bash
grim ~/screenshot.png # Pantalla completa
grim -g "$(slurp)" ~/screenshot.png # Selección de área
```

### Portales Wayland

Necesarios para compartir pantalla, portapapeles y diálogos de archivos:

```bash
sudo pacman -S xdg-desktop-portal-hyprland
```

---

## Instalación de paquetes

### Repositorios oficiales (pacman)

```bash
sudo pacman -S nombre-paquete # Instalar
sudo pacman -Syu # Actualizar todo el sistema
sudo pacman -Rns nombre-paquete # Desinstalar (con dependencias huérfanas)
sudo pacman -Ss término # Buscar paquetes
```

### AUR — Arch User Repository

Para paquetes mantenidos por la comunidad, instala un ayudante como **yay**:

```bash
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
```

Uso de yay:

```bash
yay -S nombre-paquete # Instalar desde AUR
yay -Syu # Actualizar AUR + sistema
```

### Compilación manual

```bash
git clone <repositorio>
cd proyecto
make
sudo make install
```

> Consulta siempre la documentación oficial del proyecto antes de compilar.

---

## Gestión de dotfiles

Este repositorio almacena configuraciones personales versionadas. Ubicaciones típicas:

```
~/.config/hypr/ → Configuración de Hyprland
~/.config/waybar/ → Barra de estado
~/.config/kitty/ → Terminal
~/.config/mako/ → Notificaciones
~/.config/wofi/ → Lanzador
~/.config/sddm/ → Gestor de sesión
```

**Objetivos:**
- Mantener configuraciones versionadas en Git
- Facilitar migraciones entre equipos
- Documentar cambios importantes
- Recuperar el entorno rápidamente en una instalación nueva

---

## Scripts personalizados

Con el tiempo se añadirán scripts para:

- Automatización de tareas repetitivas
- Configuración inicial de sistemas nuevos
- Gestión de copias de seguridad
- Personalización del entorno
- Mantenimiento y limpieza del sistema

---

## Documentación oficial

| Recurso | Enlace |
|---------|--------|
| Arch Wiki | https://wiki.archlinux.org |
| Guía de instalación | https://wiki.archlinux.org/title/Installation_guide |
| Recomendaciones generales | https://wiki.archlinux.org/title/General_recommendations |
| Pacman | https://wiki.archlinux.org/title/Pacman |
| Mantenimiento del sistema | https://wiki.archlinux.org/title/System_maintenance |
| Hyprland Wiki | https://wiki.hyprland.org |
| SDDM | https://wiki.archlinux.org/title/SDDM |

---

## Tareas post-instalación

```bash
# Actualización completa del sistema
sudo pacman -Syu

# Herramientas básicas esenciales
sudo pacman -S git base-devel curl wget unzip

# Instalar controladores gráficos (AMD/Intel/NVIDIA según tu hardware)
# AMD:
sudo pacman -S mesa vulkan-radeon
# Intel:
sudo pacman -S mesa vulkan-intel
# NVIDIA:
sudo pacman -S nvidia nvidia-utils
```

Otras tareas recomendadas:
- [ ] Configurar audio (PipeWire)
- [ ] Configurar Bluetooth
- [ ] Instalar fuentes (nerd fonts)
- [ ] Configurar impresión (CUPS)
- [ ] Crear copias de seguridad (Timeshift / rsync)
- [ ] Instalar herramientas de desarrollo

---

<div align="center">

## Aquí comienza mi viaje personal

*Este proyecto está en constante evolución.*

Cada nueva configuración, ajuste, script o descubrimiento se documentará aquí como parte de un proceso continuo de aprendizaje. Algunas soluciones serán experimentales, otras se convertirán en parte permanente del sistema — pero todas contribuirán a entender mejor cómo funciona Arch Linux.

Si encuentras alguna mejora, detectas errores o quieres compartir ideas, cualquier contribución es bienvenida.

**Bienvenido a MyLinux.**

</div>
