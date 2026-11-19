# M365 Outage Check

A read-only PowerShell tool that tells you in about 30 seconds whether Outlook, Teams, SharePoint, OneDrive or
Copilot is down because of a **Microsoft-side outage** or because of **your own PC or network**.

## About this tool
When Microsoft 365 suddenly stops working, the first thing everyone wants to know is whether the problem is
Microsoft's or their own PC's and network's. During a large outage, such as the worldwide one on 12–13 November 2026,
many people start "fixing" their PCs: they repair Office, rebuild their Outlook profile or change their password.
If the cause is on Microsoft's side, none of this helps. Worse, it can erase the copy of received mail saved on the PC,
leaving Outlook unusable for a while even after Microsoft has recovered. This tool answers the question in about
30 seconds, so people can stop guessing and stop changing things.

The way it works is simple. First it checks whether the PC can reach the internet at all, using Windows' own
connectivity test. Then it contacts the public "front door" of each service (sign-in, Exchange / Outlook, Teams,
Microsoft 365 / Copilot, SharePoint and OneDrive) without signing in, and looks at how each one answers. When a
service does not answer, it digs one level deeper to find out whether the network connection itself failed or
whether Microsoft's server dropped the connection during the secure handshake. It also checks that Microsoft's
certificate comes from a public certificate authority, which reveals whether an inspection device sits in the path.
Finally, it gives one plain-language verdict.

As a real example, during the November 2026 outage the internet, sign-in, Teams and SharePoint all answered
normally, but Outlook's servers closed every connection in the middle of the secure handshake. The tool reported
"Microsoft-side outage likely", and the same failure from a phone on mobile data confirmed it. No PC settings
needed to change at all.

The tool changes nothing on the PC and never uses a user name or password. Its limit is that it can only see the
front doors before sign-in. Failures that happen after sign-in (for example, the Copilot page opens but returns
`HTTP 504`) are not detected; for those, check Microsoft's official status page.

Staff only need to double-click a `.cmd` file. The helpdesk can collect the result file as evidence, or use
`-RepeatMinutes` to re-check at a fixed interval and see the moment a service comes back.

## How to use (staff)
1. On this page, click the green **Code** button → **Download ZIP**
2. Right-click the ZIP → **Properties** → tick **Unblock** at the bottom → OK (prevents a security warning when you run it)
3. Extract the ZIP and double-click **`M365 Outage Check.cmd`**
4. Read the **"Verdict:"** line in the black window

| Verdict | Meaning | What to do |
|---|---|---|
| Microsoft-side outage likely | The internet works, a Microsoft front door fails | Leave the PC as it is and wait. Use the phone for urgent matters |
| Our side suspected | Fails at internet / DNS / proxy / certificate | Contact the helpdesk / network team |
| Front doors OK | Nothing wrong on our side | If it still does not work, contact the helpdesk |

**Do NOT, during an outage:** reinstall the PC, repair or reinstall Office, rebuild the Outlook profile, change your
password, or sign in again and again. These erase the saved copy of your mail (.ost) and leave Outlook unusable even after recovery.

## What it checks
1. **Control test**: can the PC reach the internet at all (Windows' own connectivity check, `www.msftconnecttest.com`)
2. **Each service's front door**: the HTTPS response code before sign-in (sign-in / Exchange / Teams / Microsoft 365 &
   Copilot / SharePoint / OneDrive). With no response, it also finds where it stops: TCP failed or TLS dropped mid-handshake
3. **Inspection-device check**: whether Microsoft's certificate comes from a public certificate authority
4. **This PC**: whether Outlook / Teams / OneDrive are running, and whether an Outlook offline cache (.ost) exists

## Safety
- Changes **nothing** on the PC. No admin rights needed.
- Sends only **unauthenticated GET requests** to Microsoft's public URLs. No user ID, password or mail content is used.
- Reads **only the host name** of SharePoint / OneDrive from the OneDrive settings (the URL containing the user's name is not used).
- Saves results only to a .txt in `%TEMP%\m365-check\`. .ost file names (which contain the mail address) are not shown.

## Limits
It checks **only the front doors before sign-in**. Failures after sign-in (for example Copilot's `HTTP 504`) are not
detected; check the official status: https://status.cloud.microsoft

## For admins
```powershell
powershell -ExecutionPolicy Bypass -File .\M365-OutageCheck.ps1                    # run once
powershell -ExecutionPolicy Bypass -File .\M365-OutageCheck.ps1 -RepeatMinutes 10  # re-check every 10 min (Ctrl+C to stop)
powershell -ExecutionPolicy Bypass -File .\M365-OutageCheck.ps1 -NoFile            # screen only
```
| Parameter | Purpose |
|---|---|
| `-RepeatMinutes N` | Re-check every N minutes; one line per round after the first (to watch for recovery) |
| `-NoFile` | Do not save a result file |
| `-SharePointHost` | e.g. `contoso.sharepoint.com` (default: detected from the OneDrive settings) |
| `-TimeoutSec` | Wait per target (default 10 s) |
| `-OutDir` | Where to save the result file (default `%TEMP%\m365-check`) |

Exit codes: `0` front doors OK / `1` Microsoft side suspected / `2` our side suspected / `3` script error
Requirements: Windows 10 / 11, Windows PowerShell 5.1 or PowerShell 7

## Files
| File | Purpose |
|---|---|
| `M365 Outage Check.cmd` | Double-click launcher for staff |
| `M365-OutageCheck.ps1` | The check itself |
| `HowTo.txt` | Short how-to for staff |
