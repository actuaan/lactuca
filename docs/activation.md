# Activation Guide

This page explains how to activate Lactuca after obtaining a license key or trial key.

---

## How activation works

On the first `import lactuca`, the library checks for a valid license in the following
order:

1. **Environment variable** `LACTUCA_LICENSE_KEY` — if set, the key is validated online
   and stored automatically (no prompt).
2. **Local license file** `license.json` — if present and valid, the library loads
   silently.
3. **Interactive prompt** — if neither of the above is found, you are asked to enter
   your key.

Activation contacts the Lactuca license server to validate the key and bind it to the
current device (hardware fingerprint). A `license.json` file is written to your
configuration directory for subsequent offline use.

### Verification checks performed on `license.json`

On each subsequent `import lactuca`, the library verifies the local file in the
following order:

1. **Expiry check** — `expires_at` must be in the future.
2. **Device fingerprint check** — the fingerprint in the file must match the current
   machine.
3. **MAC integrity check** — a device-bound HMAC-SHA256 code covers `expires_at`,
   `tier`, `last_validated_at`, and `last_process_heartbeat`.  A missing `mac` field
   raises `LicenseTamperedError` **[LAC-3003]** (auto-recoverable); a mismatch raises
   **[LAC-3004]** (auto-recoverable).
4. **Clock rollback check** — the system clock must not be more than 60 seconds behind
   `last_validated_at`.  A larger rollback raises `LicenseTamperedError`
   **[LAC-3005]** (requires restoring the system clock).
5. **Ed25519 signature verification** — the server-issued cryptographic signature is
   verified last.  A missing or invalid signature raises `LicenseTamperedError`
   **[LAC-3001]** (auto-recoverable) or **[LAC-3002]** (auto-recoverable).

`license.json` is device-bound and cannot be shared between machines. Copying it to
another device triggers **[LAC-3004]** or **[LAC-2002]** (fingerprint mismatch).
Recovery using the stored key applies only on the **same** device; on a new machine,
activate independently or use the {ref}`device transfer procedure <device-transfer>`.

---

## License CLI quick reference

Use the license CLI when you want explicit, script-friendly checks without waiting for
the next automatic revalidation window.

```bash
python -m lactuca              # defaults to activate (same as the line below)
python -m lactuca activate
python -m lactuca license refresh [--json]
python -m lactuca license status [--json]
python -m lactuca license doctor [--json]
```

### When to use each command

- `python -m lactuca` (no subcommand): **identical** to `python -m lactuca activate` — the
  `activate` subcommand is assumed when omitted.
- `activate`: verify an existing license (same checks as `import lactuca`) and print a
  **single status line to stdout** when the license is already valid; otherwise open the
  interactive menu to enter a key or request a trial.
- `license refresh`: force online synchronization now (recommended after support confirms
  a server-side adjustment).
- `license status`: inspect local state only (no network call).
- `license doctor`: run guided diagnostics (local checks + connectivity).

### Import vs CLI — where messages appear

| Action | Already-valid license | First activation / re-activation |
|---|---|---|
| `import lactuca` | Silent on stdout; expiry reminder on **stderr** when ≤30 days remain | Interactive menu (`[T/K/Q]` or `[K/Q]`) |
| `python -m lactuca` or `python -m lactuca activate` | **One** consolidated status line on **stdout** (tier, expiry date, renewal URL) | Interactive menu, then confirmation on stdout |

The CLI skips the import-time license gate so verification runs once, in the
`activate` handler. Expiry reminders are not duplicated on stderr and stdout in the
same command.

> **Non-interactive CLI:** `python -m lactuca activate` works without a TTY when
> `license.json` already exists or `LACTUCA_LICENSE_KEY` is set. Without either, the
> command exits with code `1` and instructions to set the environment variable.

### Exit codes — `python -m lactuca` / `activate`

The `activate` command uses a simplified exit scheme for interactive use (not the
structured codes used by `license` subcommands):

| Code | Meaning |
|---|---|
| `0` | License verified or activated; user cancelled or declined activation (`ActivationRequiredError`); trial request submitted (`already_printed=True`); or `KeyboardInterrupt` |
| `1` | Any other licensing error (`LactucaLicenseError`); or non-interactive use without a license file and without `LACTUCA_LICENSE_KEY` |

On success with an **already active** license, the command exits `0` after printing
a status line to stdout (expiry date and renewal URL when applicable).

On success after **entering a new key** interactively, the command exits `0` after
printing the activation confirmation to stdout.

### Exit codes — `python -m lactuca license …`

Structured exit codes for `license refresh`, `license status`, and `license doctor`:

| Code | Meaning |
|---|---|
| `0` | Success |
| `2` | Network error (server unreachable) |
| `3` | No key available |
| `4` | License expired |
| `5` | License revoked, suspended, or permanently deleted on the license server |
| `6` | Invalid/unsupported server status |
| `7` | License command usage error |
| `10` | Unexpected internal error |

### Example outputs

Text mode — license already active (≤30 days remaining):

```text
[Lactuca] License active (individual). Expires in 12 days (2026-06-30). Renew at: https://www.lactuca.io/pricing
```

Text mode — license already active (≤7 days remaining; urgent wording):

```text
[Lactuca] License active (individual). WARNING: expires in 5 day(s) (2026-06-22). Renew now: https://www.lactuca.io/pricing
```

Text mode — interactive activation just completed:

```text
License activated. Lactuca is ready.
```

Text mode — license refresh:

```text
License synchronized successfully.
```

Doctor offline:

```text
Could not reach license server during diagnosis.
```

JSON mode:

```json
{"status": "network_error", "code": 2, "message": "Could not reach license server during diagnosis.", "expires_at": null, "tier": null, "recommendation": "Check network access and run 'python -m lactuca license doctor' again.", "checks": [{"check": "license_file", "result": "ok", "detail": "license.json found"}]}
```

---

## Step-by-step: first activation

If you already have a license key, skip to [Entering your key](#entering-your-key).
If you need a trial key, see [Requesting a free trial](#requesting-a-free-trial) below.

(requesting-a-free-trial)=
### Requesting a free trial

Run `import lactuca` without a key. When the activation prompt appears, select
option **[T]** to request a free 30-day trial. You will be asked for your email
address and your name (optional). A confirmation step lets you correct the email
before the request is sent. Before the request is submitted, the EULA agreement
URL is displayed — submitting your email constitutes acceptance.
The trial key is sent to your inbox within a few minutes.

> **Device limit**: a Trial license can be activated on **1 device** only. If you
> need to move the trial to a different machine, contact
> [support@lactuca.io](mailto:support@lactuca.io).

> **Note**: The `[T]` option only appears on **first install** (when no prior
> activation has occurred on this machine). If you are re-activating an existing
> license, the prompt shows `[K/Q]` only — enter your existing key using option
> `[K]`.

Once you have your key, re-run `import lactuca` and choose **[K]** to enter it.

### Common trial errors

**`A trial already exists for this email or device`**

A 30-day trial has already been issued to this email address or to this machine.
Check your inbox for the original trial key email. If you cannot find it, contact
[support](mailto:support@lactuca.io) or [purchase a license](pricing).

**`This device has an active paid license`**

This machine already has an active paid Lactuca license associated with it. Choose
option **[K]** and enter your license key instead of requesting a trial.

**`The license on this device has been revoked or suspended`**

The license associated with this machine has been suspended or revoked. Contact
[support@lactuca.io](mailto:support@lactuca.io) for assistance.

(entering-your-key)=
### Entering your key

#### Windows

1. Open a terminal (PowerShell, Command Prompt, or Git Bash) and activate your Python environment.
2. Run `python -m lactuca` (or `import lactuca` in a Python session).
3. When the activation prompt appears, select the option to enter an existing key
   and paste your license key. The license file is written to
   `%LOCALAPPDATA%\lactuca\license.json`.
4. Subsequent `import lactuca` calls are silent on stdout; use `python -m lactuca` to
   print the current license status.

#### macOS

1. Open Terminal and activate your Python environment.
2. Run `python -m lactuca` (or `import lactuca` in a Python session).
3. When the activation prompt appears, select the option to enter an existing key
   and paste your license key. The license file is written to
   `~/Library/Application Support/lactuca/license.json`.
4. Subsequent `import lactuca` calls are silent on stdout; use `python -m lactuca` to
   print the current license status.

#### Linux

1. Open a terminal and activate your Python environment.
2. Run `python -m lactuca` (or `import lactuca` in a Python session).
3. When the activation prompt appears, select the option to enter an existing key
   and paste your license key. The license file is written to
   `~/.config/lactuca/license.json`.
4. Subsequent `import lactuca` calls are silent on stdout; use `python -m lactuca` to
   print the current license status.

#### Jupyter notebook or JupyterLab

1. In a notebook cell, run `import lactuca`.
2. The activation menu appears directly in the cell output — no terminal needed.
3. Select `[T]` for a free trial or `[K]` to enter an existing key.
4. After activation, subsequent cells import silently.

---

## License file locations

| OS | Default path |
|---|---|
| Windows | `%LOCALAPPDATA%\lactuca\license.json` |
| macOS | `~/Library/Application Support/lactuca/license.json` |
| Linux | `~/.config/lactuca/license.json` |

You can override the directory with the `LACTUCA_CONFIG_DIR` environment variable.

---

(server-mode-and-cicd)=
## Server mode and CI/CD

For shared servers (JupyterHub, HPC clusters) or automated pipelines where interactive
prompts are not available, set the `LACTUCA_LICENSE_KEY` environment variable before
importing Lactuca:

```bash
export LACTUCA_LICENSE_KEY="XXXX-XXXX-XXXX-XXXX"
python -c "import lactuca"
```

The library validates and stores the key automatically on first run, then uses the
local `license.json` on subsequent imports (no network call unless the 30-day
revalidation period elapses).

**CI/CD (GitHub Actions example)**:

```yaml
- name: Run actuarial tests
  env:
    LACTUCA_LICENSE_KEY: ${{ secrets.LACTUCA_LICENSE_KEY }}
  run: pytest
```

Store your key in the repository's secret store, never in plain text.

**Shared server (admin sets once, all users benefit)**:

```bash
# Admin: activate once with a shared config directory
export LACTUCA_CONFIG_DIR=/etc/lactuca
export LACTUCA_LICENSE_KEY="XXXX-XXXX-XXXX-XXXX"
python -c "import lactuca"    # writes /etc/lactuca/license.json
chmod 644 /etc/lactuca/license.json

# All users: set LACTUCA_CONFIG_DIR to point to the shared location
# (add to /etc/environment or equivalent)
LACTUCA_CONFIG_DIR=/etc/lactuca
```

One activation is consumed for the server host, regardless of the number of concurrent
users. Ensure the device limit of your tier covers the number of server hosts.

---

## Offline grace period

Lactuca enforces two independent offline grace periods:

**License validation (revalidation every 30 days)**

The license is revalidated online every 30 days. If the network is unavailable
at revalidation time, the library continues operating — printing a warning to
`stderr` on each `import lactuca` startup — until the license's local expiry date.
There is no additional offline cutoff beyond `expires_at`; the binding limit is your
subscription or trial end date.

If a prior `license refresh` or `license doctor` run recorded that the license was
revoked or permanently deleted on the server, the next `import lactuca` **forces an
immediate online check** and blocks access when the network is available — even if
the 30-day interval has not elapsed.

If the expiry date is reached while offline, the library raises a
`LicenseExpiredError` and stops. Renew at the [Pricing page](pricing).

**Process heartbeat (keep-alive every 10–20 minutes)**

Each running Lactuca process sends a keep-alive heartbeat to the license server
approximately every **10 minutes** on single-seat tiers (Trial, Individual,
Academic) and every **20 minutes** on Team and Enterprise. The interval is
derived from the license server's process policy (minimum 5 minutes). **OEM
licenses** do not use floating-seat heartbeats under the normal contractual
setup (see the Licensing FAQ — OEM and third-party distribution).

A running process is **never interrupted** by network unavailability. However,
**new startups** on finite-pool tiers are permitted offline for up to **3 days**
from the last successful server contact. After 3 days, a new `import lactuca`
raises `LicenseSeatExhaustedError` until connectivity is restored. In practice,
the 3-day heartbeat limit is the binding constraint for most users.

---

## Expiry warnings

When a valid license is approaching its expiry date, the Software reminds you to renew.
Where the message appears depends on how you start Lactuca:

| Startup | Channel | Wording |
|---|---|---|
| `import lactuca` (library use) | **stderr** | Short reminder (`Your license expires in N days…`) |
| `python -m lactuca` / `activate` | **stdout** | Consolidated status line (`License active… Expires in N days…`) |

There are two urgency levels (consistent with section 11.2 of the EULA):

- **30 days or fewer remaining** — informational reminder.
- **7 days or fewer remaining** — escalated to an urgent `WARNING` (CLI status line and
  import-time stderr).

Normal computation continues unaffected in both cases. Renew through the
Lemon Squeezy customer portal (link in your subscription confirmation email)
or at the [Pricing page](pricing).

---

## Jupyter and non-interactive environments

The activation menu (`[T/K/Q]` or `[K/Q]`) appears in **any environment that
supports `input()`**: terminal, Jupyter, JupyterLab, VS Code Notebooks, Spyder,
and PyCharm. No special setup is required; simply run `import lactuca` and follow
the on-screen prompt.

In **headless environments** (CI, Docker, stdin redirected to `/dev/null`), the
menu is printed but `input()` fails immediately with `EOFError`. The library
converts this to an `ActivationRequiredError` with an actionable message:

- **First install** (no prior activation on this machine): message includes the
  free-trial URL and instructions to set `LACTUCA_LICENSE_KEY`.
- **Re-activation** (configuration directory exists but `license.json` is missing):
  message instructs to set `LACTUCA_LICENSE_KEY` or run
  `python -m lactuca activate`.

For CI/CD pipelines, always set `LACTUCA_LICENSE_KEY` as a secret — see
[Server mode and CI/CD](#server-mode-and-cicd).

---

## Troubleshooting

**`LicenseInvalidError` — device not recognised**

The hardware fingerprint stored in `license.json` does not match this device.
This happens when the file is copied from another machine or the hardware
changes significantly. Delete `license.json` and re-activate on this device
(consumes one device slot on your license), or use the
{ref}`device transfer procedure <device-transfer>`.

**`LicenseInvalidError` — activation limit reached**

Your license has reached the maximum number of activated devices. See the
{ref}`device transfer FAQ <device-transfer>` for instructions on deactivating
an unused device, or contact [support@lactuca.io](mailto:support@lactuca.io).

**`LicenseSeatExhaustedError`**

All concurrent session slots for your license are in use. This can happen for two
reasons:

- **Too many parallel processes**: another machine or process has consumed all available
  session slots. Wait for a session to close, or see the
  {ref}`FAQ <seat-exhausted>` for options.
- **Offline too long**: no successful server contact in the last 3 days. Restore
  network connectivity and try again.

**`LicenseTamperedError`**

The license file has been modified manually or is corrupted. The library
attempts **automatic online recovery** by re-validating your stored key with the
license server and rewriting `license.json`. No manual action is required in most
cases.

If recovery fails (for example, no network connection), run:

```bash
python -m lactuca license doctor
python -m lactuca license refresh
```

If recovery still fails, delete `license.json` manually and re-activate. On
re-import, the activation prompt shows `[K/Q]` (no trial option, since a prior
activation exists on this machine). Enter your license key to restore access.

**`LicenseRevokedError`**

Your license has been suspended, revoked, or permanently deleted on the license
server. Contact [support@lactuca.io](mailto:support@lactuca.io) if you believe this
is an error. Run `python -m lactuca license refresh` to confirm the server
status before contacting support.

**`LicenseExpiredError`**

Your license or trial has expired. Renew your subscription or purchase a new plan at
the [Pricing page](pricing).

---

## Error reference

For a complete list of license-related errors and their meaning, see
{doc}`errors_reference`.
