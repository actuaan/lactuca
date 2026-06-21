# End User License Agreement

**Lactuca — Life Actuarial Calculation Library for Python**

*Effective date: 2026-04-20. Last updated: 2026-04-21.*

---

**Licensor identification (Art. 10 Ley 34/2002, LSSI-CE):**

> Alberto Aragoneses Nebreda (trading as **Actuaan**)
> NIF: 09180714B
> Address: Francesc Pérez Cabrero, 7, 08021, Barcelona, SPAIN
> Email: [support@lactuca.io](mailto:support@lactuca.io)
> Website: [www.lactuca.io](https://www.lactuca.io)
>
> Alberto Aragoneses Nebreda is a self-employed professional (*autónomo*)
> registered with the Spanish Social Security (*RETA*) and tax authority
> (*AEAT*). For the purposes of the Spanish Information Society Services Act
> (Ley 34/2002, LSSI-CE), Alberto Aragoneses Nebreda is the *prestador de
> servicios de la sociedad de la información*.

By downloading, installing, activating, or using Lactuca ("**Software**"),
you ("**Licensee**") agree to be bound by this End User License Agreement
("**Agreement**"). If you do not agree, do not install or use the Software.

---

## 0. Parties and Contractual Structure

### 0.1 Payment and Merchant of Record

Purchases of **paid** Lactuca licenses (Individual, Team, and monthly
Enterprise plans) are processed by **Lemon Squeezy** (LSFS Inc.), which
acts as the Merchant of Record for those transactions. The commercial
relationship governing payment, invoicing, VAT, and refunds applicable to
the payment transaction is between Licensee and Lemon Squeezy. Lemon
Squeezy's own terms of service and privacy policy apply to that payment
transaction.

**Free Trial licenses** are available at no charge and may be requested
through the Software's activation prompt at first launch. They are not
processed through Lemon Squeezy, and no payment data is collected or
processed by Licensor for the Trial tier.

**Academic & Community licenses** are granted directly by Licensor by
email request ([support@lactuca.io](mailto:support@lactuca.io))
and are not processed through Lemon Squeezy. No payment data is collected
or processed by Licensor for these tiers.

**Enterprise annual contracts** are entered into directly between Licensee
and Licensor by means of a separate written agreement (Order Form or Master
License Agreement). Lemon Squeezy is not involved in those transactions
(see §11.5).

In the event of any conflict between this Agreement and Lemon Squeezy's
terms regarding the scope of the license or the permitted use of the
Software, this Agreement prevails with respect to the license.

---

## 1. Grant of License

Subject to the terms of this Agreement and payment of applicable fees (or
acceptance of a free trial or academic grant), Licensor grants Licensee a
limited, non-exclusive, non-transferable, non-sublicensable license to:

(a) install and use the Software on Licensee's own devices for any
    professional, academic, or research purpose; and

(b) integrate the Software into Licensee's own applications, systems,
    pipelines, or products, including applications that are distributed
    or sold to third parties, provided that:

    (i)  the Software is not made available to end users as a standalone
         actuarial library or package — it must remain embedded and
         inaccessible as an independent component; and

    (ii) a valid Lactuca license covers every machine (device) on which
         the Software is installed and executes. The applicable licensing
         model depends on the deployment scenario:

         - **Direct installation** (the Software is installed on each
           user's own machine): each device requires a license under
           an appropriate tier (Individual, Team, or Enterprise).

         - **Server or shared deployment** (the Software is installed on
           a server or shared infrastructure and accessed by users through
           an application, API, or pipeline without being installed on
           their individual devices): the license applies to the server
           device(s) on which the Software runs. An Enterprise license
           covers all of Licensee's internal users accessing the Software
           through such shared infrastructure, subject to the device
           limit of the tier. No individual license is required for each
           end user solely because they interact with an application that
           calls the Software server-side.

         The technical license enforcement mechanism (§5) validates the
         license on the machine where the Software is installed. The
         Software will not function on any unlicensed device.

Licensee's right to integrate and distribute under (b) does not extend to:

- redistributing the Software (or any part thereof) as a standalone
  library, package, or SDK, whether free of charge or commercially;
- creating or commercialising a product that is substantially derived
  from, or directly competes with, the Software as an actuarial
  calculation library.

**OEM and redistribution to external users — separate contract required.**

Use of the Software in any product or service distributed or made available
to users **external to Licensee’s own organisation** constitutes **OEM use**
and is **strictly prohibited** under any standard license tier without a
separate written OEM Agreement. See §16 for the full definition, compliance
obligations, and how to obtain an OEM license.

The number of licensed users and devices is determined by the tier
acquired:

| Tier | Activation pool | Concurrent sessions |
|---|---|---|
| **Trial** | 1 | 1 |
| **Individual** | 1 | 1 |
| **Team** | 10 | 10 |
| **Enterprise** | 50 | 50 |
| **Academic & Community** | 1 | 1 |
| **OEM** | per agreement | unlimited |

"**Device**" means a single physical or virtual machine, including servers
and container hosts. Shared servers and container environments count as one
device per host, regardless of the number of concurrent processes or users
on that host.

"**Licensee's organisation**" means, for Team and Enterprise licensees,
the legal entity that entered into this Agreement, including any subsidiary
or affiliated entity in which that legal entity holds more than 50% of the
voting rights or ownership interest. Use of the Software by employees or
contractors of such majority-controlled entities constitutes internal use
and does not require a separate license, provided total activation slots
across all included entities remain within the limits of the acquired tier.
Entities in which Licensee holds 50% or less are not covered and require
a separate license.

The **activation pool** is the technically enforced device limit (via
Keygen.sh): each activated device consumes one slot. The **concurrent
sessions** limit is a separately enforced concurrent-session tracking
constraint: only the stated number of instances of the Software may run
simultaneously across all activated devices at any one time. An instance that
cannot obtain a session slot fails to start with an error message indicating
all seats are in use. The session slot is released automatically when the
instance exits normally, or when the license server detects the instance has
stopped sending keep-alive signals (see §6.1). Any tier may cover a shared server for internal
users — the server consumes one activation slot, and internal users
accessing Lactuca through that server do not each require a separate
license key (see §1(b)(ii)). Users external to Licensee's own organisation
are not covered by any standard tier; see §16.

---

## 2. Free Trial

### 2.1 Trial terms

Licensor offers a free 30-day trial of the Software ("**Trial**"). The
Trial is available to any Licensee who has not previously activated a paid
license or a prior Trial using the same email address or on the same
device.

During the Trial, all provisions of this Agreement apply, except that:

(a) no payment is required; and

(b) the Trial automatically expires 30 days after the license key is
    issued; the Software will cease to function after expiry.

### 2.2 No automatic conversion

The Trial does **not** automatically convert into a paid subscription upon
expiry. Licensee must separately purchase a paid plan to continue using
the Software after the Trial period. No charge is made at the end of the
Trial without Licensee's affirmative act of purchase.

### 2.3 Trial limitations

The Trial is limited to 1 device activation and 1 concurrent session. All restrictions
in §3 apply during the Trial. Trial licenses do not include upgrade rights
beyond the trial period.

### 2.4 One trial per person and per device

The Trial is limited to one per person and per device. Attempts to
circumvent this limit (e.g. by registering multiple email addresses or
manipulating hardware identifiers) constitute a material breach of this
Agreement.

### 2.5 Personal data during the Trial

The Trial is not processed through Lemon Squeezy. By requesting a trial key
through the Software's activation prompt, Licensee **explicitly consents**
(GDPR Art. 6(1)(a))
to the processing of the following personal data, each for the specific purpose
indicated:

- **Email address**: to deliver the trial key and manage the trial period.
- **Name** (optional — only processed if provided): to personalise the
  trial key delivery email.
- **Hardware fingerprint hash** (§5.2): to enforce the per-device activation
  limit.
- **Connection metadata transmitted in outbound HTTP requests** during
  trial registration (to the Vercel-hosted trial webhook) and during
  license activation and keep-alive (to the Keygen.sh API, §5.6) —
  comprising the license identifier, hardware fingerprint hash,
  timestamps, and the IP address necessarily included in any outbound
  HTTP request — for the purpose of delivering the license and
  enforcing the concurrent-session limit during the trial period.

Each purpose is separately consented to. Consent to each purpose is
required for the Trial to function as described.

This consent may be withdrawn at any time by contacting
[support@lactuca.io](mailto:support@lactuca.io).
Withdrawal of consent will result in immediate deactivation of the Trial
key. To facilitate withdrawal, Licensee may also use the opt-out link
included in the trial key delivery email. Pursuant to Art. 7(3) GDPR,
withdrawal of consent shall not affect the lawfulness of processing based
on consent before its withdrawal.

### 2.6 Withdrawal right and Trial

The Trial is provided free of charge; no payment obligation arises and no
right of withdrawal under Art. 16(m) of Directive 2011/83/EU is applicable
to the Trial itself. If Licensee subsequently purchases a paid subscription,
§9 of this Agreement applies to that purchase.

---

(eula-restrictions)=
## 3. Restrictions

Licensee may not:

(a) Copy, modify, merge, publish, or distribute the Software except as
    expressly permitted herein. **Nothing in this clause prohibits Licensee
    from making a single backup copy of the Software to the extent
    permitted by Art. 5(2) of Directive 2009/24/EC (Art. 99(a) TRLPI), as
    a mandatory right that cannot be excluded by contract.**

(b) Reverse-engineer, decompile, disassemble, or otherwise attempt to
    derive the source code, algorithms, or cryptographic material from any
    compiled binary component of the Software (`.so`, `.pyd`, or any other
    compiled extension), **except solely to the extent required by mandatory
    applicable law that cannot be excluded by agreement (including the
    interoperability right under Art. 6 of Directive 2009/24/EC, subject
    to its statutory conditions). Before exercising any such right,
    Licensee shall first request the relevant technical information from
    Licensor in writing.**

(c) Circumvent, disable, or tamper with any license validation mechanism,
    digital signature, hardware fingerprint check, or concurrent-session
    keep-alive signal embedded in or used by the Software. **This prohibition
    does not apply to the extent strictly necessary to exercise mandatory
    rights under applicable law (including the interoperability and backup
    rights under Arts. 5–6 of Directive 2009/24/EC).**

(d) Transfer, assign, sublicense, rent, lease, lend, or otherwise convey
    the license or any activation key to any third party. Individual and
    Academic & Community licenses are strictly personal. Team and Enterprise
    licenses may not be transferred outside the contracting organization.

(e) Use the Software for any unlawful purpose or in violation of any
    applicable law or regulation.

---

(4-academic-community-license-restrictions)=
## 4. Academic & Community License Restrictions

Licenses issued under the **Academic & Community** tier are granted
exclusively for non-commercial, academic, research, or open-source
community use. The following uses are **strictly prohibited** under an
Academic & Community license:

- Client-facing consulting engagements or deliverables billed to third
  parties.
- Integration into production systems used by or on behalf of commercial
  entities.
- Any activity that generates revenue, directly or indirectly.

Violation of this clause constitutes a material breach and results in
immediate revocation of the license. No refund or compensation is payable
as no payment was made for this tier.

---

## 5. Activation, Hardware Fingerprint, and Technical Protection Measures

### 5.1 Technical protection measures

The Software incorporates a license management system (powered by
Keygen.sh) that requires a valid license key to operate. The following
technical protection measures are in place:

| Measure | Purpose |
|---|---|
| Hardware fingerprint (one-way hash) | Enforce per-device activation limits |
| Digitally signed local license file | Prevent license file tampering |
| Periodic online validation (every 30 days) | Detect unauthorised key sharing |
| Concurrent session tracking (keep-alive approximately every 10–20 min, tier-dependent) | Enforce concurrent session limits per tier |
| Compiled-only binary distribution (no source code) | Protect proprietary implementation |

### 5.2 Hardware fingerprint

Activation involves the collection of a hardware fingerprint: a one-way
cryptographic hash derived from hardware attributes of the device (MAC
address, hostname, CPU architecture). The **raw hardware values are never
transmitted** — only the irreversible hash is sent to and stored by
Keygen.sh. The resulting hash constitutes **pseudonymous data** within the
meaning of Art. 4(5) GDPR: it cannot directly identify Licensee as an
individual without access to additional information that Licensor does not
retain.

### 5.3 Interoperability

The Software is compatible with:

- **Operating systems**: Windows 10/11 (x86-64), macOS 11+ (ARM64), Linux
  (glibc 2.28+, x86-64)
- **Python versions**: 3.12 and above
- **Dependencies**: installed automatically by pip; the current list and
  minimum version requirements are published in the
  [installation guide](https://www.lactuca.io/latest/user_guide/getting_started.html#installation)

The Software does not require any system-level driver or kernel extension.
It does not interoperate with any third-party actuarial software.

### 5.4 Hardware replacement

If Licensee replaces or substantially modifies a device's hardware, the
existing activation on that device will be invalidated. Licensee may
deactivate devices and re-activate on new hardware as described in the
[activation documentation](activation). The total number of simultaneously
active devices may not exceed the limit for the applicable tier at any
time.

### 5.5 Privacy

Full details of what personal data is collected and how it is processed are
set out in the [Privacy Policy](privacy), which forms an integral part of
this Agreement.

### 5.6 Personal data transmitted during concurrent session tracking (keep-alive)

In addition to the activation-time data described in §5.2, each running
instance of the Software transmits a keep-alive HTTP request (*heartbeat*)
to the Keygen.sh API approximately every 10 minutes on single-seat tiers and every
20 minutes on Team and Enterprise (§6.1(b)). Each such
request contains:

- The license key identifier (or a non-reversible hash thereof);
- The hardware fingerprint hash (§5.2) of the device running the process;
- A timestamp of the request; and
- The IP address of the device, which is necessarily included in any
  outbound HTTP request.

No raw hardware values, user names, email addresses, or application-layer
data are transmitted in heartbeat requests.

**Legal basis:** For paid licensees this processing is based on
**Art. 6(1)(b) GDPR** (performance of the contract). For Trial licensees
the basis is **Art. 6(1)(a) GDPR** (consent given under §2.5).
Keygen.sh processes this data solely as a **data processor** within the
meaning of Art. 28 GDPR pursuant to a Data Processing Agreement with
Licensor. The transfer to Keygen.sh (United States) is carried out on the
basis of **Standard Contractual Clauses** (Commission Implementing Decision
(EU) 2021/914). Licensee acknowledges that this functionality is provided
by **Keygen.sh (Keygen LLC)** under its own [Terms of Service](https://keygen.sh/terms)
and [Privacy Policy](https://keygen.sh/privacy). Full details of retention
periods and sub-processor arrangements are set out in the [Privacy Policy](privacy),
which forms an integral part of this Agreement (§5.5).

If Licensor becomes aware of a material change to Keygen.sh's data
practices that reduces the level of protection described in this section,
Licensor will notify active subscribers by email within **30 days** of
becoming aware. Enterprise subscribers on an annual direct contract who do not wish
to continue under modified conditions may terminate within 30 days of notification and
receive a pro-rata refund of any prepaid annual fee (§13.2).

---

## 6. Offline Grace Periods and Service Continuity

### 6.1 Offline grace periods

The Software maintains two independent offline grace periods:

(a) **License validation grace period (30-day revalidation interval)**: the
    Software attempts to revalidate the license online approximately every
    30 days. If no internet connection is available at revalidation time, the
    Software may continue to operate while the locally stored license has not
    yet reached its expiry date, subject to periodic warnings on startup. There
    is no additional offline cutoff beyond the stored expiry date. Once the
    expiry date stored locally is exceeded, the Software will not operate
    regardless of network availability.

(b) **Concurrent session grace period (3 days)**: each running instance
    of the Software sends a keep-alive signal to the license server
    approximately every 10 minutes on single-seat tiers and every 20 minutes on Team
    and Enterprise to maintain its active session. If the server cannot
    be reached:

    - A **process already running** at the time of network failure
      continues to operate without interruption for its full lifetime.

    - A **new startup** of the Software is permitted without network
      access for up to **3 days** from the last successful server
      contact. After 3 continuous days without connectivity, new startups
      will fail with a connection error until the network is restored.

Both grace periods are independent. In practice, the concurrent session
grace period (3 days) is the binding constraint when the network is
unavailable for an extended time.

### 6.2 Service continuity commitment

Licensor undertakes to maintain the online license validation service
(currently provided by Keygen.sh) for the duration of any active paid
subscription. In the event that Licensor is unable to maintain the
validation service (whether due to discontinuation of the Software, a
change of license management provider, or any other reason), Licensor will:

(a) provide Licensee with at least **90 days' prior written notice** by
    email; and

(b) at Licensee's option, either: (i) issue a perpetual offline activation
    that does not require online validation for the remainder of the paid
    period; or (ii) offer a pro-rata refund of any unexpired prepaid
    subscription.

### 6.3 Provider migration

If Licensor migrates to a new license management provider, existing license
keys will remain valid without requiring Licensee to re-purchase. Licensor
will provide migration instructions at least 30 days before any key
migration is required.

---

## 7. Updates and Version Policy

Paid subscriptions include all minor and patch releases within the
subscribed major version. Major version upgrades may require a separate
license or subscription renewal. Licensor reserves the right to define the
boundary between minor and major releases. Free Trial licenses do not
include upgrade rights beyond the Trial period.

Notwithstanding the above, **security updates** necessary to maintain the
conformity of the Software will be supplied to consumer licensees in the
EU/EEA without additional charge for the duration of the active
subscription, regardless of whether they are classified as minor or major
releases, as required by Art. 8 of Directive 2019/770/EU (implemented in
Art. 115 quater TRLGDCU).

---

## 8. Suspension and Termination for Cause

### 8.1 Grounds

Licensor may suspend or terminate this Agreement, with immediate effect
and written notice to Licensee, if Licensee:

(a) materially breaches any provision of this Agreement and, where the
    breach is capable of remedy, fails to remedy it within **14 calendar
    days** of written notice from Licensor specifying the breach;

(b) deliberately circumvents, disables, or tampers with any license
    validation or copy-protection mechanism;

(c) provides materially false information during the trial registration
    or purchase process; or

(d) intentionally and repeatedly exceeds the device-count or
    concurrent-session limits of the licensed tier, or deliberately
    circumvents the concurrent-session tracking mechanism described in §5.1 and §6.1(b)
    (for example, by patching or disabling the keep-alive signal mechanism)
    in order to obtain more concurrent sessions than permitted.

A single unintentional excess of the device limit (e.g. caused by hardware
failure and replacement) does not constitute grounds for termination;
Licensor will first request that Licensee deactivate the excess device.

### 8.2 Refund on termination for cause

Where Licensor terminates this Agreement pursuant to §8.1:

(a) **Monthly subscribers**: no refund is payable; the license remains
    active until the end of the current billing period, after which access
    is revoked.

(b) **Annual direct-contract subscribers** (Enterprise and OEM annual contracts
    only): Licensor shall refund the pro-rata unused portion of the prepaid annual
    fee, calculated from the date of termination, **except** where the
    termination is caused by deliberate circumvention (§8.1(b)), the
    provision of materially false information (§8.1(c)), intentional
    circumvention of the concurrent-session tracking mechanism (§8.1(d)),
    or OEM misuse in violation of §16 (use of the Software to serve users
    external to Licensee's own organisation without an OEM Agreement), in
    which case no refund is payable.

### 8.3 Notification

Licensor will notify Licensee by email of any suspension or termination.
Where the breach is capable of remedy under §8.1(a), Licensor will provide
the 14-day cure notice before taking any action that disables the Software.

---

## 9. Right of Withdrawal (EU and EEA Consumers — Paid Licenses Only)

*This section applies exclusively to paid license purchases processed
through Lemon Squeezy (Individual and Team plans, and monthly Enterprise
plan). It does not apply to Free Trial activations (see §2.6),
Academic & Community licenses (no payment), or Enterprise annual contracts
(see §11.5).*

### 9.1 Statutory right of withdrawal

Consumers resident in the European Union or the European Economic Area
normally have a statutory 14-day right to withdraw from distance contracts
under Directive 2011/83/EU and the applicable national implementing
legislation (in Spain: Real Decreto Legislativo 1/2007, Arts. 68–79
TRLGDCU).

### 9.2 Waiver upon activation or download of digital content

However, pursuant to Art. 16(m) of Directive 2011/83/EU (and Art. 103(m)
TRLGDCU for consumers in Spain), the right of withdrawal does not apply to
contracts for the supply of digital content not supplied on a tangible
medium where:

(a) performance has begun with the consumer's **prior express consent**; and

(b) the consumer has **acknowledged** that they thereby lose the right of
    withdrawal.

By completing the purchase through Lemon Squeezy and activating or
downloading the Software, Licensee:

(i)  gives **prior express consent** to the immediate supply of the digital
     content (Lactuca); and

(ii) **acknowledges** that, once the license key has been delivered and the
     download or activation process has commenced, the right of withdrawal
     is lost.

These acknowledgements are recorded by Lemon Squeezy at the time of
checkout and are available on request from
[support@lactuca.io](mailto:support@lactuca.io).

**For the avoidance of doubt, the waiver set out in this §9.2 does not
affect or limit Licensee's rights in respect of lack of conformity of the
Software under Directive 2019/770/EU and §10.1 of this Agreement. Those
conformity rights are independent of the right of withdrawal and remain
fully unaffected.**

### 9.3 Refund before activation

If Licensee has purchased a license but has not yet activated or downloaded
the Software, Licensee may request a full refund within 14 calendar days of
purchase by contacting
[support@lactuca.io](mailto:support@lactuca.io). Once
activation has commenced, the waiver in §9.2 applies.

### 9.4 Consumers outside the EU/EEA

For consumers outside the EU/EEA, refund eligibility is governed by Lemon
Squeezy's standard refund policy, which applies as Merchant of Record.
Licensor's refund obligations are limited to those imposed by mandatory law
in the relevant jurisdiction.

---

## 10. Warranties and Limitation of Liability

### 10.1 Legal conformity guarantee (EU/EEA consumers)

Where Licensee is a consumer within the meaning of Directive (EU) 2019/770
and applicable national law (in Spain: Arts. 115 bis and following of
TRLGDCU as amended by RDL 7/2021), Licensor warrants that the Software will
conform to its description and will be fit for the actuarial calculation
purposes set out in the documentation for the duration of the active
subscription. In the event of a lack of conformity, Licensee is entitled
to:

(a) have the non-conformity remedied free of charge within a reasonable
    time; and

(b) if the non-conformity is not remedied within a reasonable time or
    cannot be remedied, a proportionate reduction in the subscription price
    or termination of the Agreement with a full refund of amounts paid.

Nothing in this Agreement limits or excludes any right or remedy available
to consumers under mandatory applicable law.

### 10.2 Professional responsibility of the Licensee (all tiers)

The Software is a professional actuarial calculation tool intended for use
by or under the direct supervision of qualified actuaries. By using the
Software, Licensee warrants that they have the professional competence
required to:

(a) understand the actuarial methods, assumptions, and formulas implemented
    in the Software, as documented in the technical documentation;

(b) assess the appropriateness of those methods and assumptions for the
    specific purpose for which the Software is being used;

(c) independently verify and validate all calculation results before
    relying on them for any professional, regulatory, statutory, or
    supervisory purpose; and

(d) take sole professional responsibility for any conclusions, opinions,
    reports, or decisions based on the Software's output.

Licensee acknowledges that the Software implements general actuarial
methods based on international actuarial practice and applicable Spanish
and international actuarial standards, but that **actuarial calculations
performed with the Software are not a substitute for an independent
actuarial opinion, review, or certification** by a duly qualified actuary.

### 10.3 No warranty for specific regulatory purposes (all tiers)

**The Software is not certified, approved, or validated by any regulatory
authority.** Licensor expressly disclaims, to the fullest extent permitted
by applicable law, any warranty or representation — express, implied, or
statutory — that the Software or any output thereof is suitable,
appropriate, or sufficient for any specific regulatory, supervisory,
statutory, or compliance purpose, including without limitation:

- **IFRS 17** (*International Financial Reporting Standard 17 — Insurance
  Contracts*): present value of future cash flows, risk adjustment,
  contractual service margin (CSM), best estimate liabilities (BEL), or
  any other IFRS 17 measurement model (GMM, PAA, VFA);

- **Solvency II**: Solvency Capital Requirement (SCR),
  Minimum Capital Requirement (MCR), best estimate of technical provisions,
  risk margin, standard formula modules (mortality, longevity, disability,
  lapse, expense), internal model validation, or ORSA;

- **Spanish technical provisions** (*Provisiones técnicas del seguro de
  vida, Reglamento de Ordenación y Supervisión de los Seguros Privados*):
  mathematical reserves (*reserva matemática*), actuarial equivalence,
  or any other provision required under ROSSP or DGSFP regulations;

- any other statutory, regulatory, supervisory, or reporting requirement
  imposed by any competent authority in any jurisdiction.

Use of the Software in connection with any of the above purposes does not
relieve Licensee of the obligation to obtain independent actuarial review
and certification where required by applicable law, regulation, or
professional standards (including the standards of the Instituto de
Actuarios Españoles or any other competent actuarial association).

Licensor's liability for any loss, damage, penalty, fine, or regulatory
sanction arising from Licensee's reliance on the Software for any of the
purposes listed above, without the independent verification required by
§10.2, is **excluded to the fullest extent permitted by applicable law**.

### 10.4 Limitation of liability (B2B and non-consumer use)
*This section applies in addition to §10.2 and §10.3, and independently
of the consumer guarantee in §10.1.*
Where Licensee uses the Software exclusively in the course of a trade,
business, or profession (i.e. as a non-consumer), the Software is provided
"as is" without any implied warranties of merchantability, fitness for a
particular purpose, or non-infringement, to the fullest extent permitted by
applicable law. Licensor's total aggregate liability for any claim arising
under or related to this Agreement shall not exceed the amounts paid by
Licensee in the **twelve months** preceding the claim.

This limitation does not apply to liability for:

(a) death or personal injury caused by Licensor's negligence;

(b) fraud or fraudulent misrepresentation;

(c) any other liability that cannot be limited or excluded by applicable
    law.

### 10.5 Availability

Licensor does not warrant uninterrupted availability of the Software or the
license validation service. Temporary unavailability of the online
validation server does not constitute a lack of conformity, provided the
offline grace period in §6.1 remains effective.

---

## 11. Subscription, Renewal, and Cancellation

### 11.1 Monthly subscriptions

Monthly subscriptions renew automatically on the same day of each calendar
month. Licensee may cancel at any time through the Lemon Squeezy customer
portal (accessible via the link in any subscription confirmation email).
Cancellation takes effect at the **end of the current billing month**; no
refund is payable for the remaining days of the current month.

### 11.2 Annual subscriptions (Individual and Team)

Annual subscriptions renew automatically 12 months after the initial
purchase date. Licensor will send a renewal reminder email at least **14
days** before the renewal date. Additionally, the Software displays an
in-app warning message when **30 days or fewer** remain before
expiry. Licensee may cancel before the renewal date through the Lemon
Squeezy customer portal; no refund is payable for the current annual
period already paid, unless §10.1 or §13.2 applies.

### 11.3 Price changes

Licensor may change subscription prices with at least **30 days' notice**
before the next renewal date. If Licensee does not accept the new price,
Licensee may cancel before the next renewal date without penalty.

### 11.4 Effect of cancellation or non-renewal

Upon cancellation or non-renewal:

(a) The license key remains active until the end of the paid period.

(b) After expiry, the Software will display a license error and cease to
    function until a new license is activated.

(c) Locally stored license files may be deactivated by Keygen.sh after the
    grace period expires.

### 11.5 Enterprise annual contracts
Enterprise subscriptions — whether billed monthly through Lemon Squeezy or
annually through a direct contract — confer **identical usage rights**: the
same activation pool (150 machine activations), the same maximum of 50
concurrent sessions, and the same permitted deployment scenarios (direct
workstation installation and server/shared deployment for internal users).
The difference between monthly and annual Enterprise is solely the billing
channel and contractual formality.
Annual Enterprise subscriptions are governed by a **separate written
agreement** (Order Form or Master License Agreement) entered into directly
between Licensee and Licensor, outside the Lemon Squeezy platform. The
terms of cancellation, renewal notice period, refund policy, and service
level commitments are set out exclusively in that separate agreement. In
the event of any conflict between this Agreement and the Enterprise
contract, the Enterprise contract prevails with respect to the Enterprise
Licensee's subscription terms.

---

## 12. Intellectual Property

The Software compiled binaries, stub files (`.pyi`), and documentation
are the exclusive intellectual property of Alberto Aragoneses Nebreda.
All rights not expressly granted herein are reserved. This Agreement does
not convey any ownership interest in the Software.

The **Lactuca table format** (`.ltk` binary format specification and all
code that reads, writes, or interprets `.ltk` files) is the exclusive
intellectual property of Alberto Aragoneses Nebreda.

The **actuarial data values** contained in the bundled `.ltk` files are
derived from public sources (including tables published by the Instituto
de Actuarios Españoles, the Dirección General de Seguros y Fondos de
Pensiones, and equivalent international bodies). Those factual values are
not claimed as intellectual property of Licensor.

The documentation (text, formulas, and examples in `docs/`) is licensed
separately under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/),
unless otherwise noted.

---

## 13. Changes to this Agreement

### 13.1 Notice of changes

Licensor may update this Agreement at any time. Licensee will be notified
of any material change:

(a) by email to the address provided at registration, at least **30 days**
    before the change takes effect; and

(b) by a prominent notice on this documentation page.

### 13.2 Acceptance or rejection by paid subscribers

If Licensee is a paid subscriber (Individual, Team, or Enterprise) and
does not accept the modified terms, Licensee may **terminate the agreement**
before the effective date of the change. In that case:

- **Monthly subscribers**: entitled to use the Software until the end of
  the current billing month; no further charges will be made.
- **Annual subscribers**: entitled to a pro-rata refund of the unused
  portion of their prepaid annual subscription, calculated from the
  effective date of termination.

Continued use of the Software after the effective date constitutes
acceptance of the updated terms, provided Licensee received the notice
under §13.1(a) and did not exercise the termination right within the
30-day notice period.

### 13.3 Non-material changes

Changes that merely correct typographical errors, update contact details,
or clarify existing terms without altering substantive rights are effective
immediately upon publication and do not require a 30-day notice period.

### 13.4 Trial and Academic & Community licensees

Trial and Academic & Community licenses are granted at no charge. Licensor
may modify the applicable terms at any time with reasonable notice. Continued
use after notice constitutes acceptance. If Licensee does not accept the
modified terms, they must cease using the Software.

For the avoidance of doubt, the personal data processed in connection with
the Trial is collected solely because it is technically necessary to deliver
and manage the Trial license — it does not constitute commercial
consideration for the Software. No marketing use, profiling, or commercial
exploitation of Trial licensee data is carried out.

---

## 14. Governing Law and Jurisdiction

### 14.1 B2B and professional use

Where Licensee uses the Software exclusively in the course of a trade,
business, or profession (i.e. as a non-consumer), this Agreement is
governed by the laws of **Spain** and the parties irrevocably submit to
the exclusive jurisdiction of the courts of **Barcelona, Spain**.

### 14.2 Consumer use (EU/EEA)

Where Licensee is a consumer within the meaning of applicable EU law, this
Agreement is governed by the laws of **Spain**. However, nothing in this
Agreement deprives Licensee of the protection afforded by mandatory
provisions of the law of the country in which Licensee habitually resides,
nor of the right to bring proceedings in the courts of that country,
pursuant to Arts. 17–19 of Regulation (EU) 1215/2012 (Brussels I bis).

### 14.3 Consumer use (outside EU/EEA)

For consumers outside the EU/EEA, the mandatory consumer protection laws of
the consumer's country of habitual residence apply to the extent that they
cannot be derogated by contract.

---

## 15. Online Dispute Resolution (EU)

The European Commission provides an Online Dispute Resolution (ODR)
platform for the out-of-court settlement of disputes between consumers and
traders established in the EU:

**ODR platform**: [https://ec.europa.eu/consumers/odr](https://ec.europa.eu/consumers/odr)

Licensor's email address for ODR purposes:
[support@lactuca.io](mailto:support@lactuca.io)

Licensor is **not affiliated with any alternative dispute resolution (ADR)
organism** registered under Directive 2013/11/EU or the applicable national
implementing legislation (in Spain: Real Decreto 231/2008 and Ley 7/2017).
Licensor will endeavour to resolve complaints amicably before any formal
proceedings are initiated.

---

## 16. License Compliance

### 16.0 Scope of this section

Concurrent-session and device-activation limits are enforced automatically
and continuously by the technical mechanisms in §5 (hardware fingerprint)
and §6.1 (concurrent session tracking). No audit of those limits is required or
intended.

This section addresses the two types of misuse that **cannot be detected
technically**: (i) use of the Software to serve users external to
Licensee's own organisation without an OEM Agreement; and (ii) use under
an Academic & Community license for commercial purposes.

### 16.1 OEM and Academic misuse

Licensee represents and warrants at all times during the term of this
Agreement that:

(a) If Licensee holds a standard tier license (Individual, Team, or
    Enterprise), the Software is not being used as a calculation engine
    in any product or service made available to users external to
    Licensee's own organisation, as defined in §1.

(b) If Licensee holds an Academic & Community license, the Software is
    not being used in any activity that generates revenue, directly or
    indirectly, as specified in §4.

Violation of either representation constitutes a material breach entitling
Licensor to terminate the license immediately under §8. OEM violations are
subject to termination without refund (§8.2(b)). To obtain an OEM license,
contact [support@lactuca.io](mailto:support@lactuca.io)
with a description of the product, the estimated number of external end
users, and the intended business model. OEM licenses are negotiated
individually.

### 16.2 Remediation

If Licensor has reasonable grounds to believe a violation of §16.1 is
occurring, Licensor will notify Licensee in writing. Licensee shall:

(a) promptly purchase the appropriate tier or OEM Agreement to cover
    actual use, retroactively from the date the violation began; and

(b) pay any fees due for the excess period at the list price applicable
    at the time of notification, without prejudice to Licensor's right
    to terminate under §8 in cases of deliberate misuse.

---

## 17. Contact

For licensing inquiries, support, or legal notices:

**Email**: [support@lactuca.io](mailto:support@lactuca.io)

**Website**: [www.lactuca.io](https://www.lactuca.io)

---

*A Spanish version of this Agreement (Contrato de Licencia de Usuario Final
en español) is available at [eula_es](eula_es).*

*Language prevalence: for **consumers habitually resident in Spain**, the
Spanish version shall prevail in the event of any discrepancy. For all
other Licensees, the English version shall prevail.*
