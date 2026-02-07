# 📘 Configuración Completa de la Máquina de Desarrollo

> **Entorno:** Windows 10/11 + WSL2 + Ubuntu 24.04  
> **Stack:** Apache, PHP 8.3, MariaDB, phpMyAdmin, mkcert (HTTPS local)

Este documento consolida:
- **Todas las correcciones reales** hechas durante la instalación
- **Errores comunes**, mensajes reales y cómo solucionarlos
- **Comandos adicionales** que normalmente no se documentan

Está pensado como **bitácora técnica + guía de referencia**.

---

## 1. Instalación de WSL y Ubuntu

### 1.1 Instalar WSL
Desde **PowerShell como Administrador**:

```powershell
wsl --install -d Ubuntu
```

Si ya estaba instalado:

```powershell
wsl --update
```

### 1.2 Verificar distros instaladas

```powershell
wsl -l -v
```

---

## 2. Primer ingreso a Ubuntu (WSL)

Al ejecutar:

```powershell
wsl -d Ubuntu
```

Verás algo como:

```
To run a command as administrator (user "root"), use "sudo <command>".
```

✅ **Esto NO es un error**, solo es un mensaje informativo.

---

## 3. Actualización del sistema

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ca-certificates gnupg lsb-release curl git unzip
```

### 🔹 Salir de la terminal
- `exit`
- o cerrar la ventana

---

## 4. Apache y PHP

### 4.1 Instalar Apache

```bash
sudo apt install apache2 -y
```

### 4.2 Instalar PHP 8.3 y extensiones

```bash
sudo apt install -y php php-cli php-common php-curl php-gd php-intl php-mbstring php-mysql php-xml php-zip
```

### 4.3 Controlar Apache en WSL

⚠️ **WSL NO usa systemd**

❌ NO usar:
```bash
systemctl restart apache2
```

✅ Usar siempre:
```bash
sudo service apache2 start
sudo service apache2 restart
sudo service apache2 reload
sudo service apache2 status
```

---

## 5. Configurar hosts en Windows

Editar como **Administrador**:

```
C:\Windows\System32\drivers\etc\hosts
```

Agregar:

```
127.0.0.1   orderdesk.local
```

💡 No usar `#` al inicio, o quedará comentado.

---

## 6. HTTPS local con mkcert

### 6.1 Instalar mkcert (Windows)

```powershell
winget install mkcert
```

Cerrar y abrir la terminal nuevamente.

### 6.2 Instalar CA local

```powershell
mkcert -install
```

### 6.3 Generar certificado

```powershell
mkcert orderdesk.local
```

Se generan:
- `orderdesk.local.pem`
- `orderdesk.local-key.pem`

---

## 7. Copiar certificados a WSL

```bash
sudo mkdir -p /etc/ssl/localcerts
sudo cp /mnt/c/Users/<TU_USUARIO>/orderdesk.local*.pem /etc/ssl/localcerts/
```

Permisos:

```bash
sudo chmod 600 /etc/ssl/localcerts/orderdesk.local-key.pem
sudo chmod 644 /etc/ssl/localcerts/orderdesk.local.pem
```

---

## 8. Apache VirtualHost HTTPS

Archivo:

```bash
sudo nano /etc/apache2/sites-available/orderdesk.conf
```

Contenido:

```apache
<VirtualHost *:443>
    ServerName orderdesk.local
    DocumentRoot /home/orderdesk/web/orderdesk.local/public_html

    SSLEngine on
    SSLCertificateFile /etc/ssl/localcerts/orderdesk.local.pem
    SSLCertificateKeyFile /etc/ssl/localcerts/orderdesk.local-key.pem

    <Directory /home/orderdesk/web/orderdesk.local/public_html>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Guardar en nano
- Guardar: `Ctrl + O` → Enter
- Salir: `Ctrl + X`

Habilitar:

```bash
sudo a2ensite orderdesk.conf
sudo a2dissite 000-default.conf
sudo service apache2 reload
```

---

## 9. Estructura tipo Servidor

⚠️ **IMPORTANTE:** Reemplazar `orderdesk` por tu usuario Linux real.

```bash
sudo mkdir -p /home/orderdesk/web/orderdesk.local/public_html
```

Permisos:

```bash
sudo chown -R orderdesk:www-data /home/orderdesk/web
sudo find /home/orderdesk/web -type d -exec chmod 775 {} \;
sudo find /home/orderdesk/web -type f -exec chmod 664 {} \;
```

---

## 10. MariaDB (MySQL)

### 10.1 Instalación

```bash
sudo apt install mariadb-server mariadb-client -y
```

### 10.2 Acceder

```bash
sudo mysql -u root
```

### 10.3 Crear base de datos y usuario

```sql
CREATE DATABASE orderdesk_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER 'orderdesk_emers'@'localhost' IDENTIFIED BY 'orderdesk123';

GRANT ALL PRIVILEGES ON orderdesk_db.* TO 'orderdesk_emers'@'localhost';

FLUSH PRIVILEGES;
EXIT;
```

❌ Error común:
```
Can't find any matching row in the user table
```

✅ Solución: crear primero el usuario.

---

## 11. phpMyAdmin

### 11.1 Instalación manual

```bash
sudo mkdir -p /usr/share/phpmyadmin
cd /usr/share/phpmyadmin
sudo wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz -O phpmyadmin.tar.gz
sudo tar -xzf phpmyadmin.tar.gz --strip-components=1
sudo rm phpmyadmin.tar.gz
```

### 11.2 Configuración

```bash
sudo cp config.sample.inc.php config.inc.php
sudo nano config.inc.php
```

Configurar `blowfish_secret`.

### 11.3 Apache conf

```bash
sudo nano /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin
sudo service apache2 reload
```

Acceder:
```
http://localhost/phpmyadmin
```

---

## 12. Errores reales y soluciones

### ❌ systemctl
```
System has not been booted with systemd
```
✅ Usar `service`.

---

### ❌ nginx no encuentra .key

Causa: nombre incorrecto (`.key` vs `-key.pem`).

✅ Solución:
- Corregir ruta en config
- Verificar con:
```bash
ls -l /etc/ssl/localcerts
```

---

### ❌ Terminal atrapada en `>`

Causa: bloque sin cerrar.

✅ Solución:
```
Ctrl + C
```

---

## 13. Limpieza y mantenimiento

Limpiar pantalla:
```bash
clear
```

Borrar historial:
```bash
history -c
rm ~/.bash_history
```

Reiniciar WSL:
```powershell
wsl --shutdown
```

---

## 14. Conclusión

Este documento refleja una **instalación real**, con errores reales y soluciones prácticas.

Sirve como:
- Guía de instalación
- Evidencia técnica
- Base para futuros despliegues

---

🚀 **Fin del documento**


## Acceso a phpMyAdmin

### Importante: Puerto configurado en Apache
Según la documentación base y la configuración realizada, Apache **escucha en el puerto 8080**, no en el puerto 80 por defecto.

Esto significa que **NO se debe ingresar únicamente a `http://localhost`**, ya que eso intenta conectarse al puerto 80 y genera el error:

```
ERR_CONNECTION_REFUSED
```

### Regla clave
> Apache solo responde en el puerto definido en la directiva `Listen`

| Configuración Apache | URL correcta |
|---------------------|-------------|
| `Listen 80` | `http://localhost` |
| `Listen 8080` | `http://localhost:8080` |

---

### Verificación del puerto configurado

```bash
sudo nano /etc/apache2/ports.conf
```

Configuración esperada:

```apache
Listen 8080
```

Guardar y salir del editor:
- **CTRL + O** → Enter (guardar)
- **CTRL + X** (salir)

---

### Verificación de Apache en ejecución

```bash
sudo service apache2 status
```

Salida esperada:
```
apache2 is running
```

---

### Verificación técnica del puerto activo

```bash
sudo ss -lntp | grep apache
```

Salida esperada:
```
LISTEN 0 4096 0.0.0.0:8080
```

---

### URL correcta de acceso

Abrir en el navegador de Windows:

- Iniciar apache con comando:
```
sudo service apache2 start
```
- Apache:
```
http://localhost:8080
```

- phpMyAdmin:
```
http://localhost:8080/phpmyadmin
```

---

### Error común y su causa

| Situación | Resultado |
|---------|----------|
| Apache en 8080 + acceso a `localhost` | ❌ Connection refused |
| Apache en 8080 + `localhost:8080` | ✅ Funciona |

---

### Alternativa (no usada en este manual)
Si se desea usar el puerto clásico:

```apache
Listen 80
```

Y reiniciar Apache:

```bash
sudo service apache2 restart
```

> **Nota:** Este manual mantiene el puerto **8080** para alinearse con la documentación base.

