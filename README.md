# Canvas — a collaborative whiteboard

A real-time collaborative whiteboard that runs in a single HTML file. **No build step, no
backend, no accounts.** Open it, draw, and share.

**Live:** https://thanushanyogavel-spec.github.io/whiteboard/

## Two ways to collaborate

**Same browser — instant.** Open the page in a second tab. Edits appear in both immediately.
This uses `BroadcastChannel`, which needs no server at all.

**Different device — peer to peer.** Click **Invite → Host**, send the generated code to the
other person, have them click **Join** and paste it, then paste their reply back. The two
browsers then talk directly over a WebRTC data channel.

That second flow is deliberately manual: WebRTC normally needs a signalling server to
exchange connection details, and this app has none. Copying two strings is the price of
having zero infrastructure.

## Features

- Infinite canvas with pan and zoom (wheel to pan, Ctrl+wheel or pinch to zoom)
- Pen, rectangle, ellipse, line, arrow, text, eraser, and a selection tool
- 8 colours, 4 stroke widths
- Undo / redo with `Ctrl+Z` and `Ctrl+Shift+Z`
- Live remote cursors with names
- Presence — see who else is on the board
- Export the current view as PNG
- Keyboard shortcuts: `P` pen, `V` select, `R` rect, `O` ellipse, `L` line, `A` arrow,
  `T` text, `E` eraser, `Space` to pan

## How the sync works

Every element carries an `id`, an `updatedAt` timestamp and an `author` id. When two peers
disagree about an element, the one with the **higher timestamp wins**; ties break on the
author id so both sides pick the same winner. That is last-writer-wins, and it is what makes
the two sides converge without a coordinator.

Deletes are tombstones — the element stays in the map with `deleted: true`, so a late
arriving update cannot resurrect it. Rendering skips tombstones.

Operations are broadcast, never state. A peer that joins mid-session sends `hello`, an
existing peer replies with a full snapshot, and the normal LWW merge settles everything.

## Limitations, honestly

- **BroadcastChannel is same-browser only.** Two tabs, same machine. Not two people on two
  laptops.
- **WebRTC needs both people online at once** and the manual code exchange. There is no
  persistent room, no history, no reconnect.
- **No persistence.** Refresh and the board is empty. Export a PNG if you want to keep it.
- **No auth.** Anyone who completes the handshake is trusted with full edit rights.
- **LWW loses concurrent edits** to the same element — if two people drag the same shape at
  once, one edit disappears. A proper CRDT would keep both.

A production version of this needs a signalling server, persistence, and per-element CRDTs
rather than last-writer-wins.

## Running it

Just open `index.html`. Or serve the folder:

```bash
python3 -m http.server 8000
```

## Tests

12 automated checks run in a headless browser with two live pages: a stroke drawn on one
appears on the other, a rectangle drawn on the second appears on the first, peer presence
registers, undo removes the element, and both pages run with zero JavaScript errors.

## Licence

MIT
