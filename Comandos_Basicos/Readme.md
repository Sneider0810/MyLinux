# Comandos Basicos de Arch Linux

Una referencia practica para entender como funcionan los comandos en Linux: que hacen, como se construyen y por que se escriben asi.

---

## Como leer un comando

Antes de memorizar comandos, conviene entender su estructura. La mayoria sigue este patron:

```
comando  [opciones]  [argumento]
   |          |           |
que herramienta  como se comporta  sobre que actua
```

Ejemplo real:

```bash
sudo pacman -Syu
```

| Parte | Valor | Significado |
|-------|-------|-------------|
| Ejecutor | `sudo` | Corre el comando como administrador |
| Herramienta | `pacman` | El gestor de paquetes de Arch |
| Opcion | `-S` | Sincronizar con repositorios |
| Opcion | `-y` | Actualizar la base de datos |
| Opcion | `-u` | Actualizar los paquetes instalados |

> Las opciones cortas (`-S`) se pueden combinar en una sola bandera (`-Syu`). Las largas (`--sync`) son equivalentes pero mas descriptivas.

---

## Por que se usa `sudo`

`sudo` permite ejecutar un comando con privilegios de administrador sin tener que iniciar sesion como root.

```bash
sudo pacman -S firefox    # Con permisos de admin
pacman -Q                 # Sin sudo, solo lectura
```

Regla practica: si el comando **modifica** el sistema (instalar, borrar, configurar servicios), necesita `sudo`. Si solo **consulta**, no.

---

## Gestion de paquetes

### pacman — El gestor oficial de Arch

#### Instalar un paquete

```bash
sudo pacman -S firefox
```

`-S` viene de *sync* (sincronizar con los repositorios y traer el paquete).

#### Actualizar todo el sistema

```bash
sudo pacman -Syu
```

Es el comando mas importante del mantenimiento diario. Siempre actualiza antes de instalar algo nuevo.

#### Buscar un paquete

```bash
pacman -Ss navegador
```

`-Ss` = *sync search*. Busca en nombre y descripcion. No necesita `sudo` porque solo lee.

#### Eliminar un paquete

```bash
sudo pacman -R firefox          # Solo el paquete
sudo pacman -Rs firefox         # El paquete + dependencias que ya no usa nadie
sudo pacman -Rns firefox        # Lo anterior + archivos de configuracion
```

Usar `-Rns` es la forma mas limpia de desinstalar.

#### Consultar paquetes instalados

```bash
pacman -Q                   # Lista todos los instalados
pacman -Qi firefox          # Informacion detallada de uno
pacman -Ql firefox          # Archivos que instalo ese paquete
pacman -Qo /usr/bin/firefox # A que paquete pertenece ese archivo
```

Ninguno necesita `sudo` porque solo leen informacion local.

#### Limpiar la cache

```bash
sudo pacman -Sc     # Elimina versiones antiguas descargadas
sudo pacman -Scc    # Elimina toda la cache (mas agresivo)
```

Util para liberar espacio en disco.

---

### yay — Paquetes del AUR

El AUR (Arch User Repository) contiene paquetes mantenidos por la comunidad. `yay` los gestiona igual que `pacman`.

```bash
yay -S visual-studio-code-bin    # Instalar desde AUR
yay -Syu                          # Actualizar sistema + AUR
yay -Ss zoom                      # Buscar en AUR
```

> Instalar `yay` por primera vez:
> ```bash
> sudo pacman -S --needed git base-devel
> git clone https://aur.archlinux.org/yay.git
> cd yay && makepkg -si
> ```

---

## Archivos y directorios

### Navegar el sistema

#### Saber donde estas

```bash
pwd
```

*Print Working Directory*. Muestra la ruta completa del directorio actual.

#### Moverse entre directorios

```bash
cd /etc              # Ruta absoluta (desde la raiz del sistema)
cd Documentos        # Ruta relativa (desde donde estas ahora)
cd ..                # Subir un nivel
cd ~                 # Ir al directorio personal
cd -                 # Volver al directorio anterior
```

#### Ver el contenido de un directorio

```bash
ls           # Listado simple
ls -l        # Con detalles (permisos, tamanio, fecha)
ls -a        # Incluye archivos ocultos (los que empiezan con .)
ls -lah      # Las tres anteriores combinadas, tamanios legibles
```

---

### Crear y eliminar

#### Crear un directorio

```bash
mkdir proyectos
mkdir -p proyectos/linux/scripts    # Crea toda la estructura de una vez
```

`-p` (*parents*) evita el error cuando los directorios intermedios no existen.

#### Crear un archivo vacio

```bash
touch notas.txt
```

Si el archivo ya existe, actualiza su fecha de modificacion sin borrar el contenido.

#### Eliminar

```bash
rm archivo.txt           # Eliminar archivo
rm -r directorio/        # Eliminar directorio y todo su contenido
rm -rf directorio/       # Lo mismo, sin pedir confirmacion
```

> `-rf` es irreversible. No hay papelera de reciclaje. Verifica bien antes de ejecutarlo.

---

### Copiar y mover

#### Copiar

```bash
cp archivo.txt respaldo.txt         # Copiar archivo
cp -r configuracion/ respaldo/      # Copiar directorio completo
cp -i archivo.txt destino/          # Pedir confirmacion si va a sobrescribir
```

#### Mover o renombrar

```bash
mv archivo.txt documentos/          # Mover a otro directorio
mv viejo.txt nuevo.txt              # Renombrar (mismo directorio)
```

`mv` hace las dos cosas: si el destino es una carpeta, mueve. Si es un nombre nuevo, renombra.

---

## Leer y editar texto

#### Ver el contenido de un archivo

```bash
cat archivo.txt          # Imprime todo el contenido de una vez
less archivo.txt         # Navegar pagina por pagina (util para archivos largos)
```

En `less`: usa las flechas para desplazarte, `/texto` para buscar, `q` para salir.

#### Editar con nano

```bash
nano archivo.txt
```

Editor simple, ideal para cambios rapidos en configuraciones.

| Atajo | Accion |
|-------|--------|
| `Ctrl + O` | Guardar |
| `Ctrl + X` | Salir |
| `Ctrl + K` | Cortar linea |
| `Ctrl + U` | Pegar |
| `Ctrl + W` | Buscar |

---

## Servicios del sistema (systemctl)

`systemctl` controla los servicios que corren en el sistema. Un servicio puede estar **activo/inactivo** y **habilitado/deshabilitado** al arranque. Son dos cosas distintas.

```bash
sudo systemctl start sshd      # Iniciar ahora
sudo systemctl stop sshd       # Detener ahora
sudo systemctl restart sshd    # Reiniciar (aplicar cambios)
sudo systemctl reload sshd     # Recargar config sin reiniciar (si el servicio lo soporta)

sudo systemctl enable sshd     # Que arranque automaticamente al iniciar el sistema
sudo systemctl disable sshd    # Que no arranque automaticamente

systemctl status sshd          # Ver estado, si esta corriendo y errores recientes
```

> `enable` y `start` son independientes. `enable` solo lo activa para el proximo arranque; `start` lo inicia ahora mismo. Para ambas cosas a la vez: `sudo systemctl enable --now sshd`

---

## Procesos

#### Ver procesos activos

```bash
ps aux          # Lista todos los procesos del sistema con detalle
```

#### Monitor en tiempo real

```bash
top             # Monitor basico interactivo
htop            # Version mejorada (requiere instalacion: pacman -S htop)
```

#### Terminar un proceso

```bash
kill 1234           # Enviar senal de terminacion al proceso con PID 1234
kill -9 1234        # Forzar terminacion (cuando el anterior no funciona)
pkill firefox       # Terminar por nombre en lugar de PID
```

Para conocer el PID de un proceso: `ps aux | grep firefox`

---

## Redes

#### Ver direcciones IP

```bash
ip addr             # Muestra todas las interfaces y sus IPs
ip addr show eth0   # Solo una interfaz especifica
```

#### Probar conectividad

```bash
ping archlinux.org          # Envia paquetes continuamente (Ctrl+C para detener)
ping -c 4 archlinux.org     # Envia solo 4 paquetes
```

#### Descargar desde terminal

```bash
curl https://archlinux.org              # Muestra el contenido en terminal
curl -O https://ejemplo.com/archivo.zip # Descarga el archivo
wget https://ejemplo.com/archivo.zip    # Alternativa a curl para descargar
```

---

## Consultar el manual de cualquier comando

Cada comando tiene su propia documentacion incluida en el sistema:

```bash
man pacman      # Manual completo de pacman
man ls          # Manual de ls
```

Dentro del manual: flechas para navegar, `/` para buscar, `q` para salir.

Para una ayuda rapida:

```bash
pacman --help
ls --help
```

---

## Referencia rapida

| Tarea | Comando |
|-------|---------|
| Actualizar el sistema | `sudo pacman -Syu` |
| Instalar un paquete | `sudo pacman -S nombre` |
| Desinstalar limpio | `sudo pacman -Rns nombre` |
| Buscar paquete | `pacman -Ss termino` |
| Donde estoy | `pwd` |
| Listar directorio | `ls -lah` |
| Crear carpetas anidadas | `mkdir -p ruta/completa` |
| Copiar directorio | `cp -r origen/ destino/` |
| Mover o renombrar | `mv origen destino` |
| Ver archivo largo | `less archivo` |
| Estado de un servicio | `systemctl status nombre` |
| Iniciar servicio ahora y al arranque | `sudo systemctl enable --now nombre` |
| Ver procesos | `ps aux` o `htop` |
| Ver IPs | `ip addr` |
