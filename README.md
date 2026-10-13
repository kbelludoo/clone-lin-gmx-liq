# clone-lin-gmx-liq

Experimental i64-scale LIN clone of [gmx-io/gmx-contracts](https://github.com/gmx-io/gmx-contracts) VaultUtils **validateLiquidation** (remaining collateral after PnL vs `marginFees`, then vs `marginFees + liquidationFeeUsd`, then `remaining * maxLeverage < size * 10000`).

- **LIN clone:** [`src/lin_gmx_liq.lin`](src/lin_gmx_liq.lin)
- **C11 oracle:** [`test/oracles/gmx_liq_c11.c`](test/oracles/gmx_liq_c11.c)
- **Proofs (results only) live in lin-open:** https://github.com/kbelludoo/lin-open
- **Event receipt:** `docs/events/EVENT_GMX_LIQ_CLONE_LIN.rulel` in lin-open

This repository is the standalone LIN copy. It is **not** a GMX Vault, **not** GLP, and does **not** claim Solidity uint256 / mainnet parity. The kernel is the GMX V1 **remaining-collateral liquidation gate**. If remaining is below computed `marginFees`, collected fees **cap to remaining**. If remaining sits between `marginFees` and `margin+liqFee`, fees stay `marginFees` (not remaining, not margin+liq). Profit is added only when `raise=0` (liquidate path). It is **not** GMX `getFundingFee`, **not** GMX `getNextFundingRate`, **not** GMX `getFeeBasisPoints` USDG tax/rebate, **not** Uniswap V2 `xy`, **not** Solidly cubic `x3y+y3x`, and **not** Maker `rpow`.

Upstream is MIT. This LIN kernel is an original uint64-scale implementation of the published `validateLiquidation` formula, not a verbatim Solidity dump. `validateLiquidation` / `remainingCollateral` / `liquidationFeeUsd` / `maxLeverage` are rejected by the Solidity transpiler (`REJ_SOLIDITY_GMX_LIQ`).

Pinned upstream: commit `fe55c4ba8911ae2378b50bfb9bece5ee50cf2af8`, `contracts/core/VaultUtils.sol` sha256 `ad831f3c8d675c42665c854368bc06696e1cba23bd3294f91aa2ce4c0a8fc495` blob `8e29c3ff19258d191f6d2459c110a917ba1f9906`.

Canonical goldens (uint64-scale, 128-bit product compare):

- remaining 900 < margin 950: state **1** fees capped **900** vs no-cap **950**
- remaining 920 in [900, 950): fees **900** vs include-liq **950**
- losses 600 > collateral 500: state **1** fees **100** (not remaining 0)
- remaining 1500 vs 50x cap on size 100000: state **2** fees **10**
- remaining 3000 healthy: state **0** fees **10**
- raise=1 omits profit remaining 1000: fail-closed **-4**; raise=0 remaining 1500: **0**
- naive i64 `4e9*3e9` flags leverage **1**; 128-bit product stays **healthy 0**
