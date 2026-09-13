# Phishing Email Analysis

Análisis OSINT de correos de phishing y spam malicioso recibidos en cuentas personales — cadenas de infraestructura, indicadores de compromiso (IOCs) y técnicas de evasión, documentados con metodología reproducible.

Cada caso sigue el mismo proceso: detección en bandeja de Spam → análisis de cabeceras y autenticación (SPF/DKIM/DMARC) → trazado de la infraestructura de destino (dominios, IPs, redirectores) → clasificación del vector de riesgo → recomendaciones. El análisis se limita en todo momento a reconocimiento pasivo y semi-activo (whois, dig, curl, nmap), sin interacción con formularios ni ejecución de contenido del atacante.

---

## Índice de casos

| Caso | Fecha | Vector | Hallazgo principal |
|---|---|---|---|
| [Lucky Creek Casino](casos/phishing-lucky-creek-2026-09/README.md) | Sep 2026 | Email spoofed → casino falso | Cadena de 5 saltos con SPF-as-a-Service (Valimail) y monetización vía red de afiliados CPA |
| [Magic Win](casos/phishing-magic-win-2026-08/README.md) | Ago 2026 | Email spoofed → casino falso | Clúster de 7 dominios sobre Leaseweb/OVH/Rumanía con kit de "domain aging" reutilizable |
| [Scareware almacenamiento en la nube](casos/phishing-scareware-cloud-storage-2026-08/README.md) | Ago 2026 | Alerta falsa de "archivos eliminándose" | DKIM válido pero DMARC en fallo por desalineación de dominio; ingeniería social basada en pánico |
| [Cheeky Wins](casos/phishing-cheeky-wins-2026-08/README.md) | Ago 2026 | Spam de afiliación de casino | Snowshoe spamming sobre TLD .us, sin evidencia de phishing dirigido |

Cada carpeta de caso incluye su propio informe completo (`README.md`) con datos del mensaje, resultado de autenticación, mapa de infraestructura, IOCs y conclusión.

---

## Metodología

1. **Recepción y clasificación previa** — correos ya filtrados como Spam por el proveedor (Gmail), nunca interactuados desde la bandeja principal.
2. **Análisis de cabeceras** — extracción de remitente real, IP de origen, resultado SPF/DKIM/DMARC y ruta de entrega.
3. **Análisis de infraestructura** — resolución DNS, WHOIS de dominios, geolocalización de IPs, fingerprinting de servidores (versión de software, certificados, cabeceras HTTP).
4. **Clasificación del vector** — se descarta o confirma phishing dirigido, suplantación de marca real, compromiso de cuenta y captura de credenciales.
5. **Documentación** — informe estructurado con tablas de IOCs reutilizables para reporte de abuso o correlación con otros casos.

Herramientas: `whois`, `dig`, `curl`, `nmap`, resolución DNS-over-HTTPS y, cuando es necesario evitar exponer la IP del investigador, `torsocks`/`proxychains4`.

---

## Alcance y aviso ético

Este repositorio documenta **análisis defensivo y pasivo** de correos ya recibidos por el propio autor. No se realizan intentos de explotación, acceso no autorizado, ni interacción con formularios de captura de credenciales de ningún sistema de terceros. Los datos de contacto y direcciones de correo de los destinatarios se anonimizan en todos los informes. El objetivo es exclusivamente educativo y de concienciación sobre patrones de phishing/spam reales.

---

## Autor

Sebastian | Cybersecurity Student
