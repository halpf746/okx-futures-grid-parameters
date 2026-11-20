# OKX Futures Grid Bot: How the Parameters Work, Which Direction to Choose, and What Happens When Price Leaves Your Range

The OKX futures grid bot is a strategy you build inside the exchange rather than a separate piece of software: you set a price range, a grid count and a leverage level, and it places buy and sell orders on futures contracts at fixed intervals inside that range. Maximum leverage is **100x**, and futures grid access is limited to accounts OKX treats as eligible users, so some readers will not see the entry at all. That eligibility check is the first thing to settle, before you spend time on grid spacing.

What follows covers the three directions and how they differ at the moment of creation, every parameter using OKX's own 50,000–100,000 USDT walkthrough, what actually happens to your position when price exits the range, and how to confirm your account can use the tool.

> **Quick version:** long grid if you expect a range with an upward lean, short if the lean is downward, neutral if you have no directional view. Pick a pair you would trade manually anyway, keep leverage at 2x–3x while you learn how the fills behave, and put a stop-loss below the lower limit rather than relying on the range itself. There is no subscription price for the bot; OKX charges trading fees on each fill, listed on its fee page. 👉 [Create an OKX account and open the trading bots panel](https://okx.com/join/CASH20)

## The current OKX bot lineup and who can use each one

Six bots appear on OKX's current trading bots overview. Two of them are futures products, and both carry the eligibility restriction.

| Bot | What it does | Who can use it | Leverage | Price | Entry |
| --- | --- | --- | --- | --- | --- |
| Spot grid | Repeated buy and sell orders inside a price range on spot pairs | All users | None | No subscription fee; trading fees on fills | [Open spot grid](https://okx.com/join/CASH20) |
| Spot DCA | Averages into a spot position, adding to it as price moves against the entry | All users | None | No subscription fee; trading fees on fills | [Open spot DCA](https://okx.com/join/CASH20) |
| Futures grid | Grid of orders on perpetual or delivery futures, in long, short or neutral mode | Eligible users only | Up to **100x** | No subscription fee; trading fees on fills | [Open futures grid](https://okx.com/join/CASH20) |
| Futures DCA | Leveraged version of DCA on futures contracts | Eligible users only | Set per strategy | No subscription fee; trading fees on fills | [Open futures DCA](https://okx.com/join/CASH20) |
| Iceberg order | Splits a large order into smaller visible pieces | All users | None | No subscription fee; trading fees on fills | [Open iceberg orders](https://okx.com/join/CASH20) |
| TWAP | Spreads one order across a time window at an average price | All users | None | No subscription fee; trading fees on fills | [Open TWAP](https://okx.com/join/CASH20) |

OKX's help center also documents recurring buy and arbitrage order bots in separate articles. The six above are the ones listed together on the current overview page, and the two futures entries are the ones with an access gate.

## Long, short or neutral: the choice that decides your starting position

The direction you pick changes what the bot holds the second it goes live, not just which way it profits.

A **long futures grid** opens long positions at market price on creation and takes profit by closing them as price rises. It suits a range you expect to hold with an upward bias.

A **short futures grid** holds only short positions and earns by closing them as price falls. It suits a range with a downward bias.

A **neutral futures grid** is the one people misread. OKX's product page describes it as holding both long and short positions, while the more detailed bot FAQ spells out the mechanics: the bot starts with no position at all, places buy orders below the current price and sell orders above it, and only opens a position when price touches a grid line. Once running, it can hold longs and shorts at the same time.

That flat start matters for two reasons. Your margin is not deployed the instant you press create, so the first hours can look like nothing is happening. And if price drifts in one direction without crossing back through the grid, you may end up with a one-sided position you did not explicitly choose.

## Every parameter, in the order you fill it in

**Lower limit and upper limit.** These bound the grid. When market price drops below the lower limit or rises above the upper limit, the bot stops placing new orders. It does not close your position and it does not shut itself down.

**Grid quantity.** The number of levels the range is divided into. OKX's example: a 100–400 range with 3 grids becomes 100–200, 200–300, 300–400.

**Arithmetic or geometric spacing.** Arithmetic keeps the same price difference between levels (1, 2, 3, 4). Geometric keeps the same ratio (1, 2, 4, 8), so the gaps widen the further you get from the starting price. The same grid count produces very different spacing under the two modes.

**Leverage.** Applied to the contracts traded inside the strategy, currently capped at **100x**.

**Invested margin.** The amount you commit to this strategy. Once the bot starts, that money is separated from your trading account and used only for the grid.

**Total amount.** Calculated as invested margin × leverage, so a 5,000 USDT margin at 2x shows as 10,000 USDT of grid capacity.

**Take-profit and stop-loss prices.** Optional. When triggered, the bot stops working and closes all positions with a market order.

**Open a position on creation.** Whether the bot takes its first position immediately at market price or waits for price to reach a grid line.

**Estimated liquidation price.** Calculated on the assumption that every long order (or every short order) in the grid fills, giving the maximum opening amount. It is a worst-case figure for a fully loaded grid, not the liquidation price of your position right now. If you read it as a live number, you will misjudge how much room you actually have.

### Running OKX's own example

Take a BTCUSDT perpetual long grid with a 50,000–100,000 USDT range, 50 grids, arithmetic spacing, 2x leverage, 5,000 USDT invested, first position opened on creation, and BTC trading at 60,100 USDT when the strategy activates.

Arithmetic spacing over a 50,000-wide range with 50 grids gives a level every 1,000 USDT: 50,000, 51,000, 52,000, up to 100,000. The bot places 50 buy orders across those levels. Because the market is already at 60,100, every buy order priced at 61,000 and above fills immediately, and each filled buy gets a sell order placed one level higher.

The end state: buy orders from 50,000 to 60,000, sell orders from 62,000 to 100,000.

From there it runs mechanically. Price drops below 60,000 and the buy there fills, with a sell order appearing at 61,000. Price rises above 62,000 and that sell fills, with a buy order appearing back at 61,000. Each completed round trip captures one grid's worth of price movement.

### Arithmetic versus geometric, with a second example

On BTC with a 25,000 lower bound, a 35,000 upper bound and 10 grids, arithmetic spacing puts each level 1,000 USDT apart. Switch to geometric and the levels near 25,000 sit close together while the levels near 35,000 spread out. If most of your expected price action happens near the current price, geometric concentration lower down can produce more fills there; arithmetic gives you the same spacing whether price is at the edge or the middle.

## What "out of range" really means for your money

When price breaks below the lower limit, the bot stops placing new orders. Your existing position stays open and keeps losing as price keeps falling. With leverage applied, that position can be liquidated. OKX's own risk note recommends setting a stop-loss price below the lower limit of the grid specifically to cover this scenario.

You have two responses once price leaves the range: terminate the strategy, which cancels every pending order, closes positions at current market price and returns the funds to your trading account, or wait for price to come back inside the range and let the grid resume placing orders.

The other exit is involuntary. If the underlying asset is suspended or delisted while the bot runs, OKX stops the bot automatically. 👉 [See the futures grid entry on OKX](https://okx.com/join/CASH20)

## Margin isolation and account-level risk

Money committed to a grid is isolated from your trading account and used only by that strategy. This cuts both ways. The grid cannot quietly drain the rest of your balance, but it also means the margin sitting in the bot is not available to support your other positions. After funding a grid, check what is left in the trading account against whatever else you are holding.

## Can your account use the futures grid?

Futures grid and futures DCA are listed as products for eligible users. Spot grid, spot DCA, iceberg and TWAP are open to all users. OKX points readers to a regional availability and eligibility page covering the EEA for the detail.

If you cannot find the futures grid entry under Trade → Trading bots, eligibility is the likely reason, and no amount of parameter tuning will change it. Confirm access before planning a strategy. There is no published country list attached to that eligibility page in the documentation, so the answer comes from your own account, not from a blog post.

## Four ways to fill in the parameters

- **Set myself.** You choose the range, grid count, spacing mode and leverage.
- **Set myself, auto fill.** The bot proposes parameters based on a backtested strategy, and you edit them.
- **AI strategy.** Parameters generated from a weekly backtest of the specific trading pair.
- **Lead bots.** Copy a running bot from a lead trader on the Marketplace in one click.

On the web: Trade → Trading bots → Marketplace → Futures grid. In the app: Trade → Trading bots → Futures grid, or Trade → select your pair → Trading bots → Grid bots → Futures grid → choose the contract.

The auto-fill and AI options save time but deserve a look before you confirm. A weekly backtest reflects the range the pair has been trading in, and a breakout the following week will push price straight through the limits the backtest chose.

## Managing a bot after it starts

Five controls sit in the bot dashboard: increase positions (raises the amount allocated per grid from that point forward), adjust margin (adds margin to reduce liquidation risk without changing the per-grid amount), stop (cancels pending orders, sells at market and returns funds to your trading account), details, and replicate parameters (rebuilds the same bot with identical settings).

Adjust margin is the one worth remembering. It is the only lever that changes your liquidation distance without changing the strategy's order sizes.

## Picking one: a short rule set

If you want exposure to range-bound price action without liquidation risk, use spot grid. The trade-off is that a position can sit underwater indefinitely instead of being closed out.

If you have a directional view and want the range to work in that direction, long or short grid, with the first position opened on creation so the bot is not idle waiting for a touch.

If you have no view but expect chop, neutral grid, accepting that it starts flat and may only fill on one side if price trends away from your entry.

And if you want to automate repeated buying rather than harvest a range, the DCA bots are the relevant products, not the grid.

Two things to confirm before you press create: that your account can access futures grid at all, and that your stop-loss sits below the lower limit rather than at it. Everything else is tuning. 👉 [Start from the OKX sign-up page and check what your account can open](https://okx.com/join/CASH20)
