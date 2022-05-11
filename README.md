# Godwoken Web3 API

A Web3 RPC compatible layer build upon Godwoken/Polyjuice.

## Development

### Config database

```bash
$ cat > ./packages/api-server/.env <<EOF
DATABASE_URL=postgres://username:password@localhost:5432/your_db
REDIS_URL=redis://user:password@localhost:6379 <redis url, optional, default to localhost on port 6379>

GODWOKEN_JSON_RPC=<godwoken rpc>
GODWOKEN_READONLY_JSON_RPC=<optional, default equals to GODWOKEN_JSON_RPC>

SENTRY_DNS=<sentry dns, optional>
SENTRY_ENVIRONMENT=<sentry environment, optional, default to `development`>,
NEW_RELIC_LICENSE_KEY=<new relic license key, optional>
NEW_RELIC_APP_NAME=<new relic app name, optional, default to 'Godwoken Web3'>

PG_POOL_MAX=<pg pool max count, optional, default to 20>
CLUSTER_COUNT=<cluster count, optional, default to num of cpus>
GAS_PRICE_CACHE_SECONDS=<seconds, optional, default to 0, and 0 means no cache>
EXTRA_ESTIMATE_GAS=<eth_estimateGas will add this number to result, optional, default to 0>
ENABLE_CACHE_ETH_CALL=<optional, enable eth_call cache, default to false>
LOG_LEVEL=<optional, allowed value: `debug` / `info` / `warn` / `error`, default to `debug` in development, and default to `info` in production>
LOG_FORMAT=<optional, allowed value: `json`>
MAX_SOCKETS=<optional, max number of httpAgent sockets per host for web3 connecting to godwoken, default to 10>
WEB3_LOG_REQUEST_BODY=<optional, boolean, if true, will log request method / body, default to false>
PORT=<optional, the api-server running port, default to 8024>
EOF

$ yarn

# For api-server & indexer
$ DATABASE_URL=<your database url> make migrate

# Only for test purpose
$ yarn workspace @godwoken-web3/api-server reset_database
```

rate limit config

```bash
$ cat > ./packages/api-server/rate-limit-config.json <<EOF
{
  "expired_time_milsec": 60000,
  "methods": {
    "poly_executeRawL2Transaction": 30,
    "<rpc method name>": <max requests number in expired_time>
  }
}
EOF
```

### Config Indexer

Sync block from godwoken.

```bash
$ cat > ./indexer-config.toml <<EOF
l2_sudt_type_script_hash=<l2 sudt validator script type hash>
polyjuice_type_script_hash=<godwoken polyjuice validator type hash>
rollup_type_hash=<godwoken rollup type hash>
eth_account_lock_hash=<eth account lock script hash>
godwoken_rpc_url=<godwoken rpc>
ws_rpc_url=<godwoken websocket rpc>
pg_url="postgres://username:password@localhost:5432/your_db"
chain_id=<godwoken chain_id in integer>
EOF
```

Or just run script, generate configs from `packages/api-server/.env` file and godwoken rpc `gw_get_node_info`.

```bash
node scripts/generate-indexer-config.js <websocket rpc url>
```

### Start Indexer

```bash
cargo build --release
./target/release/gw-web3-indexer
```

### Start API server

```bash
yarn run build:godwoken
yarn run start
```

#### Start in production mode

```bash
yarn run build && yarn run start:prod
```

#### Start via pm2

```bash
yarn run build && yarn run start:pm2
```

#### Start using docker image

```bash
docker run -d -it -v <YOUR .env FILE PATH>:/godwoken-web3/packages/api-server/.env  -w /godwoken-web3  --name godwoken-web3 nervos/godwoken-web3-prebuilds:<TAG> bash -c "yarn workspace @godwoken-web3/api-server start:pm2"
```

then you can monit web3 via pm2 inside docker container:

```bash
docker exec -it <CONTAINER NAME> /bin/bash
```
```
$ root@ec562fe2172b:/godwoken-web3# pm2 monit
```
Normal mode: http://your-url/

Eth wallet mode: http://your-url/eth-wallet (for wallet like metamask, please connect to this url)

WebSocket url: ws://your-url/ws

### Docker Prebuilds

local development:

```sh
make build-test-image # (tag: latest-test)
```

push to docker:

```sh
make build-push # needs login, will ask you for tag
```

resource:

- docker image: https://hub.docker.com/repository/docker/nervos/godwoken-web3-prebuilds
- code is located in `/godwoken-web3` with node_modules already installed and typescript compiled to js code.


### RPC Docs

Get RPC docs at [RPCs doc](docs/apis.md)
