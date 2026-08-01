import idaapi
import time
import hashlib
import ctypes
from ctypes import wintypes
import threading

user32 = ctypes.windll.user32
kernel32 = ctypes.windll.kernel32

user32.FindWindowW.argtypes = [wintypes.LPCWSTR, wintypes.LPCWSTR]
user32.FindWindowW.restype = wintypes.HWND
user32.SetWindowTextW.argtypes = [wintypes.HWND, wintypes.LPCWSTR]
user32.SetWindowTextW.restype = wintypes.BOOL
user32.GetWindow.argtypes = [wintypes.HWND, wintypes.UINT]
user32.GetWindow.restype = wintypes.HWND
user32.IsWindowVisible.argtypes = [wintypes.HWND]
user32.IsWindowVisible.restype = wintypes.BOOL
user32.GetWindowTextLengthW.argtypes = [wintypes.HWND]
user32.GetWindowTextLengthW.restype = ctypes.c_int
user32.GetWindowTextW.argtypes = [wintypes.HWND, wintypes.LPCWSTR, ctypes.c_int]
user32.GetWindowTextW.restype = ctypes.c_int
user32.EnumWindows.argtypes = [ctypes.c_void_p, wintypes.LPARAM]
user32.EnumWindows.restype = wintypes.BOOL
user32.GetWindowThreadProcessId.argtypes = [wintypes.HWND, ctypes.POINTER(wintypes.DWORD)]
user32.GetWindowThreadProcessId.restype = wintypes.DWORD

GW_OWNER = 4

def get_current_pid():
    return kernel32.GetCurrentProcessId()

def find_main_ida_window():
    target_pid = get_current_pid()
    result_hwnd = 0

    def enum_callback(hwnd, lParam):
        nonlocal result_hwnd
        if result_hwnd:
            return False
        if not user32.IsWindowVisible(hwnd):
            return True
        owner = user32.GetWindow(hwnd, GW_OWNER)
        if owner:
            return True
        pid = wintypes.DWORD()
        user32.GetWindowThreadProcessId(hwnd, ctypes.byref(pid))
        if pid.value != target_pid:
            return True
        length = user32.GetWindowTextLengthW(hwnd)
        if length == 0:
            return True
        buf = ctypes.create_unicode_buffer(length + 1)
        user32.GetWindowTextW(hwnd, buf, length + 1)
        if "IDA" in buf.value:
            result_hwnd = hwnd
            return False
        return True

    EnumWindowsProc = ctypes.WINFUNCTYPE(ctypes.c_bool, wintypes.HWND, wintypes.LPARAM)
    callback = EnumWindowsProc(enum_callback)
    user32.EnumWindows(callback, 0)
    return result_hwnd

def set_random_title():
    timestamp = str(time.time()).encode('utf-8')
    new_title = hashlib.sha256(timestamp).hexdigest()[:32]
    hwnd = find_main_ida_window()
    if hwnd:
        user32.SetWindowTextW(hwnd, new_title)
        return True
    return False

class TitleRandomizerPlugmod(idaapi.plugmod_t):
    def __init__(self):
        super().__init__()
        print("[TitleRandomizer] Infinite loop started (title update every 0.2s)")
        self.interval = 0.0
        self.running = True
        self._schedule_update()

    def _schedule_update(self):
        if not self.running:
            return
        self.timer = threading.Timer(self.interval, self._update)
        self.timer.daemon = True
        self.timer.start()

    def _update(self):
        set_random_title()
        self._schedule_update()

    def run(self, arg):
        pass

class TitleRandomizerPlugin(idaapi.plugin_t):
    flags = idaapi.PLUGIN_FIX | idaapi.PLUGIN_HIDE
    comment = "Randomizes IDA window title continuously"
    help = "No help"
    wanted_name = "Title Randomizer"
    wanted_hotkey = ""

    def init(self):
        return TitleRandomizerPlugmod()

    def run(self, arg):
        pass

    def term(self):
        pass

def PLUGIN_ENTRY():
    return TitleRandomizerPlugin()
