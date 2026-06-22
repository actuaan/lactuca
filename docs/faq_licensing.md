# Licensing FAQ

Answers to common questions about Lactuca licenses, activation, and account management.

---

## Trials

### How do I start a free trial?

Run `import lactuca` in Python. If no license is found, the activation prompt
appears with the option **[T] Request a free 30-day trial**. Enter your name
(optional) and email address; a confirmation step lets you correct the address
before the request is sent. Your trial key will arrive by email within a few
minutes. No credit card required.

Once you receive the key, run `import lactuca` again and choose **[K]** to enter it.

### How long is the trial?

30 days after the license key is issued (not from when the email is received). The expiry
clock starts at the moment the key is created on the license server, which is typically
within seconds of the trial request.

### Can I get a second trial?

No. One trial per person and per device. If your trial has expired, purchase a paid plan
from the [Pricing page](pricing).

### I chose `[T]` to start a trial but got an error saying my device has an active license. What should I do?

Your machine already has a paid Lactuca license registered. Choose option **[K]**
at the activation prompt and enter your license key. If you cannot find your key,
check your original purchase confirmation email from Lemon Squeezy. If you need
further help, contact [support@lactuca.io](mailto:support@lactuca.io).

### Can I use the trial for commercial work?

Yes — trial licenses allow full commercial use during the 30-day period.

---

## Activation

### How do I activate my license?

Run `python -m lactuca` (or `import lactuca`) and enter your key at the prompt.
With an already-valid license, `python -m lactuca` prints a one-line status summary.
See the full [Activation Guide](activation) for step-by-step instructions for Windows,
macOS, and Linux.

### Support confirmed a license adjustment, but my local expiry did not update yet. What should I do?

Run a manual synchronization:

```bash
python -m lactuca license refresh
```

This forces an immediate online sync and updates local `license.json` when the server
returns `VALID` or `EXPIRING`.

### How do I quickly inspect local license state?

Use:

```bash
python -m lactuca license status
```

This command reads local state only and does not call the license server.

### How do I run guided license diagnostics?

Use:

```bash
python -m lactuca license doctor
```

For automation pipelines, use JSON output:

```bash
python -m lactuca license doctor --json
```

### Where is `license.json` stored?

| OS | Default path |
|---|---|
| Windows | `%LOCALAPPDATA%\lactuca\license.json` |
| macOS | `~/Library/Application Support/lactuca/license.json` |
| Linux | `~/.config/lactuca/license.json` |

You can override the directory with the `LACTUCA_CONFIG_DIR` environment variable.

### Can I activate on multiple devices?

Yes, up to the device limit of your plan (Trial: 1; Individual: 1; Team: 10;
Enterprise: 50; Academic: 1). Each physical or virtual machine counts as one device.

(device-transfer)=
### I upgraded my computer. How do I transfer my license to the new machine?

1. On your **old machine**, delete `license.json` (see paths above). This does not free
   the device slot automatically; contact [support@lactuca.io](mailto:support@lactuca.io)
  to release it. For paid monthly or annual plans, device slots also reset on renewal.
2. On your **new machine**, run `import lactuca` and enter your key. Activation proceeds
   normally if a slot is available.

If you have reached your device limit and need to activate on a new device immediately,
contact [support@lactuca.io](mailto:support@lactuca.io) with the
subject "Device transfer request".

### Can I copy `license.json` to another machine?

No. `license.json` is device-bound: it contains a device-specific integrity code (MAC)
computed from the hardware fingerprint of the machine that activated it. Copying the
file to a different machine fails integrity checks (**[LAC-3004]** or fingerprint
mismatch **[LAC-2002]**). Online recovery using the stored key runs only on the
**same** device (tampered local file, not a cross-host copy). On another machine,
delete the copied file and activate independently, or follow the
{ref}`device transfer procedure <device-transfer>`.

Each machine must activate independently.  Activating on a new device consumes one
slot from your license's device pool.

(license-tampered-faq)=
### I see `LicenseTamperedError`. What does it mean?

`LicenseTamperedError` is raised when the local `license.json` file fails one of the
built-in integrity checks.  There are four variants:

| Error code | Meaning | Auto-recoverable? |
|---|---|---|
| **LAC-3001** | Missing `signed_data` or `signature` fields (schema version mismatch or manual edit) | Yes — online recovery using stored key |
| **LAC-3003** | Missing `mac` integrity field (old file or manual edit) | Yes — online recovery using stored key |
| **LAC-3004** | MAC mismatch — fields modified after writing, or file copied from another device | Same device only — online recovery using stored key; cross-host copy → **[LAC-2002]** |
| **LAC-3005** | System clock moved back past the last validated date | No — restore the system clock |

For **LAC-3001, LAC-3003, and LAC-3004** on the **same** device: Lactuca attempts
online recovery and rewrites `license.json` when recovery succeeds. Cross-host copies
raise **[LAC-2002]** instead. In non-interactive environments (servers, CI/CD), ensure
`LACTUCA_LICENSE_KEY` is set so same-device recovery can proceed without a prompt.

If recovery fails, run:

```bash
python -m lactuca license doctor
python -m lactuca license refresh
```

If it still fails, delete `license.json` manually and re-activate.

For **LAC-3005**: the library cannot fix the system clock on your behalf.  Sync your
system clock (e.g. `w32tm /resync` on Windows, or enable NTP on Linux/macOS) and retry.

See {ref}`lac-3003-mac-missing`, {ref}`lac-3004-mac-mismatch`,
and {ref}`lac-3005-clock-rollback` in the Error Reference for full details.

### I see `LicenseInvalidError: Your license has reached the maximum number of activations.` What now?

Your license has no free device slots. Options:

- **Deactivate an unused device**: contact [support@lactuca.io](mailto:support@lactuca.io)
  and provide the hostname(s) to deactivate.
- **Upgrade your plan**: Team (10 device pool) and Enterprise (50 device pool) support
  significantly larger activation pools.

(seat-exhausted)=
### I see `LicenseSeatExhaustedError`. What now?

All concurrent session slots allowed by your plan are currently in use. This is
different from the device activation limit — it means too many Python processes are
running Lactuca at the same time across your activated devices.

**Short-term options:**

- **Wait for a seat to free up.** Seats are released automatically when a running
  process exits cleanly. If a process crashed or was killed, the seat is released
  after the heartbeat lease expires (30 minutes for Individual/Trial/Academic; 60
  minutes for Team/Enterprise).
- **Check for stuck processes.** Look for background scripts, Jupyter kernels, or
  scheduled jobs running Lactuca that you may have forgotten about.

**Long-term options:**

- **Upgrade to Team or Enterprise.** These tiers allow 10 and 50 concurrent sessions
  respectively, which is suitable for shared server deployments and teams running
  parallel jobs.
- **Restructure your pipeline.** If you are running many short actuarial tasks in
  parallel, consider batching them within a single process.

---

## Plans and billing

### What does "named user" mean?

A named user is a specific individual authorised to use the Software under the
Licensee’s license. The “named user” count per tier is a **contractual limit**
accepted by the Licensee in the EULA.

Technical enforcement operates on two complementary axes:

1. **Machine activations**: each tier has an activation pool (Individual: 1; Team: 10;
  Enterprise: 50; Trial/Academic: 1). Every device (workstation, server, VM, Docker
   host) on which Lactuca is activated consumes one slot.

2. **Concurrent sessions (process leasing)**: the license system also enforces a limit
   on how many Python processes may run Lactuca **simultaneously** across all activated
   devices. Limits are: Individual/Trial/Academic: **1 concurrent session**; Team: **10
   concurrent sessions**; Enterprise: **50 concurrent sessions**.

   When the concurrent session limit is reached, any additional process that tries to
   import Lactuca will raise `LicenseSeatExhaustedError`. The seat is released
   automatically when the process exits or after the heartbeat lease expires (30 minutes
   for single-user tiers; 60 minutes for Team/Enterprise).

For workstation deployments, each user typically activates on their own machines. For
shared server deployments, the server consumes one activation slot and all internal
users on that server share it — no individual license key is required per user. Access
control on the server is the Licensee’s own responsibility (LDAP, roles, etc.).

### What happens if I cancel my subscription?

Your license remains active until the end of the period already paid. After that,
`import lactuca` raises `LicenseExpiredError`. Your data and calculations are unaffected
— only the ability to run Lactuca is restricted.

### Can I switch from Individual to Team?

Yes. Purchase the Team plan; you will receive a new key. Activate with the new key on
each device. The old Individual key will be revoked at the end of its paid period.
Previously generated outputs (spreadsheets, reports) are unaffected.

### Are minor updates included?

Yes. All minor and patch releases within the same major version are included in your
subscription. Major version upgrades may require a separate renewal; this policy will
be communicated in advance before the first major release.

### What payment methods are accepted?

Lemon Squeezy (our payment provider) accepts all major credit and debit cards. Bank
transfers and invoicing are available for Enterprise customers; contact us for details.

---

## Offline use

### What is the grace period?

Lactuca enforces two independent grace periods when the network is unavailable:

**License validation grace period**

Lactuca attempts to revalidate the license online every 30 days. If the network is
unavailable at revalidation time, the library continues operating — printing a warning
to `stderr` on each startup — until the license's local expiry date:

```
[Lactuca] Could not reach license server. Continuing with locally validated license.
```

There is no hard offline cutoff for the validation check; operation continues with
the warning until `expires_at` is reached. Normal computations continue unaffected.

**Process heartbeat grace period (3 days)**

Each running Lactuca process sends a short keep-alive heartbeat to the license server
approximately every **10 minutes** on single-seat tiers and every **20 minutes** on
Team and Enterprise (derived from the license server's process policy). If the server
cannot be reached, heartbeat failures are silently
ignored — **a running process is never interrupted** by network unavailability.
However, **new startups** of Lactuca are permitted offline for up to 3 days from the
last successful server contact. After 3 days, a new `import lactuca` will raise
`LicenseSeatExhaustedError` until the network is restored.

Both grace periods are independent. The binding practical limit is the **3-day
heartbeat grace**: new startups of Lactuca are blocked after 3 days without
connectivity. License revalidation has no hard offline cutoff — the library continues
running with a warning on each startup until `expires_at` is reached.

### Does the grace period apply to cancelled or revoked licenses?

No. Once a cancellation or revocation event is processed by our server, the next time
the network is available the license is blocked immediately. The grace period only
applies when the network is physically unreachable (timeout). It does not extend the
validity of an expired or revoked license.

A **permanent revocation** on the license server (for example after subscription
cancellation or account closure) removes the license record entirely. Lactuca
detects this on the next online check. Running `python -m lactuca license refresh`
or `license doctor` records that outcome; the **next** `import lactuca` with network
access forces an immediate check and blocks access even if the 30-day revalidation
window has not elapsed.

For payment or account issues that may be temporary, contact
[support@lactuca.io](mailto:support@lactuca.io) — support can restore access when
appropriate.

### I work on a corporate network with no internet access. What should I do?

Ask your IT administrator to allow outbound HTTPS (port 443) to the Lactuca license
server (`api.keygen.sh`).
Lactuca contacts the server at each startup (to acquire a session seat) and sends
keep-alive requests approximately every 10–20 minutes during operation (tier-dependent).
Heartbeat failures
are silently tolerated — a running process is never interrupted — but new startups
require a successful server connection within the last **3 days**. Every **30 days**
the library attempts online revalidation; if the network is unavailable at that time,
computation continues with a startup warning until the local license expiry date
(`expires_at`) — there is no separate 30-day offline cutoff beyond that date. If
persistent internet access is not possible, contact [support@lactuca.io](mailto:support@lactuca.io)
to discuss options for your organization.

---

## Servers, Docker, and CI/CD

### How do I use Lactuca in Docker?

Pass the license key as an environment variable:

```dockerfile
ENV LACTUCA_LICENSE_KEY="XXXX-XXXX-XXXX-XXXX"
```

Or inject it at runtime:

```bash
docker run -e LACTUCA_LICENSE_KEY="XXXX-XXXX-XXXX-XXXX" my-image
```

**Important — device fingerprint and container identity**: Lactuca uses the container's
MAC address as part of its device fingerprint. By default, Docker assigns a random MAC
to every new container, so each container recreation is treated as a different device.
This means:

- A `license.json` stored on a persistent volume will produce a `LicenseInvalidError`
  (error code `LAC-2002`) when the container is recreated, because the new container's
  fingerprint no longer matches the stored one.
- The activated device slot from the previous container is not released automatically.

**Recommended setup for persistent container deployments**:

1. Fix the container's MAC address so it is stable across recreations:

   ```bash
   docker run --mac-address 02:42:ac:11:00:02 \
              -e LACTUCA_LICENSE_KEY="XXXX-XXXX-XXXX-XXXX" \
              -v /data/lactuca:/var/lib/lactuca \
              -e LACTUCA_CONFIG_DIR=/var/lib/lactuca \
              my-image
   ```

2. Mount `LACTUCA_CONFIG_DIR` on a persistent volume so the activated `license.json`
   survives container recreations.

Without both steps, Lactuca will re-activate on every container restart, consuming
a new device slot each time. The previous slot is not released automatically and must
be freed manually from the Lactuca account portal.

**Ephemeral containers (CI/CD, batch jobs)**: if the container is intentionally
short-lived and never reused, set `LACTUCA_LICENSE_KEY` as an environment variable
and do not persist `LACTUCA_CONFIG_DIR`. Each run will activate and consume a device
slot — ensure your license tier has enough device slots for the expected concurrency.
See [How do I use Lactuca in a CI/CD pipeline?](#cicd-pipeline-faq)
for a CI-specific approach that avoids this problem.

**Kubernetes**: the same principles apply. Fix the pod's MAC address via a network
annotation and mount `LACTUCA_CONFIG_DIR` on a `PersistentVolumeClaim`.

(cicd-pipeline-faq)=
### How do I use Lactuca in a CI/CD pipeline?

Store your key as a repository secret and inject it as an environment variable:

```yaml
# GitHub Actions example
- name: Run actuarial tests
  env:
    LACTUCA_LICENSE_KEY: ${{ secrets.LACTUCA_LICENSE_KEY }}
  run: pytest
```

See the [Activation Guide](server-mode-and-cicd) for more details.

### Can multiple users on a JupyterHub share one license?

Yes — this is "server mode". The administrator activates the license once with a shared
`LACTUCA_CONFIG_DIR`, and all users on the server read the same `license.json`. This
consumes one device slot on the license (the server host). See the
[Activation Guide](server-mode-and-cicd).

Note that **concurrent session limits apply per license**, regardless of how many users
access the server. Each active JupyterHub kernel that has imported Lactuca occupies one
concurrent session slot. Users on a shared Individual or Academic license are therefore
limited to 1 simultaneous kernel with Lactuca loaded; Team and Enterprise licenses
allow 10 and 50 simultaneous sessions respectively.

### Can I install Lactuca on a calculation server used by my whole team?

Yes. Any tier allows installing Lactuca on a shared server for internal users. The
activation consumes one slot from the license’s pool — the same as a workstation.
In addition, **concurrent session limits** apply to the number of processes that may
use Lactuca simultaneously on that server:

- **Individual**: permits 1 named user and **1 concurrent session**. Server-side use is
  allowed but limited to that same user's own calculations, one at a time.
- **Team**: one server activation shares the 10-device pool; up to **10 concurrent
  sessions** across the server — suitable for a team running calculations in parallel.
- **Enterprise**: up to 50 activations — suitable for multi-server infrastructure with
  up to **50 concurrent sessions** across all activated devices combined.

Internal users accessing Lactuca through the server do not need their own key. Users
**external to your organisation** are not covered by any standard tier — see
[OEM and third-party distribution](#oem-and-third-party-distribution) below.

---

## Academic & Community licenses

### Who qualifies for a free Academic & Community license?

Individuals using Lactuca for non-commercial, academic, research, or open-source
community purposes — including students, educators, independent academic researchers,
and open-source contributors. See [§4 of the EULA](4-academic-community-license-restrictions)
for the full eligibility criteria and restrictions.

### Is commercial use allowed with an Academic license?

**No.** Academic & Community licenses are strictly non-commercial. Consulting work,
client deliverables, and production systems are not permitted. Violations result in
immediate revocation. See the [EULA](eula).

### How do I apply for an Academic license?

Send an email to [support@lactuca.io](mailto:support@lactuca.io?subject=Academic%20%26%20Community%20License%20Request)
with your name, institution, institutional email, and intended use. Licenses are issued
within 2–3 business days.

---

(oem-and-third-party-distribution)=
## OEM and third-party distribution

### I want to integrate Lactuca into software I sell to my clients. What license do I need?

You need an **OEM license** — a separate written agreement with Licensor. Standard
tiers (Individual, Team, Enterprise) do not permit using Lactuca as a backend engine
for products distributed to users external to your organisation.

OEM licenses are negotiated individually and include:

- **No concurrent session limit** — process leasing (heartbeat) is disabled for OEM
  deployments, so your product can run unlimited parallel Lactuca instances without
  triggering `LicenseSeatExhaustedError`.
- **No heartbeat requirement** — ideal for server-side or embedded deployments where
  maintaining a persistent internet connection per process is impractical.
- Activation pools and device limits set by agreement.

Contact [support@lactuca.io](mailto:support@lactuca.io?subject=OEM%20License%20Enquiry)
with a description of your product and intended use.

### Does an Enterprise license cover a SaaS platform I build on top of Lactuca?

**No.** An Enterprise license covers up to 50 device activations for **internal** named
users of your organisation, with up to 50 concurrent sessions. If your SaaS platform
serves Lactuca calculations to your own clients
(users external to your organisation), that is OEM use and requires a separate OEM
agreement, regardless of the number of end users.

---

## Privacy and data

### What data does activation collect?

- Your email address (for license management and support)
- A hardware fingerprint: a one-way cryptographic hash derived from hardware attributes
  (MAC address, hostname, CPU architecture). The raw values are never transmitted —
  only the irreversible hash is stored. This hash constitutes **pseudonymous data**
  (GDPR Art. 4(5)): it cannot directly identify you as an individual.

Full details are in the [Privacy Policy](https://www.lactuca.io/privacy).

### Where is the data stored?

License data is stored on [Keygen.sh](https://keygen.sh) servers (United States).
Transfers to Keygen.sh are governed by Standard Contractual Clauses (EU Commission
Decision 2021/914). Email addresses used during the trial sign-up are also stored on
Keygen. Payment data is processed and stored by
[Lemon Squeezy](https://lemonsqueezy.com) — Lactuca/Actuaan never sees your card
details.

---

## Still have questions?

Contact us at [support@lactuca.io](mailto:support@lactuca.io).
