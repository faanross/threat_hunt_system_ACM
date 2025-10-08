# The Five Critical Indicators: Your Analytical Framework

Now that you understand the philosophy and approach - network for breadth, endpoint for depth, and 80/20 rapid triage - here is a concrete framework for conducting investigations.

Through extensive experience and research in network threat hunting and endpoint analysis, five specific indicators have proven to be particularly diagnostic. These are the characteristics of a process or binary that, when evaluated together, provide the clearest and fastest path to accurate decision-making. **Critically, all five of these indicators are available through passive analysis** - primarily through Sysmon Event ID 1 (Process Creation) logs.

## The Five Indicators

Here I'll introduce the five indicators without going into any real depth of the "what, why, and how" - that will be discussed in a section dedicated to each respective indicator.

### 1. Binary Location

- This asks the question - where on the file system is the executable that created this process?
- Is it in `C:\Windows\System32` where we expect core Windows files to reside?
- Is it in `C:\Program Files` where installed applications belong?
- Or is it in `C:\Users\Doodiebref\AppData\Local\Temp`, the temporary folder often used by malware droppers?

### 2. Process Name

- This examines what the process is called.
- Is it named `chrome.exe` suggesting it's the Google Chrome browser?
- Is it `svchost.exe` suggesting it's a Windows system service host?
- Or is it something generic like `update.exe` or randomly named like `xyz123.exe`?

### 3. Hash/ImpHash

- This involves cryptographic hashes of the binary file, captured by Sysmon at process creation.
- The SHA256 hash provides a unique fingerprint that can be checked against databases of known malware.
- The ImpHash (Import Hash) is a more sophisticated hash that can link different variants of the same malware family together even when the main file hash has been changed.

### 4. Command Line

- This looks at how the process was launched - what parameters or arguments were passed to it when it started executing.
- A process might be legitimate, but suspicious command-line arguments can indicate abuse or malicious intent.

### 5. Parent Process

- This identifies which process created or "spawned" the process you're investigating.
- Windows has predictable parent-child relationships - for example, processes you launch by double-clicking icons should have `explorer.exe` as their parent.
- Unusual parent-child relationships, like for example "PowerShell spawning PowerShell" can indicate malicious activity.

## Why These Five?

You might wonder: why these specific five indicators? Why not three, or seven, or ten? The answer lies in a combination of **diagnostic value**, **accessibility**, and **efficiency**.

These five indicators are highly diagnostic. Each one, individually, can sometimes be enough to make a determination.

They're also readily accessible through passive analysis. You can obtain all five indicators from Sysmon Event ID 1 logs without ever touching the potentially malicious binary or process. This is what makes them ideal for network threat hunters operating under the passive-first principle.

Finally, they're efficient. You can evaluate all five indicators within a target timeframe of 5 to 15 minutes per investigation. They don't require specialized tools or extensive technical expertise to interpret. An analyst with proper training can quickly assess these five indicators and reach a confident determination.
