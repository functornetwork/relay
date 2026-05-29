# Deploy the BNB-testnet relay to Railway

Uses `Dockerfile.railway` (same Rust build as `Dockerfile`, plus it bakes in
`relay.yaml` and binds `0.0.0.0:$PORT`).

1. **Before deploying**, set `relay.yaml`'s chain-97 `endpoint:` to a reliable
   (ideally paid) BSC-testnet RPC — the public one can be flaky for a relay.
   Contract addresses (orchestrator/proxy/simulator/funder/escrow) are already
   in `relay.yaml`.
2. **New service** → Deploy from the `ithaca-relay` GitHub repo.
3. **Settings → Build → Dockerfile Path** = `Dockerfile.railway`.
   (Rust build takes ~10–20 min on the first deploy.)
4. **Add Postgres** (Railway → New → Database → PostgreSQL) for production
   persistence.
5. **Variables** (the relay reads these from env):

   | Variable | Value |
   | --- | --- |
   | `RELAY_MNEMONIC` | signer mnemonic (the relay derives signer EOAs from it) |
   | `RELAY_FUNDER_SIGNER_KEY` | funder signing key (sponsors user operations) |
   | `RELAY_DB_URL` | `${{Postgres.DATABASE_URL}}` |

   `PORT` is injected by Railway and passed to `--http.port`.
6. **Deploy**, then **Networking → Generate Domain** → your relay URL, e.g.
   `https://functor-relay.up.railway.app`. Health: `POST /` with
   `{"jsonrpc":"2.0","id":1,"method":"health","params":[]}`.
7. **Fund on BNB testnet** (ongoing):
   - the relay's signer EOAs (derived from `RELAY_MNEMONIC`) — they pay gas
   - the SimpleFunder contract `0xb248602EAadd9c3e2Db4575C4e4d58003b7a2740` —
     sponsors user ops
   Metrics (incl. balances) are exposed on the metrics port.
8. **Wire the apps** to this URL — see the SDK note below (override
   `BNB_TESTNET.portoRelayUrl`).
