# 🧭 Street Manual

A field guide for the city — crafting recipes, benches, drop offs and bus blocks — in one place,
shared live with the crew.

It exists because the crafting spreadsheet kept losing things. The part a spreadsheet can't do is
the part that matters: this follows a recipe down through every sub-craft on its own, so a nested
ingredient never goes missing again.

Street Manual is a single self-contained HTML file. No build step, no framework, no server of your
own — it runs entirely in the browser and stores shared data in a free
[Firebase Firestore](https://firebase.google.com/products/firestore) database.

**Live site:**https://alphadivine.github.io/street-manual/

---

## ✨ What's in it

### Catalogue
Every item, searchable by name, category, bench **or ingredient** — "show me everything that eats
copper wire". Filter chips for craftable / materials / needs attention. Each card shows the bench,
craft time and level requirement.

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

### Drop Offs & Bus Blocks
The chase spots the crew otherwise keeps in their heads. Each one holds a name, area/landmark,
screenshots, tags and a short "how to run it" note. Bus blocks also record **what it blocks** and
**what you need** — so mid-chase you can tell at a glance whether a spot fits the situation.

Tags (`heli-proof`, `needs 2 cars`, `night only`) become the filter chips at the top of each tab.

---

## 🧠 The rules the engine follows

These are the things that quietly go wrong in a spreadsheet:

- **Yields** — a recipe that makes 5 only runs twice for 9. Times are per *run*, not per unit.
- **Nesting** — sub-crafts are expanded recursively and their materials rolled into one list.
- **Stock** — on-hand counts are consumed at every level. Already have 2 weapon parts? It won't send
  you out for the steel. (On-hand counts stay on your own device even in shared mode — they're
  yours, not the crew's — and stay editable for read-only users.)
- **Loops** — if two recipes ever reference each other, it's caught and flagged rather than hanging.
- **Unknowns** — an ingredient you name that isn't listed yet is created automatically as a raw
  material, so the catalogue fills itself in as you type.
- **Honest blanks** — a missing value shows as `—`, never as `0`.

### What "Needs attention" means

It flags things that make the app give you a **wrong or incomplete answer**, never blank optional
fields:

- a recipe with **no bench** — the bench route can't place it
- a recipe with **no craft time** — time totals come out short
- a recipe naming an **ingredient that isn't listed** — it can't be broken down

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

---

## 🖼️ How screenshots are stored

Built on the assumption there will be a lot of them:

- An upload is downscaled to ~1400px JPEG, stepped down further if still large. A 2560×1440 grab
  lands around 200 KB.
- A **small thumbnail (~25 KB) lives on the spot record**; the full-size image sits in its own
  record and is only fetched when you open that spot. The Drop Offs and Bus Blocks tabs stay fast
  no matter how many pictures pile up.
- A Firestore record caps at 1 MB — the step-down keeps every image comfortably under it.
- Deleting a spot, or removing an image from one, deletes the stored image too. No orphans.
- Up to 8 images per spot. You can paste a Discord/imgur **link** instead of uploading, which uses
  no storage at all.

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
yourname.github.io
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

- **Mobile:** add to home screen. Laid out for a phone, no horizontal scroll.
- **Money tracking** is off by default — prices are hard to pin down in the city. The fields still
  exist, collapsed under **Money** in the edit form; **Data → Display → Track money** brings the
  costs, sell prices and profit figures back. Per device, so turning it on doesn't force it on
  anyone else.
- **A second manual:** change `BOARD_ID` to run a separate one off the same Firebase project. Give
  it its own `config/access` code.
- **Keyboard:** `/` focuses search, `Esc` closes any panel or screenshot.
- **Browser cache** is stubborn after a redeploy — `Ctrl+Shift+R`.
