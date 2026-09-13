# Informe de análisis de correo electrónico sospechoso

**Analista:** SpainCheck Pro
**Fecha del análisis:** 13 de septiembre de 2026
**Destinatario del correo:** [dirección anonimizada — cuenta personal de Gmail]
**Clasificación previa:** Spam (Gmail)

---

## 1. Resumen ejecutivo

Se ha analizado un correo electrónico recibido en la bandeja de Spam de Gmail, con asunto **"Most men are missing these 7"**, que promociona un suplemento de "energía masculina" mediante marketing de afiliación (afiliado a comisión, no venta directa). El análisis de cabeceras confirma **suplantación del campo "From" visible** (marcado por Gmail como enviado a través de un remitente de confianza delegado, técnica de "sender delegation" que oculta el remitente técnico real), **fallo de DKIM por clave inexistente** y **fallo de DMARC**, junto con relleno masivo de texto ("hashbusting"/"text stuffing") reciclado de al menos 4-5 fuentes no relacionadas para evadir filtros bayesianos de spam.

No se trata de un ataque dirigido: es una campaña de **spam de afiliación de suplementos ("male enhancement")**, uno de los verticales de spam más antiguos y persistentes en volumen, distribuido mediante infraestructura de envío dedicada y desechable.

---

## 2. Datos del mensaje

| Campo | Valor |
|---|---|
| ID de mensaje | `<UCNCY.HFZQM.qmail@localhost.localdomain>` |
| Fecha de creación | 13/09/2026, 12:31 (hora del servidor emisor) |
| Fecha de entrega | Entregado en 541 segundos (~9 minutos) |
| De (visible) | `"'Get Your Fire Back'" <fqesupportmyyk@jxicsmlmfhbecezxfljwjfwe.com>` |
| Para | sebastianbish@gmail.com |
| Asunto | Most men are missing these 7 |
| Reply-To | `reply@dfmhr.88offic.uic.edu` |
| Return-Path (SMTP MAIL FROM) | `Return@k3wnykip71njocbw-hjofazhubfe.leasterianism.me.uk` |
| IP de origen | 107.155.68.163 |
| rDNS de la IP | `107-155-68-163.static.hvvc.us` |
| Servidor de entrega interno | `pdr8-services-05v.prod.PYY28AGM.org` (qmail sobre Postfix) |

---

## 3. Resultado de autenticación de correo

| Mecanismo | Resultado | Interpretación |
|---|---|---|
| **SPF** | PASS (IP 107.155.68.163, dominio `leasterianism.me.uk`) | El servidor está autorizado a enviar en nombre de `leasterianism.me.uk` — infraestructura propia del operador, no evidencia de legitimidad. |
| **DKIM** | **permerror** ("no key for signature") — dominio `jxicsmlmfhbecezxfljwjfwe.com` | La firma DKIM referencia un selector (`default`) sin clave pública publicada en el DNS del dominio firmante. Indicativo de dominio desechable creado sin mantenimiento correcto de su propia infraestructura de autenticación. |
| **DMARC** | **FAIL** | Consecuencia directa del fallo de DKIM y de la falta de alineación entre el dominio visible (`jxicsmlmfhbecezxfljwjfwe.com`) y el dominio técnico de envío (`leasterianism.me.uk`). |

**Conclusión de este apartado:** cadena de tres dominios distintos en una sola transacción SMTP (dominio DKIM firmante, dominio Return-Path/SPF, dominio del rDNS de la IP), un patrón característico de infraestructura de envío masivo rotada y desacoplada de cualquier marca o identidad estable.

---

## 4. Anatomía de la infraestructura de envío

### 4.1 Delegación de remitente ("Sender Delegation")

Gmail marca la cabecera con `X-Google-Sender-Delegation: sebastianbish@gmail.com Trusted Sender`. Esto **no** significa que el destinatario haya enviado el correo: es un artefacto de cómo Gmail procesa ciertos encabezados de remitente delegado cuando el "From" visible no coincide con el dominio autenticado — Gmail intenta asociar el mensaje a una identidad reconocible en lugar de mostrar el dominio técnico real, lo cual, paradójicamente, puede hacer que el mensaje parezca más confiable de lo que es.

### 4.2 Tres dominios, tres roles, cero relación de marca

| Dominio | Rol | Función |
|---|---|---|
| `jxicsmlmfhbecezxfljwjfwe.com` | Firmante DKIM (roto) | Cadena alfanumérica de 26 caracteres sin estructura de marca — generación algorítmica |
| `leasterianism.me.uk` | Return-Path / SPF | Registrado el **9 de diciembre de 2024** (Namecheap) — dominio "envejecido" ~21 meses antes de este envío, mismo patrón de *domain aging* observado en los casos Magic Win y Lucky Creek de este repositorio |
| `88offic.uic.edu` | Reply-To | Subdominio de **University of Illinois Chicago** (institución académica real, TLD `.edu`) usado como buzón de respuesta — abuso de infraestructura de terceros (posible subdominio mal configurado o apropiado) para prestar una falsa señal de legitimidad institucional a cualquier respuesta que el destinatario intente enviar |

### 4.3 IP de envío: hosting dedicado, no residencial

La IP `107.155.68.163` resuelve por rDNS a `107-155-68-163.static.hvvc.us`, perteneciente a **Hivelocity, Inc.** (AS29802), proveedor de servidores dedicados/VPS con centros de datos en Dallas (TX). Es infraestructura de hosting comercial alquilada para el envío, no un servidor comprometido de terceros ni una IP residencial — coherente con una operación de afiliación que opera su propia infraestructura de disparo de correo a escala.

---

## 5. Análisis del enlace de destino

```
https://storage.googleapis.com/jhiugiiiiiiiiiii/INTLsend.html#/redirect.html?od=1syr6aa6cda39ae92_vl_Salesvl_1nq4.ujzsvi.C0000rk1ka42upo00v_x12150.k1ka4MGl5cjZyLTFscG1oNTM0c1snQ
```

- Mismo patrón de evasión que los casos Cheeky Wins, Magic Win y Lucky Creek de este repositorio: alojamiento en `storage.googleapis.com` (Google Cloud Storage) para heredar la reputación de dominio confiable de Google y evadir filtros de reputación/DNSBL, mientras el contenido real del bucket está bajo control del operador de la campaña.
- El fragmento tras `#/redirect.html?od=...` contiene un token largo (`_vl_Salesvl_...`) — estructura típica de tracking de afiliado multi-nivel (identificador de oferta + sub-afiliado + click ID), compatible con un modelo de comisión por venta/lead en una red de afiliados de suplementos.
- El bucket incluye una segunda URL de "Unsubscribe" con la misma estructura de tracking, lo cual —como en el caso Cheeky Wins— sirve principalmente para confirmar que la dirección está activa.
- No se accedió al enlace por precaución, siguiendo la misma metodología pasiva del resto de casos de este repositorio.

---

## 6. Indicadores de ingeniería social

- Ángulo de "energía/vitalidad masculina" con lenguaje de urgencia sutil ("most men are missing something... they have no idea it's the reason their fire went out").
- Cuerpo del mensaje breve y persuasivo, en inglés, sin ningún dato personal del destinatario — saludo ausente, texto genérico aplicable a cualquier receptor.
- Dirección postal física de "opt-out" (1525 Aviation Blvd. #247, Redondo Beach, CA) — inclusión típica para aparentar cumplimiento de la ley CAN-SPAM estadounidense, aunque no garantiza legitimidad de la oferta subyacente.

---

## 7. Text stuffing / relleno anti-filtrado

El cuerpo HTML del mensaje contiene, tras el bloque visible, una cantidad masiva de contenido oculto (no renderizado en el cliente de correo) diseñado para diluir la "huella" de spam ante filtros bayesianos y de aprendizaje automático:

- Un artículo completo reciclado sobre cultivo de cítricos en maceta (limoneros Meyer, kumquats, calamondin) — contenido genérico de jardinería sin relación alguna con el asunto del correo.
- Fragmentos de al menos **tres plantillas de correo legítimas distintas** intercaladas sin relación entre sí: un email de confirmación de YNAB ("budgeting glory... The YNAB Team"), un email de bienvenida al programa de fidelización Enterprise Rent-A-Car ("Enterprise Plus membership"), y un email de confirmación de demo de la empresa de software Event Temple ("Dylan Basile", "requesting a demo").
- Bloques largos de cadenas alfanuméricas aleatorias en mayúsculas y minúsculas sin significado semántico, generadas para inflar la entropía del mensaje.
- Un bloque de arte ASCII oculto (banner decorativo tipo "figlet").
- Fragmentos de texto político/institucional descontextualizado (referencias a "Brookings", "Eisenhower Foundation", "Kerner Commission") insertados como relleno temático adicional.

Este patrón —reciclaje de plantillas ajenas + relleno temático inconexo + arte ASCII— es una huella de **kit de spam genérico**, coherente con la observada en el caso Lucky Creek de este mismo repositorio (que reutilizaba plantillas de Enterprise Rent-A-Car, IBM Cloud, Podio, Fastly, Parsec y Wistia). La reaparición de la plantilla de **Enterprise Rent-A-Car** en ambos casos, meses después y en campañas de temática distinta (casino vs. suplementos), sugiere que ambos operan sobre el mismo kit de generación de spam o una fuente de plantillas recicladas compartida entre distintos afiliados.

---

## 8. Descarte y evaluación de vectores de riesgo

| Vector | ¿Presente? | Justificación |
|---|---|---|
| Ataque dirigido (spear phishing) | No | Cuerpo del mensaje completamente genérico, sin datos personales del destinatario |
| Suplantación de marca real | Parcial | Abuso del subdominio `.edu` de UIC como buzón de Reply-To, sin suplantar directamente a la institución en el cuerpo del mensaje |
| Compromiso de la cuenta del destinatario | No | La etiqueta "Trusted Sender" de Gmail es un artefacto de procesamiento de cabeceras, no evidencia de acceso a la cuenta |
| Phishing de credenciales / formulario de login | No verificado | Enlace no visitado por precaución; el patrón (bucket de Google Storage + redirector con tracking) es consistente con landing de venta de producto, no de captura de credenciales |
| Malware / adjuntos | No | Sin archivos adjuntos en el mensaje |

---

## 9. Conclusión final

El correo corresponde a una campaña de **spam de afiliación de suplementos ("male enhancement")** con infraestructura de envío dedicada (Hivelocity, hosting comercial en Dallas) y un patrón de dominios de usar-y-desechar consistente con el resto de casos documentados en este repositorio: un dominio "envejecido" 21 meses (`leasterianism.me.uk`) para el envío técnico, un dominio DKIM roto y sin mantenimiento para la firma visible, y abuso de un subdominio `.edu` legítimo (`uic.edu`) como buzón de respuesta para aparentar legitimidad institucional.

El hallazgo más relevante es la reutilización de la misma plantilla de relleno ("Enterprise Rent-A-Car") ya observada en el caso Lucky Creek, lo que sugiere un ecosistema de kits de spam compartidos entre operadores de campañas de temática distinta (casino online vs. suplementos), más que un actor aislado.

**Nivel de riesgo:** bajo-medio. No hay evidencia de intento de captura de credenciales ni de malware; el riesgo principal es la posible pérdida económica si el destinatario completa una compra a través del enlace de afiliado, dado que el producto y su eficacia no han sido verificados de ninguna forma en este análisis.

**Recomendación:** mantener el correo en Spam, marcar como "Denunciar phishing" para reforzar el filtrado de Google, no interactuar con el enlace de "Discover The 7 Ingredients" ni con el de "Unsubscribe", y no responder al buzón `reply@dfmhr.88offic.uic.edu` bajo ninguna circunstancia.

---

## 10. Indicadores de compromiso (IOCs)

```
Dominios:
jxicsmlmfhbecezxfljwjfwe.com
leasterianism.me.uk
k3wnykip71njocbw-hjofazhubfe.leasterianism.me.uk
88offic.uic.edu (subdominio abusado, no controlado por el atacante)

IP:
107.155.68.163 (Hivelocity Inc., Dallas TX, AS29802)

URLs:
https://storage.googleapis.com/jhiugiiiiiiiiiii/INTLsend.html

Remitente:
fqesupportmyyk@jxicsmlmfhbecezxfljwjfwe.com

Reply-To abusado:
reply@dfmhr.88offic.uic.edu
```

---

*Informe generado como parte del análisis OSINT rutinario de correo entrante — SpainCheck Pro.*

## Autor

Sebastian | Cybersecurity Student
