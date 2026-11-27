# Outfit Decider: problems to fix

We fix these one at a time. Tick each box when it's done and checked on the phone.

| # | Problem | Status |
|---|---------|--------|
| 1 | Can't tell which item the app means (several "black shirts") | ⬜ |
| 2 | Back gesture/button closes the installed app | ✅ Done (check on phone) |
| 3 | No dark mode | ✅ Done (check on phone) |
| 4 | People dress for where they're going, not the weather where they are | ⬜ |
| 5 | Colour-tone-wise recommendations (friend's idea) | ⬜ |
| 6 | "Start over" should become "Reroll"; no separate Reroll button | ✅ Done (check on phone) |
| 7 | No undo after an accidental reroll | ✅ Done (check on phone) |
| 8 | "No more combinations" should be a clear pop-up in the middle of the screen | ✅ Done (check on phone) |
| 9 | Fit and silhouette make pairings look wrong even when colours work | ⬜ |
| 10 | Colour theory needs more rules and more accuracy | ⬜ |

---

### 1. Can't tell which item the app means
**Problem:** With several items of the same colour and type (three black shirts), a name and swatch aren't enough to know which piece the app is recommending.
**Fixed looks like:** Every item is instantly recognisable. Most likely this means a photo per item (taken with the phone camera, stored on the device). Clearer names and details (fabric, brand, pattern) could help as well.

### 2. Back gesture closes the app
**Problem:** In the installed app, Android's back gesture or button exits the app instead of going back a step.
**Fixed looks like:** Back closes an open sheet or pop-up first, then returns to the previous tab. It only exits from the starting screen.

### 3. Dark mode
**Problem:** The app has only a light theme.
**Fixed looks like:** The app follows the phone's dark/light setting automatically, optionally with a manual switch, and every screen, sheet and swatch stays readable.

### 4. Dress for the destination, not the current weather
**Problem:** The weather chips assume you dress for where you are now. People actually dress for where they're going and its temperature (a cold office, an air-conditioned mall, an outdoor event).
**Fixed looks like:** Decide asks about the place or plan (or its temperature) rather than the local weather, and the engine uses that.

### 5. Colour-tone-wise recommendations
**Problem:** There's no way to ask for an outfit in a particular tone or mood.
**Fixed looks like:** You can choose a tone (for example monochrome, earthy, pastel, dark, bright, warm or cool) and get outfits that match it. This needs clarifying with you before building.

### 6. Start over → Reroll
**Problem:** "Start over" and "Reroll" overlap and clutter the screen.
**Fixed looks like:** After the first outfit, one button gets the next outfit. There's no separate Reroll.

### 7. Undo an accidental reroll
**Problem:** A mis-tap on reroll loses an outfit you liked.
**Fixed looks like:** A Back/Undo control returns to the previous outfit, several steps back if needed.

### 8. "No more combinations" pop-up
**Problem:** When every combination has been shown, the message is easy to miss and doesn't explain why.
**Fixed looks like:** A clear dialog in the centre of the screen says you've seen every good combination, explains why (closet size, items in the wash, weather or occasion filters) and offers next steps.

### 9. Fit and silhouette clash
**Problem:** Colours pair well, but the combined shape or vibe looks wrong (for example a slim top with baggy pants, or a formal shirt with athletic shoes).
**Fixed looks like:** Items record fit (slim, regular, relaxed or oversized) and style (formal, smart-casual, casual, sporty or street), and the engine scores how well the silhouette and style match, not just the colours. The existing silhouette logic needs auditing first.

### 10. Tighter colour theory
**Problem:** Some colour judgements are inaccurate or too lenient.
**Fixed looks like:** More rules (undertone matching, value contrast, how many accents, pattern and colour, and so on), checked against real outfits you mark as wrong. Collecting examples of bad picks will guide this.
