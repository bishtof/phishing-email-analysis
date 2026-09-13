# Caso: Spam/Phishing "Lucky Creek Casino" — SPF-as-a-Service + Red de Afiliados CPA

**Analista:** SpainCheck Pro
**Fecha de detección:** 13 de septiembre de 2026
**Vector:** Email (spam masivo, capturado en carpeta de Spam)
**Categoría:** Publicidad ilegal de casino online / abuso de red de afiliados CPA

---

## 1. Resumen ejecutivo

Se recibió un correo de spam suplantando remitente propio ("[destinatario]"), anunciando un bono de bienvenida de un casino online ("Lucky Creek") con código promocional. El análisis de cabeceras, DNS e infraestructura revela una cadena de al menos **cinco saltos** diseñada para evadir filtros antispam, sandboxes de análisis automatizado y herramientas de investigación (incluido Tor), monetizada a través de una **red de afiliados CPA (Cost-Per-Action)**.

No se trata de un ataque dirigido: es una operación de **spam masivo con técnicas de evasión de nivel medio-alto**, propia de un afiliado de marketing que opera a escala, no de un actor que persigue específicamente a la víctima.

---

## 2. Cadena de ataque reconstruida

```
[1] Email spoofed
    From: "[destinatario]" <xnrsupporthvv@hapjwnbruntzoqekfhwiiwlh.com>
    (dominio inexistente — spoofing puro en el campo From)
        │
        ▼
[2] Dominio de envío real (Return-Path)
    myrmecophagous-plethysmograph.airportance.uk
    SPF gestionado vía Valimail Instant SPF (SaaS legítimo de terceros)
    IP de envío: 185.174.29.68 (rDNS: media.bmofg.com — probable resto huérfano)
        │
        ▼
[3] Enlace de clic ("PLAY NOW")
    storage.googleapis.com/69clck87terrasa/alectormancy.html#<token>
    Geobloqueado contra tráfico Tor/datacenter ("not available in your location")
        │
        ▼
[4] Redirector desechable
    http://drilarif.org/<token>
    Apache 2.4.52 + PHP 7.4.33 (Ubuntu) — script de redirección simple
        │
        ▼
[5] Red de afiliados CPA
    https://Fwave.o18a.com/c?o=...&m=...&a=...&aff_sub1=...&aff_sub2=...&aff_sub3=...
    Tras proxy Cloudflare (IP de origen no expuesta por diseño)
        │
        ▼
[destino final no confirmado — landing de casino, servido tras el tracking CPA]
```

---

## 3. Indicadores de compromiso (IOCs)

| Tipo | Valor | Rol |
|---|---|---|
| Dominio (spoof, no registrado) | `hapjwnbruntzoqekfhwiiwlh.com` | Suplantación en campo `From:` |
| Dominio (real, envío) | `airportance.uk` | Dominio raíz, registrado 01-oct-2024 (Namecheap) |
| Subdominio de campaña | `myrmecophagous-plethysmograph.airportance.uk` | SPF dinámico vía Valimail |
| IP de envío | `185.174.29.68` | Corelux Ltd, Turquía — AS51559 |
| IP de cobertura SPF | `89.116.156.223` | Lithuanian Radio and TV Center, Lituania — AS46475 |
| Bucket de redirección | `storage.googleapis.com/69clck87terrasa/` | Alojamiento intermedio, geobloqueado |
| Dominio redirector | `drilarif.org` | Registrado 04-may-2026 (Namecheap), WHOIS redactado |
| IP del redirector | `5.199.136.89` | — |
| Red de afiliados | `o18a.com` / `Fwave.o18a.com` | Tras Cloudflare (NS: karl/lana.ns.cloudflare.com) |

---

## 4. Hallazgos técnicos destacados

### 4.1 SPF-as-a-Service (Valimail) como cobertura de spam
El subdominio de campaña usa un registro SPF con macros de expansión dinámica:
```
v=spf1 a:%{i}.secured.%{h} include:%{i}._ip.%{h}._ehlo.%{d}._spf.vali.email redirect=%{h} ~all
```
Esto corresponde al servicio legítimo **Valimail Instant SPF**, normalmente usado por empresas para gestionar SPF sin listar IPs fijas en DNS. El operador de la campaña lo emplea para poder rotar IPs de envío sin modificar el DNS del dominio raíz cada vez, manteniendo la reputación del dominio principal (`airportance.uk`, con 2 años de antigüedad) mientras los subdominios de campaña absorben el desgaste.

### 4.2 Domain aging
`airportance.uk` se registró en octubre de 2024 — casi dos años antes de su uso detectado en esta campaña. Es una técnica deliberada para evitar la desconfianza que los filtros antispam aplican a dominios recién creados.

### 4.3 Geobloqueo anti-análisis
El bucket de Google Cloud Storage devuelve `AccessDenied` / *"not available in your location"* cuando se accede vía Tor o con ciertos user-agents/IPs de datacenter, pero sirve el contenido normalmente desde una IP residencial. Es una técnica de evasión activa contra sandboxes automatizados e investigadores.

### 4.4 Reparto geográfico de infraestructura
El envío se divide entre Turquía (IP real) y Lituania (cobertura SPF), dificultando que un único proveedor o autoridad de abuso pueda cortar toda la operación de una vez.

### 4.5 Monetización vía red de afiliados CPA
El destino final no es el casino directamente, sino una plataforma de tracking de afiliados (`o18a.com`) con parámetros `o=`, `m=`, `a=` (oferta/medio/afiliado) y tres `aff_sub` de subseguimiento — el modelo de negocio típico de "aff hackers" que envían spam masivo para cobrar comisión por cada clic/registro/depósito que logran derivar a ofertas de casino, sin que el propio casino gestione el envío.

### 4.6 Relleno de texto ("text stuffing") con contenido ofensivo
El cuerpo HTML del correo contiene, oculto en comentarios y bloques no renderizados, fragmentos reciclados de al menos 6-7 plantillas de spam no relacionadas (Enterprise Rent-A-Car, YNAB, IBM Cloud, Podio, Fastly, Parsec, Wistia) y un bloque de texto con contenido antisemita usado como relleno para evadir filtros bayesianos de spam. Esto confirma el uso de un kit de spam genérico y reciclado, no una plantilla hecha a medida.

---

## 5. Autenticación del correo original

| Mecanismo | Resultado |
|---|---|
| SPF | PASS (para el dominio de envío, no para el `From:` visible) |
| DKIM | FAIL (`permerror` — sin clave para un dominio inexistente) |
| DMARC | FAIL |

---

## 6. Limitaciones de la investigación

- La IP de origen real de `o18a.com` / `Fwave.o18a.com` no pudo determinarse: el dominio está detrás de Cloudflare, que por diseño oculta la IP del servidor de origen frente a resolución DNS pasiva.
- - El rDNS `media.bmofg.com` sobre `185.174.29.68` corresponde a un dominio corporativo legítimo (CSC Corporate Domains + Akamai) sin relación aparente con la campaña — probablemente un registro PTR huérfano de una reasignación de IP anterior, no una pista útil.
  - - El destino final tras la red de afiliados (la landing real del casino) no se confirmó, para evitar exponer la máquina de análisis a contenido activo no verificado.
   
    - ---

    ## 7. Metodología

    Investigación realizada con herramientas OSINT estándar (`whois`, `dig`, `curl`, `nmap`) desde Kali Linux (WSL), con tráfico enrutado a través de Tor (`torsocks` / `proxychains4`) en las fases de contacto directo con la infraestructura del atacante, para evitar exponer la IP del investigador. Se usó DNS-over-HTTPS (Cloudflare) como alternativa cuando la resolución DNS clásica no era compatible con el proxy SOCKS.

    **Nota metodológica:** en ningún momento se accedió a la landing final del casino ni se interactuó con formularios, y todas las peticiones HTTP se limitaron a obtención de cabeceras y HTML estático sin ejecución de scripts del lado del atacante.

    ---

    ## 8. Recomendaciones

    1. Reportar abuso a los contactos identificados:
    2.    - `report@abuseradar.com` (rango 89.116.156.0/24, Lituania)
          -    - `abuse@corelux.com.tr` (rango 185.174.29.0/24, Turquía)
               -    - `abuse@namecheap.com` (registrador de `airportance.uk` y `drilarif.org`)
                    - 2. No interactuar con el enlace desde ningún dispositivo con sesión de correo o navegador activa.
                      3. 3. Marcar como spam/phishing en el proveedor de correo (ya realizado).
                         4. 
