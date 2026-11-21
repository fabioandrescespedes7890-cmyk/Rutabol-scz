# Rutabol-scz — Servidor DNS Autoritativo BIND9 en Docker

Este repositorio contiene la configuración del proyecto **Rutabol-scz** junto con la infraestructura de resolución de nombres local mediante un servidor **DNS Autoritativo BIND9** desplegado en Docker (`rutabol-dns`).

---

## Estructura del Servicio DNS

- **`dns/named.conf`**: Configuración principal de BIND9 y declaración de la zona maestra `rutabol.local`.
- **`dns/zones/db.rutabol.local`**: Archivo de zona que contiene los registros `SOA`, `NS`, `A` (`@`, `ns1`, `app`, `api`) y `CNAME` (`www`).
- **`compose.yaml`**: Archivo de orquestación Docker que define el contenedor `rutabol-dns` mapeando los puertos `53/udp` y `53/tcp`.

---

## Preguntas y Respuestas de la Consigna

### Que contiene un archivo de zona
Un archivo de zona es un archivo de texto plano que define los registros DNS de un dominio o subdominio bajo la autoridad de un servidor. Contiene:
- **Directivas globales:** Como `$TTL` (*Time To Live*), que define el tiempo en segundos que los clientes o servidores recursivos pueden mantener en caché las respuestas de la zona.
- **Registro `SOA` (*Start of Authority*):** Establece los parámetros principales de la zona, como el servidor de nombres primario, el correo del administrador, el número de serie (`Serial`), y los intervalos de actualización (`Refresh`), reintento (`Retry`), expiración (`Expire`) y caché negativa.
- **Registros `NS` (*Name Server*):** Indican qué servidores de nombres son autoritativos para responder consultas sobre el dominio.
- **Registros de recursos (`A`, `AAAA`, `CNAME`, `MX`, `TXT`, etc.):** Asocian nombres con direcciones IPv4 (`A`) o IPv6 (`AAAA`), crean alias hacia otro nombre canónico (`CNAME`) o almacenan información adicional de la zona.

### Por que se utiliza el Serial SOA
El número **Serial** del registro `SOA` (formateado comúnmente como `YYYYMMDDnn`, por ejemplo `2026112101`) funciona como el **control de versión** del archivo de zona:
- **Sincronización entre servidores:** Permite que los servidores DNS secundarios (esclavos) comparen su número de serie con el del servidor primario; si el del primario es mayor, detectan que hubo modificaciones y solicitan una actualización de zona.
- **Control y trazabilidad de cambios:** Obliga y facilita llevar un registro ordenado cada vez que se edita la zona, evitando que cambios nuevos pasen desapercibidos.

### comprobar registros DNS
Los registros DNS se comprueban realizando consultas directas al servidor DNS (en el puerto `53` UDP/TCP) mediante herramientas de diagnóstico de red:
- **Con `nslookup` (en Windows o Linux):**
  ```powershell
  nslookup -type=SOA rutabol.local 127.0.0.1
  nslookup app.rutabol.local 127.0.0.1
  nslookup www.rutabol.local 127.0.0.1



 ### ¿Por qué el DNS por sí solo no publica una aplicación web?
El DNS cumple únicamente la función de traducir nombres de dominio a direcciones IP (y otros registros de red); no aloja ni sirve aplicaciones web por sí mismo debido a que:
No transporta ni procesa tráfico web: DNS no entrega páginas HTML, archivos estáticos, respuestas HTTP/HTTPS ni ejecuta lógica de backend o APIs.
Requiere un servidor web o aplicación en ejecución: Hace falta un servidor o servicio (como Nginx, Apache, Next.js o NestJS) escuchando en los puertos correspondientes (80, 443, 3000, etc.) y atendiendo las solicitudes de los clientes.
Depende de la red y del acceso al destino: Resolver un nombre a una IP solo indica dónde conectarse; para que la aplicación abra en el navegador se necesita conectividad de red, puertos abiertos y el servicio activo en esa dirección.