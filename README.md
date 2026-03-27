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
<svg viewBox="0 0 1440 700" width="100%" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Ethereum staking after Pectra flow">
  <defs>
    <marker id="arrow" markerWidth="8" markerHeight="8" refX="7" refY="4" orient="auto">
      <path d="M0,0 L8,4 L0,8 z" fill="#64748b"/>
    </marker>
  </defs>

  <!-- Background -->
  <rect x="20" y="20" width="1400" height="660" rx="18" fill="#ffffff" stroke="#e2e8f0"/>

  <!-- Section labels -->
  <rect x="70" y="60" width="250" height="34" rx="10" fill="#eff6ff" stroke="#bfdbfe"/>
  <text x="88" y="82" font-size="20" fill="#1e3a8a" font-family="Arial, sans-serif" font-weight="700">Execution inclusion</text>

  <rect x="390" y="60" width="300" height="34" rx="10" fill="#fff7ed" stroke="#fed7aa"/>
  <text x="408" y="82" font-size="20" fill="#9a3412" font-family="Arial, sans-serif" font-weight="700">Explicit pending queue</text>

  <rect x="760" y="60" width="560" height="34" rx="10" fill="#f0fdf4" stroke="#bbf7d0"/>
  <text x="778" y="82" font-size="20" fill="#166534" font-family="Arial, sans-serif" font-weight="700">Activation flow</text>

  <!-- Top row -->
  <rect x="70" y="130" width="240" height="120" rx="14" fill="#ffffff" stroke="#cbd5e1"/>
  <text x="90" y="165" font-size="24" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">1. Deposit tx included</text>
  <text x="90" y="195" font-size="16" fill="#475569" font-family="Arial, sans-serif">Tx is in a valid EL block</text>
  <text x="90" y="220" font-size="16" fill="#475569" font-family="Arial, sans-serif">If the block finalizes, tx finalizes</text>
  <text x="90" y="242" font-size="16" fill="#2563eb" font-family="Arial, sans-serif">Not a queue</text>

  <rect x="390" y="130" width="240" height="120" rx="14" fill="#ffffff" stroke="#fdba74"/>
  <text x="410" y="165" font-size="24" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">2. pending_deposits</text>
  <text x="410" y="195" font-size="16" fill="#475569" font-family="Arial, sans-serif">Explicit queue in beacon state</text>
  <text x="410" y="220" font-size="16" fill="#475569" font-family="Arial, sans-serif">Deposit appended by CL processing</text>
  <text x="410" y="242" font-size="16" fill="#c2410c" font-family="Arial, sans-serif">First real waiting stage</text>

  <rect x="710" y="130" width="240" height="120" rx="14" fill="#ffffff" stroke="#cbd5e1"/>
  <text x="730" y="165" font-size="24" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">3. Deposit applied</text>
  <text x="730" y="195" font-size="16" fill="#475569" font-family="Arial, sans-serif">Validator created or updated</text>
  <text x="730" y="220" font-size="16" fill="#475569" font-family="Arial, sans-serif">Balance may already be visible</text>
  <text x="730" y="242" font-size="16" fill="#2563eb" font-family="Arial, sans-serif">Still not active</text>

  <rect x="1030" y="130" width="240" height="120" rx="14" fill="#ffffff" stroke="#86efac"/>
  <text x="1050" y="165" font-size="24" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">4. Enter activation queue</text>
  <text x="1050" y="195" font-size="16" fill="#475569" font-family="Arial, sans-serif">activation_eligibility_epoch set</text>
  <text x="1050" y="220" font-size="16" fill="#475569" font-family="Arial, sans-serif">Implicit queue begins here</text>
  <text x="1050" y="242" font-size="16" fill="#15803d" font-family="Arial, sans-serif">Not a separate list</text>

  <!-- Arrows -->
  <path d="M310 190 L390 190" stroke="#94a3b8" stroke-width="3" fill="none" marker-end="url(#arrow)"/>
  <path d="M630 190 L710 190" stroke="#94a3b8" stroke-width="3" fill="none" marker-end="url(#arrow)"/>
  <path d="M950 190 L1030 190" stroke="#94a3b8" stroke-width="3" fill="none" marker-end="url(#arrow)"/>

  <!-- Right side -->
  <rect x="1090" y="320" width="220" height="96" rx="14" fill="#ffffff" stroke="#86efac"/>
  <text x="1110" y="355" font-size="24" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">5. Wait</text>
  <text x="1110" y="383" font-size="16" fill="#475569" font-family="Arial, sans-serif">Need finalization</text>
  <text x="1110" y="405" font-size="16" fill="#475569" font-family="Arial, sans-serif">and churn room</text>

  <rect x="1090" y="470" width="220" height="96" rx="14" fill="#ffffff" stroke="#86efac"/>
  <text x="1110" y="505" font-size="24" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">6. activation_epoch set</text>
  <text x="1110" y="533" font-size="16" fill="#475569" font-family="Arial, sans-serif">Activation scheduled</text>
  <text x="1110" y="555" font-size="16" fill="#475569" font-family="Arial, sans-serif">by protocol rules</text>

  <rect x="1090" y="610" width="220" height="50" rx="14" fill="#f0fdf4" stroke="#86efac"/>
  <text x="1170" y="643" font-size="26" fill="#166534" font-family="Arial, sans-serif" font-weight="800">7. ACTIVE</text>

  <path d="M1270 250 L1270 320" stroke="#94a3b8" stroke-width="3" fill="none" marker-end="url(#arrow)"/>
  <path d="M1200 416 L1200 470" stroke="#94a3b8" stroke-width="3" fill="none" marker-end="url(#arrow)"/>
  <path d="M1200 566 L1200 610" stroke="#94a3b8" stroke-width="3" fill="none" marker-end="url(#arrow)"/>

  <!-- Bottom cards -->
  <rect x="70" y="320" width="450" height="250" rx="16" fill="#fff7ed" stroke="#fed7aa"/>
  <text x="95" y="360" font-size="28" fill="#9a3412" font-family="Arial, sans-serif" font-weight="700">A. pending_deposits</text>
  <text x="95" y="396" font-size="18" fill="#7c2d12" font-family="Arial, sans-serif">• Explicit queue: state.pending_deposits</text>
  <text x="95" y="430" font-size="18" fill="#7c2d12" font-family="Arial, sans-serif">• Storage limit: 134,217,728</text>
  <text x="95" y="464" font-size="18" fill="#7c2d12" font-family="Arial, sans-serif">• Processing cap: 16 per epoch</text>
  <text x="95" y="498" font-size="18" fill="#7c2d12" font-family="Arial, sans-serif">• Also bounded by activation/exit churn</text>
  <text x="95" y="532" font-size="18" fill="#7c2d12" font-family="Arial, sans-serif">• Deposit may wait here after EL finality</text>

  <rect x="560" y="320" width="460" height="250" rx="16" fill="#f0fdf4" stroke="#bbf7d0"/>
  <text x="585" y="360" font-size="28" fill="#166534" font-family="Arial, sans-serif" font-weight="700">B. Activation queue</text>
  <text x="585" y="396" font-size="18" fill="#166534" font-family="Arial, sans-serif">• Implicit queue, not a dedicated state list</text>
  <text x="585" y="430" font-size="18" fill="#166534" font-family="Arial, sans-serif">• Starts when activation_eligibility_epoch is set</text>
  <text x="585" y="464" font-size="18" fill="#166534" font-family="Arial, sans-serif">• Then waits for finalization and churn</text>
  <text x="585" y="498" font-size="18" fill="#166534" font-family="Arial, sans-serif">• Throughput cap: 256 ETH per epoch</text>
  <text x="585" y="532" font-size="18" fill="#166534" font-family="Arial, sans-serif">• Roughly 8 × 32 ETH validators at cap</text>

  <!-- Timing note -->
  <rect x="560" y="590" width="750" height="56" rx="12" fill="#f8fafc" stroke="#cbd5e1"/>
  <text x="585" y="624" font-size="18" fill="#334155" font-family="Arial, sans-serif">
    Timing note: deposit can be applied first, while activation_eligibility_epoch remains FAR_FUTURE until the next registry-updates pass.
  </text>
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
