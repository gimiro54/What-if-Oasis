# What-if-Oasis
What if we actually do it? Why not? 


## 0. Prime directive: flexibility, forever

The one goal that outranks all others: **the base must absorb the future at the lowest possible cost.**

Test for every decision: *"In 15 years, Unreal Engine 7 ships, or a new headset, or a new kind of game appears. How many lines of the base must change?"* The only acceptable answer is **zero**. Adding something new means writing an *adapter* or *plugin*, never editing the base.

### Flexibility laws (apply to every module, forever)

| # | Law | What it means in practice |
|---|---|---|
| F1 | **Adapters, not integrations** | The base never "supports Unreal". It defines an Engine Adapter contract. Unreal 7 = one new adapter that passes the conformance suite. |
| F2 | **Contracts over code** | Modules talk through versioned, language-neutral schemas. Any module can be rewritten in any language if it passes the tests. |
| F3 | **Everything is versioned** | Every message, file, package and API carries a version. Old and new coexist through negotiated versions. |
| F4 | **Ignore the unknown, never reject it** | Readers skip unrecognized fields and extensions (forward compatibility). Unknown *required* extensions fail gracefully with a clear message. |
| F5 | **Negotiate capabilities, never assume them** | Clients and planets declare what they can do; the system picks the best common ground. No hardcoded device or engine checks. |
| F6 | **Namespaced extension points** | Every contract has an extension registry (`vendor.feature.v1`). New features register; they do not fork. |
| F7 | **Fallbacks are mandatory** | Anything rich must carry a simpler representation, down to plain text. This gives hardware reach and accessibility from the same mechanism. |
| F8 | **Policy is data, not code** | Speeds, limits, fees, thresholds and rules are parameters governed by votes, not constants buried in code. |
| F9 | **Deprecate, never delete** | Old versions get long, public sunset windows. Data from year 1 must still open in year 20. |
| F10 | **No privileged parties** | No engine, device, vendor, language or server is special. The official reference implementation is just one implementation. |
| F11 | **Small waist** | The fewer things the base defines, the fewer things can become obsolete. Resist adding to the base. |
| F12 | **Design for replacement** | Every module has a documented "how to replace me" section. |

### Immutable core **[INVARIANT]**

Only a handful of things are frozen. All are about *safety of people and money*, never about features:

1. Users own their keys; nobody can seize an identity or balance.
2. Ledger arithmetic: no double-spend, no minting outside the issuance rules, full verifiable history.
3. The protocol and reference code stay open source and forkable.
4. No mandatory central operator, vendor or server.
5. Nothing installs or executes without explicit user permission and sandboxing.
6. Users can export all their data and leave.
7. The invariants can only change through the slowest, most demanding process in the system (section 12).

---

## 1. The layer model

```
L6  COMMUNITY        votes, proposals, bounties, working groups, forks
L5  PLANETS          games / worlds (any engine) + their items & rules
L4  CARRY-ALONG      avatar, outfit, emotes, Pocket skins, universal items
L3  CLIENTS          Pocket app, lobby, renderers, device shells
L2  SERVICES         relays, storage, indexers, hosts, bridges (all optional, all replaceable)
L1  CONTRACTS        C1..C12  <-- the thin waist: versioned specs + conformance tests
L0  CONSTITUTION     invariants + governance tiers
```

Rules:
- Each layer depends only on the layers beneath it.
- L1 is the product. L0 is the promise. Everything else can be replaced.
- L2 services are *conveniences*: any user can run their own, and clients work with any set of them.

---

## 2. The twelve contracts (L1)

Each contract = a spec, a schema, a conformance test suite, a reference implementation and a "how to replace me" note.

| ID | Contract | Purpose |
|---|---|---|
| C1 | Identity | Who you are, keys, names, recovery |
| C2 | Ledger | Currency, balances, items, trades |
| C3 | Package | How anything is distributed and installed |
| C4 | Simulation & Scene | Shared truth of a world; how it's described |
| C5 | Governance | Proposals, votes, reputation |
| C6 | Network | Discovery, transport, hosting, replication |
| C7 | Device & Input | Abstracting hardware, inputs and outputs |
| C8 | Engine Adapter | How any engine plugs into C4 |
| C9 | Social & Discover | Feed, messages, showcase, reviews |
| C10 | Permissions & Safety | Sandbox, capabilities, filtering, reporting |
| C11 | Storage & Persistence | Where world state and assets live |
| C12 | Evolution | Versioning, compatibility, upgrades, deprecation |

### C1 — Identity
- Identity = a public key. The app holds the private key; the user can hold several *personas* (different avatars/names) under one master identity.
- **Key hierarchy**: master key (rarely used, offline-capable) → device keys (one per phone/PC, revocable) → session keys (short-lived, used in-world). A stolen device is revoked without losing the account.
- **Recovery**: social recovery (N-of-M trusted contacts), hardware backup, encrypted backup. Mandatory in the spec, because lost keys are the top cause of lost accounts.
- **Names**: human-readable names resolve to keys through a pluggable naming layer (several resolvers allowed). Names are optional; keys are the truth.
- **Signature agility**: the signature algorithm is a field in every signed object, so we can migrate to new cryptography (including post-quantum) without breaking old data. This is F3/F9 applied to crypto.
- **Linked proofs** (optional, user-controlled): attach claims (e.g., proof of personhood, age bracket) from any provider. The base never requires them, but voting weight and features may use them.

### C2 — Ledger
- **Accounts** are identities. **Assets** are either *fungible* (OSC) or *non-fungible/semi-fungible* (items).
- **Transaction types**: transfer, swap, escrow-open, escrow-release, escrow-refund, mint (issuer only), burn, grant (treasury), extension-defined types.
- **Asset issuers**: each planet (or any identity) can register as an issuer for item classes. The ledger records ownership; the issuer's planet honors it. Issuers are rate-limited and carry reputation.
- **Atomic trades**: any cross-asset trade settles as an atomic swap or via escrow, so the buyer never pays without receiving and vice versa. Cross-planet delivery uses the same mechanism: ledger ownership changes only when the planet confirms delivery.
- **Single source of truth for tradable value**: if an item can be traded, the ledger is its authoritative record. Planet-local copies are views. Non-tradable things stay local and stay out of the ledger.
- **Dispute window**: escrows can carry a time-boxed dispute process (community arbiters, pluggable).
- **Issuance policy is data (F8)**: how OSC is created, who receives it, caps, halving or none. This is deliberately undecided; the *accounting* is frozen, the *policy* is a governed parameter.
- **Fees**: a small protocol fee on trades goes to a treasury (governed). Fee rates are parameters.
- **Settlement is pluggable**: the ledger interface can be backed by a federated signed log first, then true consensus later, with no changes to anything above it.
- **Auditability**: every balance can be proven from signed history. Light clients verify with proofs, not trust.

### C3 — Package
A **package** is the universal unit of distribution: planets, carry-along items, plugins, adapters, codecs, UI themes, translations, tools. One format for everything.
- **Manifest**: id, version, author key, license, dependencies (with version ranges), declared capabilities needed (permissions), supported tiers, entry points, size budget, content hashes.
- **Content-addressed**: files are named by hash, so they can be fetched from any source (peers, relays, mirrors) and verified.
- **Signed**: by the author key; signatures chainable (author → reviewer → community-trusted).
- **Layered downloads**: base manifest first; heavy assets stream on demand; the user never downloads what they don't use.
- **Sandboxed execution**: logic ships as WASM (or an adapter-hosted binary under stricter rules); see C10.
- **Reproducible builds** are encouraged and flagged in the manifest, so anyone can verify that the source matches the binary.
- **Uninstall is complete**: packages declare where they store data; removal deletes it.
- **Package kinds** are an open registry: `planet`, `carry-along`, `adapter`, `codec`, `plugin`, `theme`, `locale`, `tool`. New kinds register via F6.

### C4 — Simulation & Scene
This is the contract that makes "pixelated friend and HD friend act identically" true.

**Two strictly separated halves:**
- **Simulation (shared truth)**: entities, positions, velocities, hitboxes, interactions, inventory changes, rules. Small, deterministic where possible, identical for everyone.
- **Presentation (local choice)**: how entities look and sound on a given device. Never affects the simulation.

**Entities & representations**
- Every entity has an ID, a transform, a *collision/interaction shape* (the simulation owns it), and an ordered list of **representations**, from richest to simplest:
  `full-3d → low-poly → billboard/sprite → icon → text description (+ audio cue)`.
- The client picks the highest representation it can render. The last (text) is **mandatory** (F7). It's also the accessibility layer and the voice/screen-reader layer.
- Hitboxes and ranges come from the simulation, never from the model. A tiny pixel avatar and a huge HD avatar have the same physical presence rules (or whatever the planet defines).

**Authority models** (planet declares which):
1. *Host-authoritative*: one elected host runs the simulation; others send inputs.
2. *Validator-replicated*: several nodes re-run the simulation and compare results; disagreements are resolved by majority.
3. *Client-authoritative with audit*: for low-stakes interactions; spot checks catch cheating.
Any event that touches the ledger **must** be validated by model 2 or stronger.

**Simulation structure**
- Fixed-rate ticks; clients interpolate/predict for smoothness; the tick rate is a planet parameter.
- Spatial partitioning ("cells") so a world can be hosted by many nodes, each owning a region.
- **State is a log of events + periodic snapshots** (replayable, auditable, rollback-able if exploited).
- Mergeable data types (CRDTs) for free-form collaborative building, so concurrent edits resolve without a boss.

**Scene description**
- A neutral, engine-independent scene format built on existing standards (glTF for models, standard formats for audio and animation, VRM-style humanoid rigs for avatars).
- Assets pass through **codecs** (C3 package kind). A new future format = a new codec plugin, nothing else.

### C5 — Governance
- **Scopes**: each decision belongs to a scope (planet, carry-along category, ecosystem, currency, protocol, constitution). Only that scope's community votes.
- **Tiers** (details in section 12): Community rules → Ecosystem rules → Protocol → Constitution, with increasing thresholds and delays.
- **Voting power** is never raw wealth. It combines account age, verified contribution, review accuracy and (optionally) personhood proofs. Weights are capped and dampened to prevent domination.
- **Proposal lifecycle**: draft → public discussion → sponsor threshold → voting window → delay → activation → review period (with automatic rollback if defined metrics break).
- **Everything executable is separated from everything voted**: a vote can approve a change, but it can never override security checks (C10) or ledger arithmetic (C2).
- **Delegation** (liquid democracy): you may delegate your vote on a topic to someone you trust and revoke at any time.
- **Parameters registry**: all tunable values (fees, speeds, limits) live here with bounds. Bounds protect against absurd values (e.g., a vote to set speed to infinity is rejected by bounds).

### C6 — Network
- **Roles** (a single machine can hold several): *light client*, *peer*, *host*, *validator*, *relay*, *storage node*, *indexer*, *seed*.
- **Transports are pluggable**: direct peer links (WebRTC), relayed links, WebSocket, and later anything new (QUIC, mesh, Bluetooth for local play). The upper layers never know which transport is in use.
- **Discovery**: distributed directory + gossip + a bootstrap list. Clients accept any number of seeds. Node #1 is just the first seed.
- **Hosting market**: hosting is a paid role. Planet treasuries and player groups pay hosts in OSC; hosts prove they store and serve data (proofs of storage/availability).
- **Host migration**: if a host drops, the next-ranked replica takes over from the latest snapshot.
- **Privacy**: IP hiding via relays; users choose between direct (faster) and relayed (more private).
- **Mobile reality**: phones are light clients by default and may opt in to light hosting while charging on Wi-Fi.
- **Resilience goal**: no single node, operator, country or company can take the network down.

### C7 — Device & Input
Handles "any hardware, working together".

**Capability profile** (declared by every client at connect time):
```
display:   [text | 2d | 3d | stereo-vr | ar-overlay | audio-only]
input:     [keyboard-mouse | gamepad | touch | vr-controllers | hand-tracking |
            eye-tracking | voice | switch/assistive | brain-computer | ...future]
compute:   {cpu, gpu, memory budget tier}
network:   {bandwidth tier, latency}
sensors:   [gps, camera, imu, ...]
haptics:   [...]
```
Profiles are open-ended lists (F6): a device class that doesn't exist today gets added later by registration.

**Intent-based input**
- Devices produce **intents**, not button presses: `move(vector)`, `look(direction)`, `interact(target)`, `grab`, `use(item)`, `speak`, `emote(id)`. Each device maps its physical inputs to intents.
- The simulation only understands intents. New hardware = a new mapper, no world changes.

**Locomotion & parity profiles (the "VR vs PC speed" question)**
- The base ships a conservative default *movement profile* per input class (e.g., walk speed, run speed, jump, turn rate) with hard bounds.
- Planets and communities may override within bounds, per input class, and may change them by vote (F8). Example: a racing planet gives VR its own comfort-adjusted profile.
- The **simulation enforces the profile**: a client claiming to move faster than its profile allows is rejected. Profiles are validated server-side by the host/validators, not trusted from the client.
- **Fairness classes**: a planet can declare competitive mode ("all input classes equalized") or open mode ("each class has its own strengths").

**Cross-device play**
- A VR user's hand-tracking produces `grab` intents; a phone user's tap produces the same `grab` intent. Both interact with the same object under the same rules.
- Presence is shared at the semantic level: "this avatar is waving", which each client renders as it can.

### C8 — Engine Adapter
Makes "every engine runs together" and "Unreal 7 later" cheap.

**Idea:** an engine is just a *program that implements the adapter interface*. The base never imports engine code.

**Adapter interface (conceptual)**
```
describe()        -> engine name/version, supported tiers, supported codecs, capabilities
load(package)     -> prepares a planet
tick(inputs)      -> advances simulation, returns state delta (for sim adapters)
render(state, profile) -> draws a view (for renderer adapters)
events            -> adapter emits entity changes & interactions in C4 format
```
- Adapters talk over a **language-neutral, process-boundary protocol** (streams/IPC/WebSocket). An Unreal 7 adapter can be a plugin inside Unreal that speaks the protocol. Nothing else changes.
- **Two roles**, which can be separate adapters:
  - *Simulation adapter*: runs the world logic (can be WASM, Godot, Unreal, custom).
  - *Renderer adapter*: draws the world (can differ from the sim engine!). This is what lets a Godot-simulated planet be viewed through a Unity or a custom pixel renderer.
- **Conformance suite**: a standard set of test scenes (movement, collision, trade, avatar swap, tier fallback). An adapter that passes the suite is "OASIS-compatible". Certification is automatic and public.
- **Cost of adding a new engine**: write one adapter + pass tests. No change to the base, no change to existing planets, no permission needed.
- **Honest limits**: heavy engines run natively on capable devices; phones may only support portable (WASM) planets or streamed/remote rendering through willing hosts. The manifest states this and the lobby shows it.
- **Cross-engine interaction**: engines never share a process. They share the C4 state stream. An object from an Unreal-simulated planet appears in another planet only through its C4 representation, never as engine objects.

**Bridge points for the future**
- New engine: new adapter.
- New renderer technology (neural rendering, holographic displays): new renderer adapter + representation type.
- New scripting language: compile to WASM, or add an interpreter adapter.

### C9 — Social & Discover
- **Feed, messages, groups, voice/video rooms** over open protocols (Nostr-style signed events for posts; end-to-end encryption for DMs). Anyone can run relays.
- **Discover (a.k.a. Showcase)** is a dedicated section for new creations (planets, items, outfits, tools). Its design is in section 8.
- **Cross-surface identity**: the same Pocket works on a real phone and inside the world. Posts and DMs flow between both.
- **Client-side filtering is a first-class feature**: block, mute, keyword filters, trusted-curator lists, per-community rules. Moderation is composable, not centralized.
- **Reports** are signed events routed to community-chosen moderators; outcomes are logged publicly.
- **Anti-spam**: posting costs something (reputation, small stake, or rate-limited quota), never a captcha.

### C10 — Permissions & Safety
- **Capability-based sandbox**: a package gets *only* what it declares and the user approves. Nothing ambient. Examples: `net.connect(host)`, `storage.local(quota)`, `mic`, `camera`, `ledger.request-payment(max)`, `read.friends-list`.
- **Payments always go through a system prompt** the package cannot draw or fake. A planet can *request* a payment; the Pocket shows it; the user signs.
- **Resource limits**: CPU, memory, bandwidth and storage budgets per package, enforced.
- **Code and asset scanning**: automated static checks and community review before a package appears in *curated* lists. Unreviewed packages can still be installed by explicit opt-in with warnings.
- **Safety is not voted**: community votes decide taste; technical checks decide safety.
- **Age and content ratings** are labels attached to packages and rooms; clients filter by user settings.
- **Emergency controls**: any user can instantly quit any planet; any planet can be force-removed locally; a signed "revoked package" list lets the community warn users about malicious packages (advisory, not forced).
- **Privacy defaults**: minimum data collection, no tracking IDs, local-first storage.

### C11 — Storage & Persistence
- **Content-addressed storage** (files by hash) with replication across storage nodes.
- **Tiers of permanence**: ephemeral (instance only), persistent (replicated), archival (pinned long-term, paid).
- **World snapshots + event logs** are stored the same way and signed by the authority model's validators.
- **Pinning market**: planet treasuries and individuals pay storage nodes to keep data alive; failure to keep data triggers slashing.
- **Backups**: every user can pull a full copy of their own data and any planet's public data.
- **Garbage collection** is policy-driven (F8), so abandoned worlds can expire unless pinned.

### C12 — Evolution
- **Semantic versioning** on every contract and package, with explicit compatibility ranges.
- **Version negotiation** during connection: peers pick the highest mutually supported version.
- **Feature flags via governance**: new behaviors activate on a scheduled date after a vote (like soft forks), never by surprise.
- **Deprecation policy**: minimum public sunset windows by tier (e.g., 12 months for protocol features).
- **Migration tools**: any breaking change ships with an automated migrator.
- **Forks are allowed and orderly**: a fork keeps identity continuity (users can choose which fork to follow). Compatibility tests tell users what works where.
- **Time capsule rule**: archived data must stay readable by a documented minimal reader.

---

## 3. Client side (L3)

### 3.1 The Pocket (always with you)
Runs on: real phone, PC, tablet, browser, and as a prop in-world.

Modules (each a plugin, F1/F6):
1. **Wallet** — balances, history, send/receive, escrow status.
2. **Market** — browse, list, bid, trade; price history; watchlist.
3. **Inventory** — all items across planets, searchable, with provenance.
4. **Wardrobe** — outfits, avatar editor, per-tier avatar previews ("how you look on a phone").
5. **Planets** — install/update/remove, sizes, permissions, ratings, tier support.
6. **Social** — feed, DMs, groups, friends, presence, voice rooms.
7. **Discover** — creation showcase + daily review duty (section 8).
8. **Governance** — proposals in your scopes, delegation, vote history.
9. **Settings & Safety** — keys, devices, recovery, filters, permissions, privacy.
10. **Developer** — package tools, logs, local sandbox, publishing.
11. **Maps & Travel** — lobby map, friends' locations, portals.
Third parties can add modules (F6) and themes.

### 3.2 The lobby & world client
- A small client core (Rust → native + WASM) with shells for desktop, mobile, web and VR.
- **Renderer tiers** T0 text/audio, T1 2D, T2 low-poly 3D, T3 full 3D, T4 VR/AR/haptic; more can be added. Tiers are labels in a list, not code branches.
- **Adaptive fidelity**: per-entity, per-distance quality selection from the representation list.
- **Accessibility by design**: screen-reader entity descriptions, subtitles, one-switch control, colorblind palettes, comfort settings in VR.

---

## 4. Planets & carry-along (L5, L4)

### Planets
- Self-contained packages implementing C4 via an adapter (portable WASM or engine).
- Declare: engines/tiers, authority model, movement profiles, permissions, item classes issued, economy hooks, rating.
- **Zones** inside a planet: *Protected* (fixed rules, nothing can be added), *Curated* (additions need votes), *Open/Chaos* (anything goes, risks accepted). Zone rules are data; the sandbox is always on regardless of zone.

### Carry-along (travels everywhere)
- Avatar rigs, outfits, accessories, emotes, voice effects, Pocket skins, universal currency items.
- **Compatibility classes**: *Universal* (works everywhere), *Adaptable* (a planet maps it to local rules), *Local* (stays in its home planet).
- **Avatar standard**: a humanoid rig + semantic slots (head, hands, back, etc.) + multi-tier representations + hitbox owned by the planet, not the avatar.
- Planets may restrict or adapt carry-along content, but cannot demand unrelated downloads (invariant).

### Planet Maker (first planet, and the reference SDK)
- Built *on top of* the public SDK; if it can't do something, neither can anyone else, which keeps the SDK honest.
- Features: import models/animations/audio; terrain and scene editor; item/rule editor; script editor (WASM-targeting languages); economy hooks wizard; permission declaration helper; tier testing (preview on simulated phone/VR); one-click package, sign, publish; local multi-client test mode.
- Collaborative editing (multiple creators, CRDT-based).
- Template library (voxel, racing, social hub, shooter, etc.).

---

## 5. Engine & hardware cookbook

### Adding Unreal Engine 7 (target: weeks, not months)
1. Read C8 spec and conformance scenes.
2. Build an Unreal 7 plugin that implements `describe/load/tick/render` and speaks the adapter protocol.
3. Run the conformance suite; fix failures.
4. Publish the adapter as a package (`kind: adapter`). Anyone who installs it can run planets that need it.
5. Done. No base change, no vote required for it to *exist* (a vote is only needed for it to be *featured*).

### Adding a new headset class
1. Register the device class and its intent mapper (C7).
2. Add a movement profile suggestion.
3. Optionally add a renderer adapter if rendering differs.

### Adding a new asset format
1. Write a codec plugin that converts it to/from the neutral scene format.
2. Publish as a package.

---

## 6. Interop between wildly different players

| Situation | How it works |
|---|---|
| HD PC player + pixelated phone player | Same simulation state; each client renders the best representation it can; hitboxes identical |
| VR player + keyboard player | Both emit intents; speed and reach governed by movement profiles |
| Text-only player + 3D player | Text client reads entity descriptions & events; can still trade, talk and act |
| Unity-simulated planet + Godot renderer | Renderer adapter reads C4 state stream; does not need to know the sim engine |
| Future device nobody has built | New capability profile + mapper |

**Parity principles**
- Gameplay-relevant facts live in the simulation and apply to everyone.
- Presentation differences are never competitive advantages unless a planet explicitly allows them.
- Planets with competitive modes can require "equalized" profiles.

---

## 7. Economy structure (details)

1. **Currency (OSC)**: universal; price discovery by players; issuance policy open (see section 15).
2. **Items**: issued by planets/identities as ledger assets with metadata: origin planet, class, rarity rules, transferability.
3. **Market**: order books and direct offers; reputation and escrow protect traders.
4. **Delivery**: planet-side agents confirm item delivery; the ledger finalizes.
5. **Anti-duplication**: every tradable item's creation is a ledger event signed by an authorized issuer; the planet cannot spawn a tradable item "off-ledger".
6. **Rate limits & issuer caps**: an issuer can mint only within parameters tied to reputation and stake; bursts trigger alerts.
7. **Exploit response**: if a planet's issuer is exploited, the community can vote to freeze *that issuer's* newly minted assets (scoped, time-limited, auditable). This never touches OSC balances or other issuers. It is the safety valve that avoids "exploits aren't our problem" turning into a collapse.
8. **Treasuries**: ecosystem treasury (fees), planet treasuries (their own funds). Spending is governed and public.
9. **Bridges** (fiat on/off-ramps, other chains) are optional external services. They do not live in the base.
10. **Fair market protections**: wash-trade detection, price-manipulation flags, visible volume and age data. Warnings, not bans.

---

## 8. Discover: the Showcase + mandatory review duty

Goal: surface good new creations, give every creator a fair shot, and keep every citizen involved, without it becoming a manipulation machine.

### 8.1 What "mandatory" really means
Software cannot force anyone to vote, and forcing creates junk votes (people click yes to get it over with). So the duty is tied to **benefits**, not punishment:

- Complete **N reviews per day** (start at small N, e.g., 5; N is a governed parameter) to keep **Citizen Standing** for that day.
- Standing unlocks: posting new creations, full voting weight in governance, daily rewards/stipends, and market listing privileges.
- Missing a day costs nothing but standing for that day; standing recovers quickly. Vacation mode pauses the duty.
- Reviews must be *honest* to count; see below.

### 8.2 How the system is built to resist abuse

| Attack / exploit | Defense |
|---|---|
| **Rubber-stamping** (click yes, yes, yes) | Hidden **gold items**: known-good and known-bad submissions mixed in. Failing gold lowers your review weight and does not count toward your duty. |
| **Brigading** (rally friends to boost/sink a post) | **Random assignment**: reviewers are *assigned* items from a pool; they cannot choose targets. Direct linking to boost isn't possible. |
| **Bandwagon** | **Blind voting**: reviewers cannot see current scores. Commit-reveal: votes are encrypted until the window closes. |
| **Fake accounts (Sybil)** | Weight grows with age, verified contribution and past accuracy. New identities have tiny weight. Optional personhood proofs add weight. Creating identities costs effort/stake. |
| **Collusion rings** | Reviewer-graph analysis flags correlated voting clusters; assignments spread reviewers across clusters; flagged clusters lose weight. |
| **Vote selling** | Commit-reveal and random assignment make it hard to know whether a buyer's target was actually reviewed by the seller; reputation staking makes selling risky. (Honest note: perfect coercion-resistance is unsolved; we make it unprofitable.) |
| **Self-promotion** | No self-review; no review of items from your close social graph or your own cluster. |
| **Spam submissions** | Submission requires a refundable deposit or reputation quota; low-effort floods burn the deposit. |
| **Plagiarism / re-uploads** | Content hashing and similarity detection against known packages; first-publisher record is on-chain. |
| **Malicious packages hidden in "good" content** | Safety scanning and sandbox are separate from votes (C10). A popular package can still be flagged unsafe. |
| **Review farms (bots)** | Reviews require engagement proof: open the sandboxed preview, minimum interaction time, answer a short structured prompt (e.g., choose tags). Not bulletproof; raises cost. |
| **Lazy/biased categories** | Assignments rotate categories so you review things outside your usual taste; you can set exclusions. |
| **Review fatigue** | Small N, skip/abstain allowed (abstain doesn't count but doesn't hurt). Quality of reviews matters more than quantity. |
| **Rich players dominate** | Weight never depends on wealth. |
| **Rank gaming by timing** | Fixed daily windows; randomness in ordering; anti-last-minute manipulation. |
| **Retaliation / harassment of creators** | Reviewers are anonymous to creators; only aggregated results are shown. |
| **Dogpiling on controversial ideas** | Reviews use multi-dimensional scoring (quality, originality, safety concern, fun), not just up/down. Polarizing items are labeled "divisive", not buried. |

### 8.3 Outputs
- Items earn placement in Showcase sections (New, Rising, Staff-free "Community Picks", by category).
- Placement never gates *installation*: anyone can still install anything unlisted (F10).
- Creators get structured feedback.
- Review accuracy builds reputation, which improves your future weight and unlocks curator roles.

### 8.4 Where this is dangerous
This system is **the most exploitable part of the design**. Plan for: a long testnet period with simulated attackers, public bug bounties, and parameters (N, weights, windows) governed and adjusted as real attacks appear.

---

## 9. Security model

| Threat | Mitigation |
|---|---|
| Stolen device | Device keys revocable; master key offline; spending limits per device |
| Lost keys | Social/hardware recovery in spec |
| Malicious package | Sandbox, capabilities, resource limits, scanning, signed revocation advisories |
| Fake payment prompts | System-drawn prompts only; packages can't render them |
| Ledger exploit | Frozen arithmetic, independent implementations, audits, conformance tests, bug bounties |
| Item duplication | Ledger-issued items only; issuer rate limits |
| Host cheating | Validator-replicated simulation for anything economic; event-log audit; rollback |
| DDoS on nodes | Many nodes; relay diversity; rate limiting; client multi-homing |
| IP exposure | Relayed connections option; no PII in protocol |
| Governance capture | Tiers, delays, scoped votes, non-wealth weight, delegation revocable, bounds on parameters |
| Spam / abuse | Costly-to-spam design; client-side filters; community moderation tools |
| Illegal content | Per-user filters, reporting, hash-list matching for known illegal media at the client/relay level (opt-in tools), no central takedown power but local legal compliance by node operators |
| Supply chain attacks | Signed releases, reproducible builds, multiple independent builders, dependency pinning |
| Protocol downgrade attacks | Version negotiation with minimum floors; crypto-agility |
| Insider (the founder) | Founder holds no special keys after bootstrap; release signing by multi-party threshold |

---

## 10. Repository & project structure

```
oasis/
  CONSTITUTION.md
  GOVERNANCE.md
  CONTRIBUTING.md
  oip/                 improvement proposals (numbered)
  specs/
    c01-identity/ ... c12-evolution/     (spec + schemas + tests + replace-me.md)
  conformance/         test suites for every contract
  core/                reference implementation (Rust → native + WASM)
  node/                reference node (relay, ledger, storage, host roles)
  pocket/              Pocket app (TS; web, mobile wrapper)
  client/              lobby + renderer shells
  adapters/            engine adapters (one folder per engine)
  codecs/              asset format codecs
  sdk/                 planet SDK + templates
  planet-maker/        the Planet Maker
  planets/             reference planets (voxel world first)
  carry-along/         avatar standard + sample outfits
  tools/               dev environment, packaging, testnet, simulators
  docs/                guides, tutorials, glossary
```

Rule: each folder has maintainers (CODEOWNERS), a README, tests and a "how to replace me" note.

---

## 11. Contribution system (anyone can work on anything)

1. **Spec-first workflow**: change a spec via OIP → update tests → update implementations.
2. **OIPs**: numbered proposals with a template; public discussion; status tracking.
3. **Tracks & working groups**: Protocol, Core, Pocket, Network, Engines, Planet SDK, Economy, Governance & Safety, Art & Content, Docs & Education, Community. Open membership, rotating leads.
4. **Public task board**: every task sized, labeled (`good-first-issue`, `help-wanted`, `spec`, `art`, `docs`, `security`), with a definition of done.
5. **Bounties**: paid from the treasury once OSC exists; before that, recognition and reputation.
6. **Contribution reputation**: merged work earns public reputation, which feeds governance weight.
7. **Multiple implementations** of each contract are encouraged and tested against the same suite.
8. **One-command dev environment**: containerized node + Pocket + test planet + simulated devices.
9. **Mentorship & docs**: every module has "start here" docs; maintainers hold office hours.
10. **Non-code roles** count equally: art, audio, translation, QA, moderation tooling, community care.
11. **Merge authority by risk tier**: docs/planets (1 reviewer) → core (2 reviewers) → protocol (OIP + conformance + 2 independent implementations) → constitution (full process).
12. **Release signing** is threshold-based (multiple signers), so no single person controls releases.
13. **Security disclosure** process and bounties from day one.
14. **Succession plan**: the founder's powers are documented and sunset by design (the "bus factor" is a feature to design out).

---

## 12. Governance tiers

| Tier | Examples | Voters | Threshold | Delay | Notes |
|---|---|---|---|---|---|
| T1 Community | planet items, zone rules, local profiles | that community | simple majority | short | fast and local |
| T2 Ecosystem | showcase params, carry-along standards, featured lists | all citizens in standing | supermajority | medium | |
| T3 Economy | fees, issuance parameters, treasury spend | all citizens + reputation weighting | high supermajority | long | bounded by parameter limits |
| T4 Protocol | new/changed contract versions | contributors + implementers + citizens (multi-house) | very high + implementer sign-off | very long | requires conformance + two implementations |
| T5 Constitution | invariants | all houses, repeated votes over a long span | near-unanimous | longest | changes are effectively hard forks |

Safety valves: bounded parameters, automatic rollback triggers, time-delayed activation, emergency pause limited to *new* issuance of an affected issuer (never to balances).

---

## 13. Roadmap (restated with exit criteria)

| Phase | Name | Deliverables | Exit criteria |
|---|---|---|---|
| 0 | Foundations | Constitution v0, vocabulary, C1–C12 drafts, OIP process, repo | Specs reviewed by outsiders; conformance skeletons exist |
| 1 | Identity + test ledger | Core lib, Node #1, Pocket web (wallet, trade) | Two strangers create identities and trade test-OSC from browsers |
| 2 | Package + first planet | Package format, sandbox, hello-planet, tiny reference planet | Same planet runs on low-end phone (T1/T2) and PC (T3) in one session |
| 3 | Simulation + adapters | C4/C7/C8 working, WASM adapter, one engine adapter (Godot) | Conformance suite passes on two adapters; cross-device play demo |
| 4 | Network | Multi-node, peer links, host migration, storage replication | Kill Node #1; lobby and a planet survive |
| 5 | Social + Discover | Feed, DMs, showcase, review duty on testnet | Simulated attack campaign fails to capture rankings |
| 6 | Governance | Proposals, scoped votes, parameters registry | A parameter change executes via vote on testnet |
| 7 | Open the doors | SDK, Planet Maker v1, docs | A stranger ships a planet unaided |
| 8 | Economy goes live | Issuance policy, treasury, hosting rewards, issuer freezes | Hosts earn OSC; audit complete |
| 9 | Heavy engines & new hardware | Unity/Unreal adapters, VR/AR, haptics | Adapters from 3 engines coexist; VR + phone + PC in one match |
| 10 | Ongoing | Evolution process (C12) | Future engines/devices added without base changes |

---

## 14. Testing and quality strategy

- **Conformance suites** for every contract (the real spec).
- **Simulated hostile network**: bots that cheat, spam, collude; run continuously on testnet.
- **Device lab**: automated tests on low-end phones, browsers, desktop, VR emulators.
- **Determinism tests** for simulation (same inputs → same state across adapters).
- **Chaos testing**: random node failures, partitions, host drops.
- **Fuzzing** for parsers (packages, manifests, messages).
- **Long-term tests**: open year-1 archives in current versions (time-capsule rule).
- **Public bug bounties** from testnet onward.
- **Independent audits** for ledger and sandbox before real value flows.

---

## 15. Open questions (decide with the community, record in OIPs)

1. OSC issuance: fixed supply, activity rewards, community-set emission, or hybrid?
2. Ledger settlement: federated log first, then which consensus?
3. Sybil resistance mix: reputation only, stakes, personhood proofs, or a combination?
4. Default review duty N and weights.
5. License: permissive (Apache-2.0/MIT) vs copyleft for the base.
6. Moderation model defaults (what clients ship with).
7. Remote/streamed rendering for weak devices: allowed as an opt-in, and how to pay hosts.
8. Legal strategy for the currency (jurisdiction, wrappers, bridge responsibility).
9. Naming/branding independent of "OASIS".
10. Where Node #1 lives, and who holds bootstrap keys until sunset.
11. Treasury initial allocation and founder compensation (if any).
12. Minimum viable set of reference planets.

---

## 16. First 90 days (builder's checklist)

**Days 1–14: Ground rules**
- [ ] Choose name, license, languages.
- [ ] Write Constitution v0 and glossary.
- [ ] Create repo skeleton and OIP template.
- [ ] Draft C1, C2, C3 and conformance skeletons.

**Days 15–45: First slice**
- [ ] Core library: keys, signing, verification, package hashing.
- [ ] Node #1: accept signed transactions, enforce arithmetic, serve packages.
- [ ] Pocket (web): identity, wallet, send/receive test-OSC.
- [ ] Package format v0 + hello-planet.

**Days 46–90: Cross-device proof**
- [ ] C4/C7 drafts; simulation in WASM; entity representations with text fallback.
- [ ] Minimal lobby client with T1 and T3 renderers.
- [ ] Two clients on different device classes in one shared room.
- [ ] Second node online; failover demo.
- [ ] Publish specs and invite first outside contributors.

---

## 17. The rules for future-us (read before every decision)

1. If it can be a plugin, it is not in the base.
2. If it can be a parameter, it is not a constant.
3. If it is tied to one engine, device or vendor, it belongs in an adapter.
4. If the user can't leave with their data, it's wrong.
5. If it requires trusting one party, redesign it.
6. If it can't degrade to text, it isn't finished.
7. If a stranger can't contribute to it without asking you, it's still too centralized.
8. If the year-20 version can't read year-1 data, we broke the promise.

   This list didnt take that long to make but my ideas stop here. Lets see where this goes and check out the reddit page because obviously all the talking will be done there.
