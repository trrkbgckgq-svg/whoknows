# How the WHOKNOWS program is tested

Everything below was run on **November 8, 2026** against the WHOKNOWS Solana program, inside a local
Solana test runtime (the same runtime Solana uses, run on our machine instead of mainnet). The raw
results are in [`test-results/`](test-results/).

These tests were run by us. They are not an independent audit.

## 1. The automated test suite: 65 tests, 0 failures

| Area | Tests | What they check |
|---|---|---|
| Pool math | 5 | Prices always add up to 100%, buying moves the odds, a buy-then-sell round trip never profits, extreme numbers never overflow |
| Attacks | 15 | Sandwich attacks, buy-and-sell in one transaction, dust trades dodging fees, split/sell/merge arbitrage, redeem and claim attacks, transfers with fees, outside burns, trading after close, last-second result flips, the fair-launch cap, extreme values, cancels paying out more than went in |
| Permissions and validation | 12 | The fee wallet is fixed per market, a blank admin is rejected, coin sides can't be faked, coin mints can't be pre-made or duplicated, only a coin's owner can redeem it, Pyth price settings are checked, the fair-launch cap is per market, cancels pay the average price and can be corrected in a fresh window |
| Money edge cases | 6 | A last-second buy can't steer a cancel payout, the creator can't set the cancel price, the cancel price reflects the whole market, dust trades still pay fees, the fair-launch cap holds even inside one transaction, burning coins outside the program only loses the burner's own value |
| Full market lifecycles | 13 | Every market opens at 50/50, trading, splitting and merging pairs, settling YES or NO, price markets settling from Pyth (and bad price data being rejected), cancels, quiet markets returning the seed, results becoming final, rules staying locked on live markets |
| Community vote (built, not switched on) | 12 | Proposals, disputes, juries, bonds and rounds stay solvent and can't be gamed; the admin and creators can't shortcut it |
| Random stress test | 1 | See section 2 |
| Test setup | 1 | Generates the fixtures the other tests use |

Results: [`test-results/2026-11-08-test-suite.txt`](test-results/2026-11-08-test-suite.txt).

## 2. The random stress test: 300,000 actions, 4,000 markets, 0 failures

The stress test opens a market with a random seed and hits it with random actions: buys and sells of
random sizes, tiny "dust" buys, coin transfers between wallets (paying the transfer fee), and jumps
in time. Then it ends the market a random way (YES, NO, expiry cancel or admin cancel) and everyone
cashes out.

**After every single action** it checks that:
- the total SOL across every wallet, the market and the fee wallet hasn't changed, and
- the pool's value never went down.

**At the end of every market** it checks that:
- losing coins can never be paid out,
- every payout is exactly right,
- the creator is paid the pool exactly once,
- at most 10 lamports (0.00000001 SOL) of rounding dust stays behind, and
- the total SOL at the end equals the total at the start.

We ran it at full scale:

| | |
|---|---|
| Markets | **4,000** |
| Random actions | **300,000** (75 per market) |
| Went through | 250,660 |
| Correctly refused | 49,340 (invalid or out-of-bounds actions the program must reject) |
| Failures | **0** |

Results: [`test-results/2026-11-08-stress-300k.txt`](test-results/2026-11-08-stress-300k.txt).
