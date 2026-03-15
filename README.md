# PolyRemoteSpy

A runtime network introspection LocalScript for Polytoria. Captures every
NetworkEvent, Signal, and chat message the client can see, shows them in a
two-panel window, and generates ready-to-run replay scripts.

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

## Controls

The window has two panels: a scrollable event list on the left and a detail
inspector on the right.

**Title bar / toolbar** — Click and drag anywhere on the title bar or the
filter toolbar (the full 54px combined height) to move the window. The area
labelled `:: drag ::` is the clearest drag zone, but the whole top bar works.
Press `[Insert]` to toggle visibility.

**`|| PAUSE` / `> RESUME`** — Freezes the list in place so you can click rows.
New events keep being captured in the background; a `+N new` counter appears.
Click again or press `>> Latest` to resume and jump to the newest event.

**`CLR`** — Clears all captured events from memory.

**`X`** — Hides the window (same as Insert).

**Filter buttons** — `ALL EVENTS`, `S -> C`, `C -> S` filter the list without
losing captured data. Chat entries only appear in ALL.

**List rows** — Click any row to load it into the detail panel. Single-click
`^` or `v` to scroll 3 rows; hold either button for continuous smooth scroll.
`>> Latest` jumps to the bottom and unpauses.

**Fields panel** — Read-only display of all detected NetMessage keys and their
values with types.

**Replay Script panel** — Read-only display of a ready-to-run Lua script.
Click **Copy Script** then press `Ctrl+A` followed by `Ctrl+C` to copy it.

**polyhack-only buttons** (only shown inside the polyhack executor):

| Button | Action |
|---|---|
| `Execute` | Runs the replay script immediately via `loadstring()` |
| `FireClick` | Calls `fireclickdetector()` on the Instance in the event's fields |
| `SendChat` | Posts `[Spy] direction eventName t=Xs` to game chat |

---

## What gets captured

### NetworkEvent  (primary capture target)

Polytoria games communicate between server and client using `NetworkEvent`
objects. Each carries a `NetMessage` with typed key-value fields.

| Direction | How it works | Label |
|---|---|---|
| Server -> Client | Server calls `InvokeClients(msg)` or `InvokeClient(msg, player)`. `InvokedClient` fires on the LocalScript. | **S->C** (green) |
| Client -> Server | Local player calls `netEvent:InvokeServer(msg)`. Captured via a metatable wrap on `InvokeServer`. | **C->S** (orange) |

**Important limitations:**
- `C->S` only captures **your own** outbound calls. Other players' calls
  happen server-side and are invisible to a LocalScript.
- The metatable hook for `C->S` may silently fail if the sandbox restricts
  metatable access, in which case only `S->C` events will be captured.

### Signal

`Signal` objects carry variadic arguments rather than a `NetMessage`. Hooked
via the same approach as NetworkEvent.

| Direction | Label |
|---|---|
| `Signal.Invoked` fires on client | **S->C** |
| Local script calls `signal:Invoke(...)` | **C->S** |

Arguments are logged as `arg1`, `arg2`, etc. with their Lua type.

### Chat

Every `Player.Chatted` event visible to the client is captured, including all
players currently in the server.

| What | Label |
|---|---|
| Any player sends a chat message | **CHAT** (blue) |

---

## Scan coverage

On startup PolyRemoteSpy scans all descendants of:

- `game["Environment"]`
- `game["ScriptService"]`
- `game["PlayerGUI"]`

`ChildAdded` watchers on each root automatically hook objects created later.
To scan a custom container, append it to `scanRoots` near the bottom of the
script:

```lua
local scanRoots = { Env }
pcall(function() table.insert(scanRoots, game["ScriptService"]) end)
pcall(function() table.insert(scanRoots, game["Hidden"]) end)  -- add custom root
```

---

## NetMessage field probing

NetMessage has no key-enumeration API — the spy must probe every key name it
wants to find. PolyRemoteSpy probes **536 key names** covering all common
patterns across Polytoria game genres. All single-letter keys (a–z) are
included, as are genre-specific names for tycoon, RPG, shooter, simulator,
and social games.

### Detection limits

| Value type | Detected | Reason |
|---|---|---|
| Non-empty string | Yes | `GetString` returns `""` for absent keys |
| Non-zero int | Yes | `GetInt` returns `0` for absent keys |
| Non-zero float | Yes | `GetNumber` returns `0` for absent keys |
| Boolean `true` | Yes | Unambiguous |
| Boolean `false` | **No** | Indistinguishable from absent key |
| Int / float `0` | **No** | Indistinguishable from absent key |
| Empty string `""` | **No** | Indistinguishable from absent key |
| Vector3, Vector2, Color, Instance | Yes | Returns `nil` for absent keys |

### Fields show "(no keys found from probe list)"

The event fired — it just uses key names not in the 536-entry probe list.
Add the missing names to `EXTRA_KEYS` near the top of the script:

```lua
local EXTRA_KEYS = {
    "purchaseId", "buildingType", "myCustomKey",
}
```

---

## Replay script format

Every captured event generates a complete, ready-to-run Lua script. The
script includes a header with the event name, direction, full path, and
capture timestamp, followed by all detected field writes and the correct
invoke call.

Example C->S (immediately runnable):

```lua
-- PolyRemoteSpy  capture
-- Event      : BuyEvent
-- Direction  : C->S
-- Path       : game["Environment"]["Tycoon 1"]["BuyEvent"]
-- Captured   : t=42.18s
--
-- HOW TO REPLAY: run this in a LocalScript or via Execute button.

local netEvent = game["Environment"]["Tycoon 1"]["BuyEvent"]
local msg = NetMessage.New()

msg:AddString("type", "buy")
msg:AddInt("itemId", 5)
msg:AddVector3("position", Vector3.New(10.00, 0.50, -3.20))

netEvent:InvokeServer(msg)
```

Example S->C (requires a server Script to replay):

```lua
-- HOW TO REPLAY: paste this into a SERVER Script (ScriptInstance).

local netEvent = game["ScriptService"]["Events"]["PlaySound"]
local msg = NetMessage.New()

msg:AddString("name", "ChaChing")
-- [Instance] "source" was: Part("Process")
-- msg:AddInstance("source", game["Environment"]:FindChild("..."))  -- resolve manually

-- From a server Script:
-- netEvent:InvokeClients(msg)                          -- all players
-- netEvent:InvokeClient(msg, game["Players"]["name"])  -- one player
```

All 8 NetMessage types are fully generated: `string`, `int`, `float`, `bool`,
`Vector3`, `Vector2`, `Color`. Instance fields are included as comments with
the captured class name and instance name, since instances cannot be
serialised to Lua source.

---

## Known limitations

| Limitation | Reason |
|---|---|
| `C->S` may not capture in all games | Requires metatable access to `NetworkEvent.__index` |
| Other players' `C->S` calls invisible | `InvokeServer` is server-side; LocalScript cannot see it |
| Keys with value `0`, `""`, or `false` not shown | Indistinguishable from absent keys in the probe API |
| Unknown key names not shown | Add them to `EXTRA_KEYS` |
| Signal args shown as `arg1`/`arg2` | Signal carries variadic values with no schema |
| Events outside the three scan roots missed | Append extra roots to `scanRoots` |

---

## Configuration

```lua
local CFG = {
    TOGGLE_KEY   = "Insert",  -- key to show/hide window
    MAX_LOGS     = 300,       -- maximum events kept in memory
    WIN_W        = 880,       -- window width in pixels
    WIN_H        = 490,       -- window height in pixels
    REFRESH_RATE = 0.30,      -- seconds between live list refreshes
}
```

---

## Polyhack executor functions used

| Function | Where used |
|---|---|
| `identifyexecutor()` | Startup detection, badge, extra buttons |
| `loadstring(src)` | Execute button — runs replay scripts live |
| `fireclickdetector(inst)` | FireClick button |
| `sendchat(msg)` | SendChat button |
| `equiptool` / `activatetool` / `unequiptool` / `serverequiptool` | Available inside generated replay scripts |
