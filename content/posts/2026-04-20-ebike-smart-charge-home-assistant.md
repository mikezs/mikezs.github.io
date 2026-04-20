---
title: "Automating E-Bike Charging with Home Assistant"
date: 2026-04-20T00:00:00+00:00
draft: false
description: "How I built a Home Assistant blueprint that charges my e-bike battery to a target level during off-peak hours — and why nothing off the shelf could do it."
tags: ["home-assistant", "automation", "devtools"]
categories: ["Engineering"]
showToc: true
---

I'm on an Octopus Go tariff, which gives me electricity at 7.5p/kWh overnight between 23:30 and 05:30. I also commute on an e-bike most days. The obvious thing to do is charge the bike overnight — but doing it *well* turns out to be more involved than it sounds.

What I wanted:

- Charge automatically during the cheap window
- Stop at 80% on normal days (better for battery longevity)
- Charge to 100% the night before a longer ride
- Not leave the charger running all night every night

Simple enough in principle. In practice, nothing I could find actually did this.

## Why off-the-shelf didn't work

### Smart plugs with schedules

The basic approach: set a smart plug timer to turn on at 23:30 and off at 05:30. Done.

Except not. The problem is that how long a charge takes depends entirely on where the battery starts. If I've done a short commute and the battery is at 70%, I need about 30–40 minutes to reach 80%. If I've done a longer ride and I'm at 30%, I need closer to 2.5 hours. A fixed timer either cuts off too early on depleted batteries or runs for hours on a nearly-full one.

There's no way to configure "stop when full" on a dumb timer.

### EV-style smart charging

Modern EV chargers handle this properly — they speak OCPP or have dedicated integrations that read state of charge directly from the car and schedule accordingly. Home Assistant has solid EV charging automations for exactly this reason.

E-bikes have none of this. There's no standard protocol, no BMS integration, no way for Home Assistant to read the battery level off the bike. The charger is just a transformer in a box. The bike has a display that shows percentage, and that's it — no API, no Bluetooth, nothing.

### Existing HA blueprints

There are a handful of e-bike charging blueprints in the community. Most work on a simple "charge for N hours" basis, which has the same problem as the fixed timer. The ones that go further typically require a battery sensor entity — which you won't have unless your bike happens to expose one over BLE (rare and brand-specific).

## The approach I went with

If I can't read the battery level from the bike, I can calculate it from the charger.

A smart plug with energy monitoring gives me cumulative kWh delivered. Given the battery capacity in Wh and the charger's efficiency, I can work out exactly how much wall energy is needed to bring the battery from its current level to the target:

```
wall_energy_needed = (target% - current%) × capacity_Wh ÷ efficiency ÷ 1000
```

So if I set the current level manually before charging (one input on a dashboard card), and set the target level (another input), the automation can watch the kWh counter and stop the plug precisely when the calculated amount of energy has been delivered.

This gives me:

- **Accurate stopping** — not time-based, energy-based
- **Variable targets** — 80% most days, flip to 100% before a big ride
- **Partial charge recovery** — if HA restarts mid-charge, it picks up from the last saved battery level and resumes; no energy is double-counted
- **Storage mode** — on non-ride nights, charge to 50% storage level to keep the battery healthy during low-use periods
- **Cost tracking** — completion notification tells me what each session cost at the configured rate
- **Sanity checks** — if the plug turns on but nothing is drawing power after 20 seconds, it turns off and notifies me (usually means I forgot to plug the charger into the bike)

## What you need

**Hardware:**
- Any smart plug with **energy monitoring** — must expose a kWh (cumulative energy) sensor and a watt (live power) sensor. I use a Shelly Plug S; TP-Link Kasa and IKEA Tretakt also work.
- Home Assistant with the plug integrated (local API preferred — no cloud dependency)

**Helpers** (create once in Settings → Helpers → + Create Helper → Number):

| Helper | Range | Step | Unit | Notes |
|---|---|---|---|---|
| Current Battery Level | 0–100 | 1 | % | Update from the bike display before charging; the automation keeps it current during a session |
| Charge Target | 0–100 | 5 | % | Change weekly — 80% normally, 100% before a long ride |

Put both on a dashboard card somewhere you'll see them.

## Installing the blueprint

Click below to import directly into your Home Assistant instance:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgist.github.com%2Fmikezs%2Fce72828069ce0acd2be1d5222b1187ab)

Or manually: **Settings → Automations & Scenes → Blueprints → Import Blueprint** and paste:

```
https://gist.github.com/mikezs/ce72828069ce0acd2be1d5222b1187ab
```

## Configuring the automation

Once imported, create an automation from the blueprint. The key fields:

**Required:**
- **Charger Switch** — the smart plug switch entity
- **Energy Sensor (kWh)** — cumulative energy entity from the plug
- **Power Sensor (W)** — live power entity from the plug
- **Current Battery Level** — the helper you created
- **Charge Target** — the other helper

**Battery settings:**
- **Battery Capacity (Wh)** — check your battery label. Common sizes: 250, 360, 500, 630 Wh
- **Charger Efficiency (%)** — start at 95. After your first completed charge, check the bike display: if it shows less than the target, lower this; if HA stopped early, raise it

**Schedule:**
- **Ride Days** — the days you want the bike ready. If charging starts at 23:30 (evening), select the *following* day — so pick Sunday to charge Saturday night ready for Sunday morning
- **Charge Start / End Times** — match your off-peak window

**Optional extras:**
- **Storage Charge** — enable to top up to 50% on non-ride nights
- **Hard Cutoff** — enable to guarantee charging stops within the cheap window even if the target hasn't been reached
- **Battery Level Reminder** — sends a notification on ride evenings reminding you to update the current level helper after your ride

## Tuning efficiency

After the first charge or two you'll be able to tell if the efficiency setting is right. Check the bike display when the completion notification arrives:

- **Display shows less than target** → charger is less efficient than assumed, lower the efficiency setting
- **HA stopped early and battery isn't full enough** → raise the efficiency setting

Most decent e-bike chargers land between 90–98%. I run mine at 93%.

## The result

I've been running this for a few months. The bike is ready at the level I want each morning, charging happened during the cheap window, and I get a notification telling me it cost 4–8p for the session depending on how depleted the battery was. The 80% default has become habitual — I flip to 100% the night before anything over about 30km.

The blueprint is on [GitHub Gist](https://gist.github.com/mikezs/ce72828069ce0acd2be1d5222b1187ab) if you want to read the full YAML or suggest improvements.
