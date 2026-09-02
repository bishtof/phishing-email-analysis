# Caso Magic Win — Actualización 2026-09-01

**Investigador:** SpainCheck Pro
**Caso original:** phishing-magic-win-2026-08 (chipelike.com / projectually.net)
**Esta sesión:** continuación 24h después, foco en atribución de infraestructura y fingerprinting del kit

---

## 1. Resumen ejecutivo

Un día después del cierre del informe original, se repitió la investigación sobre el clúster de 7 dominios para verificar estabilidad de la infraestructura y buscar atribución adicional. Se confirma que la operación **rota infraestructura de forma parcial y frecuente** (un nodo desactivado en <24h), se identifica un **nuevo nodo en China** no documentado previamente, y se obtiene un **fingerprint fiable del kit** (favicon MD5 idéntico + PHP 7.4.33 constante) que permite futura correlación de nuevos dominios sin depender del WHOIS.

También se confirma una técnica de **cloaking sistemático**: la raíz de cada dominio sirve una fachada de noticias (scraper dinámico de NYT), mientras que `/index.php` sirve un formulario de captura de email — aunque este último resultó ser decorativo (`action="#"`, sin JS de envío), no funcional.

---

## 2. Estado de infraestructura (comparativa 24h)

| Dominio | IP documentada ayer | IP hoy | Cambio | Estado del vhost |
|---|---|---|---|---|
| chipelike.com | 164.132.40.127 (OVH, FR) | 164.132.40.127 | Sin cambio de DNS | **Desactivado** — el servidor ya no sirve el vhost del kit; responde con el vhost por defecto (`dev.military-zone.com`, vecino inocente) tanto en HTTP como HTTPS con SNI forzado |
| projectually.net | — | **144.79.119.154** | **Nuevo nodo** | Activo — cert `*.projectually.net` renovado 17-ago-2026 |
| cosdmandola.com | 209.58.144.208 (Leaseweb, US) | 209.58.144.208 | Sin cambio | Activo — **certificado caducado** desde 20-abr-2026 (4 meses sin renovar) |
| spluay.com | 86.107.246.19 (Rumanía) | 86.107.246.19 | Sin cambio | Activo |
| marganish.com | 209.58.153.197 | 209.58.153.197 | Sin cambio | No verificado en detalle esta sesión |
| electinent.pro | 209.58.144.208 | 209.58.144.208 | Sin cambio | Comparte IP con cosdmandola.com |
| controllegal.org | 209.58.153.200 | 209.58.153.200 | Sin cambio | No verificado en detalle esta sesión |

**Lectura:** el bloque Leaseweb (209.58.144.208 / .153.197 / .153.200 / .153.199) se mantiene estable — es la columna vertebral de la operación. El nodo OVH (chipelike) se apagó sin liberar el DNS. Aparece un nodo nuevo en China que no formaba parte del mapeo original.

---

## 3. Nuevo nodo — Shenzhen, China

- **IP:** 144.79.119.154 (sirve `projectually.net`, el landing final del embudo)
- **Bloque:** 144.79.118.0 – 144.79.119.255 (CIDR 144.79.0.0/16, legacy APNIC re-anunciado bajo CNNIC)
- **Organización:** GUANGYUN (SHENZHEN) CO., LTD.
- **Stack:** Apache/2.4.6 (CentOS), PHP 7.4.33
- **Reverse DNS:** sin PTR (habitual en hosting chino de bajo coste)
- Amplía la huella geográfica de la operación más allá de EE.UU./Francia/Rumanía documentada originalmente.

---

## 4. Fingerprint del kit (IOC reutilizable)

| Indicador | Valor |
|---|---|
| Favicon MD5 | `c6acedaff906029fc5455d9ec52c7f42` — idéntico byte a byte en cosdmandola.com, projectually.net y spluay.com |
| Favicon MurmurHash3 (formato Shodan) | `1274734426` — pendiente de consultar en `http.favicon.hash:1274734426` (sin acceso a Shodan en esta sesión) |
| Versión PHP constante | `7.4.33` en los 3 nodos verificados, pese a distintos SO base (Ubuntu/CentOS) y distintas versiones de Apache |
| Plantilla base | **"Introspect" by TEMPLATED** (templated.co), licencia Creative Commons Attribution 3.0 — plantilla legítima y gratuita reutilizada como fachada visual |
| Carpeta de assets | `/assets/ayt/` — sin significado operativo identificado, cosmético |
| robots.txt | `Disallow: /` en todos los nodos verificados — bloqueo deliberado de indexación |

---

## 5. Cloaking por ruta (hallazgo nuevo de esta sesión)

Comportamiento idéntico confirmado en `cosdmandola.com`, `spluay.com` y `projectually.net`:

- **`https://<dominio>/`** → fachada de noticias, scraper dinámico de titulares del New York Times en tiempo real (confirmado con artículos del mismo día de la consulta), organizados por categoría (ej. `feed.php` sirve subsección "NYT > Technology")
- **`https://<dominio>/index.php`** → formulario "SUBMIT YOUR APPLICATION NOW!" con campo de email y botón "Subscribe"

**Verificación de funcionalidad del formulario:** `action="#"`, `method="post"`, sin JavaScript de intercepción de submit. Es decorativo — un envío real no se transmite a ningún backend visible. Forma parte del atrezzo heredado de la plantilla, no está activo como mecanismo de captura.

**Interpretación:** el patrón sugiere una arquitectura de "gate" — la ruta raíz existe para superar inspección automática (crawlers, escáneres de reputación, revisores humanos casuales), mientras el tráfico dirigido desde las campañas de email probablemente use rutas o parámetros de tracking distintos (ver hallazgo del caso paralelo Lussurio/TrueFortune del mismo día: redirección vía fragmento `#od=...` no interpretado por curl).

---

## 6. Identidades WHOIS — confirmación de generador automático

Comparativa de los 7 dominios registrados vía Namecheap:

| Dominio | Fecha de creación | Nombre de registrante | Email de contacto (dominio ficticio) |
|---|---|---|---|
| chipelike.com | 2024-11-07 | Harley Richard | techsupport@cougintrof.com |
| cosdmandola.com | 2025-03-10 | Isaias Landry | domaintech@agilevsky.com |
| spluay.com | 2026-01-26 | Ilene Ankunding | danika58@cavitamily.me |
| projectually.net | 2024-10-09 | Tania Villa | domainadmin@miroknal.com |
| marganish.com | 2024-03-07 | Freddy Mcdonald | abusecomplaints@combingership.info |
| controllegal.org | 2025-04-19 | — | — |
| electinent.pro | — | — | — |

**Verificación DNS de los 5 dominios de contacto:** los cinco (`cougintrof.com`, `agilevsky.com`, `cavitamily.me`, `miroknal.com`, `combingership.info`) devuelven **NXDOMAIN** — nunca han existido en DNS. Confirma que son cadenas generadas automáticamente en el momento del registro, no identidades reales ni siquiera reutilizadas manualmente. Rango de fechas de registro: marzo 2024 – enero 2026 (casi 2 años de operación continua, no una compra en lote única).

---

## 7. Historial (Wayback Machine)

- `cosdmandola.com`: **un único snapshot**, 11-jul-2025 (4 meses después del registro, 6 meses antes de la activación como phishing). Confirma la estrategia de "domain aging" — el dominio se mantuvo deliberadamente fuera de radar, sin generar tráfico ni backlinks, antes de su activación.

---

## 8. Hilos cerrados sin hallazgo adicional

- MySQL (3306/tcp) en el nodo OVH: puerto abierto a nivel TCP pero handshake filtrado — no explotable trivialmente.
- Paths comunes expuestos (`config.php`, `.git/HEAD`, `.env`, `admin/`, etc.): 404 limpio en todos — despliegue del kit sin restos obvios de archivos sensibles.
- `README.md` / `LICENSE` de la carpeta de assets: 404 — el operador eliminó los ficheros de la plantilla original al desplegar.
- Reverse-IP sobre el nodo chino: sin resultados en hackertarget (posible limitación del servicio, no necesariamente ausencia de más dominios).

---

## 9. Pendiente para próxima sesión

- Consultar el hash de favicon (`1274734426`) en Shodan (`http.favicon.hash:1274734426`) cuando haya acceso — puede revelar dominios/IPs adicionales del mismo operador no descubiertos por WHOIS/DNS.
- Verificar estado actual de `marganish.com`, `electinent.pro`, `controllegal.org` con el mismo nivel de detalle (favicon, PHP, cloaking por ruta) que los otros tres.
- Seguir el hash de fragmento `#od=...` del caso Lussurio/TrueFortune (sesión paralela del mismo día) para confirmar si comparte generador de tracking con Magic Win.
- Reintentar VirusTotal con API key válida para contrastar reputación de todos los IOCs.

---

## 10. IOCs consolidados de esta sesión

```
Dominios activos:
  cosdmandola.com    -> 209.58.144.208 (Leaseweb US)
  spluay.com         -> 86.107.246.19  (Rumanía)
  projectually.net   -> 144.79.119.154 (Guangyun Shenzhen, China)
  marganish.com      -> 209.58.153.197 (Leaseweb US)
  electinent.pro     -> 209.58.144.208 (Leaseweb US)
  controllegal.org   -> 209.58.153.200 (Leaseweb US)

Dominio desactivado:
  chipelike.com      -> 164.132.40.127 (OVH FR) [DNS vivo, vhost apagado]

Dominios de contacto WHOIS falsos (NXDOMAIN confirmado):
  cougintrof.com, agilevsky.com, cavitamily.me, miroknal.com, combingership.info

Favicon:
  MD5:    c6acedaff906029fc5455d9ec52c7f42
  MMH3:   1274734426

Fingerprint técnico:
  PHP 7.4.33 (constante en todos los nodos activos verificados)
  Plantilla: Introspect by TEMPLATED (templated.co, CC-BY 3.0)
```
