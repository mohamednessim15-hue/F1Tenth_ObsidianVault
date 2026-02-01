# Resetting Ubuntu

```
If you prefer staying in a terminal (PowerShell or Command Prompt), you can "unregister" the distribution. This is instantaneous.

1. Open **PowerShell** as Administrator.
    
2. Run this command to see your installed distro name: `wsl --list`
    
3. Run the following to wipe it (replace `Ubuntu` with your distro name if it's different): `wsl --unregister Ubuntu`
    
4. To start over, just type `ubuntu` in PowerShell or launch it from your Start Menu. It will behave as if it’s being installed for the first time, asking you for a **new username and password**.
   
   
   GEMINI RESPONSE
```
so to completely reset ubuntu, i did:
1- `wsl --unregister Ubuntu-22.04`
2- opening ubuntu again from its windows app icon.
