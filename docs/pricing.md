---
html_theme.sidebar_secondary.remove: true
---

# Pricing

**Lactuca** is a high-performance life actuarial library for Python. All paid plans are
billed via [Lemon Squeezy](https://lemonsqueezy.com) (Merchant of Record; VAT handled
automatically for all jurisdictions). Prices shown are **ex-VAT**.

---

## Plans

```{raw} html
<!-- EULA acceptance modal -->
<div id="lac-eula-modal" role="dialog" aria-modal="true" aria-labelledby="lac-modal-title"
  style="display:none;position:fixed;inset:0;z-index:9999;background:rgba(0,0,0,0.55);align-items:center;justify-content:center">
  <div style="background:#fff;border-radius:10px;max-width:540px;width:92%;padding:2em 2em 1.6em;box-shadow:0 8px 32px rgba(0,0,0,0.22);position:relative">
    <h3 id="lac-modal-title" style="margin:0 0 0.7em;font-size:1.15em">Before you continue</h3>
    <p style="margin:0 0 1em;font-size:0.93em;line-height:1.5">Please review and accept the Lactuca license terms before proceeding to checkout.</p>
    <label style="display:flex;align-items:flex-start;gap:0.65em;cursor:pointer;font-size:0.93em;line-height:1.5;margin-bottom:1.2em">
      <input type="checkbox" id="lac-modal-check" style="margin-top:3px;flex-shrink:0" onchange="lacModalCheckChange()">
      <span>I have read and accept the
        <a href="eula.html" target="_blank" rel="noopener">End User License Agreement</a>
        (<a href="eula_es.html" target="_blank" rel="noopener">versi&#xF3;n en espa&#xF1;ol</a>)
        and acknowledge that this software requires periodic internet access
        (at least every 3&nbsp;days) to maintain the license.</span>
    </label>
    <div style="display:flex;gap:0.8em;justify-content:flex-end">
      <button onclick="lacModalClose()" style="padding:0.5em 1.2em;border:1px solid #ccc;border-radius:5px;background:#fff;cursor:pointer;font-size:0.93em">Cancel</button>
      <a id="lac-modal-continue" href="#" target="_blank" rel="noopener" onclick="lacModalClose()"
        style="padding:0.5em 1.4em;border-radius:5px;background:#0d6efd;color:#fff;text-decoration:none;font-size:0.93em;opacity:0.45;pointer-events:none"
        aria-disabled="true">Continue to checkout &#x2192;</a>
    </div>
  </div>
</div>
```

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card} Individual
**€69 / month**

- 1 user
- Up to 3 activated devices
- **1 concurrent session** (one instance at a time)
- All minor & patch releases included
- Online license validation; 30-day offline grace period

```{raw} html
<div style="margin-top:1em">
  <button class="sd-btn sd-btn-primary lac-buy-btn"
    onclick="lacModalOpen('https://lactuca-store.actuaan.com/checkout/buy/c39f4a38-3b1e-40ad-85df-90c051bd4fef')"
    style="cursor:pointer;border:none">Buy monthly</button>
</div>
```
:::

:::{grid-item-card} Team
**€580 / month**

- Up to 10 users
- Up to 3 activated devices per user (30 total)
- **Up to 10 concurrent sessions** across all devices
- All minor & patch releases included
- Online license validation; 30-day offline grace period

```{raw} html
<div style="margin-top:1em">
  <button class="sd-btn sd-btn-primary lac-buy-btn"
    onclick="lacModalOpen('https://lactuca-store.actuaan.com/checkout/buy/5bdb0aca-e75c-4515-bcb3-c756aff211d9')"
    style="cursor:pointer;border:none">Buy monthly</button>
</div>
```
:::

:::{grid-item-card} Enterprise
**€1,650 / month**

- Up to 150 device activations (3 per user for up to 50 users)
- **Up to 50 concurrent sessions** across all activated devices
- Covers workstation installation and server/shared deployments for internal users
- Monthly (Lemon Squeezy) and annual (direct contract) plans include identical usage rights
- All minor & patch releases included
- Online license validation; 30-day offline grace period
- Annual billing available on request (contact us for a quote)

```{raw} html
<div style="margin-top:1em">
  <button class="sd-btn sd-btn-primary lac-buy-btn"
    onclick="lacModalOpen('https://lactuca-store.actuaan.com/checkout/buy/c026505b-1bb6-46e4-a250-eb890a57006d')"
    style="cursor:pointer;border:none">Buy monthly</button>
</div>
```
:::

:::{grid-item-card} Academic & Community
**Free**

- Students, educators, independent researchers, open-source contributors, and beta testers
- 1 named user, up to 2 activated devices, **1 concurrent session**
- **Non-commercial use only** — see the [EULA](eula) for restrictions
- Apply by email — no credit card required

```{raw} html
<div style="margin-top:1em">
  <a class="sd-btn sd-btn-outline-secondary" href="mailto:support-lactuca@actuaan.com?subject=Academic%20%26%20Community%20License%20Request">Apply by email</a>
</div>
```
:::

::::

> **VAT**: Lemon Squeezy adds the applicable VAT for your jurisdiction at checkout
> (e.g. 21% in Spain). Enterprise annual billing is handled via direct invoice
> outside the Lemon Squeezy platform; contact us for a quote.

:::{tip} Early Bird — Founding Members
Apply discount code **`FOUNDING`** at checkout on the Individual monthly plan to lock
in **€49 / month permanently** — the discount applies to every renewal for as long as
your subscription remains active. Limited to the first **20 subscribers** in total.
:::

---

## Free 30-Day Trial

**No credit card required. No commitment.**

Start a free 30-day trial to evaluate Lactuca before purchasing. The trial gives you
full access to all features with 2 device activations (laptop + workstation).

Trials are requested directly from the activation prompt when you first import the
library. Install Lactuca and run:

```python
import lactuca
```

The activation prompt will appear:

```
Lactuca requires a license key to run.

  [T] Request a free 30-day trial (no credit card required)
  [K] Enter an existing license key
  [Q] Quit

Choice [T/K/Q]: T
```

Enter your name (optional) and email address. Your trial key will arrive by email
within a few minutes. Once you have it, run `import lactuca` again and choose **[K]**
to enter the key.

See the [Activation Guide](activation) for step-by-step instructions.

---

## Academic & Community Process

1. Send an email to [support-lactuca@actuaan.com](mailto:support-lactuca@actuaan.com?subject=Academic%20%26%20Community%20License%20Request) with:
   - Your full name
   - Your institution
   - Your institutional email address (`.edu`, `.ac.uk`, `.upm.es`, etc.)
   - Brief description of intended use (thesis, research, teaching, open-source project)
2. Licenses are reviewed and issued within 2–3 business days.
3. The license is valid for 1 year and can be renewed following the same process.

Academic licenses are **non-commercial only** — see §3 of the [EULA](eula) for the
full restrictions.

---

## Feature Comparison

| Feature | Trial | Individual | Team | Enterprise | Academic |
|---|:---:|:---:|:---:|:---:|:---:|
| All actuarial table types | ✅ | ✅ | ✅ | ✅ | ✅ |
| Life, disability, exit tables | ✅ | ✅ | ✅ | ✅ | ✅ |
| Annuities & insurances | ✅ | ✅ | ✅ | ✅ | ✅ |
| Multiple-decrement models | ✅ | ✅ | ✅ | ✅ | ✅ |
| Generational mortality | ✅ | ✅ | ✅ | ✅ | ✅ |
| Offline grace period (30 d) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Concurrent sessions | 1 | 1 | ≤ 10 | ≤ 50 | 1 |
| Commercial use | ✅ | ✅ | ✅ | ✅ | ❌ |
| Number of users | 1 | 1 | ≤ 10 | ≤ 50 | 1 |
| Devices per user | 2 | 3 | 3 | 3 | 2 |
| Duration | 30 days | Monthly | Monthly | Monthly / direct annual | 1 year |

---

## FAQ

**Can I upgrade from Individual to Team?**
Yes. Purchase the Team plan and activate with your new key. See the
[FAQ](faq_licensing) for the full upgrade procedure.

**What happens when my trial expires?**
`import lactuca` raises `LicenseExpiredError`. Purchase any paid plan to continue.

**Is VAT included in the prices shown?**
No. Prices shown are ex-VAT. Lemon Squeezy adds the applicable VAT for your
jurisdiction at checkout.

**Can I use Lactuca in a CI/CD pipeline?**
Yes — set the `LACTUCA_LICENSE_KEY` environment variable. See the
[Activation Guide](server-mode-and-cicd).

---

## OEM and SaaS Licensing

If you want to embed Lactuca as a calculation engine in a product or service
distributed or made available to your own clients or external users, you need
an **OEM license** — a separate written agreement with Licensor.

Standard license tiers (Individual, Team, Enterprise) do not cover OEM use.

**Typical OEM scenarios:**

- A SaaS platform running Lactuca server-side for your customers.
- An application or API sold to third parties that calls Lactuca internally.
- Any product where your clients benefit from Lactuca without obtaining their
  own Lactuca license directly from Licensor.

To enquire about OEM licensing, contact
[support-lactuca@actuaan.com](mailto:support-lactuca@actuaan.com?subject=OEM%20License%20Enquiry)
with a description of your product and estimated scale.

```{raw} html
<script>
function lacModalOpen(url) {
  var modal = document.getElementById('lac-eula-modal');
  var chk = document.getElementById('lac-modal-check');
  var btn = document.getElementById('lac-modal-continue');
  chk.checked = false;
  btn.style.opacity = '0.45';
  btn.style.pointerEvents = 'none';
  btn.setAttribute('aria-disabled', 'true');
  btn.href = url;
  modal.style.display = 'flex';
  document.body.style.overflow = 'hidden';
}
function lacModalClose() {
  document.getElementById('lac-eula-modal').style.display = 'none';
  document.body.style.overflow = '';
}
function lacModalCheckChange() {
  var chk = document.getElementById('lac-modal-check');
  var btn = document.getElementById('lac-modal-continue');
  if (chk.checked) {
    btn.style.opacity = '';
    btn.style.pointerEvents = '';
    btn.removeAttribute('aria-disabled');
  } else {
    btn.style.opacity = '0.45';
    btn.style.pointerEvents = 'none';
    btn.setAttribute('aria-disabled', 'true');
  }
}
document.addEventListener('keydown', function(e) {
  if (e.key === 'Escape') { lacModalClose(); }
});
document.getElementById('lac-eula-modal').addEventListener('click', function(e) {
  if (e.target === this) { lacModalClose(); }
});
</script>
```
