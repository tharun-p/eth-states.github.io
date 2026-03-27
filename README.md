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
<svg viewBox="0 0 1720 980" width="100%" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Ethereum staking after Pectra flow">
  <defs>
    <marker id="arrow" markerWidth="10" markerHeight="10" refX="8.5" refY="5" orient="auto">
      <path d="M0,0 L10,5 L0,10 z" fill="#64748b"/>
    </marker>
  </defs>

  <!-- Background -->
  <rect x="24" y="24" width="1672" height="932" rx="22" fill="#ffffff" stroke="#dbe4ee" stroke-width="2"/>

  <!-- Section pills -->
  <rect x="86" y="70" width="290" height="42" rx="14" fill="#e9f2ff" stroke="#9ec5ff" stroke-width="2"/>
  <text x="108" y="98" font-size="22" fill="#1d4ed8" font-family="Arial, sans-serif" font-weight="700">Execution inclusion</text>

  <rect x="452" y="70" width="340" height="42" rx="14" fill="#fff1e7" stroke="#ffc48d" stroke-width="2"/>
  <text x="474" y="98" font-size="22" fill="#c2410c" font-family="Arial, sans-serif" font-weight="700">Explicit pending queue</text>

  <rect x="868" y="70" width="760" height="42" rx="14" fill="#ebfff2" stroke="#9fe3b4" stroke-width="2"/>
  <text x="890" y="98" font-size="22" fill="#15803d" font-family="Arial, sans-serif" font-weight="700">Activation flow</text>

  <!-- Top flow boxes -->
  <rect x="86" y="150" width="280" height="150" rx="18" fill="#ffffff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="110" y="192" font-size="28" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">1. Deposit tx included</text>
  <text x="110" y="232" font-size="18" fill="#475569" font-family="Arial, sans-serif">Tx is in a valid EL block</text>
  <text x="110" y="262" font-size="18" fill="#475569" font-family="Arial, sans-serif">If the block finalizes, tx finalizes</text>
  <text x="110" y="288" font-size="18" fill="#2563eb" font-family="Arial, sans-serif">Not a queue</text>

  <rect x="446" y="150" width="300" height="150" rx="18" fill="#ffffff" stroke="#ffb672" stroke-width="2"/>
  <text x="470" y="192" font-size="28" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">2. pending_deposits</text>
  <text x="470" y="232" font-size="18" fill="#475569" font-family="Arial, sans-serif">Explicit queue in beacon state</text>
  <text x="470" y="262" font-size="18" fill="#475569" font-family="Arial, sans-serif">Deposit appended by CL processing</text>
  <text x="470" y="288" font-size="18" fill="#ea580c" font-family="Arial, sans-serif">First real waiting stage</text>

  <rect x="826" y="150" width="300" height="150" rx="18" fill="#ffffff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="850" y="192" font-size="28" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">3. Deposit applied</text>
  <text x="850" y="232" font-size="18" fill="#475569" font-family="Arial, sans-serif">Validator created or updated</text>
  <text x="850" y="262" font-size="18" fill="#475569" font-family="Arial, sans-serif">Balance may already be visible</text>
  <text x="850" y="288" font-size="18" fill="#2563eb" font-family="Arial, sans-serif">Still not active</text>

  <rect x="1206" y="150" width="340" height="150" rx="18" fill="#ffffff" stroke="#8ee1a9" stroke-width="2"/>
  <text x="1230" y="192" font-size="28" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">4. Enter activation queue</text>
  <text x="1230" y="232" font-size="18" fill="#475569" font-family="Arial, sans-serif">activation_eligibility_epoch set</text>
  <text x="1230" y="262" font-size="18" fill="#475569" font-family="Arial, sans-serif">Implicit queue begins here</text>
  <text x="1230" y="288" font-size="18" fill="#15803d" font-family="Arial, sans-serif">Not a separate list</text>

  <!-- Top row arrows -->
  <path d="M366 225 L446 225" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>
  <path d="M746 225 L826 225" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>
  <path d="M1126 225 L1206 225" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>

  <!-- Bottom left cards -->
  <rect x="86" y="390" width="560" height="300" rx="20" fill="#fff5ee" stroke="#ffc48d" stroke-width="2"/>
  <text x="116" y="442" font-size="34" fill="#b45309" font-family="Arial, sans-serif" font-weight="700">A. pending_deposits</text>
  <text x="116" y="492" font-size="20" fill="#9a3412" font-family="Arial, sans-serif">• Explicit queue: state.pending_deposits</text>
  <text x="116" y="536" font-size="20" fill="#9a3412" font-family="Arial, sans-serif">• Storage limit: 134,217,728</text>
  <text x="116" y="580" font-size="20" fill="#9a3412" font-family="Arial, sans-serif">• Processing cap: 16 per epoch</text>
  <text x="116" y="624" font-size="20" fill="#9a3412" font-family="Arial, sans-serif">• Also bounded by activation/exit churn</text>
  <text x="116" y="668" font-size="20" fill="#9a3412" font-family="Arial, sans-serif">• Deposit may wait here after EL finality</text>

  <rect x="694" y="390" width="620" height="300" rx="20" fill="#effdf3" stroke="#9fe3b4" stroke-width="2"/>
  <text x="724" y="442" font-size="34" fill="#15803d" font-family="Arial, sans-serif" font-weight="700">B. Activation queue</text>
  <text x="724" y="492" font-size="20" fill="#166534" font-family="Arial, sans-serif">• Implicit queue, not a dedicated state list</text>
  <text x="724" y="536" font-size="20" fill="#166534" font-family="Arial, sans-serif">• Starts when activation_eligibility_epoch is set</text>
  <text x="724" y="580" font-size="20" fill="#166534" font-family="Arial, sans-serif">• Then waits for finalization and churn</text>
  <text x="724" y="624" font-size="20" fill="#166534" font-family="Arial, sans-serif">• Throughput cap: 256 ETH per epoch</text>
  <text x="724" y="668" font-size="20" fill="#166534" font-family="Arial, sans-serif">• Roughly 8 × 32 ETH validators at cap</text>

  <!-- Right side vertical flow -->
  <rect x="1356" y="390" width="250" height="120" rx="18" fill="#ffffff" stroke="#8ee1a9" stroke-width="2"/>
  <text x="1384" y="435" font-size="28" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">5. Wait</text>
  <text x="1384" y="476" font-size="18" fill="#475569" font-family="Arial, sans-serif">Need finalization</text>
  <text x="1384" y="506" font-size="18" fill="#475569" font-family="Arial, sans-serif">and churn room</text>

  <rect x="1356" y="566" width="250" height="120" rx="18" fill="#ffffff" stroke="#8ee1a9" stroke-width="2"/>
  <text x="1384" y="611" font-size="28" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">6. activation_epoch set</text>
  <text x="1384" y="652" font-size="18" fill="#475569" font-family="Arial, sans-serif">Activation scheduled</text>
  <text x="1384" y="682" font-size="18" fill="#475569" font-family="Arial, sans-serif">by protocol rules</text>

  <rect x="1356" y="742" width="250" height="100" rx="18" fill="#ebfff2" stroke="#8ee1a9" stroke-width="2"/>
  <text x="1432" y="802" font-size="34" fill="#15803d" font-family="Arial, sans-serif" font-weight="800">7. ACTIVE</text>

  <!-- Right flow arrows -->
  <path d="M1546 300 L1546 390" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>
  <path d="M1481 510 L1481 566" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>
  <path d="M1481 686 L1481 742" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrow)"/>

  <!-- Timing note -->
  <rect x="694" y="748" width="620" height="94" rx="16" fill="#f8fbff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="724" y="786" font-size="18" fill="#334155" font-family="Arial, sans-serif" font-weight="700">Timing note</text>
  <text x="724" y="818" font-size="18" fill="#334155" font-family="Arial, sans-serif">
    Deposit can be applied first, while activation_eligibility_epoch remains FAR_FUTURE
  </text>
  <text x="724" y="844" font-size="18" fill="#334155" font-family="Arial, sans-serif">
    until the next registry-updates pass.
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
