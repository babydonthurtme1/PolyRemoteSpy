# PolyRemoteSpy

A runtime network introspection LocalScript for Polytoria. Captures every
NetworkEvent, Signal, and chat message the client can see, shows them in a
two-panel GUI, and generates ready-to-run replay scripts.

---

## Installation

1. Open Polytoria Creator for the place you want to inspect.
2. Insert a **LocalScript** inside `game["ScriptService"]`.
3. Paste the full contents of `PolyRemoteSpy.lua` into it.
4. Press **Test Game**. The spy window appears centred on screen.

> Running inside the **polyhack** executor (`identifyexecutor() == "elcapor"`)
> unlocks the Execute, FireClick, and SendChat buttons and shows a purple
> polyhack badge in the title bar.

---

## What gets captured

### NetworkEvent  (primary capture target)

Polytoria games communicate between server and client using `NetworkEvent`
objects. Each carries a `NetMessage` with typed key-value fields.

| Direction | How it works | Label |
|---|---|---|
| Server -> Client | Server calls `InvokeClients(msg)` or `InvokeClient(msg, player)`. The `InvokedClient` event fires on the LocalScript. | **S->C** (green) |
| Client -> Server | Local player calls `netEvent:InvokeServer(msg)`. Captured via a metatable wrap on `InvokeServer`. | **C->S** (orange) |

**Important limitations:**
- `C->S` only captures **your own** outbound calls. Other players' `InvokeServer`
  calls happen server-side and are invisible to a LocalScript.
- `InvokeClients` and `InvokeClient` cannot be captured at the source (they are
  server-exclusive methods). We only see their arrival as `InvokedClient`.
- The metatable hook for `C->S` is a best-effort approach. It may silently fail
  if the Polytoria sandbox restricts metatable access, in which case only `S->C`
  events will be captured for that NetworkEvent.

### Signal

`Signal` objects (class `Signal`) carry variadic arguments rather than a
`NetMessage`. They are hooked in the same way:

| Direction | Label |
|---|---|
| `Signal.Invoked` fires on client | **S->C** |
| Local script calls `signal:Invoke(...)` | **C->S** |

Because Signal arguments have no defined schema, fields are logged as
`arg1`, `arg2`, etc. with their Lua type and `tostring()` value.

### Chat

Every `Player.Chatted` event that fires on the client is captured. This
includes messages from all players in the server (chat is replicated to all
clients).

| What | Label |
|---|---|
| Any player sends a chat message | **CHAT** (blue) |

---

## Scan coverage

On startup and whenever new children are added, PolyRemoteSpy scans:

- `game["Environment"]` and all descendants
- `game["ScriptService"]` and all descendants
- `game["PlayerGUI"]` and all descendants
- `ChildAdded` watchers on all three roots catch objects created after startup

Any `NetworkEvent` or `Signal` found anywhere in these trees is hooked
automatically. If a game stores its events in a custom container outside
these three roots, it will not be scanned. You can add extra roots by
appending to `scanRoots` near the bottom of the script.

---

## NetMessage field probing

NetMessage has no key-enumeration API — you cannot ask "what keys does this
message contain?" The spy must probe every key name it wants to find.

PolyRemoteSpy probes **300+ key names** covering the most common patterns
across all Polytoria game genres: tycoon games, RPGs, shooters, simulators,
social games, and more. All single-letter keys (a–z) are included.

### Probe detection limits

| Value | Detected? | Why |
|---|---|---|
| Any non-empty string | Yes | `GetString` returns `""` for missing keys |
| Any non-zero int | Yes | `GetInt` returns `0` for missing keys |
| Any non-zero float | Yes | `GetNumber` returns `0` for missing keys |
| `true` boolean | Yes | Unambiguous |
| `false` boolean | No | `GetBool` returns `false` for missing keys — indistinguishable |
| Int/float with value `0` | No | Same as missing key — indistinguishable |
| String with value `""` | No | Same as missing key |
| Vector3, Vector2, Color, Instance | Yes | Returns `nil` for missing keys |

### Adding custom key names

If a game uses key names not in the probe list, add them near the top of
the script:

```lua
local EXTRA_KEYS = {
    "purchaseId", "buildingType", "myCustomKey",
}
```

To find what keys a game uses: pause the spy, inspect a captured event, and
look for patterns in nearby game scripts or the console output.

### Why fields show "(no keys found from probe list)"

The captured message exists — the event fired. The game just uses key names
not in the 300+ probe list. Use `EXTRA_KEYS` to add them once you identify
the names by reading the game's LocalScripts or observing multiple events.

---

## GUI reference

```
+--[PolyRemoteSpy]--[0 hooks]--[0 captured]--[ drag ]--[|| PAUSE][CLR][X]--+
| [ALL EVENTS] [S -> C] [C -> S]                         [Insert] toggle    |
+---[list]---+--------------------------------------------------------------+
| S->C  Ev   | S->C  EventName                               t=12.34s       |
| C->S  Ev   | game["ScriptService"]["Events"]["EventName"]                  |
| CHAT  Name | FIELDS                                                        |
| ...        |   [string]  key  =  "value"                                   |
|            |   [V3]  position  =  (1.00,2.00,3.00)                        |
|            | REPLAY SCRIPT  --  click inside, Ctrl+A, Ctrl+C              |
|            |   -- PolyRemoteSpy [S->C]                                     |
|            |   local netEvent = game["ScriptService"]["Events"]["Ev"]     |
|            |   local msg = NetMessage.New()                                |
|            |   msg:AddString("key", "value")                               |
|            |   -- S->C: use InvokeClient from a server Script to replay   |
+--[^][v][>> Latest][+N new]--+--[Copy Script][Execute][FireClick][SendChat]+
```

### Title bar

| Control | Action |
|---|---|
| `[ drag ]` strip | Click and drag to move the window |
| `[Insert]` key | Toggle window visibility |
| `|| PAUSE` / `> RESUME` | Freeze the list so you can click rows. New events accumulate and show a `+N new` counter. |
| `CLR` | Clear all captured events |
| `X` | Hide the window (same as Insert) |

### Filter bar

| Button | Shows |
|---|---|
| `ALL EVENTS` | Every captured event |
| `S -> C` | Server-to-client NetworkEvent/Signal traffic only |
| `C -> S` | Client-to-server traffic only |

Chat events are always visible in ALL and are hidden by S->C / C->S filters.

### List panel (left)

Rows are colour-coded by direction. Click any row to inspect it in the
detail panel. The list auto-scrolls to newest events when not paused.
Use `^` / `v` (hold for smooth scroll) or `>> Latest` to navigate.

### Detail panel (right)

Displays the selected event's full instance path, all probed NetMessage
fields with types and values, and a generated replay script.

**Copy Script** — focuses the script text box. Press `Ctrl+A` then `Ctrl+C`
to copy the full replay script to your clipboard.

### polyhack-only buttons

These buttons only appear when running inside the polyhack executor.

| Button | Function |
|---|---|
| `Execute` | Runs the replay script immediately via `loadstring()` |
| `FireClick` | Calls `fireclickdetector()` on the Instance in the event's fields (or the event's parent if no Instance field is found) |
| `SendChat` | Posts `[Spy] direction eventName t=Xs` to game chat via `sendchat()` |

---

## Replay script format

For a `S->C` event the script reminds you that replaying requires calling
`InvokeClient` from a server Script (you cannot call it from a LocalScript).

For a `C->S` event the script is immediately executable:

```lua
-- PolyRemoteSpy [C->S]
local netEvent = game["Environment"]["Tycoon 1"]["BuyEvent"]
local msg = NetMessage.New()
msg:AddString("type", "buy")
msg:AddInt("itemId", 5)
netEvent:InvokeServer(msg)
```

Paste this into a LocalScript or (if you have polyhack) use the Execute
button to fire it directly.

---

## Known limitations

| Limitation | Reason |
|---|---|
| `C->S` may not capture in all games | Requires metatable access to `NetworkEvent.__index`, which some sandbox configs block |
| Other players' `C->S` calls invisible | `InvokeServer` only fires server-side; LocalScript cannot observe it |
| Unrecognised NetMessage keys missing | No key enumeration API; only probe-list keys are shown |
| Signal args shown as `arg1`/`arg2` | Signal carries variadic Lua values with no schema |
| Events outside the three scan roots missed | Only `Environment`, `ScriptService`, `PlayerGUI` are scanned |

---

## Configuration

At the top of the script, the `CFG` table controls behaviour:

```lua
local CFG = {
    TOGGLE_KEY   = "Insert",  -- key to show/hide window
    MAX_LOGS     = 300,       -- maximum events kept in memory
    WIN_W        = 880,       -- window width in pixels
    WIN_H        = 490,       -- window height in pixels
    REFRESH_RATE = 0.30,      -- seconds between live list updates
    ...
}
```

To scan additional roots, append to `scanRoots` near the end of the script:

```lua
local scanRoots = { Env }
pcall(function() table.insert(scanRoots, game["ScriptService"]) end)
pcall(function() table.insert(scanRoots, game["Hidden"]) end)  -- custom root
```

---

## Polyhack executor functions used

| Function | Used for |
|---|---|
| `identifyexecutor()` | Detect polyhack, show badge, enable extra buttons |
| `loadstring(src)` | Execute replay scripts via the Execute button |
| `fireclickdetector(inst)` | FireClick button |
| `sendchat(msg)` | SendChat button |
| `equiptool` / `activatetool` / `unequiptool` | Available in generated replay scripts |
| `serverequiptool` | Available in generated replay scripts |
