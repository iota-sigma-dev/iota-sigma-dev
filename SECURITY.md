# 🛡️ Política de Seguridad de la Información (SECURITY.md)
**Gobernanza de Vulnerabilidades, Privacidad de Datos y Modelado de Amenazas**

Este documento define el marco de políticas de seguridad aplicadas de manera transversal en todos los desarrollos, arquitecturas y operaciones de este repositorio.

---

## 🎯 1. Modelado de Amenazas (Threat Modeling)
Cada cambio en la arquitectura o adición de componentes críticos debe ser pre-evaluado utilizando la metodología **STRIDE** para identificar y mitigar riesgos en la fase de diseño (Shift-Left Security):

| Amenaza (STRIDE) | Descripción del Riesgo | Controles de Mitigación Transversales |
| :--- | :--- | :--- |
| **S**poofing (Suplantación) | Acceso no autorizado simulando identidades legítimas. | Autenticación robusta via IdP, tokens JWT cifrados y Cloudflare Access 2FA. |
| **T**ampering (Manipulación) | Modificación maliciosa de datos en tránsito o reposo. | Integridad de base de datos, firmas en webhooks, cifrado AES-256-GCM. |
| **R**epudiation (Repudio) | Imposibilidad de auditar y trazar acciones críticas. | Audit logs inmutables en PostgreSQL, logs de nginx e ingestión IDS en CrowdSec. |
| **I**nformation Disclosure (Fuga) | Filtración de datos confidenciales o sensibles (B2B/B2C). | Aislamiento estricto de base de datos mediante **Row-Level Security (RLS)**. |
| **D**enial of Service (DoS) | Agotamiento de recursos que provoque caída de servicio. | Rate limiting en Nginx Ingress, control de concurrencia/colas FIFO en Redis/BullMQ. |
| **E**levation of Privilege (Elevación) | Explotación de brechas para obtener accesos de administración. | Principio de Menor Privilegio (Least Privilege), Docker Rootless, segregación de DB. |

---

## ⚖️ 2. Apetito de Riesgo y Ciclo de Vida de Vulnerabilidades (SLAs)

Los ecosistemas operan bajo una postura de apetito de riesgo **Bajo/Conservador** para cualquier amenaza que comprometa la integridad de los datos de los usuarios, el acceso a infraestructuras críticas (Zero-Trust) o la continuidad del negocio. Se asume un entorno hostil por defecto. No se toleran vulnerabilidades críticas o altas explotables en el perímetro expuesto sin controles compensatorios inmediatos.

La gestión de vulnerabilidades se fundamenta en la combinación de **CVSS v4.0** (Severidad Técnica Intrínseca) y **EPSS (Exploit Prediction Scoring System)** (Probabilidad de Explotación Real en el Mundo Real), optimizando la prioridad de remediación frente al impacto de negocio (**FAIR**).

### SLAs de Parcheo y Remediación (NIST SP 800-40)

| Severidad CVSS v4.0 | Probabilidad EPSS | Apetito de Riesgo | SLA Máximo de Mitigación |
| :--- | :--- | :--- | :--- |
| **Crítica (9.0 - 10.0)** | ≥ 10% | **Cero Tolerancia** | ≤ 48 Horas |
| **Alta (7.0 - 8.9)** | ≥ 5% | **Bajo** | ≤ 7 Días |
| **Media (4.0 - 6.9)** | < 5% | **Moderado** | ≤ 30 Días |
| **Baja (0.1 - 3.9)** | Cualquiera | **Aceptado** | Ciclo de Release Estándar |

*Nota:* Si un hallazgo tiene una severidad **Alta** pero un **EPSS < 1%**, el riesgo puede ser temporalmente aceptado o mitigado mediante controles compensatorios (ej. reglas WAF específicas) extendiendo el SLA de remediación hasta 15 días tras análisis formal de GRC.

---

## 🔒 3. Privacidad y Protección de Datos por Diseño (GDPR / PCI-DSS)
1. **Minimización de Datos:** No se recopila ni almacena información de identificación personal (PII) innecesaria. Los datos temporales y volcados de OCR se sanitizan localmente en memoria antes de su vectorización.
2. **Cifrado en Reposo y Tránsito:** Todas las conexiones transitan por canales TLS 1.3. Las credenciales BYOT (Bring Your Own Token) y secretos de integración se cifran simétricamente en base de datos usando `aes-256-gcm`.
3. **Borrado Seguro y Retención:** Alineado con GDPR, se implementan procedimientos automatizados de depuración y purga transaccional (`rebuild.sh` y scripts SQL atómicos) para asegurar que la eliminación de un archivo borre simultáneamente todos los metadatos y vectores asociados en la base de datos documental (`pgvector`).
4. **Segregación de Entornos:** Las automatizaciones e integraciones complejas operan sobre bases de datos físicamente aisladas (ej. `n8n_db` separada de la base transaccional), limitando el impacto lateral ante un compromiso de credenciales.

---

## 📞 4. Reporte de Vulnerabilidades (Disclosure Policy)
Si descubre alguna vulnerabilidad de seguridad en cualquiera de nuestros sistemas, por favor no abra un issue público. Reporte el hallazgo de forma segura:
- **Canal Oficial:** Envíe un correo electrónico con los detalles técnicos a: `security@iota-sigma.dev`
- **Trazabilidad:** Proporcione pasos detallados para reproducir el fallo, capturas de pantalla o código de prueba (PoC) de forma confidencial.
- **Compromiso:** Responderemos a su reporte en un plazo de 24 horas y nos comprometemos a mantenerlo informado durante el proceso de remediación hasta su divulgación coordinada.
