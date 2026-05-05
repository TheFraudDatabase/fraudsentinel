# Autonomyx Fraud Sentinel — Pattern Registry

> This file is the authoritative, append-only list of fraud patterns.
> Never delete or renumber a pattern. Add new ones at the bottom.
> Each pattern must have: ID, name, description, triggers, risk level, indicators, notes.

---

## PATTERN-001 · Scarcity Manipulation

**Description:** Artificial urgency created through scarcity claims to pressure a target into
a hasty decision — financial, credential-sharing, or behavioural.

**Triggers:**
- First scarcity message from an account with no prior scarcity history
- Spike: 3+ accounts posting similar scarcity language within a 1-hour window
- Scarcity claim cannot be verified against any real inventory or deadline

**Risk Level:** Medium (isolated) → High (coordinated)

**Indicators:**
- Language: "only X left", "expires soon", "limited offer", "act now", "last chance"
- Account age > 30 days with zero prior scarcity messaging
- Identical or near-identical text across multiple accounts
- Scarcity paired with a payment or credential request

**Notes:** Often precedes phishing or payment fraud. Escalate if coordinated.

---

## PATTERN-002 · Dormant Identity Resurrection

**Description:** A previously inactive account, agent, or identity suddenly reactivates —
often with changed profile metadata and high-stakes first actions.

**Triggers:**
- Identity inactive for ≥ 14 days, then becomes active
- First post-resurrection action is financial, credential-seeking, or urgency-driven
- Profile metadata (name, avatar, bio, contact) changed within 24h of reactivation

**Risk Level:** High

**Indicators:**
- Last-active gap ≥ 14 days
- New device fingerprint or IP on reactivation
- Profile changes within 24h of reactivation
- First message requests money, login, OTP, or personal data
- Account was previously flagged (any status)

**Notes:** Common in account takeover and impersonation attacks. Cross-reference with
PATTERN-003 (Impersonation) when profile changes are detected.

---

## PATTERN-003 · Velocity Spike

**Description:** An identity initiates an unusually high number of transactions, messages,
or actions in a short time window — far outside their established baseline behaviour.

**Triggers:**
- 3+ payment attempts within 10 minutes from the same identity
- Message volume 5x above the identity's 7-day average within 1 hour
- Rapid sequential actions targeting multiple different recipients

**Risk Level:** Medium (isolated) → Critical (cross-account)

**Indicators:**
- Transaction or action count exceeds 3x the identity's rolling 7-day average
- Multiple failed attempts followed by a successful one (probing)
- Same amount sent to multiple different recipients in quick succession
- Actions span multiple platforms simultaneously

**Notes:** Often indicates automated fraud tooling or account takeover. Combine with
PATTERN-004 (new device) for higher confidence. Escalate immediately if cross-account.

---

## PATTERN-004 · New Device or Location on High-Stakes Action

**Description:** An identity accesses a platform from a new device fingerprint, IP address,
or geographic location — and immediately performs a high-stakes action (payment, credential
change, data export) without any warm-up activity.

**Triggers:**
- New device/IP detected AND first action is financial or credential-related
- Geographic location change of >500km since last session
- Device fingerprint not seen in the identity's last 30 days of activity

**Risk Level:** High

**Indicators:**
- New device fingerprint on first high-stakes action
- IP geolocation mismatch vs. identity's established location pattern
- No warm-up browsing — goes straight to payment or settings
- Login time outside identity's normal active hours
- VPN or Tor exit node detected

**Notes:** Standalone new device is Medium risk. New device + immediate high-stakes action
= High. Cross-reference with PATTERN-002 (dormant resurrection) — both often occur together
in account takeover scenarios.

---

## PATTERN-005 · Known Bad Actor Network (Graph Linkage)

**Description:** An identity has a direct or indirect connection to a previously confirmed
fraud actor — through shared payment destinations, referral chains, device fingerprints,
or communication patterns.

**Triggers:**
- Identity sends funds to or receives funds from a flagged identity
- Identity shares a device fingerprint with a flagged identity
- Identity was referred by or refers others to a flagged identity
- 2-hop graph connection to a confirmed fraud event

**Risk Level:** Medium (2-hop) → High (direct connection) → Critical (confirmed mule)

**Indicators:**
- Shared UPI ID, bank account, or wallet address with a flagged actor
- Shared device ID or cookie fingerprint with a flagged actor
- Referral chain traces back to a known fraud campaign
- Funds pass through identity without retention (immediate re-transfer) — mule pattern

**Notes:** Requires graph traversal across fraud_event records. Query SurrealDB for
shared identity_id, device_id, or payment_destination fields. Even if the identity
itself has no direct fraud history, network linkage is high-signal.

---

## PATTERN-006 · Time-of-Day Anomaly

**Description:** An identity performs high-stakes actions at times significantly outside
their established active hours — a strong signal of account takeover or automated fraud.

**Triggers:**
- High-stakes action (payment, credential change) between midnight and 5am local time
  for an identity that has never been active in that window
- Action timestamp falls outside the identity's 30-day active hour distribution by >3 sigma
- Sudden shift in active timezone without prior travel signal

**Risk Level:** Medium (time anomaly alone) → High (combined with new device or dormancy)

**Indicators:**
- Action time outside identity's established active hours (derived from 30-day history)
- No prior activity in the same time window across 30+ days of history
- Timezone of action inconsistent with identity's registered location
- Rapid session — login, action, logout in under 60 seconds at unusual hour

**Notes:** Standalone time anomaly = Medium. Combine with PATTERN-004 (new device) or
PATTERN-002 (dormant) to escalate to High/Critical. Banks like HDFC use this as a
primary signal — Fraud Sentinel should too.

---

## PATTERN-007 · Low-Value Probe Transaction

**Description:** A fraudster or compromised account sends a very small amount (₹1–₹500 or
equivalent) to a target or through a mule — deliberately staying below bank alert thresholds
to test liveness, build transaction history, or establish trust before a large-value attack.

**Triggers:**
- Transaction of ≤ ₹500 (or currency equivalent) to a new, unverified recipient
- Series of small transactions to the same recipient building up over days
- Small transaction to a recipient who has never interacted with the identity before
- Small amount sent immediately after account reactivation (PATTERN-002)

**Risk Level:** Low (single probe) → High (series building toward large transaction)

**Insight:** Most bank fraud systems (including HDFC) set amount-based thresholds —
e.g. warn at ₹5,000+. This creates a blind spot. **Fraud Sentinel warns based on
recipient identity risk, not transaction amount.** A ₹1 transfer to a suspicious
identity is as dangerous as a ₹1,00,000 transfer.

**Indicators:**
- Transaction amount ≤ ₹500 to a first-time recipient
- Recipient identity has fraud flags or network linkage (PATTERN-005)
- Pattern of escalating small amounts to same recipient over 3–7 days
- Small transaction immediately followed by a request for confirmation ("did you get it?")
  — social engineering to establish trust

**Notes:** This is the most under-detected pattern in traditional banking fraud systems.
Fraud Sentinel should flag ANY transaction — regardless of amount — when the recipient
identity carries risk signals. Amount is irrelevant. Identity risk is everything.

---

<!-- NEW PATTERNS APPENDED BELOW THIS LINE -->

---

## PATTERN-008 · Social Engineering Deepfake

**Description:** An identity uses AI-generated voice, video, or text to impersonate a trusted figure — CEO, family member, authority — to manipulate the target into a financial action.

**Triggers:**
- First communication from an "authority" figure via new channel
- Voice/video call requested followed immediately by payment/credential request
- Text pattern matches known deepfake prompts ("can you hear me", emergencyscenario language)
- Recipient has no prior direct communication history with the "sender"

**Risk Level:** High → Critical (when combined with urgency)

**Indicators:**
- Voice quality artifacts (breathing patterns, audio glitches)
- Lip-sync mismatch in video calls
- Urgency + authority impersonation combined
- Request comes after social media reconnaissance of target
- Known deepfake tool signatures in metadata

**Notes:** Requires audio/video analysis integration. Partner with anti-deepfake services (e.g., Reality Defender, Intel's FakeCatcher). Cross-reference with PATTERN-001 (social engineering scarcity).

---

## PATTERN-009 · Cryptocurrency Mixer/Tumbler Laundering

**Description:** Cryptocurrency transactions routed through mixing or tumbling services to obscure the source of funds — a primary money laundering method for fraud proceeds.

**Triggers:**
- Transaction to/from a known mixer/tumbler address (e.g., Tornado Cash, Sinbad.io)
- Multiple small transactions consolidating into a single large withdrawal (peeling chain)
- Funds flowing through 3+ hops within 30 minutes
- Wallet with no prior crypto history suddenly receives and immediately forwards funds

**Risk Level:** High → Critical

**Indicators:**
- Destination address flagged in blockchain analytics (Chainalysis, Elliptic)
- Sudden activity from dormant wallet
- Transaction pattern matches known mixer "peeling" behavior
- Gas fee patterns indicating automated laundering
- Cross-border movement immediately after receipt

**Notes:** Integrate with blockchain analytics providers. Requires wallet tagging database. Critical for exchanges and DeFi platforms.

---

## PATTERN-010 · SIM Swap / Number Porting Fraud

**Description:** A fraudster transfers a victim's phone number to a SIM card under their control — enabling OTP bypass, account takeover, and financial fraud.

**Triggers:**
- SIM swap detected via carrier notification API (supported carriers)
- Number porting request to a new carrier within 24 hours of suspicious activity
- SMS OTP requested immediately after number migration
- Multiple accounts suddenly associated with the same device fingerprint post-swap

**Risk Level:** Critical

**Indicators:**
- Carrier signals SIM swap event
- Location change >500km within 5 minutes (impossible travel)
- New device + new number combination
- SMS delivery failure pattern before swap
- Banking app tokenrefresh attempted post-swap

**Notes:** Requires carrier API integrations. Some carriers offer dedicated fraud notification endpoints. Cross-reference with PATTERN-004 (new device) for account takeover detection.

---

## PATTERN-011 · Synthetic Identity Fraud

**Description:** A fraudster builds a fake identity over time using partial real data (SSN fragments, fake addresses) — eventually establishingcredit enough for large loans or account takeovers.

**Triggers:**
- Identity uses different SSN fragments across multiple accounts
- Address changes frequently without physical verification
- Account age suddenly becomes "established" (180+ days) after 30 days
- Credit utilization pattern indicates "sleeping giant" — sudden max utilization
- Multiple identity fragments linked via device/IP fingerprint clustering

**Risk Level:** High → Critical (when credit is established)

**Indicators:**
- SSN validation shows mismatch on file with address
- Multiple applications across platforms with slight variations in PII
- Device fingerprint + IP cluster across seemingly unconnected accounts
- First major credit usage after prolonged dormancy
- Velocity: 3+ financial applications within 24 hours

**Notes:** This is one of the hardest fraud types to detect. Synthetic identities are built to pass traditional KYC. Cross-reference with PATTERN-002 (dormant resurrection) and PATTERN-005 (network linkage).

---

## PATTERN-012 · Merchant Fraud / Fake Ecommerce

**Description:** A fake online store collects payments for goods that are never delivered — or delivers counterfeit/far inferior items.

**Triggers:**
- New merchant registered with no prior web presence
- Pricing significantly below market (30%+ discount)
- Domainregistration < 90 days old
- No legitimate contact information (email is generic or disposable)
- Multiple chargebacks within 30 days of merchant onboarding
- Return address matches other flagged merchants

**Risk Level:** High → Critical (when scale increases)

**Indicators:**
- Domain age < 90 days via WHOIS
- No social media presence or recent account creation
- Pricing 30%+ below Amazon/Flipkart benchmarks
- Multiple similar complaints in consumer forums
- Same IP/subnet as previously flagged merchants
- Payment to new merchant, then immediate bulk purchases

**Notes:** Integrate with consumer complaint APIs (ScamDB, Trustpilot scraping). Cross-reference with PATTERN-005 (network linkage) for merchant rings.

---

## PATTERN-013 · Investment Ponzi / Pyramid Scheme

**Description:** An investment opportunity promises unrealistically high returns — funded by new investors rather than actual profits.

**Triggers:**
- Referral structure with >2 levels (MLM/pyramid indicator)
- Returns promised exceed 15% monthly (impossible in legitimate markets)
- Emphasis on recruitment over product/service
- No verifiable physical business premises
- Target is specifically offered to previously defrauded victims (re-victimization)

**Risk Level:** Critical

**Indicators:**
- Referral commission >10% at Level 2+
- "Guaranteed returns" language in promotions
- Testimonials only (no verifiable third-party audits)
- Domain registered < 6 months ago
- Target demographic includes elderly or financially vulnerable
- Multiple small investors aggregated into single large wallet

**Notes:** Cross-reference with PATTERN-001 (scarcity - "limited time"). Integrate with SEBI/SEC alert feeds. Escalate to authorities (MED) for Criticalrisk events.

---

## PATTERN-014 · Romance Fraud / Long-Con Scam

**Description:** A fraudster builds a romantic relationship over weeks/months — then requests money for emergencies, travel, or medical expenses.

**Triggers:**
- Relationship established entirely online (dating app, social media)
- First financial request after 30+ days of consistent communication
- Uses excuses to avoid video calls (always "camera broken", "work issues")
- Requests money via gift cards, cryptocurrency, or wire transfer to atypical destinations
- Profile shows inconsistent details over time

**Risk Level:** High → Critical (can be extremely high loss)

**Indicators:**
- Never video called in 30+ days of communication
- Financial requests escalate (>3 requests)
- Requests via gift cards (Apple, Google Play) or crypto
- Claims to be overseas (military, business, "working on rig")
- Has existing "victim list" markers (other fraud events under same identity)
- Uses emergency scenarios (hospital, visa, legal)

**Notes:** One of the highest-average-loss fraud types. Target demographic often elderly or lonely. Cross-reference with PATTERN-002 (dormant identity) if account was previously inactive. Cross-reference with PATTERN-005 (network linkage) for coordinated rings.

---

## PATTERN-015 · Invoice Manipulation / Billing Fraud

**Description:** A fraudster intercepts or alters legitimate invoices — changing payment details to redirect funds to controlled accounts.

**Triggers:**
- Banking detail change on existing vendor invoice
- Invoice from known vendor but with new payment routing
- Email thread shows subtle domain change (@company.com vs @cornpany.com)
- Invoice amount variance >10% from historical without explanation
- Multiple invoices for same amount submitted simultaneously

**Risk Level:** High → Critical (B2B)

**Indicators:**
- Vendor bank account change via email (not portal)
- Invoice from vendor with no recent communication
- Amount matches historical but routing is new
- Email header shows redirection (BCC to fraudster)
- New vendor with existing employee as "reference"

**Notes:** Primarily B2B. Cross-reference with PATTERN-005 (network linkage) for organized billing rings. Partner with accounting systems for verified bank changes.

---

## PATTERN-016 · Account Takeover (ATO) Recovery Pattern

**Description:** After an account takeover is detected, fraudsters execute a predictable recovery flow — attempting to regain access through social engineering or technical manipulation.

**Triggers:**
- Failed login >5 followed by password reset request within 10 minutes
- New device added within 1 hour of account takeover detection
- Recovery email/phone changed immediately after takeover
- "Customer service" called from compromised number within 24 hours post-takeover

**Risk Level:** High

**Indicators:**
- 5+ failed logins → password reset pattern
- New device fingerprint added post-takeover
- Contact info changed post-takeover
- Call from newly-linked phone number
- Attempted to remove account alerts/2FA

**Notes:** Post-incident detection. Requires integration with account security events. Cross-reference with PATTERN-004 (new device) and PATTERN-002 (dormant resurrection).

---

## PATTERN-017 · Family/Friend Impersonation (Reactivation)

**Description:** A known contact's identity is reactivated by a fraudster after the original account was dormant — now requesting emergency funds.

**Triggers:**
- Contact previously dormant >30 days becomes active
- First message is financial request or urgent scenario
- Communication pattern differs from historical (>30% text deviation via NLP)
- Urgency language not present in prior communications

**Risk Level:** High → Critical (when combined with financial emergency scenario)

**Indicators:**
- 30+ day dormancy gap
- Style mismatch in communication (NLP analysis)
- Urgency not present in prior history
- New payment details differ from historical
- Profile metadata changed (new photo, bio) within 24h

**Notes:** Advanced version of PATTERN-002. Use NLP/ML for communication style analysis. Cross-reference with digital footprint from PATTERN-005.
