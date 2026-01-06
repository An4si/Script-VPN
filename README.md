# Script-VPN

# 🌐 THM-HTB VPN Automator

![Bash](https://img.shields.io/badge/Language-Bash-4EAA25?style=flat&logo=gnu-bash&logoColor=white)
![Security](https://img.shields.io/badge/Field-Pentesting-red)

Este script ha sido diseñado para simplificar y automatizar la conexión a redes privadas virtuales (VPN) de plataformas como **TryHackMe** o **HackTheBox**. Olvídate de gestionar procesos eternos que a veces se quedan colgados y no sabes si estas conectado bien o mal o teber una terminal bloqueada solo para mantener la conexión.

## 🚀 ¿Por qué usar este script?

Normalmente, al ejecutar `sudo openvpn archivo.ovpn`, la terminal se queda ocupada y, si la conexión falla o se queda en segundo plano, es difícil de gestionar. Este script resuelve esos problemas:

- **Ejecución en segundo plano (Modo Daemon):** Conecta y libera tu terminal instantáneamente.
- **Gestión de conflictos:** Cierra automáticamente cualquier sesión de OpenVPN previa para evitar errores de interfaz.
- **Verificación en tiempo real:** No solo lanza el comando; espera y confirma que la interfaz `tun0` ha recibido una IP válida.
- **Diagnóstico rápido:** Si la conexión no se establece en 20 segundos, te avisa del error.

## 🛠️ Instalación y Uso

### Opcion 1 (La mas recomendable y sencilla)

1. **Clona el repositorio:**
   ```bash
   git clone https://github.com/An4si/Script-VPN.git
   ```
2. **Entra en dicha carpeta:**
   ```bash
      cd Script-VPN
   ```
3. **Dale persmisos de ejecucion:**
   ```bash
   chmod +x vpn_connect.sh
   ```
   
   Una vez tengas el script clonado en tu VM y le hayas dado permisos de ejecucion **NO TENDRAS QUE TOCAR NADA DE DENTRO DEL SCRIPT.**
   Lo unico que tendras que hacer en tu terminal es ejecutar mi script + tu VPN descargada previamente de cualquer plataforma de CTF's como podria ser TryHackMe, HackTheBox...
4. **Ejecucion del comando:**
   ```bash
   ./vpn_connect.sh ruta/de/tu/archivo.ovpn
   ```

5. **Desconectando Script VPN de forma segura**
   ```bash
   ./vpn_connect.sh stop
   ```
<br>
<br>

**¿Qué pasa dentro del script?** 
<br>
<br>
El script toma tu vpn descargada previamnente en dicha plataforma (ejemplo.ovpn) y lo guarda en la variable $1. Luego, cuando llega a la parte de sudo openvpn --config "$VPN_FILE", el script entiende que debe usar el archivo VPN de dicha paltaforma que te hayas descargado.
<br>
<br>
<br>

### Opción 2: (Si prefieres dejar una VPN concreta fija o por si la anterior opcion falla.)
Si por ejemplo solo usas HackTheBox y quieres ejecutar el script simplemente poniendo ```./vpn_connect.sh``` sin escribir nada más o si la opcion 1 falla lo unico que  tendrías que hacer es editar esta parte del script:


```python

VPN_FILE=$1
##Deberías borrar esa línea y poner la ruta de su archivo:

VPN_FILE="/home/tuusuario/descargas/htb.ovpn"
```


Espero que te sirva para que puedas acceder sin molestias a cualquier VPN de nuestras increibles plataformas de CTF's.

Feliz Hacking!🕵️‍♂️


## Script

```python
#!/bin/bash

# --- AUTOMATIZACIÓN VPN BY AN4SI ---

# Comprobar si se ha pasado un argumento
if [ -z "$1" ]; then
    echo "[!] USO: $0 <ruta_archivo_vpn.ovpn> o $0 stop"
    exit 1
fi

# Opción para desconectar
if [ "$1" == "stop" ]; then
    echo "[-] Cerrando conexión VPN..."
    sudo killall openvpn 2>/dev/null
    echo "[+] Desconectado correctamente."
    exit 0
fi

VPN_FILE=$1

# 1. Comprobar si el archivo existe
if [ ! -f "$VPN_FILE" ]; then
    echo "[!] ERROR: No se encuentra el archivo: $VPN_FILE"
    exit 1
fi

# 2. Limpiar y ejecutar
sudo killall openvpn 2>/dev/null
sudo openvpn --config "$VPN_FILE" --daemon

echo -n "[-] Negociando conexión"

# 3. Bucle de espera
for i in {1..10}; do
    IP_VPN=$(ip addr show tun0 2>/dev/null | grep "inet " | awk '{print $2}' | cut -d/ -f1)
    if [ ! -z "$IP_VPN" ]; then
        echo -e "\n=========================================="
        echo "    ✅ ESTADO: CONECTADO"
        echo "    Tu IP VPN: $IP_VPN"
        echo "=========================================="
        exit 0
    fi
    echo -n "."
    sleep 2
done

echo -e "\n[!] ERROR: La interfaz tun0 no se levantó."
```
