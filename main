-- ╔═══════════════════════════════════════════════════════════════════╗
-- ║          PolyRemoteSpy  v7.1  –  Polytoria LocalScript            ║
-- ║  Full coverage: NetworkEvent + Signal + Chat · drag fix           ║
-- ╚═══════════════════════════════════════════════════════════════════╝
-- Lua 5.2 | Polytoria API
--
-- DRAG:
--   Click-and-hold anywhere on the title bar, then move the mouse.
--   Input.MousePosition is screen-space Y-DOWN (0,0 = top-left).
--   Win.PositionOffset is UI-space Y-UP, so the Y delta is negated.
--   Input.GetMouseButton(0) is checked every frame as a safety release.
--
-- POLYHACK FUNCTIONS (polytoria-executor docs):
--   loadstring(src)              compile Lua string into callable function
--   identifyexecutor()           returns "elcapor"
--   sendchat(msg)                send chat via ChatService
--   fireclickdetector(instance)  fire click detector on an Instance
--   equiptool(tool)              equip tool locally
--   unequiptool(tool)            unequip tool locally
--   activatetool(tool)           activate a tool (CmdActivate)
--   serverequiptool(player,tool) server-side equip
--   poop()                       debug – returns "Hello from C++!"

-- ============================================================
--  POLYHACK DETECTION
-- ============================================================
local IS_POLYHACK = false
pcall(function() IS_POLYHACK = (identifyexecutor() == "elcapor") end)

-- ============================================================
--  CONFIG
-- ============================================================
local CFG = {
    TOGGLE_KEY   = "Insert",
    MAX_LOGS     = 300,
    WIN_W        = 880,
    WIN_H        = 490,
    TITLE_H      = 30,
    TOOLBAR_H    = 24,
    ROW_H        = 28,
    ROW_INNER_H  = 27,
    LIST_W       = 238,
    REFRESH_RATE = 0.30,
    BADGE_W      = 34,
    BADGE_H      = 16,
}

local CONTENT_Y   = CFG.TITLE_H + CFG.TOOLBAR_H
local CONTENT_H   = CFG.WIN_H   - CONTENT_Y
local SCROLLBTN_H = 24
local LIST_USABLE = CONTENT_H   - SCROLLBTN_H
local MAX_ROWS    = math.floor(LIST_USABLE / CFG.ROW_H)
local DETAIL_X    = CFG.LIST_W  + 2
local DETAIL_W    = CFG.WIN_W   - DETAIL_X - 4

-- ============================================================
--  SERVICES
-- ============================================================
local PlayerGUI = game["PlayerGUI"]
local Env       = game["Environment"]

-- ============================================================
--  COLOURS
-- ============================================================
local C = {
    bg          = Color.New(0.058,0.058,0.088,0.97),
    titleBg     = Color.New(0.082,0.082,0.130,1.00),
    titleDragging=Color.New(0.108,0.108,0.175,1.00),
    tbBg        = Color.New(0.068,0.068,0.108,1.00),
    listBg      = Color.New(0.050,0.050,0.080,1.00),
    detailBg    = Color.New(0.060,0.060,0.095,1.00),
    rowA        = Color.New(0.072,0.072,0.115,1.00),
    rowB        = Color.New(0.088,0.088,0.140,1.00),
    rowSel      = Color.New(0.120,0.190,0.350,1.00),
    rowSelBdr   = Color.New(0.330,0.560,1.000,1.00),
    scrollBg    = Color.New(0.065,0.065,0.110,1.00),
    divider     = Color.New(0.150,0.150,0.248,1.00),
    border      = Color.New(0.190,0.190,0.308,1.00),
    panelDiv    = Color.New(0.170,0.170,0.268,1.00),
    sectBg      = Color.New(0.078,0.078,0.130,1.00),
    codeBg      = Color.New(0.042,0.042,0.068,1.00),
    s2cFg       = Color.New(0.250,0.910,0.450,1.00),
    c2sFg       = Color.New(1.000,0.670,0.190,1.00),
    s2cBadge    = Color.New(0.058,0.210,0.095,1.00),
    c2sBadge    = Color.New(0.268,0.135,0.025,1.00),
    filtOn      = Color.New(0.150,0.260,0.480,1.00),
    filtOff     = Color.New(0.085,0.085,0.155,1.00),
    txtMain     = Color.New(0.925,0.925,0.965,1.00),
    txtDim      = Color.New(0.430,0.450,0.565,1.00),
    txtDragIdle = Color.New(0.280,0.290,0.380,1.00),
    txtDragAct  = Color.New(0.948,0.585,0.138,1.00),
    txtPath     = Color.New(0.390,0.615,0.995,1.00),
    txtField    = Color.New(0.545,0.860,0.585,1.00),
    txtCode     = Color.New(0.745,0.850,0.968,1.00),
    txtSect     = Color.New(0.470,0.495,0.648,1.00),
    phBadge     = Color.New(0.138,0.070,0.260,1.00),
    phText      = Color.New(0.728,0.468,0.990,1.00),
    btn         = Color.New(0.105,0.105,0.188,1.00),
    btnBdr      = Color.New(0.188,0.188,0.338,1.00),
    btnPause    = Color.New(0.530,0.318,0.045,1.00),
    btnResume   = Color.New(0.068,0.298,0.108,1.00),
    btnRed      = Color.New(0.380,0.078,0.078,1.00),
    btnCopy     = Color.New(0.095,0.188,0.365,1.00),
    btnCopyOk   = Color.New(0.058,0.248,0.095,1.00),
    btnExec     = Color.New(0.148,0.090,0.278,1.00),
    btnExecOk   = Color.New(0.055,0.208,0.128,1.00),
    btnFire     = Color.New(0.268,0.098,0.098,1.00),
    pending     = Color.New(0.948,0.585,0.138,1.00),
    none        = Color.New(0,0,0,0),
}

-- ============================================================
--  Y-UP HELPERS
-- ============================================================
local function yup(pH, topY, h) return pH - topY - h end
local function V(x, y)          return Vector2.New(x, y) end

-- ============================================================
--  WIDGET FACTORIES
-- ============================================================
local function mkView(par, x, topY, w, h, pH, bg, br, bw, cr, clip)
    local v = Instance.New("UIView", par)
    v.PivotPoint=V(0,0); v.PositionRelative=V(0,0)
    v.PositionOffset=V(x, yup(pH,topY,h)); v.SizeRelative=V(0,0); v.SizeOffset=V(w,h)
    v.Color=bg or C.none; v.BorderColor=br or C.none; v.BorderWidth=bw or 0
    v.CornerRadius=cr or 0; v.ClipDescendants=clip or false
    return v
end

local function mkLabel(par, txt, x, topY, w, h, pH, col, fs, jt, va)
    local l = Instance.New("UILabel", par)
    l.PivotPoint=V(0,0); l.PositionRelative=V(0,0)
    l.PositionOffset=V(x, yup(pH,topY,h)); l.SizeRelative=V(0,0); l.SizeOffset=V(w,h)
    l.Color=C.none; l.BorderColor=C.none; l.BorderWidth=0; l.ClipDescendants=true
    l.Text=txt; l.TextColor=col or C.txtMain; l.FontSize=fs or 10; l.MaxFontSize=fs or 10
    l.Font=TextFontPreset.RobotoMono; l.AutoSize=false
    l.JustifyText=jt or TextJustify.Left; l.VerticalAlign=va or TextVerticalAlign.Middle
    return l
end

local function mkBtn(par, txt, x, topY, w, h, pH, bg, fs)
    local b = Instance.New("UIButton", par)
    b.PivotPoint=V(0,0); b.PositionRelative=V(0,0)
    b.PositionOffset=V(x, yup(pH,topY,h)); b.SizeRelative=V(0,0); b.SizeOffset=V(w,h)
    b.Color=bg or C.btn; b.BorderColor=C.btnBdr; b.BorderWidth=1; b.CornerRadius=3
    b.Text=txt; b.TextColor=C.txtMain; b.FontSize=fs or 9; b.MaxFontSize=fs or 9
    b.Font=TextFontPreset.RobotoMono
    b.JustifyText=TextJustify.Center; b.VerticalAlign=TextVerticalAlign.Middle; b.AutoSize=false
    return b
end

local function mkInput(par, x, topY, w, h, pH, bg, fs, multi)
    local i = Instance.New("UITextInput", par)
    i.PivotPoint=V(0,0); i.PositionRelative=V(0,0)
    i.PositionOffset=V(x, yup(pH,topY,h)); i.SizeRelative=V(0,0); i.SizeOffset=V(w,h)
    i.Color=bg or C.codeBg; i.BorderColor=C.divider; i.BorderWidth=1; i.CornerRadius=2
    i.Text=""; i.TextColor=C.txtCode; i.FontSize=fs or 9; i.MaxFontSize=fs or 9
    i.Font=TextFontPreset.RobotoMono; i.AutoSize=false
    i.IsMultiline=multi or false; i.IsReadOnly=false
    i.JustifyText=TextJustify.Left; i.VerticalAlign=TextVerticalAlign.Top
    return i
end

local function hRule(par, topY, pH)
    local v = Instance.New("UIView", par)
    v.PivotPoint=V(0,0); v.PositionRelative=V(0,0)
    v.PositionOffset=V(0, yup(pH,topY,1)); v.SizeRelative=V(1,0); v.SizeOffset=V(0,1)
    v.Color=C.divider; v.BorderWidth=0
    return v
end

-- ============================================================
--  ROOT WINDOW  (centred; PositionOffset accumulates drag)
-- ============================================================
local Win = Instance.New("UIView", PlayerGUI)
Win.Name="PolyRemoteSpy"; Win.PivotPoint=V(0.5,0.5); Win.PositionRelative=V(0.5,0.5)
Win.PositionOffset=V(0,0); Win.SizeRelative=V(0,0); Win.SizeOffset=V(CFG.WIN_W,CFG.WIN_H)
Win.Color=C.bg; Win.BorderColor=C.border; Win.BorderWidth=1; Win.CornerRadius=6
Win.ClipDescendants=false
local WH = CFG.WIN_H

-- ────────────────────────────────────────────────────────────
--  TITLE BAR  ← also the drag handle
-- ────────────────────────────────────────────────────────────
local TitleBar = mkView(Win, 0, 0, CFG.WIN_W, CFG.TITLE_H, WH, C.titleBg, C.none, 0, 6, false)
local TH = CFG.TITLE_H

local titleStartX = 8
if IS_POLYHACK then
    local phBg = mkView(TitleBar, 6, 6, 68, 18, TH, C.phBadge, C.none, 0, 3, false)
    mkLabel(phBg, "polyhack", 0, 0, 68, 18, 18, C.phText, 7,
            TextJustify.Center, TextVerticalAlign.Middle)
    titleStartX = 78
end

mkLabel(TitleBar, "PolyRemoteSpy", titleStartX, 0, 148, TH, TH,
        C.txtMain, 11, TextJustify.Left, TextVerticalAlign.Middle)

local HookLbl = mkLabel(TitleBar, "0 hooks", titleStartX+151, 0, 82, TH, TH,
                         C.txtDim, 8, TextJustify.Left, TextVerticalAlign.Middle)
local CapLbl  = mkLabel(TitleBar, "0 captured", titleStartX+236, 0, 104, TH, TH,
                         C.txtDim, 8, TextJustify.Left, TextVerticalAlign.Middle)

-- Drag status hint (idle: dim, dragging: amber)
local DragHint = mkLabel(TitleBar, ":: drag ::", titleStartX+344, 0, 92, TH, TH,
                           C.txtDragIdle, 8, TextJustify.Left, TextVerticalAlign.Middle)

local PauseBtn = mkBtn(TitleBar, "|| PAUSE",  CFG.WIN_W-210, 5, 100, 20, TH, C.btnPause, 9)
local ClearBtn = mkBtn(TitleBar, "CLR",       CFG.WIN_W-106, 5,  42, 20, TH, C.btn,      9)
local CloseBtn = mkBtn(TitleBar, "X",         CFG.WIN_W- 60, 5,  28, 20, TH, C.btnRed,   9)

hRule(TitleBar, TH-1, TH)

-- ────────────────────────────────────────────────────────────
--  TOOLBAR
-- ────────────────────────────────────────────────────────────
local Toolbar = mkView(Win, 0, CFG.TITLE_H, CFG.WIN_W, CFG.TOOLBAR_H, WH, C.tbBg, C.none, 0, 0, false)
local TBH = CFG.TOOLBAR_H
hRule(Toolbar, TBH-1, TBH)

local BtnAll = mkBtn(Toolbar, "ALL EVENTS",  6,   3, 82, 18, TBH, C.filtOn,  8)
local BtnS2C = mkBtn(Toolbar, "S -> C",      92,  3, 60, 18, TBH, C.filtOff, 8)
local BtnC2S = mkBtn(Toolbar, "C -> S",      156, 3, 60, 18, TBH, C.filtOff, 8)
mkLabel(Toolbar, "[Insert] toggle", CFG.WIN_W-120, 3, 114, 18, TBH,
        C.txtDim, 8, TextJustify.Right, TextVerticalAlign.Middle)

-- ────────────────────────────────────────────────────────────
--  PANEL DIVIDER
-- ────────────────────────────────────────────────────────────
mkView(Win, CFG.LIST_W, CONTENT_Y, 2, CONTENT_H, WH, C.panelDiv)

-- ────────────────────────────────────────────────────────────
--  LIST AREA
-- ────────────────────────────────────────────────────────────
local LIST_TOP  = CONTENT_Y
local ListOuter = mkView(Win, 0, LIST_TOP, CFG.LIST_W, LIST_USABLE, WH,
                          C.listBg, C.none, 0, 0, true)

-- ────────────────────────────────────────────────────────────
--  FIXED ROW POOL  (created once, text-only updates)
-- ────────────────────────────────────────────────────────────
local rowSlots = {}

for slot = 0, MAX_ROWS-1 do
    local RH = CFG.ROW_INNER_H

    local btn = Instance.New("UIButton", ListOuter)
    btn.PivotPoint=V(0,0); btn.PositionRelative=V(0,0)
    btn.PositionOffset=V(0, yup(LIST_USABLE, slot*CFG.ROW_H, RH))
    btn.SizeRelative=V(0,0); btn.SizeOffset=V(CFG.LIST_W,RH)
    btn.Color=(slot%2==0) and C.rowA or C.rowB
    btn.BorderColor=C.none; btn.BorderWidth=0; btn.CornerRadius=0
    btn.Text=""; btn.FontSize=1; btn.AutoSize=false; btn.ClipDescendants=true

    local cv = Instance.New("UIView", btn)
    cv.PivotPoint=V(0,0); cv.PositionRelative=V(0,0); cv.PositionOffset=V(0,0)
    cv.SizeRelative=V(1,1); cv.SizeOffset=V(0,0)
    cv.Color=C.none; cv.BorderWidth=0; cv.ClipDescendants=true

    local bdgBg = Instance.New("UIView", cv)
    bdgBg.PivotPoint=V(0,0); bdgBg.PositionRelative=V(0,0)
    bdgBg.PositionOffset=V(3, yup(RH,6,CFG.BADGE_H))
    bdgBg.SizeRelative=V(0,0); bdgBg.SizeOffset=V(CFG.BADGE_W,CFG.BADGE_H)
    bdgBg.Color=C.s2cBadge; bdgBg.BorderWidth=0; bdgBg.CornerRadius=2; bdgBg.ClipDescendants=false

    local bdgLbl = Instance.New("UILabel", bdgBg)
    bdgLbl.PivotPoint=V(0,0); bdgLbl.PositionRelative=V(0,0); bdgLbl.PositionOffset=V(0,0)
    bdgLbl.SizeRelative=V(1,1); bdgLbl.SizeOffset=V(0,0)
    bdgLbl.Color=C.none; bdgLbl.BorderWidth=0; bdgLbl.ClipDescendants=false
    bdgLbl.Text="S->C"; bdgLbl.TextColor=C.s2cFg; bdgLbl.FontSize=7; bdgLbl.MaxFontSize=7
    bdgLbl.Font=TextFontPreset.RobotoMono; bdgLbl.AutoSize=false
    bdgLbl.JustifyText=TextJustify.Center; bdgLbl.VerticalAlign=TextVerticalAlign.Middle

    local nameLbl = Instance.New("UILabel", cv)
    nameLbl.PivotPoint=V(0,0); nameLbl.PositionRelative=V(0,0)
    nameLbl.PositionOffset=V(CFG.BADGE_W+6, yup(RH,3,21)); nameLbl.SizeRelative=V(0,0)
    nameLbl.SizeOffset=V(CFG.LIST_W-CFG.BADGE_W-58,21)
    nameLbl.Color=C.none; nameLbl.BorderWidth=0; nameLbl.ClipDescendants=true
    nameLbl.Text=""; nameLbl.TextColor=C.s2cFg; nameLbl.FontSize=9; nameLbl.MaxFontSize=9
    nameLbl.Font=TextFontPreset.RobotoMono; nameLbl.AutoSize=false
    nameLbl.JustifyText=TextJustify.Left; nameLbl.VerticalAlign=TextVerticalAlign.Middle

    local timeLbl = Instance.New("UILabel", cv)
    timeLbl.PivotPoint=V(0,0); timeLbl.PositionRelative=V(0,0)
    timeLbl.PositionOffset=V(CFG.LIST_W-52, yup(RH,3,21)); timeLbl.SizeRelative=V(0,0)
    timeLbl.SizeOffset=V(50,21)
    timeLbl.Color=C.none; timeLbl.BorderWidth=0; timeLbl.ClipDescendants=false
    timeLbl.Text=""; timeLbl.TextColor=C.txtDim; timeLbl.FontSize=7; timeLbl.MaxFontSize=7
    timeLbl.Font=TextFontPreset.RobotoMono; timeLbl.AutoSize=false
    timeLbl.JustifyText=TextJustify.Right; timeLbl.VerticalAlign=TextVerticalAlign.Middle

    local sep = Instance.New("UIView", btn)
    sep.PivotPoint=V(0,0); sep.PositionRelative=V(0,0); sep.PositionOffset=V(0,0)
    sep.SizeRelative=V(0,0); sep.SizeOffset=V(CFG.LIST_W,1)
    sep.Color=C.divider; sep.BorderWidth=0; sep.ClipDescendants=false

    btn.Visible = false

    rowSlots[slot+1] = {
        btn=btn, clipV=cv, bdgBg=bdgBg, bdgLbl=bdgLbl,
        nameLbl=nameLbl, timeLbl=timeLbl, wired=false
    }
end

-- ────────────────────────────────────────────────────────────
--  SCROLL BAR
-- ────────────────────────────────────────────────────────────
local SCROLL_TOP = LIST_TOP + LIST_USABLE
local ScrollBar  = mkView(Win, 0, SCROLL_TOP, CFG.LIST_W, SCROLLBTN_H, WH,
                           C.scrollBg, C.none, 0, 0, false)
hRule(ScrollBar, 0, SCROLLBTN_H)

local BtnUp     = mkBtn(ScrollBar, "^",           4,  4, 28, 16, SCROLLBTN_H, C.btn, 9)
local BtnDown   = mkBtn(ScrollBar, "v",          36,  4, 28, 16, SCROLLBTN_H, C.btn, 9)
local BtnLatest = mkBtn(ScrollBar, ">> Latest",  68,  4, 80, 16, SCROLLBTN_H, C.btn, 8)
local PendLbl   = mkLabel(ScrollBar, "", 152, 5, CFG.LIST_W-156, 14, SCROLLBTN_H,
                           C.pending, 8, TextJustify.Left, TextVerticalAlign.Middle)

-- ────────────────────────────────────────────────────────────
--  DETAIL PANEL
-- ────────────────────────────────────────────────────────────
local DH = CONTENT_H
local DW = DETAIL_W

local DetailBg = mkView(Win, DETAIL_X, CONTENT_Y, DW+4, DH, WH, C.detailBg, C.none, 0, 0, false)

local HDR_H  = 36
local DdgBg  = mkView(DetailBg, 8, 9, 38, 18, DH, C.s2cBadge, C.none, 0, 3, false)
local DdgLbl = mkLabel(DdgBg, "S->C", 0, 0, 38, 18, 18, C.s2cFg, 8,
                        TextJustify.Center, TextVerticalAlign.Middle)
local DName  = mkLabel(DetailBg, "-- select an event --", 54, 0, DW-108, HDR_H, DH,
                        C.txtDim, 12, TextJustify.Left, TextVerticalAlign.Middle)
local DTime  = mkLabel(DetailBg, "", DW-50, 0, 48, HDR_H, DH,
                        C.txtDim, 8, TextJustify.Right, TextVerticalAlign.Middle)
hRule(DetailBg, HDR_H, DH)

local PATH_TOP = HDR_H+1; local PATH_H = 18
local DPath = mkLabel(DetailBg, "", 8, PATH_TOP, DW-10, PATH_H, DH,
                       C.txtPath, 8, TextJustify.Left, TextVerticalAlign.Middle)
hRule(DetailBg, PATH_TOP+PATH_H, DH)

local F_TOP = PATH_TOP+PATH_H+1; local F_HDR_H = 16; local F_BODY_H = 86
mkView(DetailBg, 0, F_TOP, DW+4, F_HDR_H, DH, C.sectBg)
mkLabel(DetailBg, "  FIELDS", 0, F_TOP, DW, F_HDR_H, DH,
        C.txtSect, 8, TextJustify.Left, TextVerticalAlign.Middle)
local DFields = mkInput(DetailBg, 4, F_TOP+F_HDR_H, DW-2, F_BODY_H, DH, C.codeBg, 9, true)
DFields.TextColor = C.txtField
DFields.IsReadOnly = true   -- display only, cannot type
hRule(DetailBg, F_TOP+F_HDR_H+F_BODY_H, DH)

local S_TOP    = F_TOP+F_HDR_H+F_BODY_H+1
local S_HDR_H  = 16
local BTN_H    = 24
local BTN_TOP  = DH - BTN_H - 2
local S_BODY_H = BTN_TOP - S_TOP - S_HDR_H - 2
mkView(DetailBg, 0, S_TOP, DW+4, S_HDR_H, DH, C.sectBg)
mkLabel(DetailBg, "  REPLAY SCRIPT  --  Copy Script button below",
        0, S_TOP, DW, S_HDR_H, DH, C.txtSect, 8, TextJustify.Left, TextVerticalAlign.Middle)
local DScript = mkInput(DetailBg, 4, S_TOP+S_HDR_H, DW-2, S_BODY_H, DH, C.codeBg, 9, true)
DScript.IsReadOnly = true   -- display only, use Copy Script to copy
hRule(DetailBg, BTN_TOP, DH)

local CopyBtn = mkBtn(DetailBg, "Copy Script",  4,   BTN_TOP+2, 92, BTN_H-4, DH, C.btnCopy, 9)
local ExecBtn = mkBtn(DetailBg, "Execute",      100, BTN_TOP+2, 76, BTN_H-4, DH, C.btnExec, 9)
local FireBtn = mkBtn(DetailBg, "FireClick",    180, BTN_TOP+2, 78, BTN_H-4, DH, C.btnFire, 9)
local ChatBtn = mkBtn(DetailBg, "SendChat",     262, BTN_TOP+2, 74, BTN_H-4, DH, C.btn,     9)
ExecBtn.Visible = IS_POLYHACK
FireBtn.Visible = IS_POLYHACK
ChatBtn.Visible = IS_POLYHACK

local DHint = mkLabel(DetailBg, "<-- Click an event row to inspect it",
                       0, 0, DW+4, DH, DH,
                       C.txtDim, 10, TextJustify.Center, TextVerticalAlign.Middle)

-- ============================================================
--  NET-MESSAGE INTROSPECTION
-- ============================================================
-- NetMessage has no key enumeration API.  We must probe every possible
-- key name.  Missing keys return a type default (e.g. "" / 0 / nil).
-- We detect "key not found" by checking for default empty values, so
-- genuine keys with value "" or 0 will be missed — this is unavoidable.
-- To help a game that uses unusual key names, add them to EXTRA_KEYS below.

-- Common single-letter keys used in tightly-packed messages
local SINGLE_KEYS = {"a","b","c","d","e","f","g","h","i","j","k","l","m",
                     "n","o","p","q","r","s","t","u","v","w","x","y","z"}

-- Main probe list — 300+ keys covering most Polytoria game genres
local PROBE_KEYS = {
    -- Core / universal
    "key","value","val","data","payload","msg","packet","raw",
    "id","Id","ID","uid","uuid","gid","nid","pid","rid","sid",
    "name","Name","label","title","tag","slug","alias","ref",
    "type","Type","kind","category","class","subtype","variant","format",
    "action","Action","command","cmd","op","method","func","call","trigger",
    "event","eventType","eventName","signal","topic","channel","room",
    "code","result","response","reply","output","answer","feedback","ack",
    "status","state","phase","stage","step","mode","flags","flag","mask",
    "error","err","reason","cause","detail","message","text","body","info",
    "enabled","disabled","active","inactive","visible","hidden","locked",
    "success","failed","valid","invalid","done","finished","started",
    "tick","timestamp","time","date","frame","duration","delay","interval",
    "index","count","num","number","total","max","min","limit","cap",
    "version","build","revision",

    -- Player / identity
    "player","playerName","playerID","playerId","username","user","userId",
    "userId","gameId","placeId","serverId","roomId","sessionId","lobbyId",
    "owner","creator","sender","source","target","recipient","dest",
    "team","teamId","faction","side","alliance","guild","clan","group",
    "role","rank","tier","grade","class","badge","title","prefix","suffix",

    -- Health / combat
    "health","hp","maxHealth","maxHp","shield","armor","defence","defense",
    "damage","dmg","dps","hit","crit","block","dodge","miss","heal","regen",
    "alive","dead","isAlive","isDead","respawn","invincible","godmode",
    "kill","killerId","victim","victimId","attacker","weapon","weaponId",
    "ammo","bullets","magazine","clip","reload","reloadTime","fireRate",
    "mana","mp","rage","energy","stamina","stamina2","fuel","charge",

    -- Economy / currency
    "cash","coins","gold","gems","tokens","credits","points","score",
    "currency","amount","price","cost","reward","bonus","penalty","tax",
    "balance","wallet","bank","income","earn","revenue","profit","loss",
    "buy","sell","trade","bid","ask","fee","tip","refund","discount",
    "quantity","qty","stack","lots","batch","bundle","set",

    -- Progression
    "level","lvl","xp","exp","experience","prestige","season","rank",
    "progress","percent","percentage","ratio","fraction","t","alpha","lerp",
    "upgrade","tier1","tier2","tier3","tier4","tier5","maxTier","upgradeId",
    "quest","mission","task","objective","challenge","achievement","badge",
    "unlocked","purchased","owned","completed","collected","activated",

    -- Tycoon / simulation (very common in Polytoria games)
    "tycoon","tycoonId","plot","plotId","base","baseId",
    "building","buildingId","buildingType","buildingLevel",
    "machine","machineId","conveyor","dropper","collector","pad",
    "button","cashier","register","station","terminal",
    "production","output","input","throughput","efficiency",
    "worker","workerCount","capacity","storage","stockpile",

    -- World / physics
    "position","pos","x","y","z","w","px","py","pz",
    "rotation","rot","rx","ry","rz","euler","yaw","pitch","roll",
    "size","scale","scaleX","scaleY","scaleZ","radius","width","height","depth",
    "velocity","vel","vx","vy","vz","speed","acceleration","force","impulse",
    "direction","dir","forward","up","right","normal","axis",
    "distance","dist","range","length","magnitude","offset","origin",
    "angle","fov","zoom","weight","mass","density","gravity",
    "grounded","flying","jumping","falling","moving","sitting","crouching",

    -- Visual / effects
    "color","colour","colorR","colorG","colorB","r","g","b","a",
    "transparency","opacity","alpha2","visible2","show","hide",
    "sound","soundId","soundName","volume","pitch","looped","loop",
    "animation","animId","animName","anim","emote","gesture",
    "effect","effectId","particle","particleId","vfx","sfx",
    "tween","tweenId","tweenTime","tweenStyle","tweenDir","easing",
    "flash","shake","pulse","spin","fade","slide","bounce","pop",

    -- Inventory / items
    "item","itemId","itemName","itemType","itemData","loot","drop","pickup",
    "tool","toolId","toolName","gear","equipment","loadout","hotbar",
    "slot","slotId","slotIndex","bag","backpack","chest","vault","shop",
    "rarity","quality","durability","condition","enchant","modifier",
    "give","take","transfer","move","swap","combine","split","merge",

    -- UI / text
    "msg2","notice","alert","warning","hint","tooltip","popup","banner",
    "prompt","question","answer2","option","choice","vote","poll",
    "chat","chatMsg","chatColor","whisper","broadcast","announce",
    "notification","notif","toast","dialog","modal","menu","tab",
    "header","footer","caption","subtitle","paragraph","line","word",

    -- Misc
    "n","v2","k2","p","q","m","s","c","d","e","f","h","o","u",
    "arg","arg1","arg2","arg3","param","param1","param2","param3",
    "extra","misc","other","custom","payload2","meta","context",
    "obj","object","inst","instance","ref2","ptr","handle",
    "list","array","map","dict","set2","pair","tuple",
    "true","false","yes","no","on","off",
}

-- Add single-letter keys to the list
for _,k in ipairs(SINGLE_KEYS) do table.insert(PROBE_KEYS, k) end

-- EXTRA_KEYS: add any game-specific key names you've identified here
local EXTRA_KEYS = {}
for _,k in ipairs(EXTRA_KEYS) do table.insert(PROBE_KEYS, k) end

-- Deduplicate the final list
local _seen = {}; local _deduped = {}
for _,k in ipairs(PROBE_KEYS) do
    if not _seen[k] then _seen[k]=true; table.insert(_deduped,k) end
end
PROBE_KEYS = _deduped

local function probeMsg(msg)
    if type(msg)~="userdata" and type(msg)~="table" then return {} end
    local out = {}
    for _, k in ipairs(PROBE_KEYS) do
        local ok, v

        -- Each type is tried in order; once a match is found the rest are skipped.
        -- No goto — fully Lua 5.1 / loadstring compatible.
        ok,v = pcall(function() return msg:GetString(k) end)
        if ok and type(v)=="string" and v~="" then
            out[k] = {t="string", v='"'..v..'"'}
        else
            ok,v = pcall(function() return msg:GetInt(k) end)
            if ok and type(v)=="number" and v~=0 then
                out[k] = {t="int", v=tostring(math.floor(v))}
            else
                ok,v = pcall(function() return msg:GetNumber(k) end)
                if ok and type(v)=="number" and v~=0 then
                    out[k] = {t="float", v=string.format("%.4g",v)}
                else
                    ok,v = pcall(function() return msg:GetBool(k) end)
                    if ok and v==true then
                        out[k] = {t="bool", v="true"}
                    else
                        ok,v = pcall(function() return msg:GetVector3(k) end)
                        if ok and v then
                            out[k] = {t="V3", v=string.format("(%.3f, %.3f, %.3f)",v.x,v.y,v.z)}
                        else
                            ok,v = pcall(function() return msg:GetVector2(k) end)
                            if ok and v then
                                out[k] = {t="V2", v=string.format("(%.3f, %.3f)",v.x,v.y)}
                            else
                                ok,v = pcall(function() return msg:GetColor(k) end)
                                if ok and v then
                                    out[k] = {t="Color", v=string.format("(%.2f, %.2f, %.2f, %.2f)",v.r,v.g,v.b,v.a)}
                                else
                                    ok,v = pcall(function() return msg:GetInstance(k) end)
                                    if ok and v then
                                        out[k] = {t="Inst", v=v.ClassName..'("'..v.Name..'")'}
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
    end
    return out
end

local function buildFieldsTxt(fields)
    local lines={}
    for k,fd in pairs(fields) do
        table.insert(lines, string.format("  [%s]  %s  =  %s", fd.t, k, fd.v))
    end
    if #lines==0 then
        return "  (no keys found from probe list)\n"
            .. "  Add custom key names to EXTRA_KEYS near the top of the script.\n"
            .. "  Note: keys with value 0, value \"\", or value false cannot be detected."
    end
    table.sort(lines); return table.concat(lines,"\n")
end

local function buildScript(path, fields, dir, evName, evTime)
    local lines = {
        "-- PolyRemoteSpy  capture",
        "-- Event      : " .. tostring(evName),
        "-- Direction  : " .. tostring(dir),
        "-- Path       : " .. tostring(path),
        "-- Captured   : t=" .. tostring(evTime) .. "s",
        "--",
    }

    if dir == "C->S" then
        table.insert(lines, "-- HOW TO REPLAY: run this in a LocalScript or via Execute button.")
        table.insert(lines, "-- The script sends the same message to the server that was originally captured.")
    elseif dir == "S->C" then
        table.insert(lines, "-- HOW TO REPLAY: paste this into a SERVER Script (ScriptInstance).")
        table.insert(lines, "-- Then call InvokeClient(msg, player) or InvokeClients(msg) to push to clients.")
    elseif dir == "CHAT" then
        table.insert(lines, "-- HOW TO REPLAY: run via Execute button (polyhack) or use sendchat().")
        table.insert(lines, "-- sendchat(\"" .. tostring(evName) .. ": <your message>\")")
        table.insert(lines, "return  -- CHAT entries have no NetworkEvent to replay")
    end

    table.insert(lines, "")
    table.insert(lines, "local netEvent = " .. tostring(path))
    table.insert(lines, "local msg = NetMessage.New()")
    table.insert(lines, "")

    -- Sort keys for deterministic output
    local sorted = {}
    for k, fd in pairs(fields) do table.insert(sorted, {k=k, fd=fd}) end
    table.sort(sorted, function(a,b) return a.k < b.k end)

    for _, pair in ipairs(sorted) do
        local k, fd = pair.k, pair.fd
        if fd.t == "string" then
            table.insert(lines, 'msg:AddString("'..k..'", '..fd.v..')')
        elseif fd.t == "int" then
            table.insert(lines, 'msg:AddInt("'..k..'", '..fd.v..')')
        elseif fd.t == "float" then
            table.insert(lines, 'msg:AddNumber("'..k..'", '..fd.v..')')
        elseif fd.t == "bool" then
            table.insert(lines, 'msg:AddBool("'..k..'", '..fd.v..')')
        elseif fd.t == "V3" then
            -- Parse "(x, y, z)" back into components for Vector3.New
            local x,y,z = fd.v:match("%(([^,]+),%s*([^,]+),%s*([^%)]+)%)")
            if x then
                table.insert(lines, 'msg:AddVector3("'..k..'", Vector3.New('..x..', '..y..', '..z..'))')
            else
                table.insert(lines, '-- [V3] msg:AddVector3("'..k..'", Vector3.New(...))')
            end
        elseif fd.t == "V2" then
            local x,y = fd.v:match("%(([^,]+),%s*([^%)]+)%)")
            if x then
                table.insert(lines, 'msg:AddVector2("'..k..'", Vector2.New('..x..', '..y..'))')
            else
                table.insert(lines, '-- [V2] msg:AddVector2("'..k..'", Vector2.New(...))')
            end
        elseif fd.t == "Color" then
            local r,g,b,a = fd.v:match("%(([^,]+),%s*([^,]+),%s*([^,]+),%s*([^%)]+)%)")
            if r then
                table.insert(lines, 'msg:AddColor("'..k..'", Color.New('..r..', '..g..', '..b..', '..a..'))')
            else
                table.insert(lines, '-- [Color] msg:AddColor("'..k..'", Color.New(r, g, b, a))')
            end
        elseif fd.t == "Inst" then
            -- Instance fields can only be serialized by path — comment with info
            table.insert(lines, '-- [Instance] "' .. k .. '" was: ' .. fd.v)
            table.insert(lines, '-- msg:AddInstance("'..k..'", game["Environment"]:FindChild("..."))  -- resolve manually')
        else
            table.insert(lines, '-- ['..fd.t..'] '..k..' = '..fd.v..' (add manually)')
        end
    end

    table.insert(lines, "")
    if dir == "C->S" then
        table.insert(lines, "netEvent:InvokeServer(msg)")
    elseif dir == "S->C" then
        table.insert(lines, "-- From a server Script:")
        table.insert(lines, "-- netEvent:InvokeClients(msg)                          -- all players")
        table.insert(lines, "-- netEvent:InvokeClient(msg, game[\"Players\"][\"name\"])  -- one player")
    end

    return table.concat(lines, "\n")
end

-- ============================================================
--  LOG STORE
-- ============================================================
local logs={}; local logVersion=0; local filterDir="ALL"

local function getPath(inst)
    local parts,cur={},inst
    while cur~=nil and cur~=game do
        table.insert(parts,1,'["'..cur.Name..'"]'); cur=cur.Parent
    end
    return "game"..table.concat(parts)
end

local function filteredLogs()
    if filterDir=="ALL" then return logs end
    local out={}
    for _,e in ipairs(logs) do if e.dir==filterDir then table.insert(out,e) end end
    return out
end

local function appendLog(dir, evName, evPath, fields)
    if #logs>=CFG.MAX_LOGS then table.remove(logs,1) end
    local t = string.format("%.2f", os.clock())
    table.insert(logs,{
        time=t, dir=dir, evName=evName, evPath=evPath, fields=fields,
        fieldsTxt=buildFieldsTxt(fields),
        script=buildScript(evPath, fields, dir, evName, t),
    })
    logVersion=logVersion+1
end

-- ============================================================
--  HOOK ENGINE
-- ============================================================
local hookedPaths={}; local hookedCount=0

local function hookNetworkEvent(netEv)
    if not netEv then return end
    local ok,cls=pcall(function() return netEv.ClassName end)
    if not ok or cls~="NetworkEvent" then return end
    local pathOk,path=pcall(getPath,netEv)
    if not pathOk or not path then return end
    if hookedPaths[path] then return end
    hookedPaths[path]=true; hookedCount=hookedCount+1

    -- S->C: InvokedClient fires on LocalScript when server pushes data down
    -- (server called InvokeClients or InvokeClient targeting this player)
    pcall(function()
        netEv.InvokedClient:Connect(function(_,msg)
            pcall(function() appendLog("S->C",netEv.Name,path,probeMsg(msg)) end)
        end)
    end)

    -- C->S: wrap InvokeServer via metatable so we capture our own outbound calls.
    -- NOTE: This only captures the LOCAL player's outbound traffic.
    -- Other players' InvokeServer calls are server-side and invisible to LocalScript.
    pcall(function()
        local mt=getmetatable(netEv); if not mt then return end
        local orig=rawget(mt,"__index"); if not orig then return end
        mt.__index=function(self,k)
            if k=="InvokeServer" then
                return function(_,msgArg)
                    pcall(function() appendLog("C->S",netEv.Name,path,probeMsg(msgArg)) end)
                    if type(orig)=="function" then return orig(self,"InvokeServer")(self,msgArg)
                    else return orig.InvokeServer(self,msgArg) end
                end
            end
            return type(orig)=="function" and orig(self,k) or orig[k]
        end
    end)
end

-- Hook a Signal instance (Polytoria Signal type: varargs Invoked/Invoke).
-- Signals carry no structured NetMessage so we log them with an empty fields table.
local hookedSignals = {}
local function hookSignal(sig)
    if not sig then return end
    local pathOk, path = pcall(getPath, sig)
    if not pathOk or not path then return end
    if hookedSignals[path] then return end
    hookedSignals[path] = true; hookedCount = hookedCount + 1

    -- Signal.Invoked fires when the signal is triggered (direction ambiguous from client;
    -- treat as S->C since client cannot call Invoke on a server-owned Signal)
    pcall(function()
        sig.Invoked:Connect(function(...)
            local args = {...}
            local fields = {}
            for i, v in ipairs(args) do
                local t = type(v)
                local display = t == "table" and "(table)" or tostring(v)
                fields["arg"..i] = { t=t, v=display }
            end
            pcall(function() appendLog("S->C", sig.Name, path, fields) end)
        end)
    end)

    -- Wrap Invoke via metatable (C->S: local script calling the signal)
    pcall(function()
        local mt = getmetatable(sig); if not mt then return end
        local orig = rawget(mt,"__index"); if not orig then return end
        mt.__index = function(self, k)
            if k == "Invoke" then
                return function(_, ...)
                    local args = {...}
                    local fields = {}
                    for i,v in ipairs(args) do
                        local t = type(v)
                        fields["arg"..i] = { t=t, v=tostring(v) }
                    end
                    pcall(function() appendLog("C->S", sig.Name, path, fields) end)
                    if type(orig)=="function" then return orig(self,"Invoke")(self, ...)
                    else return orig.Invoke(self, ...) end
                end
            end
            return type(orig)=="function" and orig(self,k) or orig[k]
        end
    end)
end

local function scanSubtree(root)
    if not root then return end
    local ok,children=pcall(function() return root:GetChildren() end)
    if not ok then return end
    pcall(function()
        local cls = root.ClassName
        if cls=="NetworkEvent" then hookNetworkEvent(root)
        elseif cls=="Signal"   then hookSignal(root)
        end
    end)
    for _,c in ipairs(children) do scanSubtree(c) end
end

local watchedRoots={}
local function watchRoot(root)
    if not root then return end
    local key=tostring(root); if watchedRoots[key] then return end
    watchedRoots[key]=true
    pcall(function()
        local function onAdded(c)
            if not c then return end
            pcall(function()
                local cls = c.ClassName
                if cls=="NetworkEvent" then hookNetworkEvent(c)
                elseif cls=="Signal"   then hookSignal(c)
                end
            end)
            pcall(function() c.ChildAdded:Connect(function(gc) onAdded(gc) end) end)
        end
        root.ChildAdded:Connect(onAdded)
    end)
end

-- ============================================================
--  RENDER STATE
-- ============================================================
local paused=false; local pendingCount=0; local viewStart=1
local displayedLogs={}; local displayVer=0; local drawnVer=-1
local lastRefreshT=0; local selectedLog=nil
local scrollHeld=0; local scrollAccum=0.0

-- ============================================================
--  DRAG STATE
-- ============================================================
local isDragging      = false
local dragStartMouse  = V(0,0)
local dragStartOffset = V(0,0)

-- Drag is wired directly on TitleBar and Toolbar so no invisible overlay
-- is needed. Both UIViews together give a 54px draggable zone.
-- Buttons are children and consume their own clicks first, so dragging
-- only activates on the empty background areas of both bars.

local function startDrag()
    isDragging      = true
    dragStartMouse  = Input.MousePosition
    dragStartOffset = Win.PositionOffset
    TitleBar.Color  = C.titleDragging
    DragHint.Text   = "  MOVING"
    DragHint.TextColor = C.txtDragAct
end

local function stopDrag()
    isDragging     = false
    TitleBar.Color = C.titleBg
    DragHint.Text  = ":: drag ::"
    DragHint.TextColor = C.txtDragIdle
end

TitleBar.MouseDown:Connect(startDrag)
TitleBar.MouseUp:Connect(stopDrag)
Toolbar.MouseDown:Connect(startDrag)
Toolbar.MouseUp:Connect(stopDrag)

-- ============================================================
--  DETAIL PANE
-- ============================================================
local function showDetail(entry)
    if not entry then
        DHint.Visible=true; DdgBg.Visible=false; DdgLbl.Visible=false
        DName.Text="-- select an event --"; DName.TextColor=C.txtDim
        DPath.Text=""; DFields.Text=""; DScript.Text=""; DTime.Text=""; return
    end
    DHint.Visible=false; DdgBg.Visible=true; DdgLbl.Visible=true
    local s2c=(entry.dir=="S->C")
    local ischat=(entry.dir=="CHAT")
    local issig=(entry.dir=="S->C" and false) -- handled below via path check; signals share S->C/C->S
    DdgBg.Color= ischat and Color.New(0.12,0.18,0.30,1) or (s2c and C.s2cBadge or C.c2sBadge)
    DdgLbl.Text=entry.dir; DdgLbl.TextColor= ischat and Color.New(0.55,0.75,1.00,1) or (s2c and C.s2cFg or C.c2sFg)
    DName.Text=entry.evName; DName.TextColor=C.txtMain
    DTime.Text="t="..entry.time.."s"
    DPath.Text=entry.evPath; DFields.Text=entry.fieldsTxt; DScript.Text=entry.script
end

-- ============================================================
--  LIST UPDATER
-- ============================================================
local function updateRows()
    local fl=displayedLogs; local count=#fl
    local maxStart=math.max(1,count-MAX_ROWS+1)
    if viewStart<1 then viewStart=1 end
    if viewStart>maxStart then viewStart=maxStart end

    for slot=0,MAX_ROWS-1 do
        local s=rowSlots[slot+1]; local idx=viewStart+slot; local e=fl[idx]
        if not e then
            s.btn.Visible=false
        else
            s.btn.Visible=true
            local s2c=(e.dir=="S->C"); local ischat=(e.dir=="CHAT"); local isSel=(e==selectedLog)
            s.btn.Color       = isSel and C.rowSel or ((slot%2==0) and C.rowA or C.rowB)
            s.btn.BorderColor = isSel and C.rowSelBdr or C.none
            s.btn.BorderWidth = isSel and 1 or 0
            local bdgBg = ischat and Color.New(0.12,0.18,0.30,1) or (s2c and C.s2cBadge or C.c2sBadge)
            local bdgFg = ischat and Color.New(0.55,0.75,1.00,1) or (s2c and C.s2cFg or C.c2sFg)
            s.bdgBg.Color     = bdgBg
            s.bdgLbl.Text     = ischat and "CHAT" or (s2c and "S->C" or "C->S")
            s.bdgLbl.TextColor= bdgFg
            s.nameLbl.Text    = e.evName
            s.nameLbl.TextColor=isSel and C.txtMain or bdgFg
            s.timeLbl.Text    = e.time.."s"
            if not s.wired then
                s.wired=true; local si=slot
                s.btn.Clicked:Connect(function()
                    local e2=displayedLogs[viewStart+si]
                    if not e2 then return end
                    selectedLog=e2; showDetail(e2); drawnVer=-1
                end)
            end
        end
    end

    HookLbl.Text=tostring(hookedCount).." hooks"
    CapLbl.Text =tostring(#logs).." captured"
    PendLbl.Text=(paused and pendingCount>0) and ("+"..tostring(pendingCount).." new") or ""
    drawnVer=displayVer
end

-- ============================================================
--  DISPLAY / PAUSE / FILTER helpers
-- ============================================================
local function jumpToBottom()
    viewStart=math.max(1,#displayedLogs-MAX_ROWS+1); drawnVer=-1
end

local function refreshDisplay()
    displayedLogs=filteredLogs(); displayVer=displayVer+1; pendingCount=0; jumpToBottom()
end

local function setPaused(v)
    paused=v
    if paused then PauseBtn.Text="> RESUME"; PauseBtn.Color=C.btnResume
    else PauseBtn.Text="|| PAUSE"; PauseBtn.Color=C.btnPause
         pendingCount=0; PendLbl.Text=""; refreshDisplay() end
end

local function setFilter(dir)
    filterDir=dir
    BtnAll.Color=(dir=="ALL")  and C.filtOn or C.filtOff
    BtnS2C.Color=(dir=="S->C") and C.filtOn or C.filtOff
    BtnC2S.Color=(dir=="C->S") and C.filtOn or C.filtOff
    selectedLog=nil; showDetail(nil); refreshDisplay(); drawnVer=-1
end

-- ============================================================
--  BUTTON WIRING
-- ============================================================
CloseBtn.Clicked:Connect(function() Win.Visible=not Win.Visible end)
PauseBtn.Clicked:Connect(function() setPaused(not paused) end)
ClearBtn.Clicked:Connect(function()
    logs={}; logVersion=0; displayedLogs={}; displayVer=displayVer+1
    viewStart=1; selectedLog=nil; pendingCount=0; showDetail(nil); drawnVer=-1
end)
BtnAll.Clicked:Connect(function() setFilter("ALL")  end)
BtnS2C.Clicked:Connect(function() setFilter("S->C") end)
BtnC2S.Clicked:Connect(function() setFilter("C->S") end)

BtnLatest.Clicked:Connect(function()
    if paused then
        paused=false; PauseBtn.Text="|| PAUSE"; PauseBtn.Color=C.btnPause
        pendingCount=0; PendLbl.Text=""
        displayedLogs=filteredLogs(); displayVer=displayVer+1
    end
    viewStart = math.max(1, #displayedLogs - MAX_ROWS + 1)
    drawnVer = -1
    updateRows()
end)

-- Single click: jump 3 rows immediately
BtnUp.Clicked:Connect(function()
    viewStart = math.max(1, viewStart - 3)
    drawnVer = -1; updateRows()
end)
BtnDown.Clicked:Connect(function()
    local maxStart = math.max(1, #displayedLogs - MAX_ROWS + 1)
    viewStart = math.min(maxStart, viewStart + 3)
    drawnVer = -1; updateRows()
end)

-- Hold: smooth continuous scroll
BtnUp.MouseDown:Connect(function()    scrollHeld=-1 end)
BtnUp.MouseUp:Connect(function()      scrollHeld=0 end)
BtnDown.MouseDown:Connect(function()  scrollHeld= 1 end)
BtnDown.MouseUp:Connect(function()    scrollHeld=0 end)

CopyBtn.Clicked:Connect(function()
    if DScript.Text=="" then return end
    -- Temporarily unlock so Focus()+Ctrl+A+Ctrl+C works, then relock
    DScript.IsReadOnly = false
    DScript:Focus()
    DScript.IsReadOnly = true
    CopyBtn.Text="Focused! Ctrl+A, C"; CopyBtn.Color=C.btnCopyOk
    wait(3); CopyBtn.Text="Copy Script"; CopyBtn.Color=C.btnCopy
end)

ExecBtn.Clicked:Connect(function()
    if not IS_POLYHACK or DScript.Text=="" then return end
    local fn; local ok=pcall(function() fn=loadstring(DScript.Text) end)
    if not ok or not fn then
        ExecBtn.Text="Parse Error"; ExecBtn.Color=C.btnRed
        wait(2); ExecBtn.Text="Execute"; ExecBtn.Color=C.btnExec; return
    end
    local ok2,err2=pcall(fn)
    ExecBtn.Text=ok2 and "Done!" or "Runtime Err"; ExecBtn.Color=ok2 and C.btnExecOk or C.btnRed
    if not ok2 then warn("[PolyRemoteSpy] Execute: "..tostring(err2)) end
    wait(2); ExecBtn.Text="Execute"; ExecBtn.Color=C.btnExec
end)

FireBtn.Clicked:Connect(function()
    if not IS_POLYHACK or not selectedLog then return end
    local inst=nil
    for _,fd in pairs(selectedLog.fields) do
        if fd.t=="Inst" then
            local iName=string.match(fd.v,'"(.+)"')
            if iName then pcall(function() inst=Env:FindChild(iName) end) end
            if inst then break end
        end
    end
    if not inst then
        pcall(function()
            local fn=loadstring("return "..selectedLog.evPath)
            if fn then local ok2,ref=pcall(fn); if ok2 and ref then inst=ref.Parent end end
        end)
    end
    if inst then
        local ok3,err3=pcall(fireclickdetector,inst)
        FireBtn.Text=ok3 and "Fired!" or "No Click"; FireBtn.Color=ok3 and C.btnExecOk or C.btnRed
        if not ok3 then warn("[PolyRemoteSpy] fireclickdetector: "..tostring(err3)) end
    else FireBtn.Text="No Inst"; FireBtn.Color=C.btnRed end
    wait(2); FireBtn.Text="FireClick"; FireBtn.Color=C.btnFire
end)

ChatBtn.Clicked:Connect(function()
    if not selectedLog then
        ChatBtn.Text="No event sel"; ChatBtn.Color=C.btnRed
        wait(1.5); ChatBtn.Text="SendChat"; ChatBtn.Color=C.btn; return
    end
    if not IS_POLYHACK then
        ChatBtn.Text="Need polyhack"; ChatBtn.Color=C.btnRed
        wait(1.5); ChatBtn.Text="SendChat"; ChatBtn.Color=C.btn; return
    end
    local msg=string.format("[Spy] %s | %s | t=%ss",selectedLog.dir,selectedLog.evName,selectedLog.time)
    local ok,err=pcall(function() sendchat(msg) end)
    if ok then
        ChatBtn.Text="Sent!"; ChatBtn.Color=C.btnExecOk
    else
        ChatBtn.Text="Chat failed"; ChatBtn.Color=C.btnRed
        warn("[PolyRemoteSpy] sendchat: "..tostring(err))
    end
    wait(2); ChatBtn.Text="SendChat"; ChatBtn.Color=C.btn
end)

Input.KeyDown:Connect(function(key)
    if key==CFG.TOGGLE_KEY then Win.Visible=not Win.Visible end
end)

-- ============================================================
--  RENDER LOOP
-- ============================================================
local SCROLL_SPEED = 0.22

game.Rendered:Connect(function()
    -- ── DRAG ─────────────────────────────────────────────────
    if isDragging then
        -- Safety cancel if mouse button released anywhere outside TitleBar
        if not Input.GetMouseButton(0) then
            isDragging=false
            TitleBar.Color=C.titleBg
            DragHint.Text=":: drag ::"
            DragHint.TextColor=C.txtDragIdle
        else
            local mouse = Input.MousePosition
            -- Screen Y is top-down; Win.PositionOffset with PositionRelative=(0.5,0.5)
            -- is ALSO top-down (positive Y = visually down).  No negation needed.
            Win.PositionOffset = V(
                dragStartOffset.x + (mouse.x - dragStartMouse.x),
                dragStartOffset.y + (mouse.y - dragStartMouse.y)
            )
        end
    end

    -- ── SCROLL ───────────────────────────────────────────────
    if scrollHeld~=0 then
        scrollAccum=scrollAccum+scrollHeld*SCROLL_SPEED
        local steps=math.floor(math.abs(scrollAccum)+0.5)
        if steps>=1 then
            viewStart=viewStart+(scrollHeld>0 and 1 or -1)*steps
            scrollAccum=0; drawnVer=-1
        end
    end

    -- ── LIVE REFRESH ─────────────────────────────────────────
    local now=os.clock()
    if paused then
        if logVersion~=displayVer then
            local pending=#filteredLogs()-#displayedLogs
            if pending~=pendingCount then
                pendingCount=pending
                PendLbl.Text=pendingCount>0 and ("+"..tostring(pendingCount).." new") or ""
            end
        end
    else
        if logVersion~=displayVer and (now-lastRefreshT)>=CFG.REFRESH_RATE then
            lastRefreshT=now; refreshDisplay()
        end
    end

    if drawnVer~=displayVer then updateRows() end
end)

-- ============================================================
--  STARTUP
-- ============================================================
local scanRoots={Env}
pcall(function() table.insert(scanRoots,game["ScriptService"]) end)
pcall(function() table.insert(scanRoots,PlayerGUI) end)

for _,r in ipairs(scanRoots) do
    pcall(function() scanSubtree(r) end)
    pcall(function() watchRoot(r) end)
end

-- ── CHAT HOOKS  (Player.Chatted on every connected player) ─────────────
-- These are not NetworkEvents, but chat is a key source of game traffic
-- so we log them as synthetic "CHAT" direction entries.
local function hookPlayerChat(player)
    pcall(function()
        player.Chatted:Connect(function(message)
            pcall(function()
                local fields = { message = { t="string", v='"'..tostring(message)..'"' } }
                local chatPath = 'game["Players"]["'..tostring(player.Name)..'"]'
                appendLog("CHAT", player.Name, chatPath, fields)
            end)
        end)
    end)
end

-- Hook players already in-game
local Players = game["Players"]
pcall(function()
    for _,p in ipairs(Players:GetPlayers()) do hookPlayerChat(p) end
end)
-- Hook future players
pcall(function()
    Players.PlayerAdded:Connect(function(p) hookPlayerChat(p) end)
end)

-- Also hook local player Chatted so own messages are captured
pcall(function()
    local lp = Players.LocalPlayer
    if lp then hookPlayerChat(lp) end
end)

showDetail(nil); updateRows()

print(string.rep("-",58))
print(string.format("  PolyRemoteSpy v7.1%s  |  %d objects hooked",
    IS_POLYHACK and "  [polyhack]" or "", hookedCount))
print("  [Insert] toggle  |  Drag title bar  |  || PAUSE then click row")
print(string.rep("-",58))
