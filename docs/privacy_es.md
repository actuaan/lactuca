# Política de Privacidad

**Lactuca — Biblioteca de Cálculo Actuarial de Vida para Python**

*Fecha de entrada en vigor: 2026-04-01. Última actualización: 2026-06-19.*
*Responsable del tratamiento: Alberto Aragoneses Nebreda, que opera bajo la marca Actuaan.*
*Contacto: [support@lactuca.io](mailto:support@lactuca.io)*

---

```{note}
Esta es la traducción oficial al español de la Política de Privacidad de Lactuca,
puesta a disposición en cumplimiento del Art. 12(1) del RGPD y del Art. 10 de la
Ley 34/2002 (LSSI-CE).

En caso de discrepancia entre esta versión y la versión en inglés, prevalecerá la
**versión en inglés** ({doc}`privacy`).
```

---

La presente Política de Privacidad explica qué datos personales se recaban al utilizar
Lactuca o al iniciar una prueba gratuita, dónde se almacenan, con qué base jurídica se
tratan y qué derechos le asisten como interesado en virtud del Reglamento General de
Protección de Datos (RGPD — Reglamento (UE) 2016/679).

---

## 1. Responsable del tratamiento

El responsable del tratamiento de todos los datos personales descritos en la presente
política es:

> Alberto Aragoneses Nebreda (Actuaan)
> NIF: 09180714B
> Dirección: Francesc Pérez Cabrero, 7, 08021, Barcelona, España
> Correo electrónico: [support@lactuca.io](mailto:support@lactuca.io)
> País: España (Unión Europea)

---

## 2. Datos que recabamos

### 2.1 Registro de prueba

Cuando solicita una prueba gratuita a través del aviso de activación al importar el
Software por primera vez, recabamos:

| Dato | Finalidad | Almacenado en |
|---|---|---|
| Dirección de correo electrónico | Entrega de la clave de prueba; gestión de licencias | Keygen.sh (a través del webhook de Vercel) |
| Nombre opcional | Personalización del correo de bienvenida | Resend — solo entrega de correo electrónico (a través del webhook de Vercel) |
| Huella digital del hardware (hash) | Prevenir el abuso de la prueba (una prueba por dispositivo) | Keygen.sh (a través del webhook de Vercel) |
| Dirección IP (inherente al HTTP saliente) | Implícita en la solicitud de red al webhook de prueba | Registros de acceso de Vercel; registros de Keygen.sh |

La **huella digital del hardware** es un hash SHA-256 derivado de los atributos de
hardware del dispositivo (dirección MAC, nombre de host, arquitectura de CPU). Los
valores brutos **nunca se transmiten** —solo se envía y almacena el hash
irreversible—. La huella digital constituye **datos seudonimizados** en el sentido del
Art. 4(5) del RGPD: no puede identificarle directamente como individuo sin acceso a
información adicional que el Licenciante no conserva.

:::{note}
**La entrega de la clave de prueba no registra un dispositivo.** Cuando solicita una
prueba, creamos una clave de licencia y almacenamos su correo y la huella digital
(hash) para evitar pruebas duplicadas. El cupo de dispositivo de su plan **no** se
consume hasta que active Lactuca con éxito en ese equipo por primera vez (véase §2.2).
:::

### 2.2 Activación y validación de la licencia

Cuando activa Lactuca (o cuando la biblioteca valida su licencia en línea), recabamos:

| Dato | Finalidad | Almacenado en |
|---|---|---|
| Clave de licencia | Autenticación de la licencia | Keygen.sh |
| Huella digital del hardware (hash) | Aplicar los límites de dispositivos | Keygen.sh |
| Nivel de licencia | Determinar el uso permitido | Keygen.sh |
| Marca de tiempo de activación | Seguimiento del período de gracia y de la expiración | Keygen.sh |
| Registro de dispositivo (hash de huella digital) | Vincular la licencia a este equipo; aplicar límites de dispositivos del plan | Keygen.sh |
| Dirección IP (inherente al HTTP saliente) | Implícita en la solicitud de red | Registros de Keygen.sh |

:::{important}
**El registro del dispositivo ocurre en la primera activación, no al emitir la clave.**
Recibir una clave de prueba o de pago no registra su ordenador. La primera activación
con éxito en un equipo crea un **registro de dispositivo** en el servidor de licencias,
asociado al hash de la huella digital de ese equipo. Cada dispositivo registrado cuenta
para el límite de su plan (por ejemplo, un dispositivo en los planes Trial e
Individual). Trasladar la licencia a otro equipo requiere liberar un cupo antes — véase
el {ref}`procedimiento de transferencia de dispositivo <device-transfer>` en la FAQ de
licencias.
:::

### 2.3 Seguimiento de sesiones simultáneas (heartbeat)

Mientras una instancia del Software está en ejecución, envía una solicitud HTTP de
keep-alive (*heartbeat*) a la API de Keygen.sh aproximadamente cada 10 minutos en
niveles de un solo puesto y cada 20 minutos en Team y Enterprise (derivado de la
política de procesos del servidor de licencias). Cada
solicitud de heartbeat contiene:

| Dato | Finalidad | Almacenado en |
|---|---|---|
| Identificador de la clave de licencia (o hash) | Identificar la sesión de licencia | Keygen.sh |
| Hash de la huella digital del hardware | Asociar la sesión con el dispositivo con licencia | Keygen.sh |
| Marca de tiempo de la solicitud | Seguimiento del keep-alive de la sesión | Keygen.sh |
| Dirección IP (inherente al HTTP saliente) | Implícita en la solicitud de red | Registros de Keygen.sh |

En las solicitudes de heartbeat no se transmiten valores de hardware brutos, nombres de
usuario, direcciones de correo electrónico ni datos de aplicaciones. **Conservación:**
Los registros de solicitudes de la API (dirección IP y marcas de tiempo) se eliminan
automáticamente transcurridos 14 días; los registros de actividad y eventos,
transcurridos 30 días; los datos de estado de la licencia, durante la vigencia de la
licencia activa.

### 2.4 Compra y facturación

Los datos de pago (número de tarjeta, dirección de facturación) son recabados
exclusivamente por **Lemon Squeezy** (nuestro Merchant of Record). Nunca tenemos acceso
ni almacenamos los datos de su tarjeta. Lemon Squeezy nos proporciona:

| Dato | Finalidad | Almacenado en |
|---|---|---|
| Dirección de correo electrónico | Entrega de la licencia; gestión de la suscripción | Keygen.sh |
| Estado de la suscripción | Renovación, suspensión y revocación de la licencia | Keygen.sh |
| País/jurisdicción | Cumplimiento del IVA | Lemon Squeezy (no transferido al Licenciante) |

### 2.5 Datos que NO recabamos

- Su nombre real más allá de la personalización opcional del correo de bienvenida
  durante el registro de la prueba (véase §2.1)
- Comportamiento de navegación, análisis de uso ni telemetría desde el interior de la
  biblioteca
- Números de tarjeta de crédito o débito (gestionados íntegramente por Lemon Squeezy)
- Contenidos de sus cálculos actuariales ni de sus archivos de datos

### 2.6 Datos obligatorios y opcionales (Art. 13(2)(e) RGPD)

La tabla siguiente indica, para cada dato personal recabado directamente de usted, si
su aportación es obligatoria y las consecuencias de no facilitarlo.

| Dato | ¿Obligatorio? | Consecuencias de no facilitarlo |
|---|---|---|
| **Dirección de correo electrónico** (registro de prueba) | Obligatorio para celebrar el acuerdo de prueba | No puede emitirse la clave de prueba; la prueba no puede iniciarse |
| **Huella digital del hardware** (activación y heartbeat) | Obligatorio para la activación y la aplicación de los límites de dispositivos y sesiones | El Software no puede activarse, validarse ni utilizarse en el dispositivo |
| **Clave de licencia** (activación y validación) | Obligatorio para el funcionamiento del Software | El Software no funcionará |
| **Nombre** (registro de prueba) | Completamente opcional | Ninguna; el correo de bienvenida se envía sin personalización |

La obligación de facilitar la dirección de correo electrónico, la huella digital del
hardware y la clave de licencia constituye un **requisito necesario para celebrar o
ejecutar el contrato** (licencia de pago o acuerdo de prueba). No es una obligación
legal impuesta por ley. El campo **nombre** es voluntario en todo momento.

---

## 3. Base jurídica del tratamiento

| Actividad de tratamiento | Base jurídica (Art. 6 RGPD) |
|---|---|
| Entrega de una licencia de pago y aplicación de sus términos | Art. 6(1)(b) — ejecución de un contrato |
| Entrega de la clave de prueba y prevención de duplicados | Art. 6(1)(a) — consentimiento (prestado en el aviso de activación) |
| Validación de la licencia y aplicación del período de gracia | Art. 6(1)(b) — ejecución de un contrato |
| Cumplimiento del IVA y mantenimiento de registros financieros | Art. 6(1)(c) — obligación legal |
| Seguimiento de sesiones simultáneas (heartbeat) | Art. 6(1)(b) — ejecución del contrato (pago); Art. 6(1)(a) — consentimiento (Prueba) |

---

## 4. Conservación de los datos

| Categoría de datos | Período de conservación |
|---|---|
| Registros de licencias activas (clave, huella digital, nivel, registros de dispositivo) | Durante la vigencia de la suscripción activa |
| Registros de licencias vencidas o revocadas | 3 años tras la expiración (ejecución del contrato y resolución de disputas) |
| Dirección de correo electrónico de la prueba | 1 año tras la expiración de la clave de prueba, o hasta que solicite la supresión |
| Registros de solicitudes de la API (dirección IP, marcas de tiempo — Keygen.sh) | 14 días (supresión automática por Keygen.sh) |
| Registros de acceso (dirección IP — webhook de prueba Vercel) | Conforme a la política de conservación de datos de Vercel (véase [vercel.com/legal/privacy-policy](https://vercel.com/legal/privacy-policy)) |
| Registros de actividad y eventos (Keygen.sh) | 30 días (supresión automática por Keygen.sh) |
| Registros de pago (en poder de Lemon Squeezy) | Según lo exigido por la legislación fiscal aplicable (normalmente 7-10 años) |

---

## 5. Subencargados del tratamiento y transferencias internacionales

Utilizamos los siguientes subencargados del tratamiento para prestar el servicio:

| Subencargado | Función | Ubicación | Mecanismo de transferencia |
|---|---|---|---|
| [Keygen.sh](https://keygen.sh) | Plataforma de gestión de licencias | Estados Unidos | Cláusulas Contractuales Tipo (CCT) |
| [Lemon Squeezy](https://lemonsqueezy.com) | Procesamiento de pagos (Merchant of Record) | Estados Unidos | Cláusulas Contractuales Tipo (CCT) |
| [Resend](https://resend.com) | Entrega de correo electrónico transaccional | Estados Unidos | Cláusulas Contractuales Tipo (CCT) |
| [Vercel](https://vercel.com) | Webhook de registro de prueba (solo enrutamiento de solicitudes) | Estados Unidos | Cláusulas Contractuales Tipo (CCT) |

Todos los subencargados del tratamiento enumerados anteriormente están obligados
contractualmente a tratar sus datos únicamente según nuestras instrucciones
documentadas y a implementar las medidas técnicas y organizativas de seguridad
adecuadas.

---

## 6. Sus derechos en virtud del RGPD

Como interesado en la UE/EEE, le asisten los siguientes derechos:

| Derecho | Descripción |
|---|---|
| **Acceso** (Art. 15) | Solicitar una copia de los datos personales que conservamos sobre usted |
| **Rectificación** (Art. 16) | Solicitar la corrección de datos inexactos |
| **Supresión** (Art. 17) | Solicitar la supresión de sus datos, sujeta a obligaciones legales de conservación |
| **Limitación** (Art. 18) | Solicitar que limitemos el tratamiento en determinadas circunstancias |
| **Portabilidad** (Art. 20) | Recibir sus datos en un formato estructurado y legible por máquina |
| **Oposición** (Art. 21) | Oponerse al tratamiento basado en intereses legítimos |
| **Retirada del consentimiento** (Art. 7(3)) | Retirar el consentimiento para el registro de la prueba en cualquier momento (no afecta al tratamiento anterior) |
| **A no ser objeto de decisiones automatizadas** (Art. 22) | Solicitar revisión humana de cualquier denegación automatizada de licencia e impugnar la decisión |

**Toma de decisiones automatizada (Art. 13(2)(f) RGPD):** El acceso a la licencia se
concede o deniega automáticamente en función del estado de su licencia y la huella
digital del hardware. Si su licencia ha vencido, está suspendida, revocada, si se ha
alcanzado el límite de dispositivos o si la huella digital no coincide con el
dispositivo activado, el acceso se deniega sin intervención humana previa. La
consecuencia es que el Software no funcionará. Este tratamiento automatizado es
necesario para la **ejecución del contrato** (Art. 22(2)(a) RGPD). Tiene derecho a
obtener intervención humana, expresar su punto de vista e impugnar cualquiera de dichas
decisiones contactando con
[support@lactuca.io](mailto:support@lactuca.io).

Para ejercer cualquiera de estos derechos, envíe un correo electrónico a
[support@lactuca.io](mailto:support@lactuca.io) con el asunto
**«Solicitud RGPD — \<derecho\>»** (p. ej., «Solicitud RGPD — Supresión»).

Para **retirar el consentimiento** para el registro de la prueba en concreto, también
puede utilizar el enlace de exclusión incluido en el correo electrónico de entrega de
la clave de prueba, que ofrece un mecanismo de retirada directo e inmediato equivalente
al canal a través del cual se prestó el consentimiento.

Responderemos en el plazo de **30 días naturales**. Si su solicitud es compleja o
numerosa, podremos ampliar este plazo otros 60 días y le notificaremos en consecuencia.

Asimismo, tiene derecho a presentar una reclamación ante la autoridad española de
protección de datos:

> **Agencia Española de Protección de Datos (AEPD)**
> Sitio web: [aepd.es](https://www.aepd.es)
> Teléfono: +34 912 663 517

---

## 7. Seguridad

Aplicamos medidas técnicas y organizativas adecuadas para proteger sus datos personales
frente a pérdidas accidentales, acceso no autorizado, divulgación o destrucción,
incluidas:

- Comunicación exclusivamente por HTTPS entre la biblioteca y los servidores de
  licencias
- Firmas digitales Ed25519 en todos los registros de licencias para evitar
  manipulaciones
- Hash de la huella digital del hardware antes de la transmisión (los datos de hardware
  brutos nunca abandonan su dispositivo)
- Claves de la API de licencias almacenadas como variables de entorno, nunca codificadas

---

## 8. Cookies y seguimiento

El sitio web de documentación de Lactuca (`www.lactuca.io`) no instala
cookies y no utiliza scripts de seguimiento ni análisis. No se recaban datos personales
por el mero hecho de visitar la documentación.

---

## 9. Menores

Lactuca es software profesional destinado a su uso por adultos en un contexto actuarial
o de investigación. No recabamos conscientemente datos personales de personas menores
de 16 años.

---

## 10. Modificaciones de esta política

Podremos actualizar la presente Política de Privacidad periódicamente. Los cambios
sustanciales serán notificados por correo electrónico (utilizando la dirección
facilitada al registrarse) y mediante un aviso destacado en esta página, al menos
30 días antes de la entrada en vigor del cambio.

El [CLUF](eula_es) incorpora la presente Política de Privacidad por referencia.

---

## 11. Contacto

Para cualquier consulta relacionada con la privacidad o para ejercer sus derechos:

**Correo electrónico**: [support@lactuca.io](mailto:support@lactuca.io)

**Prefijo del asunto**: `Solicitud RGPD —`

---

*La versión en inglés de esta Política de Privacidad está disponible en
[privacy](privacy). En caso de discrepancia, prevalece la versión en inglés.*
