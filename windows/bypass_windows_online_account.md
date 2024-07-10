### Bypass Windows Secure Boot+TPM

In Command Prompt, type

bash
regedit


Once the Registry Editor is open, expand the following tree:

HKEY_LOCAL_MACHINE\SYSTEM\Setup

Right-click Setup and select New –> Key. Name the new key LabConfig

In the right hand pane (under LabConfig), create the following two DWORD (32-bit) entries:

bash
New DWORD (32-bit) value with the name BypassTPMCheck. Enter its value as 1

New DWORD (32-bit) value with the name BypassSecureBootCheck. Enter its value as 1
