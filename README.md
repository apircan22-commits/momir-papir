# Momir Papir

A single-file web app for playing **Momir Basic** — the Magic: The Gathering format where you summon random creatures — with a thermal receipt printer.

Pick a mana value, tap Summon, and a random creature from Magic's entire history gets dithered and printed on your 58mm thermal printer. You never see the card before it prints. Tear it off, put it on the battlefield, and keep playing.

## How It Works

1. **Open `momir-papir.html`** on your phone's browser
2. **Connect your thermal printer** via Bluetooth (tested on Denver 58mm)
3. **Print emblems** from the home screen — one per player as a rules reminder
4. **Tap Play**, pick a mana value (0–16), and hit **Summon**
5. The app fetches a random creature from Scryfall, dithers it to 1-bit black and white, and sends it to the printer
6. **Battlefield log** tracks everything you've summoned with peek and reprint buttons

## Features

- **Floyd-Steinberg dithering** tuned for thermal paper legibility (gamma 0.82, threshold 120)
- **Double-faced card support** — transform cards automatically detected, both faces printed as separate print jobs, peek overlay has a flip button
- **Reprint** any card from the battlefield log
- **Dark/light theme** with persistence
- **Fully offline-capable** after first load (only Scryfall API calls need network)
- **Zero dependencies** — single HTML file, no build step, no server

## Momir Basic Rules

Each player starts with **24 life** and a **60-card deck of basic lands** (12 of each type, or any mix).

Once per turn, as a sorcery: pay {X}, discard a card, and create a token copy of a random creature with mana value X.

That's it. No other spells. Just lands, random creatures, and combat.

## Technical Details

### Printing

- Card images are rendered at **503px wide** (63mm at 203dpi, standard MTG card width on 58mm paper with margins)
- Images are converted to grayscale, gamma-corrected, then dithered using Floyd-Steinberg error diffusion
- Print output uses `window.print()` with dynamically injected `@page` and `page-break-after` styles
- Each card is wrapped in a `.receipt` div with a dashed separator line
- `@page { margin: 0 0 50mm 0 }` ensures the thermal printer advances enough paper past the last card
- `page-break-after: always` on every receipt (including the last) forces the printer to feed through completely

### Double-Faced Cards

Scryfall returns DFCs with `image_uris: null` and two entries in `card_faces[]`, each with their own image URIs. The app detects this and:

- Prints each face as a **separate print job** (avoids thermal printer cutoff issues with multi-page continuous prints)
- Waits for the first print to complete via `afterprint` event before sending the second
- Shows a ↷ badge on DFC names throughout the UI
- Peek overlay includes a flip button with a rotation animation

Note: **Meld** cards (like Gisela + Bruna → Brisela) are *not* DFCs — they have normal `image_uris` at the top level and print as single-sided cards. This is correct behavior since you'd need both specific halves to meld.

### Dithering Parameters

| Parameter | Value | Effect |
|-----------|-------|--------|
| Gamma | 0.82 | Slight midtone lift for thermal paper |
| Brightness | +5 | Minimal boost |
| Threshold | 120 | Below center — biases toward black for readable text |

These were tuned specifically for the Denver 58mm thermal printer. If text is too faint, lower the threshold. If too dark/muddy, raise it.

### API

All card data comes from the [Scryfall API](https://scryfall.com/docs/api):

- `GET /cards/search?q=momir+vig+t:vanguard` — loads the Momir Vig emblem on startup
- `GET /cards/random?q=t:creature+mv={X}+-is:funny` — summons a random creature at the chosen mana value

The `-is:funny` filter excludes Un-set and joke cards.

## Thermal Printer Notes

Tested with a **Denver 58mm Bluetooth thermal printer** on Android. Key learnings:

- The printer's "copies" feature in the system print dialog **does not work** — it squishes all copies onto one page. Use the in-app copies stepper for emblems instead.
- CSS `padding` on print elements gets stripped by some thermal printer drivers. The reliable fix is `@page` margin combined with `page-break-after:always`.
- Separate print jobs for multi-card prints (DFCs) are more reliable than multi-receipt single jobs.
- `image-rendering: pixelated` on the receipt images prevents the browser from anti-aliasing the dithered output.

## Future Ideas

- **Pod mode** — host on a local server, players join on their phones, single shared print queue on one printer via WebSockets
- **Draft mode** — Scryfall-powered booster draft with on-screen picking and batch print of final deck

## License

Personal project. Card data and images are provided by [Scryfall](https://scryfall.com) under their terms of use. Magic: The Gathering is a trademark of Wizards of the Coast.
