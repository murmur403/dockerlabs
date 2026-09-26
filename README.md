# 🐳 DockerLabs Writeups

Mis soluciones de las máquinas de [DockerLabs](https://dockerlabs.es/).

Práctica de pentesting en entornos controlados: reconocimiento, explotación
y escalada de privilegios.

---

## 📚 Índice de máquinas

| Máquina | Dificultad | Técnicas principales | Writeup |
|---------|-----------|----------------------|---------|
| Vacaciones | Fácil | Hydra, pivot por correo, GTFObins (ruby) | [ver](./vacaciones.md) |
| Obsession | Fácil | _pendiente_ | [ver](./obsession.md) |
| Tproot | Medio | _pendiente_ | [ver](./tproot.md) |



---

## 🛠️ Herramientas utilizadas

![nmap](https://img.shields.io/badge/-nmap-blue)
![hydra](https://img.shields.io/badge/-hydra-red)
![gobuster](https://img.shields.io/badge/-gobuster-orange)
![Burp Suite](https://img.shields.io/badge/-Burp%20Suite-purple)
![GTFObins](https://img.shields.io/badge/-GTFObins-green)

- **Reconocimiento:** `nmap`, `whatweb`
- **Enumeración web:** `gobuster`, `ffuf`, `curl`
- **Fuerza bruta:** `hydra`
- **Proxy/Intercept:** Burp Suite
- **Escalada:** GTFObins, `sudo -l`, `linpeas`

---

## 📖 Estructura de cada writeup

Cada archivo `.md` sigue esta plantilla:

1. **Resumen** — dificultad, SO, técnicas.
2. **Reconocimiento** — nmap, puertos, servicios.
3. **Enumeración** — web, usuarios, pistas.
4. **Explotación** — credenciales, acceso inicial.
5. **Escalada de privilegios** — vector y exploit.
6. **Evidencia** — captura de `whoami: root`.
7. **Lecciones aprendidas** — qué se practicó.

---

## ⚠️ Disclaimer

Todos los writeups son de máquinas **intencionadamente vulnerables**
desplegadas en local con [DockerLabs](https://dockerlabs.es/).
No se ha atacado ningún sistema real sin autorización.

---

**Autor:** [Murmur](https://github.com/murmur403)
