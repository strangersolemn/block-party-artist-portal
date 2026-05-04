# Inscription Architecture Test

End-to-end validation that the parent + child + Nexus + recursive-ownership architecture works on Bitcoin mainnet **before committing the real production parent inscription.**

No Bitcoin node required — uses a web-based inscriber.

## What this tests

| Behaviour | Test |
|---|---|
| Parent renders splash + studio | Open parent on ordinals.com, click ENTER STUDIO |
| Recursive endpoint reachable | Auto-tested at studio load — `/r/blockheight` |
| Nexus loads from inscription | Click CONNECT WALLET — Nexus module imported via dynamic `import()` |
| Wallet connect read-only | Approve Xverse / UniSat / Leather popup |
| Self-discovery | Parent reads its own inscription ID from `location.pathname` |
| Children enumeration | Parent pages through `/r/children/{self}/{page}` |
| Ownership lookup | Each child queried via `/r/inscription/{id}` → `data.address` |
| Squad filter | Children whose owner address matches connected wallet show in YOUR SQUAD |
| Click-to-play | AudioContext beep + visual pulse |

The parent has a built-in test report at the bottom: `TEST · self:ok · /r:ok · nexus:ok · wallet:ok · children:ok · owners:ok · play:ok`

## Files (download from GitHub Pages)

| File | Size | Direct download URL |
|---|---|---|
| `mock-parent.html` | ~17 KB | https://strangersolemn.github.io/block-party-artist-portal/inscription-test/mock-parent.html |
| `mock-delegate.html` | ~1.8 KB | https://strangersolemn.github.io/block-party-artist-portal/inscription-test/mock-delegate.html |

Right-click → "Save As" both files to your computer (or `curl` them down).

## Cost

At sub-1 sat/vB current fees:

| Step | Approx |
|---|---|
| Parent (~17 KB) | ~£0.40 fee |
| 5 children (~1.8 KB each) | ~£0.20 fee |
| Inscriber service fee + dust | ~£3-5 |
| **Total** | **~£5** |

## Procedure

### STEP 1 — Sanity-check the UI in your browser first (no chain needed)

Open this URL on your laptop:

```
https://strangersolemn.github.io/block-party-artist-portal/inscription-test/mock-parent.html
```

What you should see:
1. Spinning logo + ENTER STUDIO button
2. Click ENTER STUDIO → studio panel opens
3. Click **DEMO** → 5 coloured circles appear in YOUR SQUAD
4. Click any circle → it pulses gold + plays a beep
5. Bottom of screen shows: `TEST · self:pending · /r:ok · nexus:pending · wallet:pending · children:pending · owners:pending · play:ok`

If that all works → UI is good. The `pending` ones become `ok` only after inscribing on mainnet (because the URL has to be `ordinals.com/content/{id}` for self-discovery to work).

### STEP 2 — Save the two files locally

In your browser, right-click each link below and choose "Save Link As" (or "Download Linked File"):

- mock-parent.html → https://strangersolemn.github.io/block-party-artist-portal/inscription-test/mock-parent.html
- mock-delegate.html → https://strangersolemn.github.io/block-party-artist-portal/inscription-test/mock-delegate.html

Or via terminal:

```bash
curl -O https://strangersolemn.github.io/block-party-artist-portal/inscription-test/mock-parent.html
curl -O https://strangersolemn.github.io/block-party-artist-portal/inscription-test/mock-delegate.html
```

### STEP 3 — Inscribe the parent

Pick any web inscriber. The flow is the same for all of them:

| Inscriber | URL | Notes |
|---|---|---|
| **inscribe.dev** | https://inscribe.dev | BlockForge's recommended tool. Supports parent inscription field + brotli compression. |
| **Mscribe Blockpad** | https://blockpad.mscribe.io | What you've used before. |
| **Ord.io** | https://ord.io/inscribe | Clean UI, supports parent field. |
| **Ordinalsbot** | https://ordinalsbot.com | Mature service, has API. |
| **Unisat inscribe** | https://unisat.io/inscribe | Direct from the Unisat extension. |

**On the inscriber:**
1. Connect your wallet (Xverse / UniSat / Leather)
2. Upload `mock-parent.html`
3. **Leave the parent field BLANK** for the parent itself
4. (Optional) tick "Brotli compression" if available — saves a few sats
5. Set fee rate to current low (1 sat/vB or whatever is showing)
6. Confirm + sign in your wallet

Wait ~10 minutes for confirmation.

**📝 COPY THE INSCRIPTION ID** from the inscriber's confirmation page (or check the inscriber's "My Inscriptions" tab). It looks like `abc123def456...i0`. **Call this `PARENT_ID`.**

### STEP 4 — Verify the parent works on chain

In your browser, open:

```
https://ordinals.com/content/{PARENT_ID}
```

Replace `{PARENT_ID}` with what you copied. (Wait a few minutes after confirmation for ordinals.com's indexer to catch up.)

Click ENTER STUDIO. The bottom test report should now read:

```
TEST · self:ok · /r:ok · nexus:pending · wallet:pending · children:pending · owners:pending · play:pending
```

`self:ok` = the parent successfully read its own inscription ID from the URL. ✓

Now click CONNECT WALLET (with Xverse / UniSat / Leather installed):
- Approve the popup
- Should show: `▣ Xverse · bc1pxyz…1234`
- Test report updates: `nexus:ok · wallet:ok`

Squad will say "no children of this parent yet" because we haven't inscribed any. That's expected. Continue to step 5.

### STEP 5 — Inscribe 5 children of the parent

Same web inscriber as before, but **this time set the parent inscription field**:

For each of the 5 children:
1. Upload `mock-delegate.html`
2. **Set parent inscription ID to your `PARENT_ID`** (this is the critical bit — without it, the children won't be linked to the parent)
3. Confirm + sign

If your inscriber supports batch upload, drop `mock-delegate.html` 5 times in one go. Otherwise repeat the single-inscription flow 5 times.

**📝 COPY ALL 5 INSCRIPTION IDS** — call them `CHILD_1` through `CHILD_5`.

Wait ~10-30 minutes for all confirmations.

### STEP 6 — Verify children on the parent

Refresh `https://ordinals.com/content/{PARENT_ID}` (hard reload: Ctrl/Cmd-Shift-R).

Click ENTER STUDIO → CONNECT WALLET → Xverse.

Now the studio should:
- Page through `/r/children/{PARENT_ID}/0`
- Find the 5 inscriptions you just made
- Check ownership of each → all 5 owned by your wallet
- Show 5 coloured circles in YOUR SQUAD

Test report:

```
TEST · self:ok · /r:ok · nexus:ok · wallet:ok · children:ok · owners:ok · play:pending
```

Click any circle → it pulses gold + beeps. → `play:ok`. **All 7 green = architecture validated.** 🎉

### STEP 7 — Cross-wallet ownership test (the critical one)

Get a destination address. Easy options:
- Open Xverse on your phone or another browser → copy your **ordinals address** (starts with `bc1p…`)
- Or use a friend's wallet
- Or generate a second address in the same wallet via Xverse's "Add account"

Send 2 of the 5 children to that destination. Most wallets and inscribers have a "send" or "transfer" feature for inscriptions:

**Via Xverse:**
1. Open the wallet → Ordinals tab
2. Find one of your child inscriptions
3. Tap → Send
4. Enter destination address
5. Confirm

**Via Unisat:**
1. Wallet → Inscriptions
2. Pick a child → Transfer
3. Enter destination address
4. Confirm

Repeat for a second child. Wait one block (~10 min).

### STEP 8 — Final test: each wallet sees only its own children

**Test A — origin wallet (the one that inscribed):**
1. Refresh `https://ordinals.com/content/{PARENT_ID}`
2. ENTER STUDIO → CONNECT WALLET (your origin wallet)
3. Squad scans 5/5 → shows **3 owned** (the ones you didn't send)

**Test B — destination wallet:**
1. Click DISCONNECT
2. CONNECT WALLET → choose the destination wallet
3. Squad scans 5/5 → shows **2 owned** (the ones you sent)

**If both wallets see the right number → ownership lookup works → entire architecture proven on mainnet.** Total spend: about £2-5.

## What success means for the real launch

If steps 1-8 all pass:

- ✅ Recursive parent/child structure works on Bitcoin mainnet
- ✅ Nexus loads from inscription and connects wallets read-only
- ✅ `/r/children/{parent}/{page}` paginates correctly
- ✅ `/r/inscription/{id}` returns the live owner address
- ✅ Filtering children by owner = the squad model works
- ✅ Click-to-play renders audio inside an inscription
- ✅ The whole stack survives if the website goes away — works directly on `ordinals.com/content/{id}`

You can then commit to the real production parent (with the full algorithm + assets) and large-scale child inscriptions with confidence.

## Failure modes to watch for

| Symptom | Likely cause | Fix |
|---|---|---|
| `nexus:fail` | Pinned inscription ID changed or unreachable | Verify `5cf27ef513…` still resolves on ordinals.com |
| `children:fail` | Children inscribed without parent field set | Re-inscribe with parent ID; old ones can't be linked retroactively |
| `owners:fail` | All children have null `address` (unconfirmed transfers?) | Wait for next block confirmation |
| `wallet:fail` | No supported wallet extension installed | Install Xverse |
| Squad shows 0 owned | Wallet you connected doesn't actually own any test children | Check on ordinals.com that the child's owner address matches |
| "no children of this parent yet" after step 5 | Children were inscribed but parent field was blank | Most common failure. Re-do step 5 with parent field filled in. |
