# Acid Frogs · Inscription Architecture Test

End-to-end validation that the parent + child + Nexus + recursive-ownership architecture works on Bitcoin mainnet **before committing the real ~£175 master parent + 10K delegates.**

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

## Files

| File | Size | Purpose |
|---|---|---|
| `mock-parent.html` | ~16 KB | The test parent inscription. Splash + studio + Nexus + squad + playback. |
| `mock-delegate.html` | ~1.8 KB | Template for child inscriptions. Self-rendering coloured shape derived from its own inscription ID. |
| `INSCRIBE_TEST.md` | this | Procedure |

Both are static HTML, no external resources beyond the inscribed Nexus module.

## Cost

At sub-1 sat/vB:

| Step | Approx cost |
|---|---|
| Parent inscription (~16 KB) | ~£0.40 |
| 5 mock-delegate inscriptions (~1.8 KB each) | ~£0.20 |
| Mainnet test fees + dust | ~£3-5 |
| **Total** | **~£5** |

Cheaper than a coffee. End-to-end mainnet validation that protects the £175+ real parent inscription.

## Procedure

### 1. Open your ord wallet

```bash
ord --bitcoin-data-dir /path/to/.bitcoin --data-dir /path/to/ord wallet balance
```

Confirm you have at least ~50,000 sats in the wallet to cover everything with headroom.

### 2. Inscribe the parent

```bash
ord wallet inscribe --fee-rate 1 --file inscription-test/mock-parent.html
```

Wait for confirmation (typically one block, ~10 min). Record the inscription ID — call it `PARENT_ID`. It looks like `abc123...i0`.

### 3. View the parent on ordinals.com

```
https://ordinals.com/content/{PARENT_ID}
```

You should see the rotating logo splash. Click ENTER STUDIO. The test report at the bottom should show:
- `self:ok` (URL contains the inscription ID)
- `/r:ok` (recursive endpoint reachable)

Click DEMO to verify the squad UI works with mock data. Click any frog → it should beep and pulse.

### 4. Inscribe 5 delegates as children of the parent

For each delegate, run:

```bash
ord wallet inscribe \
  --fee-rate 1 \
  --parent {PARENT_ID} \
  --file inscription-test/mock-delegate.html
```

Or use a YAML batch (faster, atomic, all in one mempool submission):

```yaml
# inscriptions-batch.yaml
mode: same-sat
parent: <PARENT_ID>
inscriptions:
  - file: inscription-test/mock-delegate.html
  - file: inscription-test/mock-delegate.html
  - file: inscription-test/mock-delegate.html
  - file: inscription-test/mock-delegate.html
  - file: inscription-test/mock-delegate.html
```

```bash
ord wallet inscribe --fee-rate 1 --batch inscriptions-batch.yaml
```

Wait for confirmations. Each delegate gets a unique inscription ID and renders a different coloured shape based on that ID.

### 5. Send delegates to a different wallet

Pick 2 of the 5 delegates and send them to a different wallet address (e.g., a fresh Xverse wallet). This sets up the cross-wallet ownership test.

```bash
ord wallet send --fee-rate 1 {DEST_ADDRESS} {DELEGATE_ID}
```

### 6. Test ownership lookup from each wallet

In a browser:

1. Open `https://ordinals.com/content/{PARENT_ID}` again
2. Click ENTER STUDIO → CONNECT WALLET (your origin wallet)
3. Wait for the scan: `scanned 5/5 · 3 owned`
4. The 3 frogs in your origin wallet appear in the squad
5. Click each — should beep with different pitches based on hue

Switch wallets:

1. Disconnect, click CONNECT WALLET, choose the destination wallet
2. Squad should now show the 2 delegates that were sent there
3. Different frogs visible from a different wallet → ownership lookup proven

### 7. Confirm the test report

Bottom of the studio should read:

```
TEST · self:ok · /r:ok · nexus:ok · wallet:ok · children:ok · owners:ok · play:ok
```

All 7 ok = entire architecture validated.

## What this proves

If all tests pass:

- ✅ Recursive parent/child structure works on Bitcoin mainnet
- ✅ Nexus loads from inscription and connects wallets read-only
- ✅ `/r/children/{parent}/{page}` paginates correctly
- ✅ `/r/inscription/{id}` returns the live owner address
- ✅ Filtering children by owner = the squad model works
- ✅ Click-to-play renders audio inside an inscription
- ✅ The whole stack survives if the website goes away — works directly on `ordinals.com/content/{id}`

We can then commit to the real master parent (4 MB algorithm + sprite sheets + audio) and 10K delegates with confidence.

## Failure modes to watch for

| Symptom | Likely cause | Fix |
|---|---|---|
| `nexus:fail` | Pinned inscription ID changed or unreachable | Verify `5cf27ef513…` still resolves on ordinals.com |
| `children:fail` | No delegates inscribed yet, or parent ID wrong | Check `/r/children/{parent}/0` directly |
| `owners:fail` | All children have null `address` (unconfirmed transfers?) | Wait for next block confirmation |
| `wallet:fail` | No supported wallet extension installed | Install Xverse |
| Squad shows 0 owned | Wallet you connected doesn't actually own any test delegates | Check on ordinals.com that the delegate's owner address matches |
