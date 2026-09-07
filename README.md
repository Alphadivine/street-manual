# 🧭 Street Manual

A field guide for the city — crafting recipes, benches, drop offs and bus blocks — in one place,
shared live with the crew.

It exists because the crafting spreadsheet kept losing things. The part a spreadsheet can't do is
the part that matters: this follows a recipe down through every sub-craft on its own, so a nested
ingredient never goes missing again.

Street Manual is a single self-contained HTML file. No build step, no framework, no server of your
own — it runs entirely in the browser and stores shared data in a free
[Firebase Firestore](https://firebase.google.com/products/firestore) database.

> **Live site:** **https://alphadivine.github.io/street-manual/**

---

## ✨ What's in it

### Catalogue
Every item, searchable by name, category, bench **or ingredient** — "show me everything that eats
copper wire". Chips filter by kind (craftable / materials / blueprints / needs attention); bench and
category are dropdowns beside the search box, so the filter row stays one line however many
categories you end up with. Clicking an active chip turns it off, and a **Clear** appears on the
right whenever anything is filtered — it resets the chips, both dropdowns and the search in one go,
and tells you how many of the total you're looking at.

Cards carry the bench, craft time and level requirement, with a ◆ by the name when a blueprint is
needed. The ingredient line clamps to two lines so every card in a row is the same height — the
grid reads as a grid rather than a ragged wall. Open an item for the full list.

### Item detail
The direct recipe, then the full raw-material rollup all the way down, the sub-crafts you have to
run first, total craft time, and everything that uses this item as an ingredient.

### Planner
Pick several things and quantities — "2 pistols, 10 lockpicks, 1 repair kit" — and get:

- one combined **shopping list** of raw materials
- **what you already have** subtracted out, including part-built components
- the **bench route in order**, deepest sub-craft first, so you never walk back to a bench twice
- total craft time and total number of craft runs

### Benches
Every crafting bench with its location in the city and everything it makes. Recipes with no bench
assigned get called out so nothing hides.

Each bench gets **its own colour**, carried onto the tag of every item made there, the bench card,
the planner's bench route and the recipe tree — so you can spot which bench an item belongs to
without reading. Eight hues in a fixed order, checked for colour-blind separation and contrast
against the dark background rather than picked by eye; the bench name is always in the tag too, so
colour is a shortcut and never the only clue. Override any of them with the swatch picker.

That colour also runs down the **left edge of the item card**, so the catalogue grid groups itself
by bench before you read a word. No stripe means it isn't crafted — a raw material or a blueprint —
or that it's made at more than one bench.

**Hacks carry their own picture.** Each hack on a heist has a screenshot slot on its row in the
editor — click it or drag an image onto it. In the heist view the picture sits beside the hack's
name, so a circle hack and a keypad are told apart by looking. One per hack; they go into backups
and are cleaned up when the hack or the heist is deleted.

Benches take **photos** as well. Clicking one opens a detail view with the gallery, its notes and
everything it makes — open to read-only crew too, so nobody has to describe where a bench is over
voice.

### Drop Offs & Bus Blocks
The chase spots the crew otherwise keeps in their heads. Each one holds a name, area/landmark,
screenshots, tags and a short "how to run it" note. Bus blocks also record **what it blocks** and
**what you need** — so mid-chase you can tell at a glance whether a spot fits the situation.

Tags (`heli-proof`, `needs 2 cars`, `night only`) become the filter chips at the top of each tab.

### Heists
Crew size, gear, the hacks to expect and the steps. Details below.

---

## 📖 The built-in guide

There's a **How to use** button in the header, and the guide **opens by itself the first time
someone loads the page** on a given browser — the people who most need it are the ones who'd never
click Help. It never appears again after that; the button reopens it any time.

It covers what each tab is for, a worked example of the Planner, a legend for reading a card
(MATERIAL / BLUEPRINT / gap badges, the ◆ blueprint mark, bench colours), how to search and filter,
and the keyboard shortcuts.

The guide **adapts to who's reading it**. A read-only viewer is told plainly why there's no **+**
button and how to unlock editing; someone already unlocked gets the section on adding items,
auto-created ingredients, craft-time units and pasting screenshots instead.

### Keeping it current

The guide is **versioned**. `GUIDE_VERSION` and a `WHATS_NEW` list sit next to `openGuide()` in the
file. Ship something a user needs to know → bump the version and add a line. Anyone who read an
older version then gets a green dot on the **How to use** button and a **New since you last looked**
block at the top of the guide; it clears once they've read it. A first-time reader never sees that
block, just the guide.

This matters because the guide only auto-opens once. Updating it without telling returning readers
would be updating it for nobody.

The test suite has a **coverage check** that fails if the guide stops mentioning any user-facing
feature — Planner, blueprints, heists, bench colours, paste, undo, the edit code and so on. It
caught a real gap the first time it ran (an editor reading the guide was never told the edit code
existed), which is exactly the sort of drift it's there to prevent.


---

## 💰 Heists


Each heist records the crew size, the gear, the hacks to expect and the steps, plus screenshots,
tags and notes.

- **Gear** is a name and a quantity. When the name matches something in the catalogue it links
  straight to it and shows which bench makes it; anything else — a getaway van, masks — stays as
  plain text and is simply listed.
- **Prep in planner** takes every piece of gear that exists in the catalogue and loads it into the
  Planner in one click: full shopping list, bench route, blueprints you're missing. Free-text gear
  is skipped, and the button only appears when there's something plannable.
- **Hacks** are a name and a count, so you know three circle hacks are coming before you start.
- **Steps** are typed one per line and rendered as a numbered list. Reordering is moving a line.

Heists live in the same Firestore collection as drop offs and bus blocks, so **no security-rules
change is needed** to add them.

---

## 🔀 More than one recipe

Some things can be made more than one way — usually different ingredients, sometimes a different
bench. An item holds as many recipes as you need.

- **One card, not two.** The catalogue still shows one *Codeine*, marked **2 ways**. Opening it
  lists every recipe with its own bench, time, yield and ingredients.
- **The Planner lets you choose.** When a planned item has alternatives, a picker appears at the top
  of the plan; switch route and the shopping list, bench stops and total time all update. Your
  choice is remembered on your own device.
- **The first recipe is the default**, so a plan always works without touching anything.
- **Adding one:** *Other ways to make it* at the bottom of the item form. Each alternative gets its
  own bench, makes-quantity, craft time and ingredient list, with its own paste box.

Existing items are untouched — the recipe already on an item **is** recipe 1, so nothing had to be
migrated and anything with a single recipe behaves exactly as before.

---

## 📐 Blueprints

Some recipes need a blueprint as well as ingredients, so blueprints are their own thing rather than
being faked as an ingredient. On a recipe they sit in an optional, collapsed **Blueprints** section
— most items won't have one.

Each blueprint on a recipe is marked one of two ways, because it changes the maths:

- **Unlock** — you own it once and can craft forever. Needed once no matter how many you're making.
- **Used up** — one (or more) goes every craft, so it scales with the quantity.

Blueprints get their own catalogue entries, auto-created when you name one, with a **Blueprint**
badge and their own filter chip — so you can record where each one drops from and search them like
anything else. They're deliberately kept out of the raw-materials shopping list; a blueprint isn't
something you pick up at the hardware store.

The planner gives them a **Blueprints needed** panel with a tick-box for the ones you own (kept on
your own device, like on-hand counts) and tells you plainly what you're missing — so you don't get
to the bench and find you can't start.

---

## ⌨️ Pasting a list of ingredients

The slowest part of keeping this up to date is typing ingredient rows one at a time, so the item
form has a **Paste a list** button. Drop in whatever shape the bench gives you:

```
20x Iron, 25x Metal Scrap, 1x Oak Plank, 8x Plastic
```

It also copes with one-per-line, `Iron x20`, `Copper: 2`, `20 Iron`, bullets, a leading
`INGREDIENTS` header, and the app's own card format (`15× Aluminium · 30× Iron …`). A name that
starts with a digit — `9mm Rounds` — survives intact. It previews how many it found before you
commit, and drops the blank row that's already sitting there.

---

## ↩️ Undo

Deleting an item, bench, spot or heist leaves an **Undo** button in the toast for seven seconds.
Stored screenshots aren't cleaned up until that window closes, so an undone delete comes back whole
— and undoing a bench deletion puts its recipes back on it too.

A recipe also can't name **itself** as an ingredient or blueprint any more; that's refused at save.
An indirect loop (A needs B, B needs A) warns and names the chain, but lets you save it in case
you're halfway through an edit.

---

## 🧠 The rules the engine follows

These are the things that quietly go wrong in a spreadsheet:

- **Yields** — a recipe that makes 5 only runs twice for 9. Times are per *run*, not per unit.
- **Nesting** — sub-crafts are expanded recursively and their materials rolled into one list.
- **Stock** — on-hand counts are consumed at every level. Already have 2 weapon parts? It won't send
  you out for the steel. (On-hand counts stay on your own device even in shared mode — they're
  yours, not the crew's — and stay editable for read-only users.)
- **Loops** — if two recipes ever reference each other, it's caught and flagged rather than
  hanging, and the warning names the chain so you can go and fix the data.
- **Unknowns** — an ingredient or blueprint you name that isn't listed yet is created automatically,
  so the catalogue fills itself in as you type.
- **Blueprints** — an unlock is needed once however many you make; one that gets used up scales with
  the craft count. Neither lands in the raw-materials list.
- **Honest blanks** — a missing value shows as `—`, never as `0`.

### What "Needs attention" means

It flags things that make the app give you a **wrong or incomplete answer**, never blank optional
fields:

- a recipe with **no bench** — the bench route can't place it
- a recipe with **no craft time** — time totals come out short
- a recipe naming an **ingredient or blueprint that isn't listed** — it can't be broken down

A raw material with no source written down is *not* flagged; it works fine everywhere. Those sit
under a separate, quiet **No source yet** chip for when you feel like tidying.

---

## 🔒 Who can edit

The manual opens **read-only**. Browsing, searching and the planner all work with no code at all.
Adding or changing anything needs the crew's **edit code**, entered once per browser via
**View only · unlock** in the header.

This is enforced by Firestore, not just hidden in the interface. The browser signs in anonymously
so Firestore has an identity to attach the permission to; entering the correct code writes an
editor record for that identity, and the rules require one for every write. Someone without the
code can open dev tools all they like and still can't write.

- **Rotate the code:** change the `code` field on `boards/main/config/access`. Devices already
  unlocked stay unlocked — clear `boards/main/editors` to actually cut them off.
- **Revoke one person:** delete their row from `boards/main/editors`.
- Clearing site data signs a browser out; that person re-enters the code.
- With no Firebase configured at all, everything is editable — locking your own local file would be
  pointless.

### Turning editing off

Click **Editing on** in the header and you get two ways to stop editing, neither of which gives up
your access:

- **Preview as viewer** — the app behaves exactly as it does for the crew: no **+** button, no Edit
  or Delete. One click on the header pill puts you back. Good for checking a change reads properly.
- **Turn editing off** — sticks across reloads until you switch it back on. Stops accidental edits
  if you leave the page open or hand the machine over. No code needed to return; this is the browser
  choosing not to edit, not losing access.

To genuinely take a device's access away, delete its row from `boards/main/editors` in the Firebase
console. Clearing that browser's site data does it too.


---

## 🖼️ How screenshots are stored

Built on the assumption there will be a lot of them:

- An upload is downscaled to ~1400px JPEG, stepped down further if still large. A 2560×1440 grab
  lands around 200 KB.
- A **small thumbnail (~25 KB) lives on the record**; the full-size image sits in its own record and
  is only fetched when you open that spot or bench. The tabs stay fast no matter how many pictures
  pile up.
- A Firestore record caps at 1 MB — the step-down keeps every image comfortably under it.
- Deleting a spot or bench, or removing an image from one, deletes the stored image too. No orphans.
- Up to 8 images each. You can paste a Discord/imgur **link** instead of uploading, which uses no
  storage at all.

At ~250 KB an image, Firestore's free 1 GiB is roughly 3,000 screenshots.

---

## 🧩 How it works

- **Frontend:** one `index.html` — HTML, CSS and vanilla JS, no dependencies bundled. The Firebase
  SDK loads from Google's CDN as an ES module.
- **Storage & sync:** Firestore, with security rules doing the read/write split. Live updates via
  `onSnapshot`.
- **Identity:** Firebase Anonymous Auth — no accounts, no passwords, just a per-browser id for the
  edit lock to hang on.
- **Offline / solo:** with no Firebase config filled in, everything falls back to `localStorage` and
  the app works fully on one device.

There is no backend to run. Hosting is just serving a static file.

---

## 🚀 Self-hosting setup

### 1. Create a Firebase project

[console.firebase.google.com](https://console.firebase.google.com) → **Create a project**. Analytics
can be off. The free **Spark** plan is enough — no card required.

### 2. Add Firestore

**Build → Firestore Database → Create database.** Leave the ID as `(default)`, pick a location
(permanent), choose **production mode** — step 5 replaces the rules anyway.

### 3. Enable anonymous sign-in

**Build → Authentication → Get started → Sign-in method → Anonymous → Enable.**

Skip this and nobody can unlock editing.

### 4. Set your edit code

In **Firestore Database → Data**, create one document by hand:

```
boards (collection)
└── main (document)          ← must match BOARD_ID in the file
    └── config (collection)
        └── access (document)
            └── code: "your-edit-code"   (string)
```

### 5. Publish the rules

**Firestore Database → Rules**, replace everything:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /boards/{board} {

      function isEditor() {
        return request.auth != null &&
          exists(/databases/$(database)/documents/boards/$(board)/editors/$(request.auth.uid));
      }
      function codeMatches() {
        return request.resource.data.code ==
          get(/databases/$(database)/documents/boards/$(board)/config/access).data.code;
      }

      // the edit code itself: the app can never read it or change it
      match /config/{doc} {
        allow read, write: if false;
      }

      // a device claims editing rights by presenting the code
      match /editors/{uid} {
        allow read:           if request.auth != null && request.auth.uid == uid;
        allow create, update: if request.auth != null && request.auth.uid == uid && codeMatches();
        allow delete:         if false;
      }

      // the manual itself: everyone reads, only editors write
      match /items/{doc}   { allow read: if true; allow write: if isEditor(); }
      match /benches/{doc} { allow read: if true; allow write: if isEditor(); }
      match /spots/{doc}   { allow read: if true; allow write: if isEditor(); }
      match /images/{doc}  { allow read: if true; allow write: if isEditor(); }
    }
  }
}
```

### 6. Register a web app and paste its config

**Project settings → General → Your apps → `</>`**. Copy four values into the `CONFIG` block near
the bottom of `index.html`:

```js
const CONFIG = {
  APP_NAME:  "Street Manual",
  APP_SUB:   "unholy",

  FIREBASE: {
    apiKey:     "",
    authDomain: "",
    projectId:  "",
    appId:      ""
  },

  BOARD_ID: "main"
};
```

`storageBucket`, `messagingSenderId` and `measurementId` aren't needed.

> The `apiKey` is a public identifier, not a secret — it ships in every Firebase web app and is
> safe in a public repo. What actually protects the data is the security rules above. The **edit
> code is not in this file**; it lives in Firestore.

### 7. Deploy

Push `index.html` to a repo and turn on **Settings → Pages → Deploy from a branch → main / root**.

Then — and this is the step everyone forgets — add your Pages host to
**Firebase → Authentication → Settings → Authorized domains**:

```
alphadivine.github.io
```

Hostname only, no `https://` and no path. Miss it and the app loads and reads fine, but unlocking
fails in a way that looks like a wrong code.

---

## 📦 Backups

**Data → Export backup (.json)** writes out items, benches, spots **and the full-size screenshots**,
so the file can get large — that's deliberate, it's a real backup. Importing puts it all back,
pictures included. There's also a flat `.csv` of the recipes if you ever want them in a spreadsheet
again.

Export is available to everyone; import and erase need the edit code.

---

## ⌨️ Odds and ends

- **Installs on a phone.** Add it to your home screen and it opens like an app — the Unholy mark as
  the icon, full screen, no address bar. iPhone: Safari → Share → *Add to Home Screen*. Android:
  Chrome menu → *Install app*. Laid out for a phone, no horizontal scroll.
- **Total craft time is a floor.** The Planner adds the craft times back-to-back; it doesn't know
  about per-bench limits, cooldowns, queues, or walking between stops. Use it to compare two routes,
  not to set a clock.
- **Money tracking** is off by default — prices are hard to pin down in the city. The fields still
  exist, collapsed under **Money** in the edit form; **Data → Display → Track money** brings the
  costs, sell prices and profit figures back. Per device, so turning it on doesn't force it on
  anyone else.
- **A second manual:** change `BOARD_ID` to run a separate one off the same Firebase project. Give
  it its own `config/access` code.
- **Keyboard:** `/` focuses search, `Esc` closes any panel or screenshot.
- **Browser cache** is stubborn after a redeploy — `Ctrl+Shift+R`.
