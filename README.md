Original download page: https://www.microsoft.com/en-us/software-download/windows11

Source ISO filename: Windows11_Client_x64_en-us_26300_9457.iso

Source ISO SHA256: `bd4307df32bc8af33b39ccecb1174aeb345386630f89a2b86c7a4e36b55ea650`

File location: `Windows11_Client_x64_en-us_26300_9457.iso\sources\install.wim\1\Windows\System32\UIAutomationCore.dll`

File SHA256: `0b6f4af885ba3c54cb945d660ce86b7dc85c9359f5bbdd98ec15b1b1ee194bac`

---

This DLL file will be used for this purpose:

Fix for application freezing/turning black after 30 mins to 4 hours of usage. While generic UI Automation crashes in Wine are known, this specific interaction with PDF-XChange Editor's background threads has not been previously documented.

Modern versions of PDF-XChange Editor suffer from a delayed crash where the application window turns completely black and freezes after prolonged use (anywhere between 30 minutes and 4 hours). The terminal logs reveal the root cause: 
`CoUnmarshalInterface StdMarshal UnmarshalInterface failed, hr 0x800706ba.`

The `WINEDEBUG=+relay` logs reveal the exact cause of this behavior:
```
0158:Call KERNEL32.K32GetProcessMemoryInfo(...)
0158:Call KERNEL32.GlobalMemoryStatusEx(...)
...
0048:err:ole:CoUnmarshalInterface StdMarshal UnmarshalInterface failed, hr 0x800706ba
```

PDF-XChange Editor runs background threads (like thread 0158 in the logs) that continuously poll the system status, query memory information (`K32GetProcessMemoryInfo`), and synchronize state with the main GUI thread via Windows UI Automation/COM infrastructure at regular intervals.

Wine's *builtin* implementation of **uiautomationcore.dll** is incomplete. Over time, it fails to handle these long-running background COM/RPC marshalling loops properly, eventually causing the internal RPC server to faint (`0x800706ba / RPC_S_SERVER_UNAVAILABLE`). Once the connection drops, the application's rendering engine loses its connection to the Wine window manager, causing the window to turn black and stop responding entirely.

To implement the fix, **UIAutomationCore.dll** needs to be renamed to lowercase (i.e. **uiautomationcore.dll**) and placed in `System32` of a Wine prefix. You may create a backup of the *builtin* file, like renaming the old file to `.dll.bak`. Placing the DLL in the directory of PDF-XChange **will not work**. The order must be `native,builtin`, which can be done with this command:
```
wine reg add "HKEY_CURRENT_USER\Software\Wine\DllOverrides" /v "uiautomationcore" /t REG_SZ /d "native,builtin" /f
```

**Simply put, any error with code `0x800706ba` is fatal. This DLL file fixes this issue and allows the application to continue gracefully.**

---

While this component is the proprietary property of Microsoft Corporation, it is distributed here purely as an isolated dependency to enable cross-platform software compatibility. It is extracted from the officially and publicly available Windows 11 ISO.
Only a single, isolated DLL file is hosted out of the entire multi-gigabyte Windows operating system. This represents a negligible, non-substantial fraction of the original work, strictly limited to what is necessary to achieve technical compatibility.
This distribution has zero market impact on the original product. The file has no standalone commercial value, cannot be used to run the Windows OS, and the source ISO containing this file is already provided by Microsoft to the general public free of charge.

**Copyright:** © Microsoft Corporation. All rights reserved.
