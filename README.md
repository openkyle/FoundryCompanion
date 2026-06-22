# FoundryCompanion

Foundry VTT module that lets the GM publish player-facing campaign data to a read-only, mobile-friendly companion website.

FoundryCompanion is built for Foundry first. Forien's Quest Log is an optional integration: if the Forien module is installed and active, FoundryCompanion publishes its quest log data; if not, the standard Foundry Journal, Contacts, Items, and Character Sheet sections still publish normally.

## What it does

- Adds a GM-only **Configure Settings > Module Settings > FoundryCompanion** panel.
- Opens a GM preview window that updates when relevant Foundry data or FoundryCompanion settings change; interval refresh is currently disabled but visible in the panel.
- Downloads a full JSON snapshot matching the companion website publish payload.
- Publishes structured JSON to an external companion website endpoint.
- Publishes standard Foundry sidebar data for Journal, Contacts, Items, and owned player character sheets.
- Optionally publishes Forien quest tabs when Forien's Quest Log is active: **Available**, **In Progress**, **Completed**, and **Failed**.
- Excludes Forien **Inactive** quests.
- Preserves Journal folder structure.
- Includes a GM setting to publish either player-visible sidebar content only, or all standard Foundry sidebar data.
- Includes **Configure Export Options** checkboxes for Enable Forien Quest Log Exporting, Journal, Contacts, Items, and Character Sheets.
- Includes optional **5e - Custom Abilities & Skills** (`dnd5e-custom-skills`) support for custom ability and skill labels.
- Includes optional **TradeHub Markets** (`tradehub-markets`) read-only export for TradeHub Capital, current location, and owned party ships with modules, HP, AC, cargo, and lightweight ship metadata.
- Lets the GM choose one Journal Entry as the **Game Session Story** source; each page exports directly as an individual rich-text chapter/session with image references.
- Game Session Story supports a Chapter/Session label setting, continuous downward reading, and bookmark/return metadata for the companion website.
- Image export mode can use image links for smaller payloads or image embedding for larger, portable payloads.
- GM preview uses simple expandable folder/group dropdowns to verify exported data.
- Contacts and character sheets are grouped by Foundry folder path.
- Items are grouped by holder, including owned character inventories and world items.
- Quest preview shows objectives.
- Export schema v8 keeps only human-readable summaries, descriptions, images, folders, objectives, rewards, story chapters, TradeHub ship summaries, and readable sheet fields.
- Exported JSON omits empty/default values, raw Foundry/Forien task objects, ownership bookkeeping, active effects, duplicate image paths, and duplicate grouped record bodies.
- Character sheet summaries export all abilities and skills found in actor data, including custom keys such as `tec`, `cua_0`, or `cus_4`. Use `summary.abilities` / `summary.skills` for compact maps and `summary.abilityDetails` / `summary.skillDetails` for labels/modifiers.
- When `dnd5e-custom-skills` is active and **Enable Custom Abilities & Skills Exporting** is checked, FoundryCompanion reads `customAbilitiesList`, `customSkillList`, `hiddenAbilities`, and `hiddenSkills` from that module's world setting.
- Forien's starred primary quest is exported with `mainQuest: true` and `badge: "Main Quest"`.
- Copies a debug sample to the clipboard so we can wire the exporter to your exact Forien data shape.

## Install for local testing

1. Copy this folder into Foundry's `Data/modules/foundry-companion`.
2. Enable **FoundryCompanion** in your world.
3. Open **Configure Settings > Module Settings > FoundryCompanion**.

## Important permission note

FoundryCompanion is a GM-side publisher. The companion website is the player-facing read-only surface.

Quest Log is optional and requires the Forien's Quest Log module to be active. When present, it is status-gated: Available, In Progress, Completed, and Failed quests are included; Inactive quests are excluded. GM Notes are not published by default.

FoundryCompanion is designed to be run by the GM only. Players should not use the Foundry-side module; they view the companion website in a read-only capacity.

Website publishing sends standard Foundry data every time. If Forien is active, it also sends the four player-facing Forien tabs and excludes the Inactive tab. Keep sensitive information out of player-facing quest fields, especially descriptions, player notes, visible objectives, and visible rewards.

Journals are always ownership-gated using the Journal ownership object. A Journal entry is exported only when `ownership.default` is Limited or higher, or when at least one non-GM player has Limited, Observer, or Owner access. GM-only ownership does not make a Journal publishable. When Foundry exposes page-level ownership, Journal pages are filtered the same way.

Game Session Story is different from the general Journal sidebar. It is an explicit GM-selected publish source, so its selected Journal pages are exported directly to the companion website without requiring Foundry player ownership. Only choose a Story Journal whose contents are intended for the companion website.

By default, Contacts and world Items are also ownership-gated. Any document visible to at least one non-GM player at Limited permission or higher is included. Character Sheets includes only `character` actors owned by at least one non-GM player.

Actor inventories are exported only for owned player characters. If an actor is not an owned player character, it can still be listed as a Contact, but its embedded items are not exported as inventory.

Use **Sidebar Data Scope** to choose exactly one mode:

- **Publish Owned/Limited sidebar data** exports only Journal, Contacts, and world Items visible to non-GM players at Limited or higher, plus character sheets owned by players.
- **Publish all sidebar data** exports all standard Foundry Actor and world Item sidebar data regardless of player permissions. Journals still require Limited-or-higher player access; Character Sheets and actor inventory still require player ownership.

Forien Quest Log still excludes Inactive quests in either mode.

Use **Image Export Mode** to choose exactly one mode:

- **Use image links** keeps payloads smaller, but the companion website must be able to load images from Foundry URLs.
- **Use image embedding** embeds base64 image data in the payload, making it more portable but much larger.

Use **Configure Export Options** in the FoundryCompanion panel to choose which sections are included in the companion website payload. Unchecked sections are omitted from navigation and exported as empty section payloads.

TradeHub Markets export is read-only. FoundryCompanion publishes TradeHub Capital, current location, and owned party ships with modules/HP/cargo metadata. It does not publish live markets, buy/sell goods, shipyard inventory, predictive rumours, transaction history, or repair/action controls.

In exported/published schema v8 payloads, folder/status groups use ID references instead of duplicating full records. Read full records from `journal.entries`, `contacts.actors`, `items.items`, `characterSheets.actors`, `questLog.quests`, `gameSessionStory.chapters`, and `tradeHub.ships`; use group `recordIds` or `questIds` to reconstruct display sections.

Journal entries, Journal pages, folders, and Game Session Story chapters include Foundry `sort`/`order` metadata when available. The companion website should preserve received array order or sort by `order`; do not alphabetize Journal pages if Foundry order matters.

## Companion website publishing

FoundryCompanion does not create a standalone public URL inside Foundry. The Foundry-side window is a GM preview and publishing panel.

For a companion website, use this flow:

1. In the companion website, create a campaign and copy its **FoundryCompanion connection key**.
2. In Foundry, paste that one value into **Companion connection key**.
3. Click **Test Connection**. FoundryCompanion calls `GET /api/foundry-companion/<token>/ping`.
4. Click **Publish to website**, or enable **Auto-publish website data**. FoundryCompanion calls `POST /api/foundry-companion/<token>/sync`.

The connection key should be unique per campaign. That token is the tenant boundary: different Foundry worlds do not overlap as long as they use different campaign keys.

Recommended connection key format:

```text
fc1_<base64url-json>
```

Where the decoded JSON is:

```json
{
  "baseUrl": "https://your-companion-site.example/api/foundry-companion",
  "token": "campaign-token"
}
```

FoundryCompanion also accepts a full URL as the one pasted value:

```text
https://your-companion-site.example/api/foundry-companion/campaign-token
```

Legacy hidden settings for separate base URL and token are still honored for existing installs, but new installs should use the single connection key.

Expected ping response:

```json
{ "ok": true, "campaign": "Starfall Rising", "campaignId": "uuid..." }
```

Published sync body:

```json
{
  "actors": [],
  "contacts": [],
  "items": [],
  "journal": [],
  "quests": [],
  "story": {},
  "gameSessionStory": {},
  "tradeHub": {},
  "ships": [],
  "meta": {
    "schemaVersion": 8,
    "summary": {},
    "navigation": [],
    "integrations": {}
  }
}
```

No Authorization header is sent; the token is part of the companion website URL path.

The **Export JSON snapshot** button still downloads the full local payload with:

```json
{
  "schemaVersion": 8,
  "module": "foundry-companion",
  "world": {
    "id": "your-world-id",
    "title": "Your World"
  },
  "title": "FoundryCompanion",
  "generatedAt": "2026-06-15T00:00:00.000Z",
  "publishMode": "player-visible-sidebar-data",
  "imageMode": "linked",
  "exportOptions": {
    "questLog": true,
    "customAbilitiesSkills": true,
    "tradeHub": true,
    "tradeHubCapital": true,
    "tradeHubShips": true,
    "journal": true,
    "contacts": true,
    "items": true,
    "characterSheets": true,
    "embedImageData": false
  },
  "imageDataEmbedded": false,
  "navigation": [
    { "id": "gameSessionStory", "label": "Game Session Story" },
    { "id": "journal", "label": "Journal" },
    { "id": "contacts", "label": "Contacts" },
    { "id": "items", "label": "Items" },
    { "id": "characterSheets", "label": "Characters & Ships" }
  ],
  "summary": {
    "quests": 0,
    "storyChapters": 3,
    "journalEntries": 12,
    "contacts": 20,
    "items": 8,
    "characterSheets": 4,
    "tradeHubShips": 1,
    "totalDocuments": 53
  },
  "questLog": {
    "enabled": false,
    "integration": "forien-quest-log",
    "includedStatuses": ["Available", "In Progress", "Completed", "Failed"],
    "quests": [],
    "sections": []
  },
  "gameSessionStory": {
    "id": "journal-entry-id",
    "title": "Session Logs",
    "configured": true,
    "publishable": true,
    "labelMode": "chapter",
    "labelSingular": "Chapter",
    "labelPlural": "Chapters",
    "readingMode": "continuous",
    "bookmarking": {
      "enabled": true,
      "storageKey": "foundryCompanionStoryBookmark"
    },
    "source": {
      "id": "journal-entry-id",
      "name": "Session Logs",
      "folderPath": "Campaign"
    },
    "chapters": [
      {
        "id": "page-id",
        "order": 1,
        "label": "Chapter 1",
        "title": "Session 1",
        "type": "text",
        "text": "<p>Rich text from the Journal page.</p>",
        "imageUrls": []
      }
    ]
  },
  "tradeHub": {
    "enabled": true,
    "active": true,
    "vehicleLabel": "Vessel",
    "capital": 125000,
    "capitalDisplay": "125,000 GP",
    "currentLocation": "Nemoï",
    "ships": [
      {
        "id": "ship-id",
        "name": "USS Light Hungry",
        "type": "vehicle",
        "imageUrl": "https://...",
        "hp": { "value": 42, "max": 60 },
        "ac": 15,
        "cargo": {
          "current": 1200,
          "max": 4000,
          "remaining": 2800,
          "capacityTons": 2
        },
        "meta": {
          "shipValue": 75000,
          "moduleValue": 25000,
          "totalValue": 100000,
          "shieldHp": { "value": 8, "max": 12 },
          "hyperdrive": "3 parsecs"
        },
        "modules": [
          {
            "id": "module-id",
            "name": "Shield Generator",
            "type": "equipment",
            "imageUrl": "https://...",
            "hp": { "value": 8, "max": 12 },
            "quantity": 1,
            "weight": 0,
            "value": 5000
          }
        ]
      }
    ]
  },
  "journal": {
    "folders": [],
    "groups": [],
    "entries": []
  },
  "contacts": {
    "folders": [],
    "groups": [],
    "actors": []
  },
  "items": {
    "folders": [],
    "groups": [],
    "items": []
  },
  "characterSheets": {
    "folders": [],
    "actors": []
  }
}
```

## Optional Forien integration

Forien's Quest Log is a separate Foundry module. FoundryCompanion does not require it. When Forien is active, FoundryCompanion reads quests from Forien's public API at `game.modules.get("forien-quest-log").public.QuestAPI.DB`.
