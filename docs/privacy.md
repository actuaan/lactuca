# Privacy Policy

**Lactuca — Life Actuarial Calculation Library for Python**

*Effective date: 2026-04-01. Last updated: 2026-06-19.*
*Data controller: Alberto Aragoneses Nebreda, operating under the brand Actuaan.*
*Contact: [support@lactuca.io](mailto:support@lactuca.io)*

---

This Privacy Policy explains what personal data is collected when you use Lactuca or
start a free trial, where it is stored, on what legal basis it is processed, and what
rights you have as a data subject under the General Data Protection Regulation (GDPR —
EU Regulation 2016/679).

---

## 1. Data controller

The data controller for all personal data described in this policy is:

> Alberto Aragoneses Nebreda (Actuaan)
> NIF: 09180714B
> Address: Francesc Pérez Cabrero, 7, 08021, Barcelona, Spain
> Email: [support@lactuca.io](mailto:support@lactuca.io)
> Country: Spain (European Union)

---

## 2. Data we collect

### 2.1 Trial registration

When you request a free trial through the activation prompt when first importing the Software, we collect:

| Data | Purpose | Stored where |
|---|---|---|
| Email address | Deliver the trial key; license management | Keygen.sh (via Vercel webhook) |
| Optional name | Personalise the welcome email | Resend — email delivery only (via Vercel webhook) |
| Hardware fingerprint (hash) | Prevent trial abuse (one trial per device) | Keygen.sh (via Vercel webhook) |
| IP address (inherent to outbound HTTP) | Implicit in network request to trial webhook | Vercel access logs; Keygen.sh logs |

The **hardware fingerprint** is a SHA-256 hash derived from hardware attributes of the
device (MAC address, hostname, CPU architecture). The raw values are **never
transmitted** — only the irreversible hash is sent and stored. The fingerprint
constitutes **pseudonymous data** within the meaning of Art. 4(5) GDPR: it cannot
directly identify you as an individual without access to additional information that
Licensor does not retain.

:::{note}
**Trial key delivery does not register a device.** When you request a trial, we create
a license key and store your email and fingerprint hash for duplicate-trial prevention.
The device slot for your plan is **not** consumed until you successfully activate
Lactuca on that machine for the first time (see §2.2).
:::

### 2.2 License activation and validation

When you activate Lactuca (or when the library validates your license online), we collect:

| Data | Purpose | Stored where |
|---|---|---|
| License key | License authentication | Keygen.sh |
| Hardware fingerprint (hash) | Enforce device limits | Keygen.sh |
| License tier | Determine permitted usage | Keygen.sh |
| Activation timestamp | Grace-period and expiry tracking | Keygen.sh |
| Device registration record (fingerprint hash) | Link the license to this device; enforce per-plan device limits | Keygen.sh |
| IP address (inherent to outbound HTTP) | Implicit in network request | Keygen.sh logs |

:::{important}
**Device registration happens at first activation, not when the key is issued.**
Receiving a trial or paid license key does not register your computer. The first
successful activation on a machine creates a **device registration record** on the
license server, associated with that machine's fingerprint hash. Each registered device
counts toward your plan's device limit (for example, one device for Trial and
Individual plans). Transferring to another computer requires freeing a slot first — see
the {ref}`device transfer procedure <device-transfer>` in the Licensing FAQ.
:::

### 2.3 Concurrent session tracking (heartbeat)

While an instance of the Software is running, it sends a keep-alive HTTP
request (*heartbeat*) to the Keygen.sh API approximately every 10 minutes on
single-seat tiers and every 20 minutes on Team and Enterprise (derived from the
license server's process policy).
Each heartbeat request contains:

| Data | Purpose | Stored where |
|---|---|---|
| License key identifier (or hash) | Identify the license session | Keygen.sh |
| Hardware fingerprint hash | Associate the session with the licensed device | Keygen.sh |
| Request timestamp | Session keep-alive tracking | Keygen.sh |
| IP address (inherent to outbound HTTP) | Implicit in network request | Keygen.sh logs |

No raw hardware values, user names, email addresses, or application data are
transmitted in heartbeat requests. **Retention:** API request logs (IP address
and timestamps) are automatically deleted after 14 days; activity and event
logs after 30 days; license state data for the duration of the active license.

### 2.4 Purchase and billing

Payment data (card number, billing address) is collected exclusively by
**Lemon Squeezy** (our Merchant of Record). We never see or store your card details.
Lemon Squeezy provides us with:

| Data | Purpose | Stored where |
|---|---|---|
| Email address | License delivery; subscription management | Keygen.sh |
| Subscription status | License renewal, suspension, and revocation | Keygen.sh |
| Country/jurisdiction | VAT compliance | Lemon Squeezy (not transferred to Licensor) |

### 2.5 Data we do NOT collect

- Your real name beyond the optional welcome-email personalisation during trial registration (see §2.1)
- Browsing behaviour, usage analytics, or telemetry from within the library
- Credit or debit card numbers (handled entirely by Lemon Squeezy)
- Contents of your actuarial calculations or data files

### 2.6 Mandatory and optional data (GDPR Art. 13(2)(e))

The table below indicates, for each personal data item collected directly from you,
whether provision is required and what happens if you do not provide it.

| Data | Required? | Consequence of not providing |
|---|---|---|
| **Email address** (trial registration) | Required to enter into the trial arrangement | No trial key can be issued; the trial cannot be started |
| **Hardware fingerprint** (activation and heartbeat) | Required for activation and for enforcement of device and session limits | The Software cannot be activated, validated, or operated on the device |
| **License key** (activation and validation) | Required for Software operation | The Software will not operate |
| **Name** (trial registration) | Entirely optional | No consequence; the welcome email is sent without personalisation |

The requirement to provide email address, hardware fingerprint, and license key is a
**requirement necessary to enter into or perform the contract** (paid license or trial
arrangement). It is not a statutory obligation imposed by law. The **name** field is
voluntary at all times.

---

## 3. Legal basis for processing

| Processing activity | Legal basis (GDPR Art. 6) |
|---|---|
| Delivering a paid license and enforcing its terms | Art. 6(1)(b) — performance of a contract |
| Trial key delivery and duplicate-trial prevention | Art. 6(1)(a) — consent (given at activation prompt) |
| License validation and grace-period enforcement | Art. 6(1)(b) — performance of a contract |
| VAT compliance and financial record-keeping | Art. 6(1)(c) — legal obligation |
| Concurrent session tracking (heartbeat) | Art. 6(1)(b) — performance of contract (paid); Art. 6(1)(a) — consent (Trial) |

---

## 4. Data retention

| Data category | Retention period |
|---|---|
| Active license records (key, fingerprint, tier, device registrations) | For the duration of the active subscription |
| Expired or revoked license records | 3 years after expiry (contract performance and dispute resolution) |
| Trial email address | 1 year after the trial key expires, or until you request deletion |
| API request logs (IP address, timestamps — Keygen.sh) | 14 days (automated deletion by Keygen.sh) |
| Access logs (IP address — Vercel trial webhook) | As per Vercel's data retention policy (see [vercel.com/legal/privacy-policy](https://vercel.com/legal/privacy-policy)) |
| Activity and event logs (Keygen.sh) | 30 days (automated deletion by Keygen.sh) |
| Payment records (held by Lemon Squeezy) | As required by applicable tax law (typically 7–10 years) |

---

## 5. Sub-processors and international transfers

We use the following sub-processors to deliver the service:

| Sub-processor | Role | Location | Transfer mechanism |
|---|---|---|---|
| [Keygen.sh](https://keygen.sh) | License management platform | United States | Standard Contractual Clauses (SCCs) |
| [Lemon Squeezy](https://lemonsqueezy.com) | Payment processing (Merchant of Record) | United States | Standard Contractual Clauses (SCCs) |
| [Resend](https://resend.com) | Transactional email delivery | United States | Standard Contractual Clauses (SCCs) |
| [Vercel](https://vercel.com) | Trial registration webhook (request routing only) | United States | Standard Contractual Clauses (SCCs) |

All sub-processors listed above are contractually bound to process your data only on our
documented instructions and to implement appropriate technical and organisational security measures.

---

## 6. Your rights under GDPR

As a data subject in the EU/EEA, you have the following rights:

| Right | Description |
|---|---|
| **Access** (Art. 15) | Request a copy of the personal data we hold about you |
| **Rectification** (Art. 16) | Request correction of inaccurate data |
| **Erasure** (Art. 17) | Request deletion of your data, subject to legal retention obligations |
| **Restriction** (Art. 18) | Request that we restrict processing in certain circumstances |
| **Portability** (Art. 20) | Receive your data in a structured, machine-readable format |
| **Object** (Art. 21) | Object to processing based on legitimate interests |
| **Withdraw consent** (Art. 7(3)) | Withdraw consent for trial registration at any time (does not affect prior processing) |
| **Not to be subject to automated decisions** (Art. 22) | Request human review of any automated license denial and contest the decision |

**Automated decision-making (Art. 13(2)(f) GDPR):** License access is granted or denied automatically based on your license status and hardware fingerprint. If your license is expired, suspended, revoked, the device limit has been reached, or the fingerprint does not match the activated device, access is denied without prior human intervention. The consequence is that the Software will not operate. This automated processing is necessary for the **performance of the contract** (Art. 22(2)(a) GDPR). You have the right to obtain human intervention, express your point of view, and contest any such decision by contacting [support@lactuca.io](mailto:support@lactuca.io).

To exercise any of these rights, send an email to
[support@lactuca.io](mailto:support@lactuca.io) with the subject
**"GDPR request — \<right\>"** (e.g. "GDPR request — Erasure").

To **withdraw consent** for trial registration specifically, you may also use the
opt-out link included in the trial key delivery email, which provides a direct and
immediate withdrawal mechanism equivalent to the channel through which consent was given.

We will respond within **30 calendar days**. If your request is complex or numerous,
we may extend this period by a further 60 days and will notify you accordingly.

You also have the right to lodge a complaint with the Spanish data protection authority:

> **Agencia Española de Protección de Datos (AEPD)**
> Website: [aepd.es](https://www.aepd.es)
> Phone: +34 912 663 517

---

## 7. Security

We implement appropriate technical and organisational measures to protect your personal
data against accidental loss, unauthorised access, disclosure, or destruction, including:

- HTTPS-only communication between the library and license servers
- Ed25519 digital signatures on all license records to prevent tampering
- Hardware fingerprint hashing before transmission (raw hardware data never leaves your device)
- License API keys stored as environment variables, never hard-coded

---

## 8. Cookies and tracking

The Lactuca documentation website (`www.lactuca.io`) does not set cookies
and does not use any tracking or analytics scripts. No personal data is collected by
simply visiting the documentation.

---

## 9. Children

Lactuca is professional software intended for use by adults in an actuarial or research
context. We do not knowingly collect personal data from anyone under the age of 16.

---

## 10. Changes to this policy

We may update this Privacy Policy from time to time. Material changes will be notified
by email (using the address provided at registration) and by a prominent notice on this
page, at least 30 days before the change takes effect.

The [EULA](eula) incorporates this Privacy Policy by reference.

---

## 11. Contact

For any privacy-related questions or to exercise your rights:

**Email**: [support@lactuca.io](mailto:support@lactuca.io)
**Subject prefix**: `GDPR request —`

---

*Una versión en español de esta Política de Privacidad está disponible en
[privacy_es](privacy_es).*
