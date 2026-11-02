# Informe de análisis de correo electrónico sospechoso — "ArterioFlush"

**Analista:** SpainCheck Pro
**Fecha de recepción:** 31 de octubre de 2026
**Fecha del análisis:** 1 de noviembre de 2026
**Destinatario del correo:** [dirección anonimizada — cuenta personal de Gmail]
**Clasificación previa:** Spam (Gmail)

> **Aviso.** Análisis basado en fuentes abiertas (cabeceras, DNS, RDAP, certificados) y en peticiones mínimas enrutadas por Tor. No se resolvió el captcha, no se introdujeron datos ni se realizó ningún pago. El mensaje original (`.eml`) se conserva en privado y no se publica. Los indicadores se muestran en formato de código para que no sean enlaces pulsables: no los copies en un navegador.

---

## 1. Resumen ejecutivo

El 31/10/2026 llegó a un buzón personal un correo que suplanta a **CBS News** y al **doctor Oz** para vender un suplemento llamado **«ArterioFlush»**. Usa testimonios falsos e insta a dejar la medicación antihipertensiva, lo que añade un **riesgo sanitario** al fraude comercial.

La infraestructura se reparte entre Namecheap, velia.net, Hivelocity, SERVERHUB y un pool de unas 30 IPs de hosters de bajo coste. El hallazgo técnico más llamativo es un **registro CNAME aparentemente erróneo en la zona de `comcast.net`** que apunta a un dominio ajeno y permite que las IPs de ese pool se presenten con un DNS inverso (PTR) que dice ser de Comcast.

No consta pérdida económica: no se pagó ni se introdujeron datos. Como no se resolvió el captcha de la página de destino, se desconocen la oferta final y el procesador de pago.

---

## 2. Datos del mensaje

| Campo | Valor |
|---|---|
| Fecha de entrega | 31/10/2026 |
| De (visible) | Dominio `mzqekgkjohmiqztsniuidjgp.com` (no registrado, NXDOMAIN) |
| Para | [dirección anonimizada] |
| Return-Path (envelope) | Subdominio de `fluctance.net` |
| HELO | `obesegrab.net` |
| IP de origen | `209.239.115.34` |
| PTR de la IP | `wholesale-vod-demo-1.comcast.net` (falseado, ver apartado 5) |
| Marca suplantada | CBS News / doctor Oz |

---

## 3. Resultado de autenticación de correo

| Mecanismo | Resultado | Interpretación |
|---|---|---|
| **SPF** | PASS (subdominio de `fluctance.net`) | El servidor emisor está autorizado por la infraestructura propia del operador; no es señal de legitimidad. El SPF de `obesegrab.net` autoriza además las IPs terminadas en `.34`, `.36` y `.38`. |
| **DKIM** | Sin clave | La firma referencia el dominio del From, `mzqekgkjohmiqztsniuidjgp.com`, que no existe, por lo que no hay clave pública que verificar. |
| **DMARC** | No consta en el informe | — |

La cabecera `X-Google-Sender-Delegation` aparece en el mensaje, pero no está cubierta por la firma DKIM (ver hipótesis en el apartado 9).

---

## 4. Cómo funciona el ataque (cadena de redirección)

```
[1] Botón del correo
    https://storage.googleapis.com/xcvdcxcd/nwdllrd.html#<parámetro personalizado>
    │
    ▼
[2] Redirector en Google Cloud Storage (107 bytes)
    Una línea de JavaScript copia lo que hay tras la # y lo añade a choracetamatoes.com
    │
    ▼
[3] choracetamatoes.com (Apache 2.4.52, PHP 7.4.33)
    Responde 200 con una falsa página de «Unsubscribe» protegida por reCAPTCHA
    Con un token inválido devuelve 404
    │
    ▼
[4] Destino real (tienda del suplemento) — no obtenido
```

**Por qué le sirve al atacante:**

- Alojar el redirector en `storage.googleapis.com` le presta la reputación de un dominio de confianza frente a filtros de reputación y DNSBL.
- La parte tras la `#` se procesa en el navegador y no viaja al servidor, así que un `curl` normal no sigue la cadena: hay que leer el HTML del redirector.
- El reCAPTCHA impide que escáneres automáticos vean el destino final.
- El parámetro tras la `#` identifica a cada destinatario. Abrir el enlace desde una conexión propia confirma al operador que el buzón está activo.

---

## 5. Hallazgo técnico destacado: el truco del DNS inverso

El servidor autoritativo de Comcast (`dns101.comcast.net`) devuelve este CNAME:

```
wholesale-vod-demo-1.comcast.net  CNAME  pmd205544tn.download.theplatform.comedgesuite.net
```

Parece una errata en la zona (falta un punto), de modo que el nombre acaba colgando de `comedgesuite.net`, un dominio que **no es de Comcast** y está registrado por un tercero (Namecheap, 22/02/2025). Ese dominio tiene un wildcard en el apex y resuelve a un pool de unas 30 IPs.

Resultado: una IP del pool puede tener como PTR `wholesale-vod-demo-1.comcast.net`, y la resolución directa de ese nombre (vía el CNAME) vuelve al pool. La comprobación de DNS inverso confirmado parece entonces superada con un nombre de Comcast, lo que da apariencia de legitimidad al enviar correo.

**Pool de IPs con PTR de Comcast:** unas 30 IPs repartidas entre Leaseweb, ColoCrossing/HostPapa, OVH, ISPpro, velia.net, SERVITRO, Starry Network, Hetzner, ServerPoint, MysticDev, gmhost y Contabo. Solo `209.239.115.34` consta como emisora de este correo.

---

## 6. Indicadores de ingeniería social

- Suplantación de un medio real (CBS News) y de una figura pública conocida (doctor Oz) para dar credibilidad.
- Testimonios falsos.
- Insta a dejar la medicación antihipertensiva, nombrándola: el riesgo no es solo económico, sino sanitario.

---

## 7. Relleno anti-filtrado

El bloque final del correo incluye texto con marcas como Enterprise, YNAB, IBM, Podio, Fastly o Parsec. Es relleno anti-Bayes para diluir la huella de spam ante los filtros. **Esas marcas no están implicadas en la campaña.**

---

## 8. Descarte y evaluación de vectores de riesgo

| Vector | ¿Presente? | Justificación |
|---|---|---|
| Suplantación de marca real | Sí | CBS News y el doctor Oz |
| Riesgo sanitario | Sí | El correo desaconseja medicación antihipertensiva por nombre |
| Seguimiento por destinatario | Sí | Parámetro personalizado tras la `#` del enlace |
| Phishing de credenciales / formulario de pago | No verificado | El destino tras el reCAPTCHA no se obtuvo |
| Pérdida económica | No consta | No se pagó ni se introdujeron datos |

---

## 9. Confirmado frente a hipótesis

Lo confirmado procede de cabeceras, DNS y RDAP guardados. Las hipótesis **no están verificadas** y no deben citarse como hechos en una denuncia.

### Confirmado

- El correo suplanta a CBS News y al doctor Oz, y desaconseja medicación antihipertensiva por nombre.
- La IP `209.239.115.34` envió el mensaje (SPF pass para el subdominio de `fluctance.net`; DKIM sin clave).
- El CNAME de Comcast apunta a `comedgesuite.net`, registrado el 22/02/2025 en Namecheap, que resuelve a unas 30 IPs; unas 25 tienen el PTR de Comcast.
- El redirector de Google Storage solo reenvía a `choracetamatoes.com`, que sirve un captcha.
- Cuatro dominios están en Namecheap con NS `registrar-servers.com`: `choracetamatoes.com`, `fluctance.net`, `obesegrab.net` y `comedgesuite.net`.

### Hipótesis (sin verificar)

- El CNAME de Comcast es una errata que el operador aprovechó. Falta el histórico DNS para saber cuándo apareció y quién lo escribió.
- SERVITRO LTD (AS213495) es una línea de investigación: comparte el bloque `5.175.234.0/24` con `fluctance.net` y su AS se creó un mes antes que `comedgesuite.net`. No hay pruebas de que pertenezca al operador.
- Las IPs del pool pueden ser servidores del operador o clientes de hosters con un rDNS ajeno. Solo la `.34` consta como emisora.
- `170.130.200.114` solo consta como host de correo de `comedgesuite.net`; no hay spam enviado desde ella.
- La cabecera `X-Google-Sender-Delegation` parece inyectada por el spammer para ganar confianza; no está en la firma DKIM.
- Compartir registrador (Namecheap) y fechas es una pista débil, porque es un registrador muy grande.

No hay vínculo con otros casos del repositorio: ni las IPs ni los ASN del pool coinciden.

---

## 10. Cronología

| Fecha | Hecho |
|---|---|
| 12/04/2024 | Registro de `fluctance.net` (Namecheap) |
| 06/09/2024 | Registro de `choracetamatoes.com` (Namecheap) |
| 22/02/2025 | Registro de `comedgesuite.net` (Namecheap) |
| 23/05/2025 | Registro de `obesegrab.net` (Namecheap) |
| 20/08/2026 | Certificado wildcard de `choracetamatoes.com` (Let's Encrypt) |
| 29/10/2026 | Subida del redirector al bucket de Google Storage |
| 31/10/2026 | Recepción del correo; el proveedor lo clasifica como spam |
| 01/11/2026 | Análisis pasivo y peticiones mínimas por Tor |

---

## 11. Indicadores de compromiso (IOCs)

| Función | Indicador | Titular / proveedor | Notas |
|---|---|---|---|
| IP emisora | `209.239.115.34` | velia.net (bloque `209.239.112.0/20`, cliente privado) | HELO `obesegrab.net`; PTR `wholesale-vod-demo-1.comcast.net` (falseado) |
| Dominio envelope | `fluctance.net` | Namecheap | Return-Path; A en `5.175.234.81` |
| Dominio HELO/SPF | `obesegrab.net` | Namecheap | SPF autoriza `.34`, `.36` y `.38`; wildcard `*.secured.obesegrab.net` hacia `.34` |
| Tracker / landing | `choracetamatoes.com` (`199.231.167.38`) | Namecheap; Hivelocity | Puerta reCAPTCHA; MX en Google Workspace |
| Dominio del CNAME | `comedgesuite.net` (`170.130.200.114`) | Namecheap; SERVERHUB | Wildcard en el apex; sirve el pool de IPs |
| Redirector | `https://storage.googleapis.com/xcvdcxcd/nwdllrd.html` | Google Cloud Storage | Subido el 29/10/2026 |
| Imagen enlazada | `https://i.imgur.com/xPHiPxr.jpg` | Imgur | Enlazada al tracker |
| From falso | `mzqekgkjohmiqztsniuidjgp.com` | No registrado (NXDOMAIN) | DKIM sin clave |

**Otros indicadores:**

- Site key de reCAPTCHA: `6Lf2Q_oUAAAAAPuHL-4TbwHMZajtdrTxLXEPfdeU` (útil para buscar otros dominios del mismo kit)
- Ruta de assets de la plantilla: `/assets/ayt/css/main.css`
- Token de verificación de Search Console del atacante: `JaDlI8P8VcFEh9nMSNhEFFjjxn8ubnvePXSsyO_Q18`

**Hashes de la evidencia (SHA256):**

| Fichero | SHA256 |
|---|---|
| Redirector (`nwdllrd.html`) | `0641a04a948dc1ca9915ee20d4f304e5b188966dbb60ec3969e6920a76bea804` |
| Página de destino (`hop1.html`) | `f87df3403bb98aceb5f3c1695c918deb02131da616d5c1e51048cb34212bea40` |

Solo se publican los hashes de ficheros que son iguales para cualquier visitante. El mensaje original y las cabeceras de las peticiones se conservan en privado.

---

## 12. Notificaciones

A 01/11/2026 los reportes están **redactados y pendientes de envío**. Ninguno consta como enviado. El orden prioriza a quien puede cortar el DNS y el envío.

| Orden | Destinatario | Objeto | Estado |
|---|---|---|---|
| 1 | Namecheap | 4 dominios | Redactado, pendiente de envío |
| 2 | Comcast (canal de seguridad) | CNAME erróneo en `comcast.net` | Redactado, pendiente de envío |
| 3 | velia.net | `209.239.115.34` y `92.204.97.212` | Redactado, pendiente de envío |
| 4 | Hivelocity | `199.231.167.38` | Redactado, pendiente de envío |
| 5 | SERVERHUB / Eonix | `170.130.200.114` | Redactado, pendiente de envío |
| 6 | INCIBE-CERT | Campaña completa con cabeceras | Pendiente |
| 7 | Google Cloud | Bucket `xcvdcxcd` | Pendiente |
| 8 | Imgur | Imagen enlazada | Pendiente |
| 9 | Hosters del pool | IPs con PTR de Comcast | Opcional; evidencia más débil |

---

## 13. Conclusión y qué aprender de este caso

El correo corresponde a una campaña de **spam de afiliación de suplementos con suplantación de marca** que, además del fraude comercial, supone un riesgo para la salud al animar a abandonar un tratamiento médico. Su infraestructura combina un redirector en una nube de confianza, una puerta reCAPTCHA contra el análisis automatizado y un CNAME erróneo en una zona ajena que presta un PTR de apariencia legítima.

- Los redirectores en nubes de confianza (Google Storage) esquivan filtros por reputación.
- La `#` de una URL se resuelve en el navegador: por eso un `curl` no basta y hay que leer el HTML.
- Un PTR no es prueba de identidad: un CNAME mal puesto en una zona ajena puede falsearlo.
- El spam sanitario que insta a dejar medicación es un riesgo real, no solo una molestia.

**Recomendación:** mantener el correo en Spam, marcarlo como "Denunciar phishing", no abrir el enlace del botón ni el de "Unsubscribe" desde ninguna conexión propia y no dejar ningún tratamiento médico por un correo publicitario: cualquier cambio de medicación debe consultarse con un profesional sanitario.

---

*Informe generado como parte del análisis OSINT rutinario de correo entrante — SpainCheck Pro.*
