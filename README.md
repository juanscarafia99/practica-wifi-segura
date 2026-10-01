# Informe de Auditoría de Red Wi-Fi Insegura

**Alumno:** Juan Scarafia
**Rol:** Auditor de seguridad junior
**Fecha:** 30/09/2026

## 1. Introducción

Cuando nos conectamos a la Wi-Fi de un bar, un aeropuerto o una plaza, compartimos el mismo medio con desconocidos. En este informe analizo qué información queda expuesta al navegar un sitio **HTTP** (sin cifrado), comparo con **HTTPS** y explico cómo una **VPN** protege el tráfico en una red pública. Termino con mis 3 Reglas de Oro.

## 2. Sitio analizado

| Dato | Valor |
|---|---|
| URL | `http://neverssl.com/` |
| Host | `neverssl.com` |
| IP del servidor | `34.223.124.45` |
| Puerto | `80` (HTTP) |
| Herramientas | Wireshark (captura en mi PC); los mismos campos se ven en DevTools (F12 → pestaña Network) |

`neverssl.com` es un sitio que funciona a propósito **sin HTTPS**, por eso sirve para ver cómo viaja una página sin cifrado.

> **Nota:** en la consigna el host de ejemplo es `http://ejemplo-inseguro.com`. En esta práctica ese rol lo cumple `neverssl.com`: ambos son sitios servidos por HTTP plano, sin cifrado, así que el análisis aplica igual a `http://ejemplo-inseguro.com`.

## 3. Evidencia observada

La primera solicitud al cargar la página es el documento HTML pedido por **HTTP**. Estos son los datos que muestra la pestaña **Network** al seleccionar esa solicitud; los tomé de mi captura de tráfico con Wireshark (frames 4346 y 4349), que registra la misma petición.

**Solicitud enviada por el navegador:**

```http
GET / HTTP/1.1
Host: neverssl.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Claude/2.16120.0 Chrome/152.0.7977.130 Safari/537.36 MSIX
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate
Accept-Language: es
```

**Respuesta del servidor:**

```http
HTTP/1.1 200 OK
Server: Apache/2.4.68 ()
Content-Type: text/html; charset=UTF-8
Content-Encoding: gzip
Content-Length: 1900
```

| Elemento | Valor observado |
|---|---|
| URL solicitada | `http://neverssl.com/` |
| Método HTTP | `GET` |
| Host | `neverssl.com` |
| Protocolo | `HTTP/1.1` sin cifrado (puerto 80) |
| Código de respuesta | `200 OK` |
| Headers visibles | `User-Agent`, `Accept`, `Accept-Language`, `Accept-Encoding`, `Connection` |

## 4. Análisis

### 4.1 ¿Qué protocolo utiliza el sitio?

Utiliza **HTTP** y no HTTPS. La URL empieza con `http://`, la conexión va al puerto **80** y el navegador marca el sitio como **"No seguro"**. No hay negociación TLS: los datos viajan en **texto plano**.

### 4.2 ¿Qué información puede observarse durante la solicitud?

- **Host:** `neverssl.com`, es decir, a qué sitio me conecté.
- **URL completa y método:** `GET /`, qué página pedí.
- **User-Agent:** sistema operativo (Windows 10 de 64 bits) y versión exacta del navegador (Chromium 152).
- **Accept-Language:** el idioma preferido (`es`).
- **Headers de la respuesta:** software del servidor (`Apache/2.4.68`), tipo y tamaño del contenido.
- **El contenido completo de la página:** con Wireshark ("Seguir secuencia HTTP") se lee el HTML entero.

### 4.3 Riesgos de navegar por HTTP en una Wi-Fi pública

- **Sniffing (intercepción):** cualquiera en la misma red puede capturar los paquetes con Wireshark y leer URLs, formularios, usuarios, contraseñas y cookies.
- **Robo de sesión:** si una cookie de sesión viaja por HTTP, el atacante puede copiarla y entrar a la cuenta sin saber la contraseña.
- **Man-in-the-Middle:** con técnicas como ARP spoofing, el atacante se hace pasar por el router. Todo el tráfico pasa por su equipo y puede **leerlo o modificarlo**, por ejemplo para inyectar malware o cambiar un enlace de descarga.
- **Evil Twin:** una red falsa con el nombre del local ("Bar_WiFi_Gratis") le da al atacante control total del tráfico desde el primer momento.
- **Perfilado:** aunque no se roben contraseñas, se ven los sitios visitados, los horarios y el dispositivo, lo que expone hábitos e identidad.
- **Una contraseña compartida no protege:** si la clave del Wi-Fi está impresa en el ticket, todos los clientes la tienen y pueden atacar a los demás desde adentro.

### 4.4 Comparación HTTP vs HTTPS

| | HTTP | HTTPS |
|---|---|---|
| Puerto | 80 | 443 |
| Cifrado | No, texto plano | Sí, TLS |
| URL, headers y contenido | Visibles | Cifrados |
| Integridad | Se puede modificar en tránsito | Se detecta cualquier cambio |
| Identidad del servidor | No se verifica | Se verifica con certificado |

Con HTTPS, un atacante solo ve la IP de destino, el nombre del sitio (SNI) y la cantidad de datos, pero no el contenido.

## 5. Cómo ayuda una VPN

- **Cifrado:** la VPN cifra todo el tráfico del dispositivo antes de que salga a la red Wi-Fi, con protocolos como WireGuard, OpenVPN o IPsec. Incluye también el tráfico HTTP y las consultas DNS, que normalmente viajan en claro.
- **Túnel seguro (encapsulamiento):** cada paquete original se **encapsula** dentro de otro paquete cifrado dirigido al servidor VPN. En la red pública solo se ve un **túnel** entre mi equipo y ese servidor.
- **Protección del tráfico:** si alguien captura con Wireshark en la cafetería, ya no ve `GET /`, ni `Host: neverssl.com`, ni el User-Agent: solo ve **ruido cifrado ilegible** hacia una única IP. Un Man-in-the-Middle o un Evil Twin no pueden leer ni modificar los datos.
- **Privacidad:** los sitios visitados quedan ocultos para la red local y para el proveedor de Internet, y los sitios ven la IP del servidor VPN en lugar de la mía.

**Límite importante:** la VPN protege el tramo desde mi equipo hasta su servidor. Desde ahí hasta el sitio, el HTTP vuelve a viajar sin cifrar, así que la VPN **no reemplaza a HTTPS**. Además, hay que elegir un proveedor de VPN confiable, porque este pasa a ver el tráfico.

## 6. Mis 3 Reglas de Oro para navegar en redes Wi-Fi públicas

1. **Solo HTTPS:** activar el "modo solo HTTPS" del navegador y nunca ingresar contraseñas ni datos personales en sitios que muestren "No seguro".
2. **VPN siempre encendida:** activar una VPN confiable antes de navegar en cualquier red que no sea la de casa, para que todo el tráfico viaje cifrado dentro de un túnel.
3. **Desconfiar de la red:** confirmar con el personal el nombre exacto del Wi-Fi para evitar un Evil Twin, desactivar la conexión automática y el uso compartido de archivos, y dejar los trámites sensibles (banco, compras) para los datos móviles.

## 7. Conclusión

La captura demuestra que con HTTP toda la conversación con el sitio se puede leer desde la misma red. HTTPS protege el contenido de cada sitio y una VPN agrega un túnel cifrado para todo el tráfico. Combinadas con buenos hábitos, reducen al mínimo el riesgo de usar una Wi-Fi pública.
