# Informe de análisis de correo electrónico sospechoso

**Analista:** SpainCheck Pro
**Fecha del análisis:** 30 de agosto de 2026
**Destinatario del correo:** [dirección anonimizada — cuenta personal de Gmail]
**Clasificación previa:** Spam (Gmail)

## 1. Resumen ejecutivo

Se ha analizado un correo electrónico recibido en la bandeja de Spam de Gmail, con asunto "Claim a 500% Deposit Bonus + 100 Free Spins", promocionando una plataforma de apuestas online ("Cheeky Wins"). El análisis de cabeceras, remitente y estructura del enlace de destino confirma que se trata de spam comercial de afiliación de casino, distribuido mediante infraestructura de envío masivo desechable, sin evidencia de phishing dirigido, robo de credenciales ni compromiso de la cuenta del destinatario.

## 2. Datos del mensaje

| Campo | Valor |
|---|---|
| ID de mensaje | `<5c0s856g.yuvy0220.b0hszq.ckguSMTPIN_ADDED_MISSING@mx.google.com>` |
| Fecha de entrega | 30/08/2026, 6:33 (entregado en 1 segundo) |
| De | Member-Bonus-Team `<ifvomcfaiimiyt.46295952857670@832zdl.drqtu0.wpawrn.us>` |
| Para | [dirección anonimizada] |
| Asunto | Claim a 500% Deposit Bonus + 100 Free Spins |
| Enviado "a través de" | graines-bulbes-semences-plantules.reveralia.info |
| IP de origen | 198.50.125.247 |

## 3. Resultado de autenticación de correo

| Mecanismo | Resultado | Interpretación |
|---|---|---|
| SPF | PASS (IP 198.50.125.247) | El servidor emisor está autorizado a enviar en nombre de wpawrn.us. Esto no valida legitimidad: es infraestructura propia del spammer, correctamente configurada solo para pasar este control. |
| DKIM | FAIL | La firma criptográfica del dominio no es válida. Indicativo de dominio desechable sin mantenimiento serio de autenticación. |
| DMARC | FAIL | Falla por el fallo de DKIM al no haber desalineación de dominio de terceros. Confirma ausencia de política DMARC estricta (p=reject) en wpawrn.us. |

**Conclusión de este apartado:** no se trata de suplantación de la dirección del destinatario (backscatter), sino de infraestructura de spam legítima para el propio operador, generada específicamente para esta campaña.

## 4. Anatomía del remitente

`ifvomcfaiimiyt.46295952857670@832zdl.drqtu0.wpawrn.us`

| Segmento | Función identificada |
|---|---|
| ifvomcfaiimiyt | Cadena alfanumérica aleatoria, sin significado léxico |
| 46295952857670 | Identificador numérico de 14 dígitos — probable ID único de destinatario/campaña para tracking de apertura y clic |
| 832zdl / drqtu0 | Subdominios de 6 caracteres alfanuméricos, generados por wildcard DNS — permiten rotar de subdominio sin perder el dominio raíz cuando uno es bloqueado |
| wpawrn.us | Dominio raíz sin vocales ni estructura de marca — patrón típico de generación algorítmica/diccionario para snowshoe spamming |

El TLD `.us` presenta históricamente una de las peores reputaciones de abuso a nivel mundial (fuente: Interisle Consulting Group, *Cybercrime Supply Chain*, 2018), lo cual es consistente con el patrón observado.

## 5. Análisis del enlace de destino

```
https://storage.googleapis.com/sd8489fd/reddd21.html#Cheeky.html?syy=1x16a82d5043411e_vl...
```

- Alojado en `storage.googleapis.com` (Google Cloud Storage): técnica de evasión que aprovecha la reputación de un dominio raíz confiable para eludir filtros de reputación/DNSBL, mientras el contenido real (bucket público) está bajo control del operador de la campaña.
- Parámetro `syy=...` de longitud considerable: probable token de tracking de afiliado/clic individual.

## 6. Indicadores de ingeniería social

- Bonificaciones desproporcionadas (500%, hasta 3.000 €, 300 giros gratis) — anzuelo de tipo "demasiado bueno para ser verdad".
- Saludo genérico ("Hello Friend") sin ningún dato personal del destinatario, lo que descarta un ataque dirigido (spear phishing) y confirma un envío masivo desde lista comprada o scrapeada.
- Enlace de "unsubscribe" presente — en campañas de esta naturaleza, suele servir para confirmar que la dirección está activa, incrementando el volumen de spam futuro en lugar de reducirlo.

## 👤 Autor

Sebastian | Cybersecurity Student
