# 🏁 F1 Next Race — iOS Compact Premium Widget

This is a script optimized for the **Scriptable** application on iOS. It displays a **medium-sized** widget on your iPhone's home screen featuring the local times for all sessions of the upcoming Formula 1 Grand Prix.

---

## 📌 Features
- 📅 Displays the Grand Prix name and the circuit.
- ⏰ Automatic conversion of all sessions (FP1, FP2, FP3, Sprint, Qualifying, and Race) to your **local time**.
- 💾 Integrated **local cache system** so the widget functions even when you don't have an internet connection.
- 🏎️ Premium visual styling with tailored color themes optimized for dark mode (`#07070c`).

---

## 🛠️ Prerequisites

1. An iOS device (iPhone or iPad).
2. Download the free **Scriptable** app from the App Store.
3. Download Apple's **Shortcuts** app (pre-installed on most iOS devices).

---

## 🌟 Source Code

```javascript
// ─────────────────────────────────────────────
// F1 NEXT RACE — COMPACT PREMIUM WIDGET
// Optimizado para widget mediano iPhone
// ─────────────────────────────────────────────

const API_URL =
  "[https://api.jolpi.ca/ergast/f1/current/next.json](https://api.jolpi.ca/ergast/f1/current/next.json)"

const widget = new ListWidget()

// ─────────────────────────────
// STYLE
// ─────────────────────────────
widget.backgroundColor = new Color("#07070c")
widget.setPadding(14, 14, 12, 14)

Request.defaultTimeoutInterval = 8

// ─────────────────────────────
// CACHE
// ─────────────────────────────
const fm = FileManager.local()

const cachePath = fm.joinPath(
  fm.documentsDirectory(),
  "f1-cache.json"
)

async function getData() {

  try {

    const req = new Request(API_URL)
    req.timeoutInterval = 8

    const data = await req.loadJSON()

    fm.writeString(
      cachePath,
      JSON.stringify(data)
    )

    return data

  } catch (e) {

    if (fm.fileExists(cachePath)) {

      return JSON.parse(
        fm.readString(cachePath)
      )
    }

    throw e
  }
}

// ─────────────────────────────
// DATA
// ─────────────────────────────
const data = await getData()

const race =
  data.MRData.RaceTable.Races[0]

// ─────────────────────────────
// HELPERS
// ─────────────────────────────

function formatSession(date, time) {

  const dt =
    new Date(`${date}T${time}`)

  return dt.toLocaleString(
    "es-ES",
    {
      weekday: "short",
      hour: "2-digit",
      minute: "2-digit"
    }
  )
}

function formatRaceDate(date) {

  return new Date(date)
    .toLocaleDateString(
      "es-ES",
      {
        day: "2-digit",
        month: "short"
      }
    )
}

function addSession(label, sessionData) {

  if (!sessionData) return

  const row = widget.addStack()
  row.layoutHorizontally()

  const left =
    row.addText(label)

  left.font =
    Font.boldSystemFont(11)

  left.textColor =
    label === "RACE"
      ? new Color("#00e5ff")
      : label.includes("SPR")
        ? new Color("#ff9f0a")
        : Color.white()

  row.addSpacer()

  const right =
    row.addText(
      formatSession(
        sessionData.date,
        sessionData.time
      )
    )

  right.font =
    Font.mediumSystemFont(10)

  right.textColor =
    new Color("#8f95b2")

  widget.addSpacer(4)
}

// ─────────────────────────────
// HEADER
// ─────────────────────────────
const header = widget.addStack()

const title =
  header.addText("F1")

title.font =
  Font.boldSystemFont(13)

title.textColor =
  new Color("#ff1744")

header.addSpacer()

const local =
  header.addText("LOCAL")

local.font =
  Font.mediumSystemFont(9)

local.textColor =
  new Color("#666a85")

widget.addSpacer(8)

// ─────────────────────────────
// GP NAME
// ─────────────────────────────
const gp =
  widget.addText(
    race.raceName.toUpperCase()
  )

gp.font =
  Font.boldSystemFont(18)

gp.textColor =
  Color.white()

gp.lineLimit = 2

widget.addSpacer(2)

// ─────────────────────────────
// CIRCUIT + DATE
// ─────────────────────────────
const info =
  widget.addText(
    `${race.Circuit.circuitName} • ${formatRaceDate(race.date)}`
  )

info.font =
  Font.mediumSystemFont(11)

info.textColor =
  new Color("#00cfff")

widget.addSpacer(12)

// ─────────────────────────────
// SESSIONS
// ─────────────────────────────

addSession("FP1", race.FirstPractice)
addSession("FP2", race.SecondPractice)
addSession("FP3", race.ThirdPractice)

if (race.SprintQualifying) {
  addSession("SPR Q", race.SprintQualifying)
}

if (race.Sprint) {
  addSession("SPR", race.Sprint)
}

addSession("QUALI", race.Qualifying)

addSession("RACE", {
  date: race.date,
  time: race.time
})

// ─────────────────────────────
// AUTO REFRESH
// ─────────────────────────────
widget.refreshAfterDate =
  new Date(
    Date.now() + 1000 * 60 * 30
  )

// ─────────────────────────────
// SHOW
// ─────────────────────────────
Script.setWidget(widget)

if (!config.runsInWidget) {
  await widget.presentMedium()
}

Script.complete()
```

## 🚀 Installation & Setup Guide

### Step 1: Add the Code to Scriptable
1. Copy the full source code of F1 script.
2. Open the **Scriptable** app on your iPhone.
3. Tap the **"+"** button in the top-right corner to create a new script.
4. Clear any default text and **paste the code**.
5. Tap the script name at the top (by default *"Untitled Script"*) and rename it to **`F1 Widget`**.
6. Tap **Done** in the top-left corner to save.

### Step 2: Place the Widget on Your Home Screen
1. Go to your iPhone Home Screen, then touch and hold any app or the background until the icons jiggle.
2. Tap the **"+"** button in the top-left corner.
3. Search for and select **Scriptable**.
4. Swipe right to find the **Medium** size widget and tap **Add Widget**.
5. Tap the newly added widget (while the icons are still jiggling) to open its configuration:
   - **Script:** Select `F1 Widget`.
   - **When Interacting:** Select *Run Script* or *Open URL* (based on your preference).
6. Tap outside the widget to exit jiggle mode.

---

## 🔄 Weekly Auto-Refresh Automation (The Fix)
*Note: iOS native background refreshing (`widget.refreshAfterDate`) is unreliable due to iOS battery and memory management restrictions. To ensure your widget seamlessly switches to the next Grand Prix, we set up a Shortcut Automation that forces a background data fetch every **Sunday at 23:59**.*

1. Open the **Shortcuts** app on your iPhone.
2. Go to the **Automation** tab (clock icon at the bottom) and tap **New Automation** (or the `+` button).
3. Select **Time of Day**.
4. Configure the following settings:
   - **Time of Day:** Set it to **23:59**.
   - **Repeat:** Select **Weekly**.
   - **Days:** Ensure **only Sunday** is selected.
   - **Execution:** Select **Run Immediately** (so it updates in the background without prompting you).
5. Tap **Next**.
6. In the action search bar, type **Scriptable** and select the **Run Script** action.
7. In the newly added block, tap the blurred *"Script"* text and choose **`F1 Widget`**.
8. Expand the action details (by tapping the small arrow icon on the action block) and ensure **"Show When Run" is turned OFF** (this prevents Scriptable from launching on your screen late at night).
9. Tap **Done** in the top-right corner.

---

## ⚙️ Technical Overview
- **Data Source:** Fetched from the open-source OpenF1 / Ergast API (`api.jolpi.ca`).
- **Cache Policy:** The script saves data to `f1-cache.json` inside Scriptable’s local document directory. If a network request fails or iOS fails to refresh natively, it safely falls back to the last known schedule fetched by your Sunday automation.
