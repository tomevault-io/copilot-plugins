## inference-angels

> You are being asked to claim an Inference Angel. This file tells you everything you need. Read it fully before you start.

# AGENTS.md · for an agent working this mine

You are being asked to claim an Inference Angel. This file tells you everything you need. Read it fully before you start.

**Live on Robinhood Chain.** Contract `0xD982d8F175BD50B976F4Ad90562c9e38200091f9`, chain id 4663. `corpus/v3/deploy.json` holds the addresses.

## What this is

7,777 angels on Robinhood Chain. Each is locked behind one to four puzzles, called trials. Solve them all and you may claim that angel. It is free apart from gas.

**There is no model to download and no setup to perform.** Every answer is a fact of arithmetic, checked by SHA-256. Nothing you write is compared against anything a model produced. Use whatever you already are.

## What you must be able to do

- Run code you write yourself. Most trials need a short script.
- Compute SHA-256 and Keccak-256.
- Decrypt AES-256-CBC.
- Normalise Unicode to NFC and NFD, and know that these differ.
- Handle Base64, Base36, hex, and Roman numerals.
- Search a 6,000 word list.
- Read a 32 by 32 grid of colours out of an SVG.

## Setup

```
npm install
```

That is all. There is nothing else.

## The shape of a puzzle

Read `corpus/v3/public/band-NN.json`. An angel's band is `ceil(tokenId / 101)`.

```json
{
  "tokenId": 7777,
  "band": 77,
  "trials": [
    { "n": 1, "kind": "cipher", "statement": "..." },
    { "n": 2, "kind": "lattice", "sealed": "<base64>", "hint": "..." },
    { "n": 3, "kind": "hidden",  "sealed": "<base64>", "hint": "..." },
    { "n": 4, "kind": "hunt",    "sealed": "<base64>", "hint": "..." }
  ],
  "finalStep": "..."
}
```

Only trial 1 is readable. The rest are sealed.

## Opening a sealed trial

Given your answer to trial `n`, open trial `n+1`:

```js
import { createHash, createDecipheriv } from 'node:crypto';
const sha = s => createHash('sha256').update(s, 'utf8').digest('hex');

function unseal(sealedB64, previousAnswer, tokenId, trialIndex) {
  const key = Buffer.from(sha(previousAnswer), 'hex');
  const iv  = Buffer.from(sha(`${tokenId}|iv|${trialIndex}`), 'hex').subarray(0, 16);
  const d   = createDecipheriv('aes-256-cbc', key, iv);
  return Buffer.concat([d.update(Buffer.from(sealedB64, 'base64')), d.final()]).toString('utf8');
}
```

`trialIndex` counts from 1 for the second trial in the list, matching the index in the `trials` array.

**Always check the result reads as English.** A wrong key usually throws on padding, but roughly one time in 256 it returns nonsense and no error. Do not treat "it decrypted" as "it worked."

## The nine kinds, and how to approach each

| Kind | Approach |
|---|---|
| **chain** | Follow the steps exactly, in order. Watch Unicode normalisation, and watch which steps say NFC and which say NFD. Some puzzles have two or three parts that are worked separately and joined with a vertical bar. |
| **rule** | Six examples of one hidden transformation. Try common ones and compositions of them: Atbash, ROT13, reverse, keep every other character, Base64, keep only letters. Test your guess against all six before you trust it. |
| **hunt** | Hash every word in the published list, or every pair, and compare the leading hex. A one-word hunt is 6,000 tries. Three words is about 2×10¹¹, so write it in a fast loop and be patient. |
| **cipher** | Vigenère, key length given. The plaintext is ordinary English built from a small vocabulary, so frequency analysis works, and so does brute force over the key space. |
| **hidden** | One word sits at a fixed spacing. Try every spacing from 2 to 12 and every starting offset, and check the result against the published word list. More than one reading may look valid, so confirm with the hash. |
| **construct** | You are given an acrostic, the word lengths, and a hash prefix. Filter the word list by first letter and length for each slot, then walk the combinations and test the hash. |
| **relic** | Fetch the named angel's picture and read the squares it names. The picture is a 32 by 32 grid drawn as SVG rectangles at 16 pixels each. Note that a rectangle can cover several squares, so expand runs. Rows count from 0 at the top, columns from 0 at the left. |
| **ottendorf** | Pairs of numbers. The first picks a word in the published list, counting from 1. The second picks a letter inside that word, counting from 1. Read the letters in order. |
| **lattice** | Small enough to brute force. Generate all arrangements, filter by each stated condition, and exactly one survives. |

## The turn some trials take

Several kinds end with a line saying that what you found is not the answer, it is the key. When you see that:

1. Take SHA-256 of what you worked out, exactly as the trial described it.
2. Use those bytes as an AES-256-CBC key.
3. The starting block is the first 16 bytes of the SHA-256 of `<tokenId>|key|<trialIndex>`.
4. What comes out is four ordinary words from the published list, separated by single spaces. Those four words are the trial's answer.

This exists because a grid with 120 arrangements, or a word from a 6,000 word list, is a set small enough to guess without reading the puzzle. The sealed phrase makes the set about 10¹⁵ instead.

## The final answer

Join the angel's number and every trial answer in order with single vertical bars and nothing else:

```
7777|first answer|second answer|third answer|fourth answer
```

SHA-256 that, in lower-case hex. That is what you submit. Check it against `corpus/v3/seeding/band-NN-hashes.json` before you spend gas.

## Claiming

```
export RPC_URL=https://rpc.mainnet.chain.robinhood.com
export PRIVATE_KEY=<a throwaway wallet key>
node miner/mine.js solve <tokenId>
```

Claiming happens in two steps, a sealed submission and then an opening, so nobody can take your answer while it waits. `mine.js` does both for you. Do not do them by hand.

Between the two steps it waits 60 blocks. The chain makes about ten blocks a second, so that is about six seconds. You will see this:

```
  sealed in block 35712877. Waiting for block 35712937... reached.
  asking the chain to accept it... accepted.
  opening it and claiming...
```

Sometimes it says the commitment is not old enough yet, and waits longer. **That is normal. It is not a failure and you must not treat it as one.** The address you connect to is a pool of computers and they do not all hold the same number of blocks yet. The tool asks the chain, for free, until the chain says yes. Let it finish. It waits up to twenty minutes and then tells you what to do next.

## Rules for you specifically

- **Never invent an answer.** If you cannot solve a trial, say so. A wrong submission wastes gas and claims nothing.
- **Watch exact text.** Answers are compared byte for byte. Trailing spaces, capital letters, and smart quotes all break the hash.
- **Do not parallelise the trials.** Trial 3 cannot be read until trial 2 is answered. You can parallelise a search inside one trial.
- **Verify before spending.** Always check your final hash against the seeding file first.
- **Ask before spending gas** unless the person you are working for has already told you to go ahead.
- **Use `miner/mine.js` as it is. Do not rewrite it.** People have rewritten it, kept its older waiting logic, and rebuilt a bug that is already fixed here. If it looks wrong to you, pull the current version first. If it is still wrong, report it. Do not reimplement the claiming steps yourself.

## If you are stuck

Work out which of these is true, in this order.

1. The sealed trial will not open. Your previous answer is wrong, or has stray whitespace.
2. It opened but reads as nonsense. Same thing. The padding passed by luck.
3. Everything opened but the final hash is wrong. One trial answer is wrong, or you joined them with the wrong separator.
4. The chain says the commitment is not old enough, or that it has not seen your commitment. Nothing is wrong. A computer in the pool is behind the others. Wait. The tool keeps asking for you. Do not restart it and do not edit it.
5. It waited twenty minutes and gave up. That endpoint is a long way behind. Your sealed answer stays good for about fourteen hours, so run the same `solve` command again later, or set `RPC_URL` to another node.
6. All of it checks out and the chain rejects it for some other reason. Re-download the corpus and run `node scripts/verify-corpus.js`.

*Ad maiorem Dei gloriam.*

---
> Source: [jacklarmer/inference-angels](https://github.com/jacklarmer/inference-angels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
