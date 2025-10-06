
# The Seven Critical Indicators: Your Analytical Framework

Now that you understand the philosophy and approach - network for breadth, endpoint for depth, and 80/20 rapid triage - here is a concrete framework for conducting investigations.

Through extensive experience and research in network threat hunting and endpoint analysis, seven specific indicators have proven to be particularly diagnostic. These are the characteristics of a process or binary that, when evaluated together, provide the clearest and fastest path to accurate decision-making.

## The Seven Indicators

Here I'll introduce the seven indicators without going into any real depth of the "what, why, and how" - that will be discussed in a section dedicated to each respective indicator.

### 1. Binary Location
- This asks the question - where on the file system is the executable that created this process?
- Is it in `C:\Windows\System32` where we expect core Windows files to reside?
- Is it in `C:\Program Files` where installed applications belong?
- Or is it in `C:\Users\Doodiebref\AppData\Local\Temp`, the temporary folder often used by malware droppers?


### 2. Process Name
- This examines what the process is called.
- Is it named `chrome.exe` suggesting it's the Google Chrome browser?
- Is it `svchost.exe` suggesting it's a Windows system service host?
- Or is it something generic like `update.exe` or randomly named like `xyz123.exe`?

### 3. Hash/ImpHash
- This involves calculating cryptographic hashes of the binary file.
- The SHA256 hash provides a unique fingerprint that can be checked against databases of known malware.
- The ImpHash (Import Hash) is a more sophisticated hash that can link different variants of the same malware family together even when the main file hash has been changed.

### 4. Strings
- This examines the human-readable text embedded within the binary.
- Strings can reveal URLs that the malware connects to, file paths it targets, commands it executes, or even error messages left behind by careless malware authors.

### 5. Command Line
- This looks at how the process was launched - what parameters or arguments were passed to it when it started executing.
- A process might be legitimate, but suspicious command-line arguments can indicate abuse or malicious intent.

### 6. Parent Process
- This identifies which process created or "spawned" the process you're investigating.
- Windows has predictable parent-child relationships - for example, processes you launch by double-clicking icons should have `explorer.exe` as their parent.
- Unusual parent-child relationships, like for example "PowerShell spawning PowerShell" can indicate malicious activity.

### 7. Digital Signatures
- This verifies whether the binary is cryptographically signed by its creator and whether that signature is valid.
- Legitimate software from major vendors is almost always digitally signed.
- Malware is usually unsigned, or occasionally signed with stolen or fraudulent certificates.

## Why These Seven?

You might wonder: why these specific seven indicators? Why not five, or ten, or twenty? The answer lies in a combination of **diagnostic** value, **accessibility**, and **efficiency**.

These seven indicators are highly diagnostic. Each one, individually, can sometimes be enough to make a determination.

They're also readily accessible. You can obtain all seven indicators relatively quickly, mostly through passive analysis techniques that don't require directly interacting with a potentially malicious process.

Finally, they're efficient. You can evaluate all seven indicators within a target timeframe of 5 to 15 minutes per investigation. They don't require specialized tools or extensive technical expertise to interpret. An analyst with proper training can quickly assess these seven indicators and reach a confident determination.

## How to Use Them Together

The power of these seven indicators comes not just from examining each one individually, but from evaluating them together and looking for patterns.

Think of each indicator as a vote. Some indicators might vote "suspicious" while others vote "legitimate." Your job is to count the votes and assess the overall picture.

For example, imagine you're investigating a process and you find:

- It's running from C:\Program Files\Google\Chrome\Application (legitimate location - vote: benign)
- It's named chrome.exe (expected name - vote: benign)
- Its SHA256 hash matches the known legitimate Google Chrome (vote: benign)
- It's digitally signed by Google LLC with a valid signature (vote: benign)
- Its parent process is explorer.exe, meaning a user launched it (vote: benign)
- Its command line is normal for Chrome (vote: benign)

In this case, every indicator votes "benign." This is very clearly legitimate Google Chrome, and you can quickly clear this alert and move on.

Now contrast that with a different scenario:

- It's running from C:\Users\JohnDoe\AppData\Local\Temp (suspicious location - vote: malicious)
- It's named chrome.exe (trying to look legitimate - vote: suspicious)
- Its hash is unknown to VirusTotal, but there are no positive detections (vote: neutral/slightly suspicious)
- It's unsigned (vote: suspicious)
- Its parent process is WINWORD.EXE, meaning Microsoft Word spawned it (vote: highly suspicious)
- Its command line is empty or contains encoded PowerShell commands (vote: highly suspicious)

In this second case, multiple indicators are voting "suspicious" or "malicious." Even though the process is named "chrome.exe" trying to appear legitimate, the combination of executing from a temporary folder, being spawned by Word, lacking a digital signature, and having a suspicious command line creates a clear pattern of malicious activity.

This is how you use the seven indicators together. You're not looking for perfection - you're looking for patterns. When multiple indicators align toward malicious activity, you can confidently escalate. When they align toward legitimate activity, you can confidently clear. And when the picture is mixed or unclear, you know to dig a bit deeper or escalate for a more experienced analyst to review.

