# Contrato de Licencia de Usuario Final

**Lactuca — Biblioteca de Cálculo Actuarial de Vida para Python**

*Fecha de entrada en vigor: 2026-04-20. Última actualización: 2026-06-22.*

---

```{note}
Esta es la traducción oficial al español del Contrato de Licencia de Usuario Final de
Lactuca, puesta a disposición del Licenciatario en cumplimiento del artículo 5 de la
Ley 7/1998 sobre Condiciones Generales de la Contratación (LCGC) y del artículo 60
del TRLGDCU.

**Prevalencia de versiones:** Para **consumidores con residencia habitual en España**,
esta versión española prevalecerá en caso de discrepancia con la versión en inglés.
Para todos los demás Licenciatarios, prevalecerá la **versión en inglés**
({doc}`eula`).
```

---

**Identificación del Licenciante (Art. 10 Ley 34/2002, LSSI-CE):**

> Alberto Aragoneses Nebreda (que opera bajo la marca **Actuaan**)
> NIF: 09180714B
> Dirección: Francesc Pérez Cabrero, 7, 08021, Barcelona, ESPAÑA
> Correo electrónico: [support@lactuca.io](mailto:support@lactuca.io)
> Sitio web: [www.lactuca.io](https://www.lactuca.io)
>
> Alberto Aragoneses Nebreda es un profesional autónomo inscrito en el Régimen Especial
> de Trabajadores Autónomos (RETA) de la Seguridad Social española y en la Agencia
> Estatal de Administración Tributaria (AEAT). A efectos de la Ley 34/2002, de 11 de
> julio, de servicios de la sociedad de la información y de comercio electrónico
> (LSSI-CE), Alberto Aragoneses Nebreda actúa como *prestador de servicios de la
> sociedad de la información*.

Al descargar, instalar, activar o utilizar Lactuca («**Software**»), usted
(«**Licenciatario**») acepta quedar vinculado por el presente Contrato de Licencia de
Usuario Final («**Contrato**»). Si no está de acuerdo, no instale ni utilice el
Software.

---

## 0. Partes y estructura contractual

### 0.1 Pago y Merchant of Record

Las compras de licencias **de pago** de Lactuca (planes Individual, Team y Enterprise
mensual) se procesan a través de **Lemon Squeezy** (LSFS Inc.), que actúa como
Merchant of Record en esas transacciones. La relación comercial relativa al pago, la
facturación, el IVA y los reembolsos aplicables a la transacción de pago es entre el
Licenciatario y Lemon Squeezy. Los términos de servicio y la política de privacidad
propios de Lemon Squeezy se aplican a dicha transacción de pago.

Las **licencias de prueba gratuita** están disponibles sin coste y pueden solicitarse
a través del aviso de activación del Software al iniciarlo por primera vez. No se
procesan a través de Lemon Squeezy y el Licenciante no recaba ni trata ningún dato de
pago en el nivel Trial.

Las **licencias Académicas y Comunitarias** son concedidas directamente por el
Licenciante mediante solicitud por correo electrónico
([support@lactuca.io](mailto:support@lactuca.io)) y no se procesan
a través de Lemon Squeezy. El Licenciante no recaba ni trata datos de pago en estos
niveles.

Los **contratos anuales Enterprise** se celebran directamente entre el Licenciatario y
el Licenciante mediante un acuerdo escrito independiente (Formulario de Pedido o
Contrato Marco de Licencia). Lemon Squeezy no interviene en dichas transacciones
(véase §11.5).

En caso de conflicto entre el presente Contrato y los términos de Lemon Squeezy en
relación con el alcance de la licencia o el uso permitido del Software, el presente
Contrato prevalecerá en lo relativo a la licencia.

---

## 1. Concesión de licencia

Con sujeción a los términos del presente Contrato y al pago de las tarifas aplicables
(o a la aceptación de una prueba gratuita o una concesión académica), el Licenciante
otorga al Licenciatario una licencia limitada, no exclusiva, intransferible y no
sublicenciable para:

(a) instalar y utilizar el Software en sus propios dispositivos para cualquier fin
    profesional, académico o de investigación; y

(b) integrar el Software en sus propias aplicaciones, sistemas, flujos de trabajo o
    productos, incluidas las aplicaciones que se distribuyen o comercializan con
    terceros, siempre que:

    (i)  el Software no se ponga a disposición de los usuarios finales como biblioteca
         o paquete actuarial independiente —debe permanecer integrado e inaccesible
         como componente independiente—; y

    (ii) una licencia Lactuca válida cubra cada máquina (dispositivo) en la que el
         Software esté instalado y se ejecute. El modelo de licencia aplicable depende
         del escenario de despliegue:

         - **Instalación directa** (el Software se instala en el dispositivo propio de
           cada usuario): cada dispositivo requiere una licencia bajo el nivel adecuado
           (Individual, Team o Enterprise).

         - **Servidor o despliegue compartido** (el Software se instala en un servidor
           o infraestructura compartida a la que acceden los usuarios a través de una
           aplicación, API o flujo de trabajo sin instalarlo en sus propios dispositivos
           individuales): la licencia se aplica al dispositivo o dispositivos servidor
           en los que se ejecuta el Software. Una licencia Enterprise cubre a todos los
           usuarios internos del Licenciatario que acceden al Software a través de
           dicha infraestructura compartida, sujeto al límite de dispositivos del nivel
           contratado. No se requiere una licencia individual para cada usuario final
           únicamente porque interactúe con una aplicación que llama al Software del
           lado del servidor.

         El mecanismo técnico de aplicación de la licencia (§5) valida la licencia en
         la máquina en la que está instalado el Software. El Software no funcionará en
         ningún dispositivo sin licencia.

El derecho del Licenciatario a integrar y distribuir según lo previsto en (b) no se
extiende a:

- redistribuir el Software (o cualquier parte del mismo) como biblioteca, paquete o
  SDK independiente, ya sea de forma gratuita o comercialmente;
- crear o comercializar un producto que sea sustancialmente derivado o que compita
  directamente con el Software como biblioteca de cálculo actuarial.

**Uso OEM y redistribución a usuarios externos — se requiere contrato independiente.**

El uso del Software en cualquier producto o servicio distribuido o puesto a disposición
de usuarios **externos a la organización del Licenciatario** constituye **uso OEM** y
está **estrictamente prohibido** bajo cualquier nivel de licencia estándar sin un
Acuerdo OEM escrito independiente. Véase §16 para la definición completa, las
obligaciones de cumplimiento y la forma de obtener una licencia OEM.

El número de usuarios y dispositivos con licencia viene determinado por el nivel
adquirido:

| Nivel | Grupo de activaciones | Sesiones simultáneas |
|---|---|---|
| **Trial** | 1 | 1 |
| **Individual** | 1 | 1 |
| **Team** | 10 | 10 |
| **Enterprise** | 50 | 50 |
| **Académica y Comunitaria** | 1 | 1 |
| **OEM** | según acuerdo | ilimitadas |

«**Dispositivo**» significa una única máquina física o virtual, incluidos servidores y
hosts de contenedores. Los servidores compartidos y los entornos de contenedores
cuentan como un dispositivo por host, independientemente del número de procesos o
usuarios simultáneos en dicho host.

«**Organización del Licenciatario**» significa, para los Licenciatarios de los niveles
Team y Enterprise, la entidad jurídica que celebró el presente Contrato, incluida
cualquier filial o entidad afiliada en la que dicha entidad jurídica ostente más del
50 % de los derechos de voto o participación en la propiedad. El uso del Software por
parte de empleados o contratistas de dichas entidades con control mayoritario constituye
uso interno y no requiere una licencia independiente, siempre que el total de plazas de
activación en todas las entidades incluidas se mantenga dentro de los límites del nivel
contratado. Las entidades en las que el Licenciatario tenga el 50 % o menos no están
cubiertas y requieren una licencia independiente.

El **grupo de activaciones** es el límite de dispositivos técnicamente aplicado
(mediante Keygen.sh): cada dispositivo activado consume una plaza. El límite de
**sesiones simultáneas** es una restricción de seguimiento de sesiones concurrentes
aplicada de forma independiente: solo el número indicado de instancias del Software
puede ejecutarse simultáneamente en todos los dispositivos activados en cualquier
momento. Una instancia que no puede obtener una plaza de sesión no se inicia y muestra
un mensaje de error indicando que todas las plazas están ocupadas. La plaza de sesión
se libera automáticamente cuando la instancia finaliza con normalidad, o cuando el
servidor de licencias detecta que la instancia ha dejado de enviar señales de
keep-alive (véase §6.1). Cualquier nivel puede cubrir un servidor compartido para
usuarios internos —el servidor consume una plaza de activación y los usuarios internos
que acceden a Lactuca a través de dicho servidor no necesitan cada uno una clave de
licencia independiente (véase §1(b)(ii))—. Los usuarios externos a la organización del
Licenciatario no están cubiertos por ningún nivel estándar; véase §16.

---

## 2. Prueba gratuita

### 2.1 Condiciones de la prueba

El Licenciante ofrece una prueba gratuita de 30 días del Software («**Prueba**»). La
Prueba está disponible para cualquier Licenciatario que no haya activado previamente
una licencia de pago o una Prueba anterior utilizando la misma dirección de correo
electrónico o en el mismo dispositivo.

Durante la Prueba, se aplican todas las disposiciones del presente Contrato, excepto
que:

(a) no se requiere ningún pago; y

(b) la Prueba expira automáticamente 30 días después de que se emita la clave de
    licencia; el Software dejará de funcionar tras la expiración.

### 2.2 Sin conversión automática

La Prueba **no** se convierte automáticamente en una suscripción de pago al expirar.
El Licenciatario debe adquirir por separado un plan de pago para continuar utilizando
el Software tras el período de Prueba. No se realizará ningún cargo al finalizar la
Prueba sin un acto afirmativo de compra por parte del Licenciatario.

### 2.3 Limitaciones de la prueba

La Prueba está limitada a 2 activaciones de dispositivo y 1 sesión simultánea. Se
aplican todas las restricciones del §3 durante la Prueba. Las licencias de Prueba no
incluyen derechos de actualización más allá del período de prueba.

### 2.4 Una prueba por persona y por dispositivo

La Prueba está limitada a una por persona y por dispositivo. Los intentos de eludir
este límite (p. ej., mediante el registro de múltiples direcciones de correo electrónico
o la manipulación de identificadores de hardware) constituyen un incumplimiento esencial
del presente Contrato.

### 2.5 Datos personales durante la prueba

La Prueba no se procesa a través de Lemon Squeezy. Al solicitar una clave de prueba a
través del aviso de activación del Software, el Licenciatario **consiente expresamente**
(Art. 6(1)(a) RGPD) el tratamiento de los siguientes datos personales, cada uno para
el fin específico indicado:

- **Dirección de correo electrónico**: para entregar la clave de prueba y gestionar el
  período de prueba.
- **Nombre** (opcional — solo se trata si se facilita): para personalizar el correo
  electrónico de entrega de la clave de prueba.
- **Hash de la huella digital del hardware** (§5.2): para aplicar el límite de
  activación por dispositivo.
- **Metadatos de conexión transmitidos en solicitudes HTTP salientes** durante el
  registro de la prueba (al webhook de prueba alojado en Vercel) y durante la
  activación y el keep-alive de la licencia (a la API de Keygen.sh, §5.6) —que
  comprenden el identificador de licencia, el hash de la huella digital del hardware,
  marcas de tiempo y la dirección IP necesariamente incluida en cualquier solicitud
  HTTP saliente— con el fin de entregar la licencia y aplicar el límite de sesiones
  simultáneas durante el período de prueba.

Cada fin cuenta con el consentimiento por separado. El consentimiento a cada fin es
necesario para que la Prueba funcione como se describe.

Este consentimiento puede retirarse en cualquier momento contactando con
[support@lactuca.io](mailto:support@lactuca.io). La retirada del
consentimiento dará lugar a la desactivación inmediata de la clave de prueba. Para
facilitar la retirada, el Licenciatario también puede utilizar el enlace de exclusión
incluido en el correo electrónico de entrega de la clave de prueba. De conformidad con
el Art. 7(3) RGPD, la retirada del consentimiento no afectará a la licitud del
tratamiento basado en el consentimiento previo a su retirada.

### 2.6 Derecho de desistimiento y prueba

La Prueba se ofrece gratuitamente; no surge ninguna obligación de pago y no es
aplicable el derecho de desistimiento previsto en el Art. 16(m) de la Directiva
2011/83/UE respecto a la Prueba en sí. Si el Licenciatario adquiere posteriormente una
suscripción de pago, se aplicará el §9 del presente Contrato a dicha compra.

---

## 3. Restricciones

El Licenciatario no podrá:

(a) Copiar, modificar, fusionar, publicar o distribuir el Software salvo en los
    términos expresamente permitidos en el presente documento. **Nada en esta cláusula
    prohíbe al Licenciatario realizar una única copia de seguridad del Software en la
    medida permitida por el Art. 5(2) de la Directiva 2009/24/CE (Art. 99(a) TRLPI),
    en tanto se trata de un derecho imperativo que no puede excluirse por contrato.**

(b) Realizar ingeniería inversa, descompilar, desensamblar o intentar de cualquier otro
    modo obtener el código fuente, los algoritmos o el material criptográfico de
    cualquier componente binario compilado del Software (`.so`, `.pyd` u otra extensión
    compilada), **salvo exclusivamente en la medida exigida por la ley imperativa
    aplicable que no pueda excluirse por acuerdo (incluido el derecho de
    interoperabilidad previsto en el Art. 6 de la Directiva 2009/24/CE, con sujeción a
    sus condiciones legales). Antes de ejercer cualquiera de esos derechos, el
    Licenciatario deberá solicitar por escrito al Licenciante la información técnica
    pertinente.**

(c) Eludir, deshabilitar o manipular cualquier mecanismo de validación de licencia,
    firma digital, verificación de la huella digital del hardware o señal de keep-alive
    de sesiones simultáneas integrada en el Software o utilizada por este. **Esta
    prohibición no se aplica en la medida estrictamente necesaria para ejercer derechos
    imperativos conforme a la ley aplicable (incluidos los derechos de
    interoperabilidad y copia de seguridad previstos en los Arts. 5 a 6 de la Directiva
    2009/24/CE).**

(d) Transferir, ceder, sublicenciar, arrendar, alquilar, prestar o transmitir de
    cualquier otro modo la licencia o cualquier clave de activación a un tercero. Las
    licencias Individual y Académica y Comunitaria son estrictamente personales. Las
    licencias Team y Enterprise no pueden transferirse fuera de la organización
    contratante.

(e) Utilizar el Software para cualquier fin ilícito o en contravención de cualquier ley
    o reglamento aplicable.

---

## 4. Restricciones de la licencia Académica y Comunitaria

Las licencias concedidas en el nivel **Académica y Comunitaria** se otorgan
exclusivamente para uso no comercial, académico, de investigación o de comunidad de
código abierto. Los siguientes usos están **estrictamente prohibidos** bajo una licencia
Académica y Comunitaria:

- Proyectos de consultoría o entregables facturados a terceros orientados al cliente.
- Integración en sistemas de producción utilizados por o en nombre de entidades
  comerciales.
- Cualquier actividad que genere ingresos, directa o indirectamente.

La infracción de esta cláusula constituye un incumplimiento esencial y da lugar a la
revocación inmediata de la licencia. No se reembolsará ni indemnizará ningún importe,
ya que no se realizó ningún pago por este nivel.

---

## 5. Activación, huella digital del hardware y medidas técnicas de protección

### 5.1 Medidas técnicas de protección

El Software incorpora un sistema de gestión de licencias (gestionado por Keygen.sh)
que requiere una clave de licencia válida para funcionar. Se aplican las siguientes
medidas técnicas de protección:

| Medida | Finalidad |
|---|---|
| Huella digital del hardware (hash unidireccional) | Aplicar los límites de activación por dispositivo |
| Archivo de licencia firmado digitalmente | Prevenir la manipulación del archivo de licencia |
| Validación en línea periódica (cada 30 días) | Detectar el uso compartido no autorizado de claves |
| Seguimiento de sesiones simultáneas (keep-alive aprox. cada 10–20 min, según nivel) | Aplicar los límites de sesiones simultáneas por nivel |
| Distribución en formato binario compilado únicamente (sin código fuente) | Proteger la implementación propietaria |

### 5.2 Huella digital del hardware

La activación implica la recogida de una huella digital del hardware: un hash
criptográfico unidireccional derivado de los atributos de hardware del dispositivo
(dirección MAC, nombre de host, arquitectura de CPU). Los **valores de hardware brutos
nunca se transmiten** —solo el hash irreversible se envía y almacena en Keygen.sh—. El
hash resultante constituye **datos seudonimizados** en el sentido del Art. 4(5) del
RGPD: no puede identificar directamente al Licenciatario como individuo sin acceso a
información adicional que el Licenciante no conserva.

### 5.3 Interoperabilidad

El Software es compatible con:

- **Sistemas operativos**: Windows 10/11 (x86-64), macOS 11+ (ARM64), Linux
  (glibc 2.28+, x86-64)
- **Versiones de Python**: 3.12 y superiores
- **Dependencias**: instaladas automáticamente por pip; la lista actual y los
  requisitos de versión mínima se publican en la
  [guía de instalación](https://www.lactuca.io/latest/user_guide/getting_started.html#installation)

El Software no requiere ningún controlador a nivel de sistema ni extensión del núcleo
del sistema operativo. No interactúa con ningún otro software actuarial de terceros.

### 5.4 Sustitución de hardware

Si el Licenciatario reemplaza o modifica sustancialmente el hardware de un dispositivo,
la activación existente en dicho dispositivo quedará invalidada. El Licenciatario podrá
desactivar dispositivos y reactivar en el nuevo hardware tal como se describe en la
[documentación de activación](activation). El número total de dispositivos activos
simultáneamente no podrá superar en ningún momento el límite del nivel aplicable.

### 5.5 Privacidad

Los detalles completos sobre qué datos personales se recaban y cómo se tratan figuran
en la [Política de Privacidad](privacy_es), que forma parte integrante del presente
Contrato.

### 5.6 Datos personales transmitidos durante el seguimiento de sesiones simultáneas (keep-alive)

Además de los datos descritos en §5.2 recabados en el momento de la activación, cada
instancia en ejecución del Software transmite una solicitud HTTP de keep-alive
(*heartbeat*) a la API de Keygen.sh aproximadamente cada 10 minutos en niveles de
un solo puesto y cada 20 minutos en Team y Enterprise (§6.1(b)). Cada
solicitud de este tipo contiene:

- El identificador de la clave de licencia (o un hash no reversible del mismo);
- El hash de la huella digital del hardware (§5.2) del dispositivo en el que se
  ejecuta el proceso;
- Una marca de tiempo de la solicitud; y
- La dirección IP del dispositivo, necesariamente incluida en cualquier solicitud
  HTTP saliente.

En las solicitudes de heartbeat no se transmiten valores de hardware brutos, nombres de
usuario, direcciones de correo electrónico ni datos de nivel de aplicación.

**Base jurídica:** Para los licenciatarios de pago, este tratamiento se basa en el
**Art. 6(1)(b) RGPD** (ejecución del contrato). Para los licenciatarios de Prueba, la
base es el **Art. 6(1)(a) RGPD** (consentimiento prestado conforme al §2.5). Keygen.sh
trata estos datos exclusivamente como **encargado del tratamiento** en el sentido del
Art. 28 RGPD, en virtud de un Acuerdo de Tratamiento de Datos con el Licenciante. La
transferencia a Keygen.sh (Estados Unidos) se realiza sobre la base de las **Cláusulas
Contractuales Tipo** (Decisión de Ejecución de la Comisión (UE) 2021/914). El
Licenciatario reconoce que esta funcionalidad es prestada por **Keygen.sh (Keygen LLC)**
bajo sus propios [Términos de Servicio](https://keygen.sh/terms) y
[Política de Privacidad](https://keygen.sh/privacy). Los detalles completos sobre los
plazos de conservación y las disposiciones relativas a los subencargados figuran en la
[Política de Privacidad](privacy_es), que forma parte integrante del presente Contrato
(§5.5).

Si el Licenciante tuviera conocimiento de un cambio material en las prácticas de datos
de Keygen.sh que reduzca el nivel de protección descrito en esta sección, el Licenciante
lo notificará a los suscriptores activos por correo electrónico en el plazo de **30
días** desde que tenga conocimiento de ello. Los suscriptores Enterprise con contrato
directo anual que no deseen continuar en las condiciones modificadas podrán rescindir
el contrato en el plazo de 30 días desde la notificación y recibirán un reembolso
prorrateado de cualquier tarifa anual prepagada (§13.2).

---

## 6. Períodos de gracia sin conexión y continuidad del servicio

### 6.1 Períodos de gracia sin conexión

El Software mantiene dos períodos de gracia sin conexión independientes:

(a) **Período de gracia de validación de licencia (intervalo de revalidación de 30 días)**:
    el Software intenta revalidar la licencia en línea aproximadamente cada 30 días.
    Si no hay conexión a internet disponible en el momento de la revalidación, el
    Software puede seguir operando mientras la licencia almacenada localmente no haya
    alcanzado su fecha de expiración, con avisos periódicos al inicio. No existe un
    corte offline adicional más allá de la fecha de expiración almacenada. Una vez
    superada la fecha de expiración almacenada localmente, el Software no funcionará
    independientemente de la disponibilidad de red.

(b) **Período de gracia de sesión simultánea (3 días)**: cada instancia en ejecución
    del Software envía una señal de keep-alive al servidor de licencias aproximadamente
    cada 10 minutos en niveles de un solo puesto y cada 20 minutos en Team y
    Enterprise para mantener su sesión activa. Si el servidor no puede ser
    alcanzado:

    - Un **proceso ya en ejecución** en el momento del fallo de red sigue funcionando
      sin interrupción durante toda su vida útil.

    - Un **nuevo inicio** del Software se permite sin acceso a la red hasta **3 días**
      desde el último contacto exitoso con el servidor. Tras 3 días continuos sin
      conectividad, los nuevos inicios fallarán con un error de conexión hasta que se
      restablezca la red.

Ambos períodos de gracia son independientes. En la práctica, el período de gracia de
sesión simultánea (3 días) es la restricción vinculante cuando la red no está disponible
durante un período prolongado.

### 6.2 Compromiso de continuidad del servicio

El Licenciante se compromete a mantener el servicio de validación de licencias en línea
(actualmente prestado por Keygen.sh) durante la vigencia de cualquier suscripción de
pago activa. En el caso de que el Licenciante no pueda mantener el servicio de
validación (ya sea debido a la descontinuación del Software, un cambio de proveedor de
gestión de licencias u otra razón), el Licenciante:

(a) proporcionará al Licenciatario al menos **90 días de aviso previo por escrito** por
    correo electrónico; y

(b) a opción del Licenciatario, bien (i) emitirá una activación offline perpetua que no
    requiera validación en línea por el período restante de pago; o bien (ii) ofrecerá
    un reembolso prorrateado de cualquier suscripción prepagada no vencida.

### 6.3 Migración de proveedor

Si el Licenciante migra a un nuevo proveedor de gestión de licencias, las claves de
licencia existentes seguirán siendo válidas sin necesidad de que el Licenciatario
realice una nueva compra. El Licenciante proporcionará instrucciones de migración al
menos 30 días antes de que se requiera cualquier migración de claves.

---

## 7. Actualizaciones y política de versiones

Las suscripciones de pago incluyen todas las versiones menores y de corrección dentro
de la versión mayor suscrita. Las actualizaciones de versiones mayores pueden requerir
una licencia o renovación de suscripción independiente. El Licenciante se reserva el
derecho de definir el límite entre versiones menores y mayores. Las licencias de Prueba
gratuita no incluyen derechos de actualización más allá del período de Prueba.

No obstante lo anterior, las **actualizaciones de seguridad** necesarias para mantener
la conformidad del Software se suministrarán a los Licenciatarios consumidores de la
UE/EEE sin cargo adicional durante la vigencia de la suscripción activa,
independientemente de si se clasifican como versiones menores o mayores, según lo
exigido por el Art. 8 de la Directiva 2019/770/UE (implementada en el Art. 115 quater
TRLGDCU).

---

## 8. Suspensión y resolución por incumplimiento

### 8.1 Causas

El Licenciante podrá suspender o resolver el presente Contrato con efecto inmediato y
notificación escrita al Licenciatario, si este:

(a) incumple de forma esencial cualquier disposición del presente Contrato y, cuando el
    incumplimiento sea susceptible de subsanación, no lo subsana en el plazo de **14
    días naturales** desde la notificación escrita del Licenciante especificando el
    incumplimiento;

(b) elude, deshabilita o manipula deliberadamente cualquier mecanismo de validación de
    licencia o de protección de copia;

(c) facilita información materialmente falsa durante el proceso de registro de la prueba
    o de compra; o

(d) supera intencionada y reiteradamente los límites de dispositivos o sesiones
    simultáneas del nivel con licencia, o elude deliberadamente el mecanismo de
    seguimiento de sesiones simultáneas descrito en §5.1 y §6.1(b) (por ejemplo,
    parcheando o deshabilitando el mecanismo de señal de keep-alive) con el fin de
    obtener más sesiones simultáneas de las permitidas.

Un único exceso involuntario del límite de dispositivos (p. ej., causado por un fallo
de hardware y su sustitución) no constituye causa de resolución; el Licenciante
solicitará en primer lugar que el Licenciatario desactive el dispositivo en exceso.

### 8.2 Reembolso en caso de resolución por incumplimiento

Cuando el Licenciante resuelva el presente Contrato conforme al §8.1:

(a) **Suscriptores mensuales**: no se realizará ningún reembolso; la licencia
    permanecerá activa hasta el final del período de facturación en curso, tras el cual
    se revocará el acceso.

(b) **Suscriptores con contrato directo anual** (únicamente contratos anuales Enterprise
    y OEM): el Licenciante reembolsará la parte prorrateada no utilizada de la tarifa anual
    prepagada, calculada desde la fecha de resolución, **salvo** que la resolución sea
    causada por elusión deliberada (§8.1(b)), facilitación de información materialmente
    falsa (§8.1(c)), elusión intencional del mecanismo de seguimiento de sesiones
    simultáneas (§8.1(d)) o uso indebido OEM en violación del §16 (uso del Software
    para dar servicio a usuarios externos a la organización del Licenciatario sin un
    Acuerdo OEM), en cuyo caso no se realizará ningún reembolso.

### 8.3 Notificación

El Licenciante notificará al Licenciatario por correo electrónico cualquier suspensión
o resolución. Cuando el incumplimiento sea susceptible de subsanación conforme al
§8.1(a), el Licenciante proporcionará el aviso de subsanación de 14 días antes de
adoptar cualquier medida que deshabilite el Software.

---

## 9. Derecho de desistimiento (consumidores de la UE y el EEE — solo licencias de pago)

*Esta sección se aplica exclusivamente a las compras de licencias de pago procesadas a
través de Lemon Squeezy (planes Individual y Team, y plan Enterprise mensual). No se
aplica a las activaciones de Prueba gratuita (véase §2.6), a las licencias Académicas y
Comunitarias (sin pago) ni a los contratos anuales Enterprise (véase §11.5).*

### 9.1 Derecho legal de desistimiento

Los consumidores residentes en la Unión Europea o el Espacio Económico Europeo disponen
normalmente de un derecho legal de desistimiento de 14 días en los contratos a
distancia, de conformidad con la Directiva 2011/83/UE y la legislación nacional de
aplicación (en España: Real Decreto Legislativo 1/2007, Arts. 68 a 79 TRLGDCU).

### 9.2 Renuncia al desistimiento al activar o descargar contenido digital

No obstante, de conformidad con el Art. 16(m) de la Directiva 2011/83/UE (y el
Art. 103(m) del TRLGDCU para los consumidores en España), el derecho de desistimiento
no se aplica a los contratos de suministro de contenido digital no facilitado en un
soporte material cuando:

(a) la ejecución haya comenzado con el **consentimiento previo y expreso** del
    consumidor; y

(b) el consumidor haya **reconocido** que, de este modo, pierde el derecho de
    desistimiento.

Al completar la compra a través de Lemon Squeezy y activar o descargar el Software, el
Licenciatario:

(i)  presta su **consentimiento previo y expreso** al suministro inmediato del contenido
     digital (Lactuca); y

(ii) **reconoce** que, una vez entregada la clave de licencia y comenzado el proceso de
     descarga o activación, el derecho de desistimiento queda extinguido.

Estos reconocimientos son registrados por Lemon Squeezy en el momento del pago y están
disponibles a petición en
[support@lactuca.io](mailto:support@lactuca.io).

**A efectos aclaratorios, la renuncia establecida en el presente §9.2 no afecta ni
limita los derechos del Licenciatario respecto a la falta de conformidad del Software
en virtud de la Directiva 2019/770/UE y del §10.1 del presente Contrato. Esos derechos
de conformidad son independientes del derecho de desistimiento y permanecen plenamente
inalterados.**

### 9.3 Reembolso antes de la activación

Si el Licenciatario ha adquirido una licencia pero no ha activado ni descargado aún el
Software, podrá solicitar un reembolso completo en el plazo de 14 días naturales desde
la compra contactando con
[support@lactuca.io](mailto:support@lactuca.io). Una vez iniciada la
activación, se aplica la renuncia del §9.2.

### 9.4 Consumidores fuera de la UE/EEE

Para los consumidores fuera de la UE/EEE, la elegibilidad para el reembolso se rige
por la política de reembolso estándar de Lemon Squeezy, que se aplica en calidad de
Merchant of Record. Las obligaciones de reembolso del Licenciante se limitan a las
impuestas por la ley imperativa de la jurisdicción correspondiente.

---

## 10. Garantías y limitación de responsabilidad

### 10.1 Garantía legal de conformidad (consumidores de la UE/EEE)

Cuando el Licenciatario sea un consumidor en el sentido de la Directiva (UE) 2019/770
y la ley nacional aplicable (en España: Arts. 115 bis y siguientes del TRLGDCU
modificado por el RDL 7/2021), el Licenciante garantiza que el Software se ajustará a
su descripción y será apto para los fines de cálculo actuarial establecidos en la
documentación durante la vigencia de la suscripción activa. En caso de falta de
conformidad, el Licenciatario tendrá derecho a:

(a) que la falta de conformidad sea subsanada de forma gratuita en un plazo razonable;
    y

(b) si la falta de conformidad no se subsana en un plazo razonable o no puede
    subsanarse, una reducción proporcional en el precio de la suscripción o la
    resolución del Contrato con el reembolso íntegro de los importes pagados.

Nada de lo contenido en el presente Contrato limita o excluye ningún derecho o recurso
disponible para los consumidores en virtud de la ley imperativa aplicable.

### 10.2 Responsabilidad profesional del Licenciatario (todos los niveles)

El Software es una herramienta de cálculo actuarial profesional destinada a su uso por
parte de actuarios cualificados o bajo su supervisión directa. Al utilizar el Software,
el Licenciatario garantiza que dispone de la competencia profesional necesaria para:

(a) comprender los métodos, supuestos y fórmulas actuariales implementados en el
    Software, tal como se documentan en la documentación técnica;

(b) evaluar la adecuación de dichos métodos y supuestos para el fin específico para el
    que se utiliza el Software;

(c) verificar y validar de forma independiente todos los resultados de los cálculos
    antes de utilizarlos para cualquier fin profesional, reglamentario, legal o de
    supervisión; y

(d) asumir la exclusiva responsabilidad profesional por las conclusiones, opiniones,
    informes o decisiones basados en los resultados del Software.

El Licenciatario reconoce que el Software implementa métodos actuariales generales
basados en la práctica actuarial internacional y las normas actuariales españolas e
internacionales aplicables, pero que **los cálculos actuariales realizados con el
Software no sustituyen a una opinión, revisión o certificación actuarial independiente**
por parte de un actuario debidamente cualificado.

### 10.3 Sin garantía para fines regulatorios específicos (todos los niveles)

**El Software no está certificado, aprobado ni validado por ninguna autoridad
regulatoria.** El Licenciante renuncia expresamente, en la máxima medida permitida por
la ley aplicable, a cualquier garantía o declaración —expresa, implícita o legal— de
que el Software o cualquier resultado del mismo sea adecuado, apropiado o suficiente
para cualquier fin regulatorio, de supervisión, legal o de cumplimiento específico,
incluyendo sin limitación:

- **NIIF 17** (*Norma Internacional de Información Financiera 17 — Contratos de
  Seguro*): valor actual de los flujos de efectivo futuros, ajuste de riesgo, margen
  de servicio contractual (MSC), mejor estimación de los pasivos (BEL) u otro modelo
  de medición de la NIIF 17 (MGM, EPS, EVA);

- **Solvencia II**: Requisito de Capital de Solvencia (SCR), Requisito
  de Capital Mínimo (MCR), mejor estimación de las provisiones técnicas, margen de
  riesgo, módulos de la fórmula estándar (mortalidad, longevidad, incapacidad,
  rescate, gastos), validación del modelo interno u ORSA;

- **Provisiones técnicas del seguro de vida español** (*Reglamento de Ordenación y
  Supervisión de los Seguros Privados*): reservas matemáticas (*reserva matemática*),
  equivalencia actuarial o cualquier otra provisión exigida por el ROSSP o las
  regulaciones de la DGSFP;

- cualquier otro requisito legal, reglamentario, de supervisión o de información
  impuesto por cualquier autoridad competente en cualquier jurisdicción.

El uso del Software en relación con cualquiera de los fines anteriores no exime al
Licenciatario de la obligación de obtener revisión y certificación actuarial
independiente cuando así lo exija la ley, la regulación o las normas profesionales
aplicables (incluidas las normas del Instituto de Actuarios Españoles o cualquier otra
asociación actuarial competente).

La responsabilidad del Licenciante por cualquier pérdida, daño, sanción, multa o
sanción regulatoria que surja de la confianza del Licenciatario en el Software para
cualquiera de los fines enumerados anteriormente, sin la verificación independiente
exigida por el §10.2, queda **excluida en la máxima medida permitida por la ley
aplicable**.

### 10.4 Limitación de responsabilidad (uso B2B y no consumidor)

*Esta sección se aplica adicionalmente a los §10.2 y §10.3, y con independencia de la
garantía para consumidores del §10.1.*

Cuando el Licenciatario utilice el Software exclusivamente en el marco de una actividad
comercial, empresarial o profesional (es decir, como no consumidor), el Software se
suministra «tal cual» sin garantías implícitas de comerciabilidad, idoneidad para un
fin particular o no vulneración, en la máxima medida permitida por la ley aplicable.
La responsabilidad total agregada del Licenciante por cualquier reclamación derivada del
presente Contrato o relacionada con el mismo no superará los importes pagados por el
Licenciatario en los **doce meses** anteriores a la reclamación.

Esta limitación no se aplica a la responsabilidad por:

(a) muerte o lesiones personales causadas por negligencia del Licenciante;

(b) fraude o declaración fraudulenta;

(c) cualquier otra responsabilidad que no pueda limitarse ni excluirse conforme a la
    ley aplicable.

### 10.5 Disponibilidad

El Licenciante no garantiza la disponibilidad ininterrumpida del Software ni del
servicio de validación de licencias. La indisponibilidad temporal del servidor de
validación en línea no constituye una falta de conformidad, siempre que el período de
gracia sin conexión del §6.1 siga siendo efectivo.

---

## 11. Suscripción, renovación y cancelación

### 11.1 Suscripciones mensuales

Las suscripciones mensuales se renuevan automáticamente el mismo día de cada mes
natural. El Licenciatario puede cancelar en cualquier momento a través del portal de
clientes de Lemon Squeezy (accesible mediante el enlace incluido en cualquier correo
electrónico de confirmación de suscripción). La cancelación surte efecto al **final del
mes de facturación en curso**; no se realizará ningún reembolso por los días restantes
del mes en curso.

### 11.2 Suscripciones anuales (Individual y Team)

Las suscripciones anuales se renuevan automáticamente 12 meses después de la fecha de
compra inicial. El Licenciante enviará un correo electrónico de aviso de renovación al
menos **14 días** antes de la fecha de renovación. Adicionalmente, el Software muestra
un mensaje de aviso en la aplicación cuando queden **30 días o menos** antes de la
expiración. El Licenciatario puede cancelar antes de la fecha de renovación a través
del portal de clientes de Lemon Squeezy; no se realizará ningún reembolso por el período
anual en curso ya pagado, salvo que resulten aplicables los §10.1 o §13.2.

### 11.3 Cambios de precio

El Licenciante podrá cambiar los precios de las suscripciones con al menos **30 días de
aviso previo** antes de la siguiente fecha de renovación. Si el Licenciatario no acepta
el nuevo precio, podrá cancelar antes de la próxima fecha de renovación sin penalización.

### 11.4 Efectos de la cancelación o la no renovación

Tras la cancelación o la no renovación:

(a) La clave de licencia permanecerá activa hasta el final del período pagado.

(b) Tras la expiración, el Software mostrará un error de licencia y dejará de funcionar
    hasta que se active una nueva licencia.

(c) Los archivos de licencia almacenados localmente podrán ser desactivados por
    Keygen.sh una vez transcurrido el período de gracia.

### 11.5 Contratos anuales Enterprise

Las suscripciones Enterprise —ya sean facturadas mensualmente a través de Lemon Squeezy
o anualmente mediante contrato directo— confieren **derechos de uso idénticos**: el
mismo grupo de activaciones (50 activaciones de máquina), el mismo máximo de 50
sesiones simultáneas y los mismos escenarios de despliegue permitidos (instalación
directa en estación de trabajo y despliegue en servidor o compartido para usuarios
internos). La diferencia entre Enterprise mensual y anual es únicamente el canal de
facturación y la formalidad contractual.

Las suscripciones anuales Enterprise se rigen por un **acuerdo escrito independiente**
(Formulario de Pedido o Contrato Marco de Licencia) celebrado directamente entre el
Licenciatario y el Licenciante, al margen de la plataforma Lemon Squeezy. Los términos
de cancelación, período de aviso de renovación, política de reembolso y compromisos de
nivel de servicio se establecen exclusivamente en dicho acuerdo independiente. En caso
de conflicto entre el presente Contrato y el contrato Enterprise, el contrato Enterprise
prevalecerá respecto a los términos de suscripción del Licenciatario Enterprise.

---

## 12. Propiedad intelectual

Los binarios compilados del Software, los archivos de definición de tipos (`.pyi`) y
la documentación son propiedad intelectual exclusiva de Alberto Aragoneses Nebreda.
Todos los derechos no concedidos expresamente en el presente documento quedan
reservados. El presente Contrato no transfiere ningún derecho de propiedad sobre el
Software.

El **formato de tablas Lactuca** (especificación del formato binario `.ltk` y todo el
código que lee, escribe o interpreta archivos `.ltk`) es propiedad intelectual exclusiva
de Alberto Aragoneses Nebreda.

Los **valores de datos actuariales** contenidos en los archivos `.ltk` incluidos se
derivan de fuentes públicas (incluidas las tablas publicadas por el Instituto de
Actuarios Españoles, la Dirección General de Seguros y Fondos de Pensiones y organismos
internacionales equivalentes). Dichos valores de hecho no se reclaman como propiedad
intelectual del Licenciante.

La documentación (texto, fórmulas y ejemplos en `docs/`) se licencia por separado bajo
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), salvo que se indique lo
contrario.

---

## 13. Modificaciones del presente Contrato

### 13.1 Aviso de modificaciones

El Licenciante podrá actualizar el presente Contrato en cualquier momento. El
Licenciatario será notificado de cualquier modificación sustancial:

(a) por correo electrónico a la dirección facilitada al registrarse, al menos **30
    días** antes de la entrada en vigor del cambio; y

(b) mediante un aviso destacado en esta página de documentación.

### 13.2 Aceptación o rechazo por parte de suscriptores de pago

Si el Licenciatario es un suscriptor de pago (Individual, Team o Enterprise) y no
acepta los términos modificados, podrá **resolver el contrato** antes de la fecha de
entrada en vigor del cambio. En ese caso:

- **Suscriptores mensuales**: tendrán derecho a utilizar el Software hasta el final del
  mes de facturación en curso; no se realizarán más cargos.
- **Suscriptores anuales**: tendrán derecho a un reembolso prorrateado de la parte no
  utilizada de su suscripción anual prepagada, calculado desde la fecha efectiva de
  resolución.

El uso continuado del Software tras la fecha de entrada en vigor implicará la aceptación
de los términos actualizados, siempre que el Licenciatario haya recibido la notificación
prevista en el §13.1(a) y no haya ejercido el derecho de resolución dentro del período
de aviso de 30 días.

### 13.3 Modificaciones no sustanciales

Las modificaciones que se limiten a corregir errores tipográficos, actualizar datos de
contacto o aclarar términos existentes sin alterar derechos sustanciales surtirán efecto
de forma inmediata tras su publicación y no requieren un período de aviso de 30 días.

### 13.4 Licenciatarios de Prueba y Académica y Comunitaria

Las licencias de Prueba y Académica y Comunitaria se conceden gratuitamente. El
Licenciante podrá modificar los términos aplicables en cualquier momento con aviso
razonable. El uso continuado tras el aviso implicará la aceptación. Si el Licenciatario
no acepta los términos modificados, deberá dejar de utilizar el Software.

A efectos aclaratorios, los datos personales tratados en relación con la Prueba se
recaban únicamente porque son técnicamente necesarios para entregar y gestionar la
licencia de Prueba —no constituyen una contraprestación comercial por el Software—. No
se realiza ningún uso con fines de marketing, elaboración de perfiles ni explotación
comercial de los datos de los Licenciatarios de Prueba.

---

## 14. Legislación aplicable y jurisdicción

### 14.1 Uso B2B y profesional

Cuando el Licenciatario utilice el Software exclusivamente en el marco de una actividad
comercial, empresarial o profesional (es decir, como no consumidor), el presente
Contrato se regirá por la legislación española y las partes se someten irrevocablemente
a la jurisdicción exclusiva de los juzgados y tribunales de **Barcelona, España**.

### 14.2 Uso por consumidores (UE/EEE)

Cuando el Licenciatario sea un consumidor en el sentido de la legislación de la UE
aplicable, el presente Contrato se regirá por la legislación española. No obstante, nada
en el presente Contrato priva al Licenciatario de la protección que le otorgan las
disposiciones imperativas de la ley del país en el que resida habitualmente, ni del
derecho a entablar procedimientos judiciales en los tribunales de dicho país, conforme
a los Arts. 17 a 19 del Reglamento (UE) 1215/2012 (Bruselas I bis).

### 14.3 Uso por consumidores (fuera de la UE/EEE)

Para los consumidores fuera de la UE/EEE, se aplicarán las leyes de protección del
consumidor imperativas del país de residencia habitual del consumidor en la medida en
que no puedan derogarse por contrato.

---

## 15. Resolución alternativa de litigios en línea (UE)

La Comisión Europea ofrece una plataforma de resolución de litigios en línea (ODR)
para la resolución extrajudicial de litigios entre consumidores y comerciantes
establecidos en la UE:

**Plataforma ODR**:
[https://ec.europa.eu/consumers/odr](https://ec.europa.eu/consumers/odr)

Correo electrónico del Licenciante para la resolución de litigios en línea:
[support@lactuca.io](mailto:support@lactuca.io)

El Licenciante **no está afiliado a ningún organismo de resolución alternativa de
litigios (RAL)** registrado conforme a la Directiva 2013/11/UE o la legislación
nacional de aplicación (en España: Real Decreto 231/2008 y Ley 7/2017). El Licenciante
procurará resolver las reclamaciones de forma amistosa antes de iniciar cualquier
procedimiento formal.

---

## 16. Cumplimiento de la licencia

### 16.0 Ámbito de esta sección

Los límites de sesiones simultáneas y activaciones de dispositivos se aplican automática
y continuamente mediante los mecanismos técnicos del §5 (huella digital del hardware)
y §6.1 (seguimiento de sesiones simultáneas). No se requiere ni se pretende ninguna
auditoría de dichos límites.

Esta sección aborda los dos tipos de uso indebido que **no pueden detectarse
técnicamente**: (i) el uso del Software para dar servicio a usuarios externos a la
organización del Licenciatario sin un Acuerdo OEM; y (ii) el uso bajo una licencia
Académica y Comunitaria con fines comerciales.

### 16.1 Uso indebido OEM y Académico

El Licenciatario declara y garantiza en todo momento durante la vigencia del presente
Contrato que:

(a) Si el Licenciatario tiene una licencia de nivel estándar (Individual, Team o
    Enterprise), el Software no se está utilizando como motor de cálculo en ningún
    producto o servicio puesto a disposición de usuarios externos a la organización del
    Licenciatario, según la definición del §1.

(b) Si el Licenciatario tiene una licencia Académica y Comunitaria, el Software no se
    está utilizando en ninguna actividad que genere ingresos, directa o indirectamente,
    tal como se especifica en el §4.

La infracción de cualquiera de las declaraciones anteriores constituye un incumplimiento
esencial que faculta al Licenciante a resolver la licencia de forma inmediata conforme
al §8. Las infracciones OEM están sujetas a resolución sin reembolso (§8.2(b)). Para
obtener una licencia OEM, contacte con
[support@lactuca.io](mailto:support@lactuca.io) con una descripción
del producto, el número estimado de usuarios finales externos y el modelo de negocio
previsto. Las licencias OEM se negocian individualmente.

### 16.2 Subsanación

Si el Licenciante tiene motivos razonables para creer que se está produciendo una
infracción del §16.1, el Licenciante notificará al Licenciatario por escrito. El
Licenciatario deberá:

(a) adquirir sin demora el nivel o Acuerdo OEM adecuado para cubrir el uso real, con
    carácter retroactivo desde la fecha en que comenzó la infracción; y

(b) pagar las tarifas devengadas por el período en exceso al precio de lista vigente en
    el momento de la notificación, sin perjuicio del derecho del Licenciante a resolver
    el contrato conforme al §8 en casos de uso indebido deliberado.

---

## 17. Contacto

Para consultas sobre licencias, asistencia técnica o notificaciones legales:

**Correo electrónico**:
[support@lactuca.io](mailto:support@lactuca.io)

**Sitio web**: [www.lactuca.io](https://www.lactuca.io)

---

*La versión en inglés del presente Contrato (End User License Agreement) está
disponible en [eula](eula).*

*Prevalencia de versiones: para **consumidores con residencia habitual en España**,
esta versión española prevalecerá en caso de discrepancia. Para todos los demás
Licenciatarios, prevalecerá la **versión en inglés**.*
