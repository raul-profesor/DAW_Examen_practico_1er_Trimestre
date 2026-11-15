# Escenario de despliegue con errores intencionados

Esta copia contiene nueve fallos para que el escenario se diagnosed y corrija.

## Errores incluidos

1. **Root incorrecto:** `puertos.daw` usa `/var/www/puertos/public` en lugar de `/var/www/puertos`.
2. **Puerto incorrecto:** el backend de `puertos.daw` escucha en 8080, pero el balanceador continúa usando el puerto 80 del upstream.
3. **Backend fuera del balanceador:** solo `web1` se incluye en `backend_ips`.
4. **Proxy inverso incorrecto:** `proxy_pass` antepone `/ruta-inexistente`.
5. **Regla de red incorrecta:** el security group del backend permite HTTP desde el propio security group del backend, no desde el del balanceador.
6. **Certificados intercambiados:** `ssl_certificate` apunta a `.key` y `ssl_certificate_key` a `.crt`.
7. **TLS obsoleto:** solo se permite `TLSv1`.
8. **Backend parado:** Nginx se detiene en el segundo backend **antes de configurar los sitios**, por lo que la máquina queda con el servicio parado desde el principio del despliegue. Su configuración sigue siendo válida y puede iniciarse correctamente con `sudo systemctl start nginx`.
9. **Página de inicio inexistente:** la configuración de los sitios pide `home.html` (`index home.html` y `try_files $uri $uri/home.html`), pero en los repositorios desplegados solo existe `index.html`. Por eso el balanceador devuelve 404 en lugar de servir la página de inicio.

## Despliegue

Desde este directorio:

```bash
make deploy
```

Terraform crea las instancias y genera `ansible/inventory.yml`. Ansible instala y configura los servicios, deja los errores presentes y finaliza sin abortar el despliegue. La comprobación `nginx -t` del balanceador se registra, pero su fallo no interrumpe el playbook porque forma parte del fallo intencionado de los certificados.

El handler `Recargar nginx backend` solo recarga nginx si el servicio ya está activo, de modo que no reactiva el segundo backend que debe quedar parado.

## Limpieza

```bash
make destroy
make clean
```
