# MoodNFT

[![CI](https://github.com/agent-mino/MoodNFT/actions/workflows/test.yml/badge.svg)](https://github.com/agent-mino/MoodNFT/actions/workflows/test.yml)

Two ERC-721 contracts that explore on-chain vs. off-chain NFT metadata. **MoodNft** stores its artwork entirely on-chain as base64-encoded SVG — no IPFS, no external dependency. **BasicNft** uses IPFS-hosted metadata as a reference counterpart.

## Contracts

### MoodNft

An NFT whose image lives on-chain and toggles between two moods.

- **Mint** — anyone can mint; new tokens start `HAPPY`
- **Flip** — `flipMood(tokenId)` switches between `HAPPY` and `SAD`; only the token owner or an approved operator can call it
- **Token URI** — fully generated on-chain: the contract encodes the SVG into a `data:image/svg+xml;base64,…` URI and wraps it in a `data:application/json;base64,…` metadata blob, so the NFT renders in any ERC-721-aware wallet with no external requests

```
mintNft() → token #N (HAPPY)
flipMood(N) → HAPPY ↔ SAD   (owner/approved only)
tokenURI(N) → data:application/json;base64,<SVG embedded>
```

### BasicNft

A minimal ERC-721 where each token's URI is supplied at mint time, pointing to an IPFS-hosted metadata file. Used as a reference implementation alongside MoodNft.

## How on-chain metadata works

```
Deploy script reads happy.svg / sad.svg from disk
  └─▶ base64-encodes each → data:image/svg+xml;base64,…
        └─▶ stored as constructor arguments in the contract

tokenURI() at query time:
  └─▶ picks the image URI for the current mood
        └─▶ encodes JSON { name, description, attributes, image }
              └─▶ wraps in data:application/json;base64,…
                    └─▶ returned to the caller (wallet, marketplace)
```

No IPFS pinning service, no metadata server, no CDN. The token survives as long as the chain does.

## Tech stack

Solidity 0.8.18 · OpenZeppelin ERC-721 · Foundry (Forge + Anvil)

## Run locally

Requires [Foundry](https://book.getfoundry.sh/getting-started/installation).

```bash
git clone --recurse-submodules https://github.com/agent-mino/MoodNFT.git
cd MoodNFT
forge build
forge test
```

## Deploy

```bash
# Local Anvil node
anvil &
forge script script/DeployMoodNft.s.sol --broadcast --rpc-url http://127.0.0.1:8545 --private-key <ANVIL_KEY>

# Sepolia testnet
forge script script/DeployMoodNft.s.sol --broadcast --rpc-url $SEPOLIA_RPC_URL --private-key $PRIVATE_KEY
```

## CI

Every push runs `forge fmt --check`, `forge build --sizes`, and `forge test -vvv`.
