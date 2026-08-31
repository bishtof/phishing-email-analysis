# Caso: Phishing "Magic Win" vía Google Cloud Storage — Clúster de dominios de "domain aging"

**Autor:** SpainCheck Pro
**Fecha de análisis:** 31 de agosto de 2026
**Tipo:** Phishing / pig-butchering de casino falso, distribuido por email spam
**Estado:** Infraestructura activa en el momento del análisis, no reportada en blocklists públicas

---

## Resumen ejecutivo

Análisis OSINT de una campaña de phishing detectada en spam de Gmail, que ofrece un falso bono de bienvenida de €2.000 en un casino online ("Magic Win") para captar credenciales/datos personales. El seguimiento de la cadena de infraestructura reveló una operación con cierta escala: **7 dominios** desplegados sobre **al menos 6 IPs contiguas** en un rango de Leaseweb, más nodos adicionales en OVH (Francia) y Rumanía, todos sirviendo el mismo kit de "domain aging" (sitios que imitan contenido legítimo — en este caso, titulares reales del New York Times — para envejecer la reputación del dominio antes de activarlo para spam/phishing).

---

## Cadena de infección

1. **Email de spam** recibido en Gmail, asunto *"€2000 was credited to your account"*, remitente falsificado simulando ser la propia dirección del destinatario.
   - Enviado desde: `kalaman-facts.chipelike.com`
      - Contenido: señuelo de casino "Magic Win", bono de bienvenida de €2.000, promoción temática de Halloween, personalización del nombre/email del destinatario con enlaces de tracking individuales por campo.

      2. **Redirección vía bucket de Google Cloud Storage mal configurado** (listado público habilitado):
         ```
            https://storage.googleapis.com/dhkfjhbfbg/helloo22f.html
               ```
                  Contenido: script de una línea que reenvía a `projectually.net` con parámetros de tracking (`?od=...`).
                     El bucket contiene también 5 imágenes JPG (assets gráficos del kit) con nombres tipo ID de campaña (`11306_49216.jpg`, `42_14236_56697.jpg`).

                     3. **Landing final**: `projectually.net`, que sirve como sitio "aged" por defecto (feed de noticias del NYT) y presumiblemente como landing de captación cuando se accede con el parámetro de tracking correcto.

                     ---

                     ## Mapa de infraestructura

                     ### Dominios identificados (mismo kit / mismo patrón de registro)

                     | Dominio | Registrador | Registrante (WHOIS) | Email de contacto | IP(s) |
                     |---|---|---|---|---|
                     | chipelike.com | NameCheap | Harley Richard (WI, US) | techsupport@cougintrof.com | 164.132.40.127 |
                     | cosdmandola.com | NameCheap | Isaias Landry (CA, US) | domaintech@agilevsky.com | 209.58.144.208 |
                     | projectually.net | NameCheap | Tania Villa (MI, US) | domainadmin@miroknal.com | 144.79.119.154/155 |
                     | spluay.com | NameCheap | — | — | 86.107.246.19/63/105 |
                     | electinent.pro | NameCheap | (redactado, RDAP) | — | 209.58.144.208, 209.58.153.197–200 |
                     | marganish.com | — | — | — | 209.58.153.197 |
                     | controllegal.org | — | — | — | 209.58.153.200 |

                     **Patrón de registro:** identidad estadounidense genérica sin ciudad real + email de contacto en un dominio "empresa" distinto y desechable en cada registro — huella de registro automatizado/kit, no registros manuales independientes.

                     ### Bloque de IPs — Leaseweb (209.58.144.0/20)

                     ```
                     209.58.144.208
                     209.58.153.197
                     209.58.153.198  (sin dominio DNS asociado)
                     209.58.153.199  ← host analizado en profundidad
                     209.58.153.200
                     ```

                     Confirmado por SPF de `electinent.pro`, que declara las 5 IPs como emisores autorizados. Todas devuelven idéntica huella de servidor (Apache/2.4.52, PHP/7.4.33) y mismo hostname interno de imagen base (`tss7335` en los hosts comprobados con `-sV`).

                     ### Otros nodos de la operación

                     - **OVH (Francia)** — `164.132.40.127`: nginx 1.24.0, coexiste con `dev.military-zone.com` (ver nota sobre military-zone.com más abajo)
                     - **BTS Telecom (Rumanía)** — `86.107.246.19/63/105`: Apache 2.4.58, cert wildcard de vida corta (3 meses)
                     - **Destino final** — `144.79.119.154/155`: CentOS/PHP 7.4.33 (proveedor sin identificar en profundidad)

                     ### Huella técnica del kit (indicador de despliegue común)

                     - Mismo CSS servido en todos los hosts con **ETag y Last-Modified idénticos** (`ETag: "112a5-6257938f4a062"`, `Last-Modified: Sun, 27 Oct 2024 18:07:17 GMT`) → despliegue desde el mismo paquete de origen.
                     - Contenido HTML generado dinámicamente en PHP consumiendo un feed en vivo del New York Times, con auto-inyección del hostname/IP propio en todo el texto legal (disclaimer, privacy policy).
                     - `robots.txt` con `Disallow: /` en todos los nodos.

                     ---

                     ## Postura de seguridad de la infraestructura comprometida/alquilada

                     - Apache 2.4.52 y PHP 7.4.33 (EOL desde noviembre 2022), Sendmail 8.15.2 — sin parchear desde su lanzamiento (~2022), decenas de CVEs acumulados según InternetDB.
                     - **Sin cabeceras de seguridad** (no HSTS, CSP, X-Frame-Options, etc.) en ningún host comprobado.
                     - Sin filtraciones de archivos de configuración/backups (`.git`, `.env`, `wp-config.php`, etc. — todo 404).
                     - Puerto 22/SSH filtrado en todos los hosts comprobados (impide confirmar si son VMs clonadas literalmente o despliegues independientes del mismo kit).
                     - SMTP (Sendmail) en `209.58.153.199` soporta AUTH (DIGEST-MD5/CRAM-MD5); no se confirmó open-relay clásico — hipótesis más plausible: credenciales SMTP comprometidas o cron/script local invocando sendmail directamente, no relay abierto a terceros.

                     ---

                     ## Nota: military-zone.com (hallazgo colateral, sin relación con la campaña)

                     `military-zone.com` (y subdominios `dev.`, `api.`, `stg.`) comparte la IP de OVH (`164.132.40.127`) con parte del clúster, pero es una **aplicación real y activa**: SPA en React con motores Babylon.js/Cesium.js y librería `milsymbol` (símbolos militares NATO APP-6) — coherente con el título "Poligon" (polígono de tiro, en polaco) y el registrante en Polonia. Correo corporativo gestionado en OVH (`mx1/2/3.mail.ovh.net`), stack de desarrollo profesional (Sentry, Socket.io, MUI). Todo apunta a **coincidencia de hosting compartido**, no a participación en la operación de phishing.

                     ---

                     ## Indicadores de compromiso (IOCs)

                     ### Dominios

                     ```
                     chipelike.com
                     cosdmandola.com
                     projectually.net
                     spluay.com
                     electinent.pro
                     marganish.com
                     controllegal.org
                     ```

                     ### IPs

                     ```
                     164.132.40.127       (OVH, FR)
                     209.58.144.208        (Leaseweb, US)
                     209.58.153.197        (Leaseweb, US)
                     209.58.153.198        (Leaseweb, US)
                     209.58.153.199        (Leaseweb, US)
                     209.58.153.200        (Leaseweb, US)
                     144.79.119.154        (destino final)
                     144.79.119.155        (destino final)
                     86.107.246.19          (BTS Telecom, RO)
                     86.107.246.63          (BTS Telecom, RO)
                     86.107.246.105         (BTS Telecom, RO)
                     ```

                     ### URLs / recursos

                     ```
                     https://storage.googleapis.com/dhkfjhbfbg/helloo22f.html
                     http://projectually.net/
                     ```

                     ### Emails de contacto de registro (huella de kit)

                     ```
                     techsupport@cougintrof.com
                     domaintech@agilevsky.com
                     domainadmin@miroknal.com
                     ```

                     ---

                     ## Conclusión

                     No se trata de un envío de spam aislado, sino de una **operación de "domain aging as a service"** con escala real: múltiples dominios, múltiples proveedores de hosting (OVH, Leaseweb, BTS Telecom), kit de despliegue reutilizable y automatizado, y un patrón de registro de dominios consistente. La infraestructura no aparece todavía en blocklists públicas (VirusTotal/URLhaus) en el momento del análisis, lo que sugiere una operación relativamente reciente o con rotación activa para evadir detección.

                     ---

                     *Análisis realizado con herramientas OSINT estándar (whois, dig, nmap, curl) — reconocimiento pasivo y semi-activo únicamente, sin intentos de acceso no autorizado a ningún sistema.*
                     
