# Ethereum Staking after Pectra: Deposit → Pending Queues → Active Validator

A clean explanation of the **post-Pectra / Electra** onboarding path for a **new validator**.

---

## Short answer

After a deposit transaction is included in a valid canonical execution block, the validator path is:

```text
EL deposit tx included
→ appended to pending_deposits
→ deposit applied to beacon state
→ activation_eligibility_epoch set
→ waits for finalization + churn
→ activation_epoch set
→ ACTIVE
```

There is **no separate queue before `pending_deposits`**.

There are **2 waiting stages that matter**:

1. **`pending_deposits`** — explicit queue in beacon state  
2. **activation queue** — implicit queue formed by validator state fields and activation rules

---

## Diagram

<div style="overflow-x:auto; margin:20px 0;">
<svg viewBox="0 0 1200 1450" width="100%" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Ethereum staking after Pectra flow">
  <defs>
    <marker id="arrow" markerWidth="10" markerHeight="10" refX="8.5" refY="5" orient="auto">
      <path d="M0,0 L10,5 L0,10 z" fill="#64748b"/>
    </marker>
  </defs>

  <rect x="24" y="24" width="1152" height="1402" rx="24" fill="#ffffff" stroke="#dbe4ee" stroke-width="2"/>

  <!-- Section pills -->
  <rect x="70" y="60" width="260" height="42" rx="14" fill="#e9f2ff" stroke="#9ec5ff" stroke-width="2"/>
  <text x="92" y="88" font-size="22" fill="#1d4ed8" font-family="Arial, sans-serif" font-weight="700">Execution inclusion</text>

  <rect x="380" y="60" width="300" height="42" rx="14" fill="#fff1e7" stroke="#ffc48d" stroke-width="2"/>
  <text x="402" y="88" font-size="22" fill="#c2410c" font-family="Arial, sans-serif" font-weight="700">Explicit pending queue</text>

  <rect x="730" y="60" width="370" height="42" rx="14" fill="#ebfff2" stroke="#9fe3b4" stroke-width="2"/>
  <text x="752" y="88" font-size="22" fill="#15803d" font-family="Arial, sans-serif" font-weight="700">Activation flow</text>

  <!-- Top flow -->
  <rect x="70" y="140" width="250" height="150" rx="18" fill="#ffffff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="92" y="182" font-size="27" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">1. Deposit tx included</text>
  <text x="92" y="222" font-size="18" fill="#475569" font-family="Arial, sans-serif">Tx is in a valid EL block</text>
  <text x="92" y="252" font-size="18" fill="#475569" font-family="Arial, sans-serif">If the block finalizes, tx finalizes</text>
  <text x="92" y="278" font-size="18" fill="#2563eb" font-family="Arial, sans-serif">Not a queue</text>

  <rect x="355" y="140" width="270" height="150" rx="18" fill="#ffffff" stroke="#ffb672" stroke-width="2"/>
  <text x="377" y="182" font-size="27" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">2. pending_deposits</text>
  <text x="377" y="222" font-size="18" fill="#475569" font-family="Arial, sans-serif">Explicit queue in beacon state</text>
  <text x="377" y="252" font-size="18" fill="#475569" font-family="Arial, sans-serif">Deposit appended by CL processing</text>
  <text x="377" y="278" font-size="18" fill="#ea580c" font-family="Arial, sans-serif">First real waiting stage</text>

  <rect x="660" y="140" width="250" height="150" rx="18" fill="#ffffff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="682" y="182" font-size="27" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">3. Deposit applied</text>
  <text x="682" y="222" font-size="18" fill="#475569" font-family="Arial, sans-serif">Validator created or updated</text>
  <text x="682" y="252" font-size="18" fill="#475569" font-family="Arial, sans-serif">Balance may already be visible</text>
  <text x="682" y="278" font-size="18" fill="#2563eb" font-family="Arial, sans-serif">Still not active</text>

  <rect x="945" y="140" width="195" height="150" rx="18" fill="#ffffff" stroke="#8ee1a9" stroke-width="2"/>
  <text x="967" y="182" font-size="22" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">4. Enter activation</text>
  <text x="967" y="210" font-size="22" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">queue</text>
  <text x="967" y="242" font-size="17" fill="#475569" font-family="Arial, sans-serif">activation_eligibility_</text>
  <text x="967" y="264" font-size="17" fill="#475569" font-family="Arial, sans-serif">epoch set</text>
  <text x="967" y="286" font-size="17" fill="#15803d" font-family="Arial, sans-serif">Implicit queue begins here</text>

  <path d="M320 215 L355 215" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>
  <path d="M625 215 L660 215" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>
  <path d="M910 215 L945 215" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>

  <!-- Vertical activation column -->
  <rect x="860" y="380" width="280" height="120" rx="18" fill="#ffffff" stroke="#8ee1a9" stroke-width="2"/>
  <text x="885" y="425" font-size="27" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">5. Wait</text>
  <text x="885" y="463" font-size="18" fill="#475569" font-family="Arial, sans-serif">Need finalization</text>
  <text x="885" y="491" font-size="18" fill="#475569" font-family="Arial, sans-serif">and churn room</text>

  <rect x="860" y="560" width="280" height="120" rx="18" fill="#ffffff" stroke="#8ee1a9" stroke-width="2"/>
  <text x="885" y="605" font-size="27" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">6. activation_epoch set</text>
  <text x="885" y="643" font-size="18" fill="#475569" font-family="Arial, sans-serif">Activation scheduled</text>
  <text x="885" y="671" font-size="18" fill="#475569" font-family="Arial, sans-serif">by protocol rules</text>

  <rect x="860" y="740" width="280" height="100" rx="18" fill="#ebfff2" stroke="#8ee1a9" stroke-width="2"/>
  <text x="936" y="802" font-size="34" fill="#15803d" font-family="Arial, sans-serif" font-weight="800">7. ACTIVE</text>

  <path d="M1042 290 L1042 380" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>
  <path d="M1000 500 L1000 560" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>
  <path d="M1000 680 L1000 740" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>

  <!-- Bottom explanation cards -->
  <rect x="70" y="380" width="740" height="320" rx="20" fill="#fff5ee" stroke="#ffc48d" stroke-width="2"/>
  <text x="100" y="432" font-size="34" fill="#b45309" font-family="Arial, sans-serif" font-weight="700">A. pending_deposits</text>
  <text x="100" y="484" font-size="22" fill="#9a3412" font-family="Arial, sans-serif">• Explicit queue: state.pending_deposits</text>
  <text x="100" y="532" font-size="22" fill="#9a3412" font-family="Arial, sans-serif">• Storage limit: 134,217,728</text>
  <text x="100" y="580" font-size="22" fill="#9a3412" font-family="Arial, sans-serif">• Processing cap: 16 per epoch</text>
  <text x="100" y="628" font-size="22" fill="#9a3412" font-family="Arial, sans-serif">• Also bounded by activation/exit churn</text>
  <text x="100" y="676" font-size="22" fill="#9a3412" font-family="Arial, sans-serif">• Deposit may wait here after EL finality</text>

  <rect x="70" y="760" width="740" height="320" rx="20" fill="#effdf3" stroke="#9fe3b4" stroke-width="2"/>
  <text x="100" y="812" font-size="34" fill="#15803d" font-family="Arial, sans-serif" font-weight="700">B. Activation queue</text>
  <text x="100" y="864" font-size="22" fill="#166534" font-family="Arial, sans-serif">• Implicit queue, not a dedicated state list</text>
  <text x="100" y="912" font-size="22" fill="#166534" font-family="Arial, sans-serif">• Starts when activation_eligibility_epoch is set</text>
  <text x="100" y="960" font-size="22" fill="#166534" font-family="Arial, sans-serif">• Then waits for finalization and churn</text>
  <text x="100" y="1008" font-size="22" fill="#166534" font-family="Arial, sans-serif">• Throughput cap: 256 ETH per epoch</text>
  <text x="100" y="1056" font-size="22" fill="#166534" font-family="Arial, sans-serif">• Roughly 8 × 32 ETH validators at cap</text>

  <!-- Timing note -->
  <rect x="860" y="910" width="280" height="170" rx="18" fill="#f8fbff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="885" y="950" font-size="22" fill="#334155" font-family="Arial, sans-serif" font-weight="700">Timing note</text>
  <text x="885" y="992" font-size="18" fill="#334155" font-family="Arial, sans-serif">Deposit can be applied first,</text>
  <text x="885" y="1022" font-size="18" fill="#334155" font-family="Arial, sans-serif">while activation_eligibility_</text>
  <text x="885" y="1048" font-size="18" fill="#334155" font-family="Arial, sans-serif">epoch remains FAR_FUTURE</text>
  <text x="885" y="1074" font-size="18" fill="#334155" font-family="Arial, sans-serif">until the next registry-updates pass.</text>
</svg>
</div>

---

## Clean state-by-state view

### 1) Deposit tx included in execution layer
Your deposit transaction is included in a valid execution-layer block.

- This is **not** a queue.
- If the block finalizes, the transaction finalizes.
- Once the consensus client processes that block, the deposit request is appended to `state.pending_deposits`.

### 2) `pending_deposits`
This is the first real queue.

- State object: `state.pending_deposits`
- It is an explicit list in beacon state
- New deposits wait here before being applied to validator state

#### Limits
- `PENDING_DEPOSITS_LIMIT = 134,217,728`
- `MAX_PENDING_DEPOSITS_PER_EPOCH = 16`

### 3) Deposit applied to beacon state
When `process_pending_deposits()` handles the entry:

- if the pubkey is new, a validator record is created
- otherwise, the existing validator balance is updated

At this point:

- the balance can already be visible in consensus state
- the validator is still not active
- `activation_eligibility_epoch` may still be `FAR_FUTURE_EPOCH`

### 4) Validator enters activation queue
The validator effectively enters the activation queue when `activation_eligibility_epoch` is set.

This happens in `process_registry_updates()` when the validator satisfies the activation-queue condition, including:

- `activation_eligibility_epoch == FAR_FUTURE_EPOCH`
- `effective_balance >= MIN_ACTIVATION_BALANCE`

#### Minimum balance
- `MIN_ACTIVATION_BALANCE = 32 ETH`

### 5) Wait for finalization
The validator must wait until:

```text
finalized_checkpoint.epoch >= activation_eligibility_epoch
```

Only then can it be assigned an `activation_epoch`.

### 6) `activation_epoch` is set
Once eligible, the validator is assigned `activation_epoch`.

### 7) Active validator
When the chain reaches `activation_epoch`, the validator becomes active and starts receiving duties.
