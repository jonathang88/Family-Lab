Family — Notas sanitizadas



Advertencia: se han eliminado o reemplazado todos los datos sensibles (IP, credenciales reales, claves privadas). Este documento es para uso educativo y publicación en GitHub.

Objetivo



Laboratorio WordPress con objetivo de aprendizaje: reconocimiento, enumeración web, acceso vía panel, subida controlada de plugin (reverse shell), post-explotación y escalada de privilegios.

Herramientas principales
Nmap, WhatWeb, Gobuster
curl, hydra (ejemplos de uso con permiso)
Herramientas para análisis de archivos: cat, strings, exiftool (si aplica)
netcat (nc), reverse shell scripts
pspy (monitorización de procesos), valgrind (uso en escalada)
Editor y transferencias: wget, chmod, scp/sftp
1) Reconocimiento y enumeración
Descubrimiento de servicio web con WordPress en /wordpress/.
Nmap (resumen): detectados 22/tcp (ssh) y 80/tcp (http - Apache).
WhatWeb detectó Apache y versión aproximada de WordPress.
Gobuster para enumeración de directorios comunes:
/wordpress, /wp-admin, /wp-content, /wp-includes, /wp-signup.php
2) Verificación de panel de administración
Comprobación de acceso con curl -I y llamadas POST simulando intentos de login.
Ejemplo de POST para analizar respuestas:
curl -X POST http://TARGET/wordpress/wp-login.php -d "log=admin&pwd=wrongpass&wp-submit=Login" -L -v -c cookies_fail.txt
Identificar patrones de éxito/fracaso por:
Texto en HTML (mensaje de error o bienvenida)
Código HTTP y headers (Location para redirecciones)
Cookies establecidas tras login
3) Fuerza bruta (documentado, no ejecutar sin permiso)
Método: análisis de patrones de respuesta y uso de herramientas como hydra.
Ejemplo documentado:
hydra -l admin -P /path/to/wordlist TARGET http-post-form "/wordpress/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Login:F=login_error"
Resultado en laboratorio: credenciales válidas encontradas (sanitizadas en este documento).
4) Explotación — subida de plugin / reverse shell (laboratorio controlado)
Se creó un plugin/archivo ZIP con reverse shell (ejercicio educativo).
Subida mediante el panel de administración de WordPress (actividad simulada en entorno controlado).
Confirmación de conexión reversa:
nc -lvnp 4444 en la máquina atacante
Shell recibido como www-data en el objetivo
Limitaciones de este shell: permisos de www-data, sin privilegios root.
5) Exploración post-explotación
Exploración básica del sistema desde el usuario www-data.
Búsqueda de archivos de interés:
find / -user father 2>/dev/null → ficheros relacionados con usuario father
Lectura de wp-config.php u otros ficheros de configuración (valores sanitizados y reemplazados con [REDACTED]).
Identificación de usuarios locales de ejemplo: father, mother, baby.
6) Escalada de privilegios (ejemplo aplicado)
Enumeración de binarios SUID y comandos con sudo permitidos.
Hallazgo clave en laboratorio: mother puede ejecutar /usr/bin/valgrind como baby sin contraseña (NOPASSWD).
Técnica aplicada:
sudo -u baby /usr/bin/valgrind /bin/bash → shell como baby.
Resultado: acceso a /home/baby, lectura de user.txt (flag: Chilatyfile, incluido aquí como ejemplo educativo).
Se documentó lectura de /etc/shadow en pruebas; los hashes han sido removidos de este archivo por seguridad.
7) Post-explotación avanzada
Identificación de procesos programados/monitoreo (por ejemplo, python /home/mother/check.py ejecutado periódicamente).
Documentación de la creación de un script check.py en /home/mother (detalles de conexión removidos).
Hallazgo de claves en /root/.ssh en el laboratorio original: claves removidas y no se publican.
8) Lecciones y recomendaciones
Validación de uploads: impedir subida arbitraria de plugins/temas sin validación y escaneo.
Permisos y sudoers: revisar entradas NOPASSWD y reducir comandos ejecutables sin contraseña.
Eliminar metadatos: eliminar EXIF o metadatos de archivos públicos que puedan filtrar información.
Monitorización: emplear detección de procesos inusuales y monitoreo de integridad (pspy, osquery, etc.).
Backups y segregación: backups seguros y separación de entornos de prueba y producción.
Notas sobre sanitización
Todas las IPs, dominios, contraseñas y claves han sido reemplazadas por [REDACTED].
No se incluyen claves privadas ni contraseñas reales en este repositorio.
Flags menores (user.txt) se mantienen como ejemplo educativo cuando no comprometen seguridad real.
