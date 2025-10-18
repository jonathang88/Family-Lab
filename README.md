# Family Lab — Notas sanitizadas

**Aviso importante**
Este repositorio contiene notas educativas sobre un laboratorio WordPress. Se han eliminado o reemplazado todos los datos sensibles (IP, credenciales, claves privadas). Este material es solo para aprendizaje en entornos controlados. **No** realices pruebas contra sistemas reales sin autorización por escrito.

## Objetivo

Laboratorio orientado a aprendizaje práctico de: reconocimiento, enumeración web, análisis de autenticación, subida controlada de un plugin (reverse shell) en un entorno de prueba, post-explotación y escalada de privilegios.

---

## Herramientas principales

* Reconocimiento y enumeración: **nmap**, **WhatWeb**, **Gobuster**
* Interacción HTTP: **curl**
* Fuerza bruta (documentado): **hydra** *(no ejecutar sin permiso)*
* Transferencias y shells: **wget**, **scp/sftp**, **netcat (nc)**, scripts de reverse shell
* Análisis local y monitoreo: **cat**, **strings**, **exiftool**, **pspy**
* Escalada/depuración: **valgrind** (uso demostrativo en laboratorio)

---

## Resumen de pasos realizados

### 1) Reconocimiento y enumeración

* Detectado servicio WordPress en `/wordpress/`.
* Nmap: puertos típicos abiertos (`22/tcp` SSH, `80/tcp` HTTP - Apache).
* Uso de WhatWeb para identificar servidor/versión y Gobuster para listar rutas comunes:

  * `/wordpress`, `/wp-admin`, `/wp-content`, `/wp-includes`, `/wp-signup.php`

### 2) Verificación del panel de administración

* Comprobación con `curl -I` y pruebas POST para observar respuestas de login.
* Ejemplo para probar credenciales incorrectas y analizar la respuesta:

```bash
curl -X POST http://TARGET/wordpress/wp-login.php \
  -d "log=admin&pwd=wrongpass&wp-submit=Login" -L -v -c cookies_fail.txt
```

* Se analizan:

  * Texto HTML (mensajes de error o bienvenida)
  * Headers HTTP (ej. `Location` en caso de redirección)
  * Cookies establecidas tras el intento de login

### 3) Fuerza bruta (documentado)

* Técnica: identificar patrón de éxito/fracaso y usar herramientas para automatizar (ej. `hydra`).
* Ejemplo documentado **para estudio** (no ejecutar sin autorización):

```bash
hydra -l admin -P /path/to/wordlist TARGET \
  http-post-form "/wordpress/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Login:F=login_error"
```

* En el laboratorio se obtuvieron credenciales válidas (estas credenciales han sido sanitizadas en este repositorio).

### 4) Explotación — subida de plugin / reverse shell (laboratorio controlado)

* Se creó un plugin ZIP con una reverse shell (uso **únicamente** en el laboratorio).
* Subida mediante el panel de administración y activación controlada.
* Confirmación de conexión inversa (en la máquina atacante):

```bash
nc -lvnp 4444
# esperar la conexión reversa desde el objetivo
```

* Resultado: shell con el usuario `www-data` (permisos limitados, no root).

### 5) Exploración post-explotación

* Desde `www-data` se buscaron archivos de interés y configuraciones:

```bash
find / -user father 2>/dev/null
# leer archivos relevantes con precaución y según reglas del laboratorio
```

* Lectura de `wp-config.php` y otros ficheros: todos los valores sensibles han sido reemplazados por `[REDACTED]`.

### 6) Escalada de privilegios (ejemplo aplicado)

* Enumeración de binarios SUID y revisado `sudo -l` para usuarios locales.
* Hallazgo: el usuario `mother` podía ejecutar `/usr/bin/valgrind` como `baby` sin contraseña (NOPASSWD).
* Técnica usada (demostrativa):

```bash
sudo -u baby /usr/bin/valgrind /bin/bash
```

* Resultado: shell como `baby`, acceso a `/home/baby` y lectura de `user.txt` (ejemplo: `Chilatyfile`).
* Observación: se consultó `/etc/shadow` en pruebas; los hashes se han removido de este documento.

### 7) Post-explotación avanzada

* Se detectaron procesos de monitoreo/cron que ejecutaban scripts (ej. `python /home/mother/check.py`).
* Se documentó la existencia de un script `check.py` cuyo contenido con direcciones reales fue removido por seguridad.
* Claves privadas encontradas en laboratorio fueron **eliminadas** de estas notas y no se publican.

---

## Lecciones y recomendaciones de seguridad

1. **Validar uploads**: impedir subida arbitraria de plugins y validar tipos y contenido antes de almacenar/activar.
2. **Revisar `sudoers`**: evitar reglas `NOPASSWD` innecesarias que permitan escalada lateral.
3. **Remover metadatos**: limpiar EXIF y metadatos en imágenes y ficheros públicos.
4. **Monitorización**: usar herramientas de detección de procesos y monitoreo de integridad (pspy, osquery, IDS).
5. **Backups y separación de entornos**: mantener entornos de prueba separados de producción y backups cifrados.

---

## Nota sobre sanitización

* Todas las IPs, dominios, credenciales, claves privadas y datos sensibles han sido reemplazados por `[REDACTED]`.
* No se incluyen claves privadas ni contraseñas reales en este repositorio.
* Las flags pequeñas (`user.txt`) pueden mantenerse como ejemplos didácticos solo si no comprometen seguridad real.

---

## Cómo subir este repositorio a GitHub (comandos rápidos)

```bash
git init
git add .
git commit -m "Add sanitized Family Lab notes"
# crear repo remoto con gh CLI o por la web, luego:
git remote add origin git@github.com:<tu-usuario>/family-lab.git
git branch -M main
git push -u origin main
```

---

Si quieres, puedo:

* Generar una versión en inglés simple para tu web.
* Añadir badges (fecha, nivel, licencia) al README.
* Preparar un `CHANGELOG.md` y un `LICENSE` (MIT) listo para incluir. ¿Cuál prefieres que haga ahora?
