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

Activation contacts the Keygen license server to validate the key and bind it to the
current device (hardware fingerprint). A `license.json` file is written to your
configuration directory for subsequent offline use.

---

## Step-by-step: first activation

If you already have a license key, skip to [Entering your key](#entering-your-key).
If you need a trial key, see [Requesting a free trial](#requesting-a-free-trial) below.

(requesting-a-free-trial)=
### Requesting a free trial

Run `import lactuca` without a key. When the activation prompt appears, select
the option to request a free 30-day trial. You will be asked for your email
address (and optionally your name). Before the request is submitted, the EULA
agreement URL is displayed — submitting your email constitutes acceptance.
The trial key is sent to your inbox within a few minutes.

Once you have your key, continue with [Entering your key](#entering-your-key).

(entering-your-key)=
### Entering your key

#### Windows

1. Open a terminal (PowerShell or Command Prompt) and activate your Python environment.
2. Run `import lactuca`.
3. When the activation prompt appears, select the option to enter an existing key
   and paste your license key. The license file is written to
   `%LOCALAPPDATA%\lactuca\license.json`.
4. Subsequent imports are silent.

#### macOS

1. Open Terminal and activate your Python environment.
2. Run `import lactuca`.
3. When the activation prompt appears, select the option to enter an existing key
   and paste your license key. The license file is written to
   `~/Library/Application Support/lactuca/license.json`.
4. Subsequent imports are silent.

#### Linux

1. Open a terminal and activate your Python environment.
2. Run `import lactuca`.
3. When the activation prompt appears, select the option to enter an existing key
   and paste your license key. The license file is written to
   `~/.config/lactuca/license.json`.
4. Subsequent imports are silent.

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
`stderr` on each startup — until the license's local expiry date. There is no
additional offline cutoff beyond `expires_at`; the binding limit is your
subscription or trial end date.

If the expiry date is reached while offline, the library raises a
`LicenseExpiredError` and stops. Renew at the [Pricing page](pricing).

**Process heartbeat (keep-alive every 5 minutes)**

Each running Lactuca process sends a keep-alive heartbeat to the license server
every 5 minutes. A running process is **never interrupted** by network
unavailability. However, **new startups** of Lactuca are permitted offline for
up to **3 days** from the last successful server contact. After 3 days, a new
`import lactuca` raises `LicenseSeatExhaustedError` until connectivity is
restored. In practice, the 3-day heartbeat limit is the binding constraint for
most users.

---

## Expiry warnings

When a valid license is approaching its expiry date, the Software prints a
warning to `stderr` on every startup. There are two urgency levels:

- **30 days or fewer remaining** — informational reminder (consistent with §11.2 of the EULA).
- **7 days or fewer remaining** — escalated to an urgent warning.

Normal computation continues unaffected in both cases. Renew through the
Lemon Squeezy customer portal (link in your subscription confirmation email)
or at the [Pricing page](pricing).

---

## Jupyter and non-interactive environments

If `import lactuca` is executed in a non-interactive environment (Jupyter kernel,
background script, CI pipeline) and no `LACTUCA_LICENSE_KEY` is set and no local
license file exists, the library raises an `ActivationRequiredError`.

To resolve: activate once in an interactive session (see above), or set
`LACTUCA_LICENSE_KEY` before starting Jupyter.

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
an unused device, or contact [support-lactuca@actuaan.com](mailto:support-lactuca@actuaan.com).

**`LicenseSeatExhaustedError`**

All concurrent session slots for your license are in use. This can happen for two
reasons:

- **Too many parallel processes**: another machine or process has consumed all available
  session slots. Wait for a session to close, or see the
  {ref}`FAQ <seat-exhausted>` for options.
- **Offline too long**: no successful server contact in the last 3 days. Restore
  network connectivity and try again.

**`LicenseTamperedError`**

The license file has been modified manually or is corrupted. Delete it and
re-activate.

**`LicenseRevokedError`**

Your license has been suspended or revoked. Contact
[support-lactuca@actuaan.com](mailto:support-lactuca@actuaan.com) if you believe this
is an error.

**`LicenseExpiredError`**

Your license or trial has expired. Renew your subscription or purchase a new plan at
the [Pricing page](pricing).

---

## Error reference

For a complete list of license-related errors and their meaning, see
{doc}`errors_reference`.
