# Starting and Stopping Ghostfolio

Ghostfolio runs in Docker on this PC. Open it at <http://localhost:3333> and log in with your security token.

## Start

1. Open **Docker Desktop** and wait until it says it is running.
2. Go to <http://localhost:3333>. The containers normally come back on their own, so it may already be up.
3. If the page does not load, open a terminal in `D:\Documents\GitHub\ghostfolio` and run:

   ```powershell
   docker compose -f docker/docker-compose.yml up -d
   ```

4. Wait about 30 seconds, then reload the page.

To skip step 1 after a reboot, turn on **Start Docker Desktop when you sign in** in Docker Desktop's settings.

## Stop

Run this from `D:\Documents\GitHub\ghostfolio`:

```powershell
docker compose -f docker/docker-compose.yml stop
```

Your data is kept. After a manual stop, Ghostfolio stays off, even after a reboot, until you run the `up -d` command again.

> **Warning:** Do not add `-v` to a `docker compose down` command. It deletes your portfolio database.

## Stock prices

Ghostfolio fetches stock prices about once an hour, and only while it is running. If it has been off, prices catch up at the next update.

### PSE price history

You don't need to do anything to update PSE prices. A fourth container, `gf-pse-sync`, starts and stops with Ghostfolio. Each time it starts, it downloads all closing prices since November 1, 2025 from PSE EDGE, plus the last 12 months of the PSEi index from EOD Historical Data, and updates them in Ghostfolio. This takes about a minute; reload the page to see the new prices.

- If you leave Ghostfolio running, it updates again each day at 6:30 PM Manila time.
- Days when Ghostfolio was off are filled in by the next update.
- A ticker shows `FAIL` if its prices don't reach back to the first week of November 2025. Its existing prices stay as they are.
- If an update fails, it tries again an hour later.

See what the last update did. Each ticker's line starts with `OK` or `FAIL`:

```powershell
docker logs --tail 25 gf-pse-sync
```

To update prices right away:

```powershell
docker restart gf-pse-sync
```

To add a ticker, add it to the list at the top of `docker/pse-sync.sh`, then run the restart command above. Its name must match the asset's name in Ghostfolio.

## If something is wrong

Run these from `D:\Documents\GitHub\ghostfolio`.

Check that all four containers (`ghostfolio`, `gf-postgres`, `gf-redis`, `gf-pse-sync`) are running:

```powershell
docker compose -f docker/docker-compose.yml ps
```

See the most recent app logs:

```powershell
docker logs --tail 50 ghostfolio
```

If you see an error about connecting to Docker, Docker Desktop is not running yet. Start it and try again.

## Market benchmarks (added November 22, 2026)

Besides your PSE holdings, Ghostfolio tracks these market symbols. Ghostfolio updates them on its own; you don't need to do anything.

| Symbol | What it is | Benchmark? |
|---|---|---|
| `GC=F` | Gold futures (USD) | Yes |
| `^GSPC` | S&P 500 | Yes |
| `GF_PSEI` (named PSEi) | PSEi index, filled by `gf-pse-sync` from EOD Historical Data (Yahoo has no PSEi history) | Yes |
| `DX-Y.NYB` | US dollar index | No, price history only |
| `^TNX` | US 10-year Treasury yield | No, price history only |
| `^VIX` | Volatility (fear) index | No, price history only |

Benchmarks appear under **Markets** and can be picked on the **Analysis** page to compare your portfolio against them. To change them, go to **Admin Control → Market Data**, open the symbol, and use its **Benchmark** setting.

### Buying-conditions check

Ask Claude "is the market good for buying?" or run `/market-check`. It uses these prices (read-only) to suggest how much to buy this month and which holdings to favour. A one-line summary also appears in Monday's morning brief.
