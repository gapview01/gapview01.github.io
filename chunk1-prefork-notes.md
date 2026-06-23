# Chunk 1 — Pre-fork onboarding · conversion notes (leverage, not rebuild)

**Artifact:** `chunk1-prefork.html` (founder-screenshot click-through, review-only — NOT production, NOT goblin_ui).
**Live:** https://gapview01.github.io/chunk1-prefork.html?v=2#0
**Scope:** the shared spine O0–O8, Welcome → "Pick your start" (the binary Crypto │ Cash fork). Both branches are later chunks.
**Source of truth:** founder screenshots `toreva-stickerverse/docs/screens-reference/onb-0X-crypto.png`, ordered per `toreva-stickerverse/docs/sitemap.md` + `tests/e2e/screen-map.ts`.

> Why this doc exists: so the founder can see Chunk 1 is mostly **UI porting onto finished machinery**, not new product wiring. Every "wire to" below is an existing, running asset. Citations are to the consolidated review `po/docs/plans/app-migration-existing-wiring-synthesis-001.md` (the "synthesis") and the screen rows in `sitemap.md` §1 / §7 + `screen-map.ts`.

---

## The one rule (synthesis TL;DR)
The real-money + identity machinery is **finished goods**. Backends (identity passkey/KYC/vault, money-truth, network-sol, receipt-model, choice-surface) are HTTP services / npm packages the app **calls via `createServerFn`** — it does not port their code. Chunk 1 is overwhelmingly a **UI-port** (Next/Day1 → TanStack Start / Stickerverse palette). The #1 risk is rebuilding what already runs. (synthesis §TL;DR, §2.)

Chunk 1 sits in synthesis **Step 2 — "Welcome + onboarding 0–7 (identity)"**, the FIRST screens to port, plus the head of **Step 3** ([O8] Pick your start). Each screen PR passes the 4 hardened gates: **G1** MEMORY/constitution audit · **G2** ≤2% visual-diff vs the founder PNG · **G3** wired+planetary (only on money screens) · **G4** vitest. (synthesis §3 Step 2/3, Appendix gates.)

---

## Per-screen wiring map

### O0 — Welcome · "Box it. Rule it. Run it." + asset rotator
- **Screenshot:** `onb-00-crypto.png`
- **Reuse:** Pure UI. Brand rules from Stickerverse `MEMORY.md` (orange robot mark, teal Sign-in / orange Sign-up, "Have an account? Sign in"); legacy frame `toreva-app.tsx frames[0]`, pattern refs `routes/welcome-proto*.tsx`.
- **Wire to:** nothing dynamic. "Sign in" → existing identity passkey sign-in challenge (see O2/O3).
- **Owners:** Front-end + Brand (mark) + **Grammar**. ⚠️ Grammar template DEFERRED (synthesis §4) — gates the visual sign-off of this + O1.
- **Cite:** synthesis §3 Step-2 row 1.

### O1 — Hi, I'm Toreva → Box / Rule / Run cards
- **Screenshot:** `onb-01-crypto.png`
- **Reuse:** Pure UI; Box/Rule/Run card pattern (`MEMORY.md`), legacy frame `toreva-app.tsx frames[1]`. No robot at step 1.
- **Wire to:** static Terms / Privacy links (already on the public site).
- **Owners:** Front-end + **Grammar** (template deferred, same as O0).
- **Cite:** synthesis §3 Step-2 row 2.

### O2 — Lock it to you (Face ID / biometric)
- **Screenshot:** `onb-02-crypto.png`
- **Reuse-as-is:** **identity** passkey challenge/verify — `identity.ts: prefetchSignInChallenge / performServerAssertion`. Real WebAuthn.
- **Critical wiring rules:** create the credential on a **fresh user gesture** (recovery-rebind memory — `navigator.credentials.create()` must not run after an awaited network call); device token + flow ID; **no localStorage token** (INV-SEC-D1-006).
- **Owners:** **Identity** + Front-end.
- **Cite:** synthesis §1a identity rows, §2 D16, §3 Step-2 row 3.

### O3 — Save a passkey (1Password / Chrome / Passwords)
- **Screenshot:** `onb-03-crypto.png`
- **Reuse-as-is:** same identity passkey credential-creation flow as O2 — this is the OS passkey-provider sheet. No parallel KYC/auth path (synthesis D16).
- **Wire to:** identity passkey register; bind **pubkey** as the passkey identity (recovery-rebind memory), not a session accountId.
- **Owners:** **Identity** + Front-end.
- **Cite:** synthesis §1a identity (passkey reuse-as-is), §2 D16.

### O4 — Locked to your face (confirmation)
- **Screenshot:** `onb-04-crypto.png`
- **Reuse:** Pure UI confirmation of the O2/O3 bind; reads identity readiness/flow state already returned by the passkey verify.
- **Owners:** Identity + Front-end.
- **Cite:** synthesis §3 Step-2 row 3 (the "Locked to your face" confirm in the O2–O4 group).

### O5 — Your name
- **Screenshot:** `onb-05-crypto.png`
- **Reuse-as-is:** **identity Tier-1 PII** capture → AES-256-GCM vault service (backend-only). Field persists via the existing identity profile/vault write, not a client store.
- **Owners:** Identity + Front-end.
- **Cite:** synthesis §1a identity (vault service / verification events), §3 Step-2 remaining row ([O5] Your name).

### O6 — Your birthday (DOB wheel)
- **Screenshot:** `onb-06-crypto.png`
- **Reuse-as-is:** same identity Tier-1 vault path as O5 (DOB is a Tier-1 KYC attribute used by `tieredModel.ts`). Wheel picker is UI; the value feeds existing KYC tiering.
- **Owners:** Identity + Compliance (Tier-1 attribute) + Front-end.
- **Cite:** synthesis §1a identity (KYC Tier1 / tieredModel.ts), §3 Step-2 ([O6] Birthday wheel).

### O7 — Recovery phrase → cloud backup → "You're backed up"
- **Screenshot:** `onb-07-crypto.png` (recovery-phrase list + "I wrote it down"). The "Are you sure / cloud backup / backed-up / backup-didn't-save" sub-states are the same flow (`preview-backup-failed.png` is the failure state).
- **Reuse-as-is:** **identity** vault + seed/backup round-trip. Two-tier Wallet Recovery Doctrine (DEC-2026-017): Tier-1 written seed backstop (mandatory) + Tier-2 opt-in biometric-gated cloud (iCloud Keychain / Google PW).
- **Critical wiring rules:** **BackupSheet must never infinite-spin** (route to recovery on failure — clobber/recovery memory); key seed storage **per pubkey** (per-wallet seed clobber memory); anti-stub round-trip test `derive(restored)==pubkey` (DEC-2026-016).
- **Owners:** **Identity** (owns) + Privacy + Sentinel.
- **Cite:** synthesis §3 Step-2 remaining ([O7] Recovery phrase → cloud backup → "You're backed up"); §1a identity.

### O8 — Pick your start (BINARY: Crypto │ Cash) — the fork
- **Screenshot:** `onb-08-crypto.png` (swipeable sticker card; "Crypto" shown, dots indicate a 2nd "Cash" card).
- **Reuse-as-is:** **choice-surface** binary `OptionSet`; binary swipeable sticker-card carousel pattern from `MEMORY.md` / `routes/pick-proto.tsx`. **tap = commit, no Continue button** (North-Star binary rule).
- **Wire to:** choice-surface `Option/OptionSet` → sets the branch; downstream Crypto vs Cash chunks consume it.
- **Owners:** Front-end + **Choice-Surface**.
- ⚠️ **Blocked on missing pick/binary card art** (synthesis §5 open-decision #2) — the swipeable sticker-card stickers ([O8]/[O9c]/[O10c]) need commissioning or a Cut-A placeholder.
- **Cite:** synthesis §3 Step-3 ([O8] Pick your start), §2 D12/D14 (card factories — don't inline), §5 #2.

---

## Cross-cutting (applies to all of Chunk 1)
- **Identity is the spine of Chunk 1.** O2–O7 are all identity-owned and **reuse-as-is** — passkey, Tier-1 PII vault, seed/backup. The synthesis duplication register flags re-implementing any of these as **HIGH** (D16: "reuse exactly; no parallel KYC paths, no localStorage token fallback").
- **No client-side model calls, no localStorage auth token, no 2nd CTA / sub-header** (parent-plan backend bar + INV-SEC-D1-006).
- **Truth-gate (downstream):** in-life access keys on **activation completion**, not funds/possession — relevant once Chunk-1 identity feeds the later activation chunks (active memory; synthesis §3 Step-4 note).
- **Deferred gating:** **grammar** template (gates G1/G2 sign-off on O0/O1) and **user-engine** review are both deferred on fleet quota (synthesis §4) — PO re-drive items, not Chunk-1 build blockers for the screenshots themselves.

---

## What this artifact is / isn't
- **Is:** a review click-through of the founder's own screenshots, so he can tap the pre-fork flow on his phone and approve the sequence + design.
- **Isn't:** production, goblin_ui, or a port. PO converts the real screens under a dark flag **after** founder approval, per the synthesis Step-2/3 ticket template + 4 gates.
