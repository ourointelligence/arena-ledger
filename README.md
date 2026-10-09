# OURO Arena ledger

Every hour, each lane of OURO Arena (https://ourosi.xyz/arena) exports its full state and the hash of that export is chained to the previous one. Every day the hour files and the chain are committed here, so anyone can check that the numbers shown on the page were not edited later.

## Layout

```
core/2026-10-10T14.json   one lane export per hour, canonical JSON (sorted keys, no whitespace)
core/chain.jsonl          one line per hour: { "ts", "sha256", "prev_sha256", "file" }
alts/...                  same for the other lanes
flow/...
```

The export is the SDK's `schemaVersion: 1` document (see https://github.com/ourointelligence/ouro/blob/main/docs/export-schema.md): the goal, every strategy ever generated with its code, every cycle with its diagnosis and rejections, and the takeoff curve.

## How to recompute the hashes

For each line of `chain.jsonl`, in order: the SHA-256 of the bytes of the named file must equal `sha256`, and `prev_sha256` must equal the `sha256` of the line before it (null on the first line). Any edit to a file, a reorder or a removed hour breaks the chain from that point on.

```bash
lane=core
prev=null
while IFS= read -r line; do
  file=$(printf '%s' "$line" | python3 -c 'import json,sys; print(json.load(sys.stdin)["file"])')
  want=$(printf '%s' "$line" | python3 -c 'import json,sys; print(json.load(sys.stdin)["sha256"])')
  wantprev=$(printf '%s' "$line" | python3 -c 'import json,sys; print(json.load(sys.stdin)["prev_sha256"])')
  have=$(sha256sum "$lane/$file" | cut -d" " -f1)
  [ "$have" = "$want" ] || echo "BAD HASH $file"
  [ "$wantprev" = "$prev" ] || echo "BAD LINK $file"
  prev=$want
done < "$lane/chain.jsonl" && echo "checked $lane"
```

Or, with the Arena code: `arena verify-ledger core` on a clone of this repository.

Paper trading only. Nothing here is financial advice or a trading signal.
