# Informe de análisis de correo electrónico sospechoso

**Analista:** SpainCheck Pro
**Fecha del análisis:** 30 de agosto de 2026
**Destinatario del correo:** [dirección anonimizada — cuenta personal de Gmail]
**Clasificación previa:** Spam (Gmail)

---

## 1. Resumen ejecutivo

Se ha analizado un correo electrónico recibido en la bandeja de Spam de Gmail, con asunto **"Los archivos se están eliminando ahora mismo"**, que simula una alerta de un proveedor de almacenamiento en la nube genérico advirtiendo de la eliminación inminente de archivos. El análisis de cabeceras, remitente y estructura del enlace de destino confirma que se trata de **scareware / phishing por miedo**, con infraestructura de envío más cuidada que casos previos (DKIM válido) pero con desalineación DMARC, campo "Para" genérico y un retraso de entrega anómalo.

---

## 2. Datos del mensaje

| Campo | Valor |
|---|---|
| ID de mensaje | `<mdbeqwsnzebvhjtfluuaosmjhcbzwn@yt8qrg0c6elwkyegim>` |
| Fecha de creación | 25/08/2026, 17:09 |
| Fecha de entrega | Entregado 387.443 segundos después (~4,5 días de retraso) |
| De | `Su_Cloud_Account <lwghoir@mlrjaztk.dm.punen.biz.id>` (marcado como "enviado por Trusted Sender") |
| Para | `me@aol.com` (dirección genérica, no la del destinatario real) |
| Asunto | Los archivos se están eliminando ahora mismo |
| IP de origen | 62.3.35.12 |

---

## 3. Resultado de autenticación de correo

| Mecanismo | Resultado | Interpretación |
|---|---|---|
| **SPF** | PASS (IP 62.3.35.12) | El servidor emisor está autorizado para `mlrjaztk.dm.punen.biz.id`. |
| **DKIM** | **PASS** | La firma criptográfica es válida — infraestructura más cuidada que en casos anteriores de spam más burdo. |
| **DMARC** | **FAIL** | Con SPF y DKIM en PASS, un fallo de DMARC indica **desalineación de dominio**: el dominio visible en el remitente no coincide exactamente con el dominio validado por SPF/DKIM a nivel técnico. |

**Conclusión de este apartado:** operación de spam más profesionalizada que casos previos analizados, pero que sigue sin superar una validación DMARC estricta, delatando el uso de dominios de envío técnicos distintos a la identidad mostrada al usuario.

---

## 4. Anomalías estructurales relevantes

### 4.1 Campo "Para" genérico

El campo "Para" muestra `me@aol.com`, una dirección que no corresponde al destinatario real. Esto es característico de envíos masivos por BCC oculto o de listas de distribución que rellenan el campo "To" visible con un valor placeholder idéntico para todos los destinatarios, ocultando así tanto la lista real de víctimas como la dirección individual en la cabecera pública del mensaje.

### 4.2 Retraso de entrega anómalo

El mensaje fue creado el 25 de agosto a las 17:09 pero entregado con un retraso de aproximadamente 4,5 días (387.443 segundos). Este patrón es atípico en correo legítimo y consistente con colas de reenvío a través de múltiples relays/proxies de spam, o con reintentos de entrega tras rebotes en servidores intermedios.

### 4.3 Dominio y subdominios

`mlrjaztk.dm.punen.biz.id` — mismo patrón de subdominios alfanuméricos aleatorios (`mlrjaztk.dm`) sobre un dominio raíz (`punen.biz.id`) observado en operaciones de spam desechable. El TLD `.biz.id` (espacio empresarial del ccTLD indonesio) es un TLD de bajo coste y escasa vigilancia, habitual en infraestructura de spam internacional.

---

## 5. Análisis del enlace de destino

```
https://storage.googleapis.com/globallyse/serkimerns.html#4FCjhN9170vMhA123kdtzomolfq79DlISVUVTPYKSUOV88531WRUM15886y5
```

Aloja el contenido en `storage.googleapis.com` (Google Cloud Storage), técnica de evasión que aprovecha la reputación del dominio raíz de Google para eludir filtros de reputación/DNSBL, mientras el contenido real (bucket público) está bajo control del operador de la campaña. Mismo patrón de evasión que en el caso de spam de casino analizado previamente.

---

## 6. Indicadores de ingeniería social

- Banner de alarma en rojo con la etiqueta "ÚLTIMA ADVERTENCIA".
- Mensaje central: "SU ALMACENAMIENTO EN LA NUBE ESTÁ LLENO".
- Amenazas de pérdida irreversible de datos: "fotos y vídeos serán eliminados", "archivos serán perdidos permanentemente", "copias de seguridad serán borradas para siempre".
- Botón de llamada a la acción en mayúsculas y color de alarma: "ACTUAR AHORA".
- Refuerzo textual de urgencia: "NO IGNORE ESTO, SUS ARCHIVOS ESTÁN SIENDO ELIMINADOS EN ESTE MOMENTO".
- Remitente ("Su_Cloud_Account") que alude genéricamente a "su proveedor cloud" sin nombrar ninguna marca real (Google Drive, iCloud, Dropbox), evitando así infracción de marca registrada mientras se apoya en que el usuario asuma que es su propio servicio.

Este perfil de presión psicológica (miedo a pérdida irreversible + urgencia temporal) es más agresivo que el observado en el caso de spam de casino, orientado a provocar un clic impulsivo sin verificación previa.

---

## 7. Descarte y evaluación de vectores de riesgo

| Vector | ¿Presente? | Justificación |
|---|---|---|
| Ataque dirigido (spear phishing) | No | Campo "Para" genérico, sin datos personales del destinatario real |
| Suplantación de marca real | No | No se nombra ningún proveedor cloud específico |
| Compromiso de la cuenta del destinatario | No | SPF/DKIM válidos corresponden al dominio propio del operador de spam, no al del destinatario |
| Phishing de credenciales / formulario de login | No verificado | El enlace de destino no fue visitado por precaución; el diseño (alarma + urgencia) es consistente con landing de "soporte técnico falso" o captura de datos, pero no se confirma directamente |
| Malware / adjuntos | No | Sin archivos adjuntos en el mensaje |

---

## 8. Conclusión final

El correo analizado corresponde a una campaña de **scareware por urgencia de almacenamiento en la nube**, con infraestructura de envío más profesionalizada que otros casos de spam básico (DKIM correctamente firmado) pero que aun así falla DMARC por desalineación de dominio, confirmando el uso de infraestructura técnica de envío distinta a la identidad mostrada al usuario.

El campo "Para" genérico (`me@aol.com`) y la ausencia total de datos personales en el cuerpo del mensaje descartan un ataque dirigido: el destinatario forma parte de una lista de distribución masiva, probablemente la misma o una similar a la usada en la campaña de spam de casino analizada previamente, dado el patrón compartido de subdominios aleatorios y abuso de `storage.googleapis.com` como alojamiento de landing.

El principal riesgo de este correo no reside en su infraestructura técnica —relativamente estándar dentro del panorama de spam desechable— sino en su **diseño de ingeniería social**, orientado deliberadamente a inducir pánico y una respuesta impulsiva sin verificación, un patrón de mayor riesgo conductual que el de una oferta promocional simple.

**Nivel de riesgo:** medio, condicionado a no interactuar con el enlace del cuerpo del mensaje. El riesgo es mayor que en casos de spam promocional simple debido al diseño de presión psicológica, aunque no se ha confirmado captura activa de credenciales en el destino.

**Recomendación:** mantener el correo en Spam, marcar como "Denunciar phishing" para reforzar el filtrado de Google, y no acceder al enlace de "Darse de baja" ni al botón "ACTUAR AHORA". Si existiera duda real sobre el estado de algún servicio de almacenamiento en la nube legítimo, verificar siempre accediendo directamente a la aplicación o sitio web oficial, nunca a través de enlaces recibidos por correo.

---

*Informe generado como parte del análisis OSINT rutinario de correo entrante — SpainCheck Pro.*

## 👤 Autor

Sebastian | Cybersecurity Student

## Autor

Sebastian | Cybersecurity Student
