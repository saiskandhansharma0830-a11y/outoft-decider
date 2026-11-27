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


---

## Possible solutions (brainstorm)

Motto: unbelievably simple, yet effective and powerful; innovative and creative. The app should feel like it knows you and talks to you. ⭐ = recommended first.

### 0. Cross-cutting: "The app that knows you" (a style profile and a voice)
- ⭐ **A 60-second style quiz on first run**, played as swipeable image-free cards: "Quiet or loud?", "Fitted or relaxed?", "Monochrome or colourful?", "Comfort or sharp?", "Do you like standing out?" The answers build a **Style DNA**: about 5 sliders such as Bold↔Calm, Fitted↔Relaxed, Classic↔Trendy, Warm↔Cool palette, and Comfort↔Polish. It's shown as a small card on My Data, and you can drag any slider to correct it.
- ⭐ **It learns quietly from what you already do.** Good/Bad, "I wore this", rerolls (a reroll is a soft "no") and Previous (going back means "I liked that") all nudge the Style DNA. This is not AI training, just transparent weights you can see and reset. It also turns the existing feedback dataset into immediate value.
- ⭐ **A voice instead of a score sheet.** Each outfit gets one short, personal line in place of bullet reasons, for example: "Navy and beige, calm and sharp, like you like it on office days." "You've worn your black jeans 3 times this week; giving them a rest." "Rain at your destination: dark bottoms won't show splashes." Short templates are filled from your Style DNA, history and context, so there's no AI model and it works offline.
- **Morning greeting** on the Decide screen: "Morning. Office again? Here's your strongest look." It remembers your usual occasion per weekday and pre-selects it.
- **Wardrobe insights** on My Data: "You never wear your red tee. Want me to suggest it more?", "Your most-loved combo: white tee + black jeans (5 Goods)", "You lean cool-toned 80% of the time."
- **Streaks and variety:** it gently avoids repeating yesterday's outfit, and says so.

### 1. Recognising which item is which
- ⭐ **Item photos.** Tap the swatch and take a picture or choose one from the gallery. The app compresses it to about 30 KB and stores it on the device (IndexedDB, not localStorage, so storage space isn't an issue). The outfit card then shows the real clothes.
- ⭐ **Auto-colour from the photo.** It samples the centre of the photo and suggests the matching colour preset ("Looks like Navy, tap to confirm"). That fixes colour accuracy at the same time.
- **Auto-names that disambiguate:** when two items share a type and colour, the app asks for one distinguishing word ("Black shirt, *linen*" / "Black shirt, *with logo*"), offering chips such as plain, logo, striped, linen, oversized and fav.
- **Little details as chips:** collar, sleeve length, pattern and brand appear as tiny tags under the name.
- **"Which one is this?" mode:** a grid of your closet with photos. Tap and hold to see the item large.
- Simplest version: distinguishing word + chips (no photos) as a 1-hour fix; photos next.

### 4. Dressing for the destination, not the weather here
- ⭐ **"Where are you heading?" replaces "Weather".** Quick chips: Office (AC), College, Outdoors, Mall/Cinema (cold AC), Gym, Night out, Travel, plus your own saved places. Each place stores a **feel** (cold/cool/mild/warm/hot) and an occasion, so one tap sets both.
- ⭐ **It learns your places:** after a few uses it pre-selects by time and day ("Monday 8 am → Office").
- **Layering logic for split days:** "Cold office, hot commute" asks for a removable hoodie layer, and the line explains it ("Hoodie for the AC, take it off outside").
- **Optional live weather at the destination** (free Open-Meteo, no key): type or pick a place and it reads that place's temperature. This is an optional extra; the app still works offline.
- **"Plans today" multi-stop:** College → Gym → Dinner finds one outfit that survives the whole day (scores the worst-case stop).
- Simplest version: rename Weather to "Where to?" with place chips that each map to a temperature feel.

### 5. Colour-tone-wise recommendations
- ⭐ **Mood chips on Decide (optional, one tap):** Calm (neutrals), Earthy (browns, olives, beige), Dark (black, navy, charcoal), Fresh (whites, light blues), Bold (one strong accent), Tonal (one colour family in shades), Monochrome. Picking one filters and boosts outfits whose palette matches. With nothing picked, behaviour is as today.
- ⭐ **Mood from your Style DNA:** if you lean calm, Calm is the default; it says so ("Going calm today, as usual").
- **"Surprise me" tone:** once a week it suggests a tone you never use but your closet supports ("You've never tried earthy; you own all of it").
- **Palette preview:** a strip of the outfit's colours named as a tone ("Earthy · warm").
- **Seasonal palettes:** a light suggestion of autumn/winter/spring/summer palettes by month.
- Simplest version: 5 tone chips that act as a filter plus a bonus to the existing score.

### 9. Fit and silhouette
- ⭐ **Two tiny fields per item:** **Fit** (Slim / Regular / Relaxed / Oversized) and **Vibe** (Smart / Casual / Sporty / Street). Each is a single tap when adding an item, and existing items get asked once ("Quick: how does this fit?").
- ⭐ **Silhouette rules** the engine adds:
  - Balance: oversized top + slim bottom ✓, slim top + wide bottom ✓, oversized + oversized ✗ unless your Style DNA is Relaxed/Street.
  - Vibe clash: smart shirt + sporty shoes ✗, street hoodie + formal pants ✗. A "vibe distance" penalty applies.
  - Tuck logic: shirt + slim pants → "tuck it in" tip.
- **Learns your bad pairs:** when you tap Bad, ask one optional question, "What was off? Colours / Fit / Vibe / Just not me", and use it to weight the rules. This also makes the dataset richer.
- **Body-shape-free:** no body questions, just how the clothes themselves fit.
- Simplest version: Fit + Vibe fields and two penalty rules (vibe clash, double-oversized).

### 10. Tighter colour theory
- ⭐ **A "why was this bad?" chip on Bad** (shared with #9) collects real failure cases; the rules get tuned against your own Bad list.
- ⭐ **New rules:**
  - Undertone match: warm-undertone neutrals (cream, camel) with warm colours; cool (pure white, grey) with cool colours.
  - The 60-30-10 proportion: the biggest area (bottom or hoodie) should be the calmest colour.
  - At most one accent; two loud colours only if they're complementary and at different lightness.
  - Value-contrast tiers: low contrast reads relaxed, high contrast reads sharp, matched to occasion and mood.
  - Shoe anchoring: shoes echo the top or bottom, or stay neutral.
  - Denim rules: Canadian tuxedo only if the washes differ clearly.
- **Self-test page (hidden):** a table of 30 reference outfits with expected tiers that is re-run after every rule change, so nothing silently regresses.
- **Confidence line:** "I'm 90% sure about this one" vs "Experimental pick", shown honestly.
- Simplest version: undertone + one-accent + shoe-anchor rules, checked against the reference table.

### Suggested order to build
1. Bad-reason chips (feed #9 and #10, very small).
2. #4 "Where to?" place chips.
3. #9 Fit + Vibe fields and rules.
4. #5 Mood chips.
5. #1 Photos + auto-colour.
6. The Style DNA quiz + personal voice lines, layered over everything above.
7. #10 rule tightening, guided by the Bad reasons collected by then.
