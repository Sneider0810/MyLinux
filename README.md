# Mi Linux

## Introducción

**Mi Linux** es un repositorio personal dedicado a documentar mi proceso de aprendizaje, exploración y personalización de **Arch Linux** con **Hyprland** como gestor de ventanas principal.

El objetivo de este proyecto es centralizar configuraciones, notas, procedimientos y herramientas utilizadas durante la construcción de un entorno Linux moderno, ligero y altamente personalizable. Además de servir como referencia personal, este repositorio también puede ayudar a otros usuarios interesados en comprender cómo construir su propio entorno basado en Arch Linux.

### Áreas principales de interés

- Instalación y mantenimiento de Arch Linux.
- Optimización del rendimiento del sistema.
- Gestión y organización de dotfiles.
- Automatización mediante scripts personalizados.
- Personalización visual y funcional de Hyprland.
- Documentación de problemas, soluciones y mejores prácticas.
- Aprendizaje continuo del ecosistema Linux.

---

# Instalación de Arch

La filosofía de este repositorio parte de una instalación mínima de Arch Linux, construyendo el sistema paso a paso para comprender cada componente instalado.

## Consideraciones previas

Antes de comenzar:

- Descargar la imagen oficial desde: https://archlinux.org/download/
- Crear un medio de instalación USB.
- Verificar si el sistema utilizará:
  - BIOS Legacy
  - UEFI (recomendado)
- Confirmar la conexión a Internet.
- Revisar la documentación oficial de Arch Linux.

## Particionado básico

La estructura puede variar según las necesidades de cada usuario.

Ejemplo para sistemas UEFI:

| Partición | Tamaño sugerido | Sistema de archivos |
|-----------|----------------|--------------------|
| EFI | 512 MB | FAT32 |
| Root (`/`) | 40 GB o más | ext4 o btrfs |
| Home (`/home`) | Resto del disco | ext4 o btrfs |
| Swap | Opcional | swap |

Herramientas comunes:

```bash
fdisk
cfdisk
parted
gdisk
```

## Instalación del sistema base

Montar las particiones correspondientes y ejecutar:

```bash
pacstrap -K /mnt base linux linux-firmware
```

Generar el archivo fstab:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Entrar al nuevo sistema:

```bash
arch-chroot /mnt
```

## Configuración inicial

Configurar:

### Zona horaria

```bash
ln -sf /usr/share/zoneinfo/Region/Ciudad /etc/localtime
hwclock --systohc
```

### Idioma

Editar:

```bash
/etc/locale.gen
```

Generar locales:

```bash
locale-gen
```

Crear:

```bash
/ etc/locale.conf
```

Ejemplo:

```text
LANG=es_ES.UTF-8
```

### Nombre del equipo

```bash
echo "mi-linux" > /etc/hostname
```

### Usuario principal

Crear usuario:

```bash
useradd -m -G wheel -s /bin/bash usuario
passwd usuario
```

Instalar sudo:

```bash
pacman -S sudo
```

Editar:

```bash
visudo
```

Descomentar:

```text
%wheel ALL=(ALL:ALL) ALL
```

### Gestor de arranque

Para sistemas UEFI:

```bash
pacman -S grub efibootmgr
```

Instalar y generar configuración:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
```

---

# Documentación de Arch

La documentación oficial es una parte fundamental de cualquier instalación de Arch Linux.

## Recursos recomendados

### Arch Wiki

https://wiki.archlinux.org

Considerada una de las mejores fuentes de documentación del ecosistema Linux.

### Guía de instalación

https://wiki.archlinux.org/title/Installation_guide

### General Recommendations

https://wiki.archlinux.org/title/General_recommendations

### Pacman

https://wiki.archlinux.org/title/Pacman

### System Maintenance

https://wiki.archlinux.org/title/System_maintenance

## Tareas comunes después de instalar Arch

- Actualizar completamente el sistema.
- Configurar red y conectividad.
- Instalar controladores gráficos.
- Configurar audio.
- Configurar Bluetooth.
- Instalar fuentes.
- Configurar impresión.
- Crear copias de seguridad.
- Instalar herramientas de desarrollo.

Actualización completa:

```bash
sudo pacman -Syu
```

Instalación de herramientas básicas:

```bash
sudo pacman -S git base-devel curl wget unzip
```

---

# Hyprland

## ¿Por qué Hyprland?

Hyprland es el gestor de ventanas en mosaico elegido para este proyecto debido a:

- Excelente rendimiento.
- Compatibilidad con Wayland.
- Configuración flexible.
- Animaciones modernas.
- Amplias posibilidades de personalización.
- Comunidad activa.

El objetivo es construir un entorno productivo, ligero y visualmente agradable.

---

# Herramientas cruciales para Hyprland

## Barra de estado

### Waybar

Muestra:

- Espacios de trabajo
- Uso de CPU
- Memoria
- Red
- Audio
- Batería
- Fecha y hora

Instalación:

```bash
sudo pacman -S waybar
```

---

## Lanzador de aplicaciones

### Wofi

```bash
sudo pacman -S wofi
```

### Rofi (Wayland)

Puede requerir versiones específicas o paquetes del AUR.

---

## Notificaciones

### Mako

```bash
sudo pacman -S mako
```

---

## Portales Wayland

```bash
sudo pacman -S xdg-desktop-portal-hyprland
```

---

## Terminales recomendadas

### Kitty

```bash
sudo pacman -S kitty
```

### Alacritty

```bash
sudo pacman -S alacritty
```

---

## Gestor de archivos

### Thunar

```bash
sudo pacman -S thunar
```

### Dolphin

```bash
sudo pacman -S dolphin
```

---

## Fondo de pantalla

### Hyprpaper

```bash
sudo pacman -S hyprpaper
```

---

## Bloqueo de pantalla

### Hyprlock

```bash
sudo pacman -S hyprlock
```

---

## Capturas de pantalla

```bash
sudo pacman -S grim slurp
```

---

# Descargar herramientas

Dependiendo del origen del paquete, existen varias opciones.

## Repositorios oficiales

La mayoría de los paquetes pueden instalarse mediante:

```bash
sudo pacman -S nombre-paquete
```

Actualizar índices y sistema:

```bash
sudo pacman -Syu
```

## AUR

Para paquetes mantenidos por la comunidad puede utilizarse un ayudante como **yay**.

Instalación típica:

```bash
yay -S nombre-paquete
```

Instalar yay:

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

## Compilación manual

Algunos proyectos requieren compilación desde código fuente.

Procedimiento general:

```bash
git clone <repositorio>
cd proyecto
make
sudo make install
```

Consultar siempre la documentación oficial del proyecto antes de compilar.

---

# Gestión de dotfiles

Este repositorio también funciona como almacenamiento central de configuraciones personales.

Ejemplos:

```text
~/.config/hypr/
~/.config/waybar/
~/.config/kitty/
~/.config/mako/
~/.config/wofi/
```

Objetivos:

- Mantener configuraciones versionadas.
- Facilitar migraciones entre equipos.
- Documentar cambios importantes.
- Recuperar configuraciones rápidamente.

---

# Scripts personalizados

Con el tiempo se añadirán scripts para:

- Automatización de tareas repetitivas.
- Configuración inicial de sistemas.
- Gestión de copias de seguridad.
- Personalización del entorno.
- Mantenimiento del sistema.

---

# Aquí comienza mi viaje personal

Este proyecto está en constante evolución.

Cada nueva configuración, ajuste, script o descubrimiento dentro del ecosistema Linux se documentará aquí como parte de un proceso continuo de aprendizaje. Algunas soluciones serán experimentales, otras se convertirán en parte permanente del sistema, pero todas contribuirán a comprender mejor cómo funciona Arch Linux y cómo construir un entorno de trabajo eficiente y personalizado.

Si encuentras alguna mejora posible, detectas errores o simplemente quieres compartir ideas, comentarios o sugerencias, cualquier contribución será bienvenida.

**Bienvenido a Mi Linux.**
