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

  <rect x="70" y="60" width="260" height="42" rx="14" fill="#e9f2ff" stroke="#9ec5ff" stroke-width="2"/>
  <text x="92" y="88" font-size="22" fill="#1d4ed8" font-family="Arial, sans-serif" font-weight="700">Execution inclusion</text>

  <rect x="380" y="60" width="300" height="42" rx="14" fill="#fff1e7" stroke="#ffc48d" stroke-width="2"/>
  <text x="402" y="88" font-size="22" fill="#c2410c" font-family="Arial, sans-serif" font-weight="700">Explicit pending queue</text>

  <rect x="730" y="60" width="370" height="42" rx="14" fill="#ebfff2" stroke="#9fe3b4" stroke-width="2"/>
  <text x="752" y="88" font-size="22" fill="#15803d" font-family="Arial, sans-serif" font-weight="700">Activation flow</text>

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

---

# Partial Deposit / Top-Up of an Existing Validator

For a **top-up** or **partial deposit** into an **already existing validator**, the queue model is simpler.

## Short answer

A top-up follows the **same deposit path** as a new validator deposit until the point where the deposit is applied:

```text
EL top-up tx included
→ appended to pending_deposits
→ process_pending_deposits()
→ existing validator found by pubkey
→ validator balance increased
```

There is **no separate top-up queue**.

The explicit queue is still:

- **`pending_deposits`**

After the top-up is applied, there is **no new activation queue** if the validator is already active. The balance is simply added to the existing validator state.

---

## Top-up diagram

<div style="overflow-x:auto; margin:20px 0;">
<svg viewBox="0 0 1600 920" width="100%" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Ethereum top-up flow after Pectra">
  <defs>
    <marker id="arrowTopup" markerWidth="10" markerHeight="10" refX="8.5" refY="5" orient="auto">
      <path d="M0,0 L10,5 L0,10 z" fill="#64748b"/>
    </marker>
  </defs>

  <rect x="24" y="24" width="1552" height="872" rx="24" fill="#ffffff" stroke="#dbe4ee" stroke-width="2"/>

  <rect x="76" y="60" width="250" height="42" rx="14" fill="#e9f2ff" stroke="#9ec5ff" stroke-width="2"/>
  <text x="98" y="88" font-size="22" fill="#1d4ed8" font-family="Arial, sans-serif" font-weight="700">Execution inclusion</text>

  <rect x="376" y="60" width="290" height="42" rx="14" fill="#fff1e7" stroke="#ffc48d" stroke-width="2"/>
  <text x="398" y="88" font-size="22" fill="#c2410c" font-family="Arial, sans-serif" font-weight="700">Shared deposit queue</text>

  <rect x="716" y="60" width="780" height="42" rx="14" fill="#ebfff2" stroke="#9fe3b4" stroke-width="2"/>
  <text x="738" y="88" font-size="22" fill="#15803d" font-family="Arial, sans-serif" font-weight="700">Existing validator state update</text>

  <rect x="76" y="150" width="250" height="145" rx="18" fill="#ffffff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="98" y="190" font-size="26" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">1. Top-up tx included</text>
  <text x="98" y="228" font-size="18" fill="#475569" font-family="Arial, sans-serif">Transaction lands in an EL block</text>
  <text x="98" y="256" font-size="18" fill="#475569" font-family="Arial, sans-serif">Request becomes processable by CL</text>
  <text x="98" y="280" font-size="18" fill="#2563eb" font-family="Arial, sans-serif">Not a queue</text>

  <rect x="376" y="150" width="290" height="145" rx="18" fill="#ffffff" stroke="#ffb672" stroke-width="2"/>
  <text x="398" y="190" font-size="26" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">2. pending_deposits</text>
  <text x="398" y="228" font-size="18" fill="#475569" font-family="Arial, sans-serif">Same explicit queue as new deposits</text>
  <text x="398" y="256" font-size="18" fill="#475569" font-family="Arial, sans-serif">No separate top-up queue exists</text>
  <text x="398" y="280" font-size="18" fill="#ea580c" font-family="Arial, sans-serif">Dequeued by epoch processing</text>

  <rect x="716" y="150" width="290" height="145" rx="18" fill="#ffffff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="738" y="190" font-size="26" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">3. Find validator</text>
  <text x="738" y="228" font-size="18" fill="#475569" font-family="Arial, sans-serif">Pubkey already exists in state</text>
  <text x="738" y="256" font-size="18" fill="#475569" font-family="Arial, sans-serif">No new validator is created</text>

  <rect x="1056" y="150" width="250" height="145" rx="18" fill="#ffffff" stroke="#8ee1a9" stroke-width="2"/>
  <text x="1078" y="190" font-size="26" fill="#0f172a" font-family="Arial, sans-serif" font-weight="700">4. Balance increased</text>
  <text x="1078" y="228" font-size="18" fill="#475569" font-family="Arial, sans-serif">increase_balance(...)</text>
  <text x="1078" y="256" font-size="18" fill="#475569" font-family="Arial, sans-serif">Existing validator gets the ETH</text>

  <rect x="1356" y="150" width="170" height="145" rx="18" fill="#ebfff2" stroke="#8ee1a9" stroke-width="2"/>
  <text x="1388" y="205" font-size="24" fill="#15803d" font-family="Arial, sans-serif" font-weight="800">5. Done</text>
  <text x="1378" y="240" font-size="18" fill="#166534" font-family="Arial, sans-serif">No new activation</text>
  <text x="1382" y="266" font-size="18" fill="#166534" font-family="Arial, sans-serif">queue if already</text>
  <text x="1404" y="290" font-size="18" fill="#166534" font-family="Arial, sans-serif">active</text>

  <path d="M326 224 L376 224" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrowTopup)"/>
  <path d="M666 224 L716 224" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrowTopup)"/>
  <path d="M1006 224 L1056 224" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrowTopup)"/>
  <path d="M1306 224 L1356 224" stroke="#7b8ca3" stroke-width="4" fill="none" marker-end="url(#arrowTopup)"/>

  <rect x="76" y="370" width="500" height="230" rx="20" fill="#fff5ee" stroke="#ffc48d" stroke-width="2"/>
  <text x="106" y="420" font-size="34" fill="#b45309" font-family="Arial, sans-serif" font-weight="700">Queue used by top-ups</text>
  <text x="106" y="472" font-size="22" fill="#9a3412" font-family="Arial, sans-serif">• Explicit queue: state.pending_deposits</text>
  <text x="106" y="520" font-size="22" fill="#9a3412" font-family="Arial, sans-serif">• Storage limit: 134,217,728</text>
  <text x="106" y="568" font-size="22" fill="#9a3412" font-family="Arial, sans-serif">• Processing cap: 16 per epoch</text>

  <rect x="616" y="370" width="430" height="230" rx="20" fill="#f8fbff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="646" y="420" font-size="34" fill="#334155" font-family="Arial, sans-serif" font-weight="700">What changes in state</text>
  <text x="646" y="472" font-size="22" fill="#334155" font-family="Arial, sans-serif">• Existing validator is matched by pubkey</text>
  <text x="646" y="520" font-size="22" fill="#334155" font-family="Arial, sans-serif">• Balance is increased</text>
  <text x="646" y="568" font-size="22" fill="#334155" font-family="Arial, sans-serif">• No new validator record is created</text>

  <rect x="1086" y="370" width="440" height="230" rx="20" fill="#effdf3" stroke="#9fe3b4" stroke-width="2"/>
  <text x="1116" y="420" font-size="34" fill="#15803d" font-family="Arial, sans-serif" font-weight="700">Activation effect</text>
  <text x="1116" y="472" font-size="22" fill="#166534" font-family="Arial, sans-serif">• Active validator: no new activation queue</text>
  <text x="1116" y="520" font-size="22" fill="#166534" font-family="Arial, sans-serif">• Inactive validator: top-up can help reach</text>
  <text x="1116" y="548" font-size="22" fill="#166534" font-family="Arial, sans-serif">  the activation threshold</text>
  <text x="1116" y="596" font-size="22" fill="#166534" font-family="Arial, sans-serif">• Deposit processing still shares churn limits</text>

  <rect x="76" y="660" width="1450" height="170" rx="20" fill="#f8fbff" stroke="#c8d6e5" stroke-width="2"/>
  <text x="106" y="708" font-size="28" fill="#334155" font-family="Arial, sans-serif" font-weight="700">Operational note</text>
  <text x="106" y="752" font-size="22" fill="#334155" font-family="Arial, sans-serif">Top-ups become much more useful for compounding 0x02 validators, because their effective balance cap can go above 32 ETH</text>
  <text x="106" y="788" font-size="22" fill="#334155" font-family="Arial, sans-serif">up to 2048 ETH. For legacy 0x01 validators, excess above the cap does not make them heavier than a normal 32 ETH validator.</text>
</svg>
</div>

---

## What “top-ups are still subject to the activation period” means

This wording can be confusing.

It does **not** mean an already active validator has to activate again.

What it means is:

- the **newly deposited ETH** does not become usable immediately
- the top-up still enters **`pending_deposits`**
- it is then processed under the same **deposit-processing and activation-side churn limits** used after Pectra / Electra
- only when that pending deposit is applied does the existing validator balance increase

So the delay applies to the **new ETH amount**, not to the validator restarting its lifecycle.

### Plain-English interpretation

For an already active validator:

```text
validator stays active
→ top-up tx is included
→ top-up waits in pending_deposits
→ top-up is processed
→ validator balance increases
```

There is **no second validator activation ceremony**.

### Why people call it the “activation period”

After Pectra / Electra, the protocol meters onboarding and deposit application by **ETH amount**, not just by validator count.

That is why docs say:

- the queue “churns by ETH amount instead of validator count”
- top-ups are still subject to the activation period

Operationally, that means the top-up can still wait because of:

- the `pending_deposits` queue
- the per-epoch deposit processing cap
- the activation / exit churn budget

### Clean distinction

**Already active validator**
- stays active the whole time
- top-up just adds balance once processed

**Not-yet-active validator**
- top-up can help it reach the minimum activation balance
- after that, the validator still follows the normal activation path

> **Note**
> A top-up is delayed as **new incoming stake**, not as a validator reactivation.  
> The validator does not “start over”; only the newly added ETH waits to be processed.

> **Tip**
> For operator UX, think of a top-up as **queued stake**, not a **queued validator**.  
> That mental model avoids most confusion around “activation period” wording.

---

## Top-up state transitions

### 1) Execution inclusion
The top-up transaction is included in an execution block.

- This is not a queue.
- Once the block is processed by the consensus client, the request is appended to `state.pending_deposits`.

### 2) `pending_deposits`
Top-ups use the same explicit queue as new validator deposits.

- There is no separate top-up queue.
- The same queue storage and dequeue limits apply.

### 3) Existing validator lookup
When the pending deposit is processed, the consensus state checks whether the pubkey already exists.

- If it exists, the validator is updated.
- If it does not exist, a new validator can be created instead.

### 4) Balance increase
For an existing validator, the top-up is applied as a balance increase.

```text
increase_balance(state, validator_index, amount)
```

### 5) Activation consequences
For an **already active** validator:

- the top-up does **not** create a new activation queue entry
- it simply raises the validator balance

For a validator that is **not yet active**:

- a top-up can help it reach the minimum activation balance
- after that, the normal activation path applies

---

## Top-up limits that still apply

The same deposit-side limits apply to top-ups:

- `MAX_DEPOSIT_REQUESTS_PER_PAYLOAD = 8192`
- `PENDING_DEPOSITS_LIMIT = 134,217,728`
- `MAX_PENDING_DEPOSITS_PER_EPOCH = 16`

Top-ups are also subject to the same balance-based churn accounting used by Electra deposit processing.

---

## Operational note: 0x01 vs 0x02

Top-ups matter much more for **compounding (`0x02`) validators** than for normal `0x01` validators.

- `0x01` validators keep the normal effective-balance cap around **32 ETH**
- `0x02` compounding validators can grow their effective balance up to **2048 ETH**
- after Pectra, new deposits / top-ups must be at least **1 ETH**

So the queue is the same, but the **effect** of the top-up is different depending on validator type.
