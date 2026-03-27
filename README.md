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

<div style="overflow-x:auto; background:#0f172a; border-radius:16px; padding:20px; border:1px solid #334155; margin:20px 0;">
<svg viewBox="0 0 1480 760" width="100%" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Ethereum staking after Pectra flow">
  <defs>
    <marker id="arrow" markerWidth="12" markerHeight="12" refX="10" refY="6" orient="auto">
      <path d="M0,0 L12,6 L0,12 z" fill="#93c5fd"/>
    </marker>
  </defs>

  <rect x="24" y="20" rx="14" ry="14" width="300" height="42" fill="#172554" stroke="#60a5fa"/>
  <text x="42" y="47" font-size="22" fill="#dbeafe" font-family="Arial, sans-serif" font-weight="700">Execution inclusion</text>

  <rect x="370" y="20" rx="14" ry="14" width="330" height="42" fill="#3f2a13" stroke="#f59e0b"/>
  <text x="388" y="47" font-size="22" fill="#ffedd5" font-family="Arial, sans-serif" font-weight="700">Explicit pending queue</text>

  <rect x="760" y="20" rx="14" ry="14" width="690" height="42" fill="#163322" stroke="#4ade80"/>
  <text x="778" y="47" font-size="22" fill="#dcfce7" font-family="Arial, sans-serif" font-weight="700">Activation flow</text>

  <rect x="24" y="100" rx="18" ry="18" width="245" height="128" fill="#1e293b" stroke="#60a5fa"/>
  <text x="44" y="136" font-size="26" fill="#eff6ff" font-family="Arial, sans-serif" font-weight="700">1. Deposit tx included</text>
  <text x="44" y="168" font-size="18" fill="#cbd5e1" font-family="Arial, sans-serif">Tx is in a valid EL block</text>
  <text x="44" y="196" font-size="18" fill="#cbd5e1" font-family="Arial, sans-serif">If the block finalizes, tx finalizes</text>
  <text x="44" y="220" font-size="17" fill="#93c5fd" font-family="Arial, sans-serif">Not a queue</text>

  <rect x="332" y="100" rx="18" ry="18" width="260" height="128" fill="#33210f" stroke="#f59e0b"/>
  <text x="352" y="136" font-size="26" fill="#fff7ed" font-family="Arial, sans-serif" font-weight="700">2. pending_deposits</text>
  <text x="352" y="168" font-size="18" fill="#fed7aa" font-family="Arial, sans-serif">Explicit queue in beacon state</text>
  <text x="352" y="196" font-size="18" fill="#fed7aa" font-family="Arial, sans-serif">Deposit appended by CL processing</text>
  <text x="352" y="220" font-size="17" fill="#fdba74" font-family="Arial, sans-serif">First real waiting stage</text>

  <rect x="654" y="100" rx="18" ry="18" width="280" height="128" fill="#1e293b" stroke="#60a5fa"/>
  <text x="674" y="136" font-size="26" fill="#eff6ff" font-family="Arial, sans-serif" font-weight="700">3. Deposit applied</text>
  <text x="674" y="168" font-size="18" fill="#cbd5e1" font-family="Arial, sans-serif">Validator created or balance updated</text>
  <text x="674" y="196" font-size="18" fill="#cbd5e1" font-family="Arial, sans-serif">Balance may already be visible</text>
  <text x="674" y="220" font-size="17" fill="#93c5fd" font-family="Arial, sans-serif">Still not active</text>

  <rect x="996" y="100" rx="18" ry="18" width="280" height="128" fill="#163322" stroke="#4ade80"/>
  <text x="1016" y="136" font-size="26" fill="#f0fdf4" font-family="Arial, sans-serif" font-weight="700">4. Enter activation queue</text>
  <text x="1016" y="168" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">activation_eligibility_epoch set</text>
  <text x="1016" y="196" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">Implicit queue begins here</text>
  <text x="1016" y="220" font-size="17" fill="#86efac" font-family="Arial, sans-serif">Not a separate list</text>

  <rect x="1298" y="100" rx="18" ry="18" width="158" height="128" fill="#163322" stroke="#4ade80"/>
  <text x="1318" y="136" font-size="26" fill="#f0fdf4" font-family="Arial, sans-serif" font-weight="700">5. Wait</text>
  <text x="1318" y="168" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">Need finalization</text>
  <text x="1318" y="196" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">and churn room</text>

  <rect x="1140" y="324" rx="18" ry="18" width="280" height="118" fill="#163322" stroke="#4ade80"/>
  <text x="1160" y="360" font-size="26" fill="#f0fdf4" font-family="Arial, sans-serif" font-weight="700">6. activation_epoch set</text>
  <text x="1160" y="392" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">Activation scheduled ahead</text>
  <text x="1160" y="418" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">by protocol rules</text>

  <rect x="1140" y="530" rx="18" ry="18" width="280" height="108" fill="#14532d" stroke="#86efac"/>
  <text x="1242" y="588" font-size="34" fill="#f0fdf4" font-family="Arial, sans-serif" font-weight="800">7. ACTIVE</text>

  <path d="M269 164 L332 164" stroke="#93c5fd" stroke-width="5" fill="none" marker-end="url(#arrow)"/>
  <path d="M592 164 L654 164" stroke="#93c5fd" stroke-width="5" fill="none" marker-end="url(#arrow)"/>
  <path d="M934 164 L996 164" stroke="#93c5fd" stroke-width="5" fill="none" marker-end="url(#arrow)"/>
  <path d="M1276 164 L1298 164" stroke="#93c5fd" stroke-width="5" fill="none" marker-end="url(#arrow)"/>
  <path d="M1377 228 L1377 324" stroke="#93c5fd" stroke-width="5" fill="none" marker-end="url(#arrow)"/>
  <path d="M1280 442 L1280 530" stroke="#93c5fd" stroke-width="5" fill="none" marker-end="url(#arrow)"/>

  <rect x="24" y="290" rx="18" ry="18" width="1040" height="390" fill="#0b1220" stroke="#334155"/>
  <text x="44" y="330" font-size="28" fill="#e2e8f0" font-family="Arial, sans-serif" font-weight="700">Queue definitions and limits</text>

  <rect x="44" y="360" rx="14" ry="14" width="470" height="290" fill="#33210f" stroke="#f59e0b"/>
  <text x="64" y="396" font-size="24" fill="#fff7ed" font-family="Arial, sans-serif" font-weight="700">A. pending_deposits</text>
  <text x="64" y="432" font-size="18" fill="#fed7aa" font-family="Arial, sans-serif">• Explicit queue: state.pending_deposits</text>
  <text x="64" y="464" font-size="18" fill="#fed7aa" font-family="Arial, sans-serif">• Storage limit: 134,217,728</text>
  <text x="64" y="496" font-size="18" fill="#fed7aa" font-family="Arial, sans-serif">• Processing cap: 16 per epoch</text>
  <text x="64" y="528" font-size="18" fill="#fed7aa" font-family="Arial, sans-serif">• Also bounded by activation/exit churn</text>
  <text x="64" y="560" font-size="18" fill="#fed7aa" font-family="Arial, sans-serif">• Deposit may wait here even after EL finality</text>

  <rect x="540" y="360" rx="14" ry="14" width="500" height="290" fill="#163322" stroke="#4ade80"/>
  <text x="560" y="396" font-size="24" fill="#f0fdf4" font-family="Arial, sans-serif" font-weight="700">B. Activation queue</text>
  <text x="560" y="432" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">• Implicit queue, not a dedicated state list</text>
  <text x="560" y="464" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">• Starts when activation_eligibility_epoch is set</text>
  <text x="560" y="496" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">• Then waits for finalization and churn</text>
  <text x="560" y="528" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">• Throughput cap: 256 ETH per epoch</text>
  <text x="560" y="560" font-size="18" fill="#bbf7d0" font-family="Arial, sans-serif">• For 32 ETH validators, roughly 8 at cap</text>

  <rect x="1092" y="474" rx="14" ry="14" width="340" height="176" fill="#172554" stroke="#60a5fa"/>
  <text x="1112" y="510" font-size="22" fill="#dbeafe" font-family="Arial, sans-serif" font-weight="700">Important timing detail</text>
  <text x="1112" y="544" font-size="18" fill="#bfdbfe" font-family="Arial, sans-serif">Deposit can be applied first,</text>
  <text x="1112" y="572" font-size="18" fill="#bfdbfe" font-family="Arial, sans-serif">while activation_eligibility_epoch</text>
  <text x="1112" y="600" font-size="18" fill="#bfdbfe" font-family="Arial, sans-serif">is still FAR_FUTURE until the next</text>
  <text x="1112" y="628" font-size="18" fill="#bfdbfe" font-family="Arial, sans-serif">registry-updates pass.</text>
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
