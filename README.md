# UTT Subgraph

Indexes events from the UTT (UTU Trust Token) smart contract on Ethereum mainnet and Sepolia testnet, using The Graph Protocol (Subgraph Studio).

## Contracts

| Network | Address |
|---|---|
| Ethereum Mainnet | `0xE62dc4c82a9749Bf2E40F73138B3CFee0a2EC89F` |
| Sepolia Testnet | `0x537BE61c5EFB865Df53CA55eeA07ceEe5d5fB162` |

## Subgraph Studio Endpoints

| Environment | Subgraph Name | Query URL |
|---|---|---|
| Testnet | `utu-protocol-testnet` | `https://api.studio.thegraph.com/query/29422/utu-protocol-testnet/version/latest` |
| Mainnet | `utu-protocol-mainnet` | `https://api.studio.thegraph.com/query/29422/utu-protocol-mainnet/version/latest` |

## Prerequisites

- Node.js and Yarn installed
- Access to the Subgraph Studio account (https://thegraph.com/studio/)
- The deploy key from Subgraph Studio (account Settings page)

## Setup

```bash
yarn install
```

Create a `.env` file in the project root (never commit this):

```
GRAPH_DEPLOY_KEY=your-deploy-key-here
```

Get the deploy key from https://thegraph.com/studio/ under your account Settings. The same key works for both testnet and mainnet subgraphs.

Load the env before deploying:

```bash
export $(cat .env | xargs)
```

## Generate Types

Run this after any changes to `schema.graphql` or `abis/UTT.json`:

```bash
yarn codegen
```

## Build

```bash
yarn build
```

## Deploy

### Testnet (Sepolia)

Uses `subgraph-testnet.yaml`:

```bash
yarn deploy
```

### Mainnet

Uses `subgraph-mainnet.yaml`:

```bash
yarn deploy-production
```

The version label is taken automatically from the `version` field in `package.json` — no manual prompt. Before deploying, bump the version in `package.json`:

```bash
yarn version --new-version 0.0.3   # or: edit package.json manually
```

If you deploy without bumping the version, the Studio will reject it if that version label already exists. Treat it the same as `npm publish` — always increment before deploying.

### Local

```bash
yarn create-local    # create the subgraph on local node
yarn deploy-local    # deploy to local node
yarn remove-local    # remove from local node
```

## After Deploying

The `version/latest` endpoint automatically points to the most recently deployed version, so the frontend does not need a URL change on each deploy.

If prod is still not reflecting after a successful deployment, the issue is likely the environment variable in the hosting platform (not the `.env_prod` file in the frontend repo). Check the hosting dashboard and ensure `VITE_GRAPHQL_ENDPOINT` matches the Studio URL above.

## Troubleshooting

### Event signature mismatch

```
Event with signature 'X' not present in ABI 'UTT'
```

The ABI was updated but the event signatures in the yaml files were not. Fix:

1. Open `abis/UTT.json` and find the updated event definition
2. Update the matching `event:` line in `subgraph-testnet.yaml`, `subgraph-mainnet.yaml`, and `subgraph.yaml`
3. Update `schema.graphql` if new fields were added to the event
4. Update the handler in `src/utt.ts` to store any new fields
5. Run `yarn codegen` then redeploy

### Deploy key not set

```
You must provide a deploy key
```

Run `yarn graph auth --studio <deploy-key>` again (see Setup above).

### Deployment exists in Studio but prod frontend shows errors

The hosting platform's environment variables may be stale. Log into the hosting dashboard, update `VITE_GRAPHQL_ENDPOINT` to the correct Studio URL, and trigger a redeploy.

### "Failed to fetch" on prod

The prod frontend is hitting a dead or wrong URL. Check the `VITE_GRAPHQL_ENDPOINT` environment variable in the hosting platform dashboard — it should point to the Studio URL, not the old hosted-service URL (`api.thegraph.com/subgraphs/name/...` is deprecated and shut down).

## Publishing to The Graph Network (optional)

Publishing makes the subgraph available on the decentralized Graph Network (indexable by multiple indexers). This is separate from deploying to Studio.

- Requires ETH on **Arbitrum One** (not ARB token, not ETH on mainnet) to pay gas
- Go to https://thegraph.com/studio/, open the subgraph, and click Publish
- Bridge ETH to Arbitrum One at https://bridge.arbitrum.io if needed
- A few dollars of ETH on Arbitrum is sufficient for the publish transaction

Deploying to Studio is sufficient for most use cases and does not require GRT or Arbitrum ETH.

## Files

| File | Purpose |
|---|---|
| `subgraph.yaml` | Default manifest (points to Sepolia) |
| `subgraph-testnet.yaml` | Testnet-specific manifest |
| `subgraph-mainnet.yaml` | Mainnet-specific manifest |
| `schema.graphql` | GraphQL entity definitions |
| `abis/UTT.json` | UTT contract ABI |
| `src/utt.ts` | Event handler mappings |
