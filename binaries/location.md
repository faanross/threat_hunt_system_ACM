
# Binary Location Analysis: Understanding Where Processes Run

## The Fundamental Principle

When investigating a suspicious process, one of the fastest and most reliable indicators you can examine is where the executable file is located on the file system. The principle is straightforward: **legitimate system software runs from protected system directories, while malware typically runs from user-writable locations.**

This pattern exists for a simple reason. Legitimate software vendors install their applications to protected directories like `C:\Program Files\` or `C:\Windows\System32\`. These locations require administrator privileges to write to, which means they're part of a proper installation process. Users can't accidentally drop files into these directories, and casual malware can't write there without first elevating privileges.

Malware, on the other hand, usually operates under the constraints of the user account it's running under. Unless the attacker has already obtained administrator access, malware is restricted to writing files in locations where the current user has write permissions - places like temporary folders, the user's profile directories, or the Downloads folder.

An executable's location tells you two critical things: first, what level of permissions were required to place it there, and second, whether its location is consistent with being legitimate software. A process named `svchost.exe` running from `C:\Windows\System32\` is exactly where you'd expect to find the legitimate Windows Service Host. That same process name running from `C:\Users\DingleberrysMcGee\AppData\Local\Temp\` is malware masquerading as a system process.



## Highly Suspicious Locations

Let's examine the locations that should immediately raise your suspicion.

### Temporary Folders

#### **Primary Locations:**

- `C:\Users\<username>\AppData\Local\Temp\`
- `C:\Windows\Temp\`

Temporary folders are the most common locations for malware droppers and payloads. These directories serve a legitimate purpose - they provide a place for programs to store files that are needed briefly during installation or operation and then deleted. However, this same characteristic makes them attractive to attackers.

The key distinction is persistence. Legitimate software might briefly use temporary folders during installation, but it doesn't run continuously from them. An installer might extract files to temp, execute them to perform the installation, and then clean up after itself. The actual installed program runs from Program Files or another proper location.

Malware behaves differently. A malicious document might drop an executable to the temp folder and launch it. That executable then maintains a persistent presence, creating network connections, establishing persistence mechanisms, and continuing to run from the temporary directory. This is the pattern you're looking for.

#### **Red Flags for Temporary Folder Execution:**

When you see a process running from a temporary folder, these characteristics strongly indicate malicious activity:

- **Persistent network connections**: Any process maintaining ongoing external connections while running from temp is highly suspicious
- **Random or generic naming**: Executables with names like `tmp8A93.exe`, `setup.exe`, or `update.exe` in temp directories often indicate malware
- **Long execution time**: A process running for more than a few minutes from temp suggests it's not a brief installer operation
- **Child process creation**: Processes spawning additional processes from temp directories indicate malicious behaviour
- **Multiple similar files over time**: If you find a pattern of different random-named executables in temp across multiple investigation periods, this suggests ongoing malware activity

#### **Legitimate Temporary Folder Use:**

To avoid false positives, understand when temporary folder use is legitimate:

- Software installers during active installation (should complete within minutes)
- Update utilities downloading and applying patches (brief execution)
- Archive extraction tools temporarily unpacking files (short-lived)

The key differentiator is duration and behaviour. Legitimate use is transient and purposeful. Malicious use is persistent and establishes ongoing presence.

#### **Example of Malicious Temporary Folder Execution:**

```
Process: svchost.exe
Path: C:\Users\john.doe\AppData\Local\Temp\svchost.exe
Parent: WINWORD.EXE
Network: Outbound HTTPS to unknown IP

Analysis: This exhibits multiple critical indicators:
- Core Windows process name (svchost.exe) in wrong location
- Spawned by Microsoft Word (document-based attack vector)
- Making external network connections
→ Assessment: HIGHLY SUSPICIOUS - Likely document-based malware
```


### Recycle Bin

#### **Location:** `C:\$Recycle.Bin\`

The Recycle Bin deserves special attention because there is absolutely zero legitimate reason for code to execute from this location. Unlike temporary folders which have legitimate uses, the Recycle Bin has no legitimate execution use case whatsoever.

This location is attractive to attackers because users rarely examine its contents in detail. Files sit in the Recycle Bin until the user explicitly empties it, providing a hiding place that might persist for extended periods. Attackers sometimes combine this with scheduled tasks or registry run keys to create persistence mechanisms that execute code from the Recycle Bin.

Your analysis here is simple: any executable running from `C:\$Recycle.Bin\` is malicious. There are no edge cases, no context dependencies, no legitimate exceptions.

#### **Example:**

```
Process: chrome.exe
Path: C:\$Recycle.Bin\S-1-5-21-xxx\chrome.exe
→ Decision: IMMEDIATE ALERT - No legitimate scenario exists for this
```



### User Profile and AppData Directories

#### **Locations:**

- `C:\Users\<username>\`
- `C:\Users\<username>\AppData\Roaming\`
- `C:\Users\<username>\AppData\Local\`

These locations require more nuanced analysis than temporary folders or the Recycle Bin. Many legitimate applications do use AppData directories, but they use them in specific ways that differ from how malware typically uses them.

Legitimate software creates proper folder structures in AppData. When you install Microsoft Teams, for example, it creates `C:\Users\[username]\AppData\Local\Microsoft\Teams\current\Teams.exe`. Notice the hierarchical structure: the vendor name (Microsoft), the product name (Teams), a version indicator (current), and then the executable.

Malware typically doesn't bother with this organizational structure. You'll find executables dropped directly into the AppData parent directories with no subfolder organization. A file like `C:\Users\john\AppData\Local\svchost.exe` is immediately suspicious - it's a Windows system process name in a user directory with no vendor folder structure.

#### **Distinguishing Legitimate from Suspicious:**

When evaluating AppData execution, consider these factors:

**Folder Structure:** Legitimate software has recognizable vendor folder hierarchies. Random executables directly in AppData parent directories are suspicious.

**File Naming:** Legitimate software uses product names. Generic names like `update.exe`, `service.exe`, or `runtime.exe` directly in AppData are red flags.

**Timestamps:** Compare file creation time to execution time. If a file was created and immediately started executing from AppData with no installation process, this suggests malicious activity.

**Vendor Recognition:** Can you identify the vendor folder as belonging to known software? Microsoft, Google, Adobe, and other major vendors are recognizable. Unknown or absent vendor folders warrant investigation.

**Examples:**

```
LEGITIMATE:
C:\Users\john\AppData\Local\Microsoft\Teams\current\Teams.exe
✓ Proper vendor folder (Microsoft)
✓ Product folder (Teams)
✓ Version folder (current)
✓ Appropriate file name

SUSPICIOUS:
C:\Users\john\AppData\Local\svchost.exe
✗ No subfolder structure
✗ System process name in user directory
✗ No vendor identification

C:\Users\john\AppData\Roaming\update.exe
✗ Generic, vague name
✗ Directly in parent directory
✗ No legitimate software context
```


### Downloads Folder

#### **Location:** `C:\Users\<username>\Downloads\`

The Downloads folder represents a common initial infection vector. Users download files here, and phishing campaigns or drive-by downloads deposit malicious executables here. However, the presence of an executable in Downloads isn't inherently suspicious - the suspicious part is what happens next.

Legitimate software downloaded to the Downloads folder follows a predictable pattern. The user downloads an installer, double-clicks it to begin installation, the installer runs briefly from Downloads, installs the actual application to Program Files, and then typically either deletes itself or remains as an inactive file in Downloads. The key point is that legitimate software doesn't continue running persistently from the Downloads folder.

Malware behaves differently. It might execute from Downloads and continue running from that location, making network connections, creating persistence mechanisms, or spawning additional processes - all while still residing in the Downloads folder.

#### **Red Flags for Downloads Folder:**

- **Persistent execution**: Process still running from Downloads days after being downloaded
- **Network connections**: Process in Downloads making external connections
- **Generic installer names**: Files named `setup.exe`, `install.exe`, `update.exe` with no vendor context
- **Disguised extensions**: Files like `document.pdf.exe` attempting to hide their executable nature
- **Immediate execution after download**: File downloaded and executed within seconds (automated, not user-initiated)

#### **Temporal Context Matters:**

Consider the timeline of events:

```
Scenario 1 - SUSPICIOUS:
Downloaded: 2025-01-15 14:23:00
First Execution: 2025-01-15 14:24:00
Current Status: Still running from Downloads (7 days later)
→ Assessment: Legitimate software should have installed to proper location

Scenario 2 - LESS SUSPICIOUS:
Downloaded: 2025-01-15 14:23:00
First Execution: 2025-01-15 14:25:00
Current Status: Installation completed, installer remains but not running
→ Assessment: Normal installer behavior, verify installed application is legitimate
```




### Public and Shared Folders

#### **Locations:**

- `C:\Users\Public\`
- `C:\ProgramData\`

These directories require the most nuanced analysis because they serve legitimate purposes for many applications, yet attackers also exploit them for persistence and system-wide access.

`C:\ProgramData\` is commonly used by major software vendors for system-wide application data, shared resources, and installation components. Adobe, Microsoft, and other legitimate vendors routinely install components here. This makes distinguishing legitimate use from malicious use more challenging.

The key is to examine the context carefully. Look for these characteristics:

**Vendor Recognition:** Can you identify the subfolder as belonging to a known, trusted vendor? `C:\ProgramData\Microsoft\` or `C:\ProgramData\Adobe\` are expected. `C:\ProgramData\WindowsUpdate\` with unusual executables inside is suspicious - it's imitating a legitimate-sounding name but isn't actually Microsoft's update infrastructure.

**Folder Organization:** Legitimate vendors create structured, organized folder hierarchies. `C:\ProgramData\Package Cache\{GUID}\vcredist.exe` shows proper organization. `C:\ProgramData\runtime.exe` with no subfolder structure is suspicious.

**File Age:** Older files in ProgramData established during system setup or software installation are more likely legitimate. Recently created files, especially with current timestamps, require scrutiny.

The `C:\Users\Public\` folder is less commonly used by legitimate software than ProgramData, so your suspicion threshold should be higher. While some applications do use it for shared resources, it's relatively uncommon. Executables running from Public folders deserve careful investigation.

#### **Evaluation Examples:**

```
LIKELY LEGITIMATE:
C:\ProgramData\Microsoft\Windows Defender\
✓ Recognized Microsoft product
✓ Expected location for Defender

C:\ProgramData\Adobe\Acrobat\
✓ Known vendor (Adobe)
✓ Proper folder structure

SUSPICIOUS:
C:\ProgramData\WindowsUpdate\svchost.exe
✗ Fake folder name mimicking legitimate Windows component
✗ System process in unusual ProgramData location

C:\ProgramData\runtime.exe
✗ No vendor folder structure
✗ Generic name
✗ Direct execution from parent directory
```

### Rarely Used System Directories

#### **Locations:**

- `C:\Windows\Tasks\`
- `C:\Windows\System32\Tasks\` (for scheduled tasks)
- `C:\Windows\Temp\`
- `C:\PerfLogs\`

Some system directories allow user writes but are rarely used for legitimate purposes, making them attractive hiding spots for malware.

`C:\PerfLogs\` is particularly noteworthy. This directory exists on Windows systems for performance logging, but it's almost never used by legitimate software. Finding an executable running from PerfLogs is highly suspicious.

`C:\Windows\Tasks\` and `C:\Windows\System32\Tasks\` contain scheduled task definitions. While legitimate tasks exist here, unexpected executables or task files warrant investigation, especially if recently created.

`C:\Windows\Temp\` falls into the same category as user temporary folders but at the system level. The same principles apply - brief, transient use might be legitimate, but persistent execution is suspicious.



## Expected Legitimate Locations

Understanding suspicious locations is only half the equation. You also need to recognize where legitimate software should be running from to avoid false positives and to identify masquerading malware.

### System Directories

#### **Core Windows Executable Locations:**

- `C:\Windows\System32\`
- `C:\Windows\SysWOW64\` (32-bit executables on 64-bit Windows)
- `C:\Windows\`

These are protected system directories containing core Windows components. Executables here are part of the operating system or critical system functions.

The critical factor with system directories is not just the location, but the combination of location and process name. Core Windows processes have specific expected locations:

- `svchost.exe` should ONLY run from `C:\Windows\System32\`
- `explorer.exe` should ONLY run from `C:\Windows\`
- `lsass.exe` should ONLY run from `C:\Windows\System32\`
- `winlogon.exe` should ONLY run from `C:\Windows\System32\`

This creates an important detection opportunity. Malware often masquerades by using the names of legitimate Windows processes, hoping to blend in. But if you find `svchost.exe` running from anywhere other than System32, you've identified masquerading malware regardless of how legitimate the process name appears.

#### **Masquerading Detection:**

```
LEGITIMATE:
Process: svchost.exe
Path: C:\Windows\System32\svchost.exe
→ Expected location for this process

MALICIOUS (Masquerading):
Process: svchost.exe
Path: C:\Windows\Temp\svchost.exe
→ ALERT: Core Windows process in wrong location - this is malware
```


### VAD Types: Understanding Memory Categories

Not all memory is created equal. The kernel categorizes memory regions into different **VAD types** based on their purpose and backing store.

#### The Four Primary VAD Types

```
┌─────────────────────────────────────────────────────────────────┐
│                         VAD TYPE MATRIX                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Type        │ Backed By         │ Common Uses                  │
│  ───────────────────────────────────────────────────────────────│
│  PRIVATE     │ Pagefile/RAM only │ • Heap allocations           │
│              │                   │ • Stack memory               │
│              │                   │ • VirtualAlloc() calls       │
│              │                   │ • Process-specific data      │
│              │                   │                              │
│  IMAGE       │ PE files on disk  │ • Executable files (.exe)    │
│              │                   │ • DLL files loaded           │
│              │                   │ • Code sections (.text)      │
│              │                   │ • Shareable across processes │
│              │                   │                              │
│  MAPPED      │ Regular files     │ • Memory-mapped files        │
│              │                   │ • Database files             │
│              │                   │ • Large data files           │
│              │                   │ • Inter-process data sharing │
│              │                   │                              │
│  SHAREABLE   │ Shared memory     │ • Named shared memory        │
│              │                   │ • IPC mechanisms             │
│              │                   │ • Multiple processes access  │
│              │                   │                              │
└─────────────────────────────────────────────────────────────────┘
```

##### **Private VADs:**

- Most common type for dynamic allocations
- Backed by the page file (virtual memory) when paged out
- Each process has its own copy - no sharing
- Examples: `malloc()`, new, `HeapAlloc()`, `VirtualAlloc()` with no file mapping

##### **Image VADs:**

- Represent PE executable files loaded into memory
- Backed by the actual .exe or .dll file on disk
- Can be shared across processes (same `kernel32.dll` mapped into many processes)
- Sections have different protections (`.text` is R-X, `.data` is RW-)

##### **Mapped VADs:**

- Created by `CreateFileMapping()` + `MapViewOfFile()`
- Backed by a regular file on disk
- Changes can be written back to the file (or not, depending on flags)
- Efficient way to work with large files without loading entirely into RAM

##### **Shareable VADs:**

- Explicitly shared between multiple processes
- Often used for inter-process communication
- Created with PAGE_READWRITE and proper sharing flags
- Multiple processes can read/write the same physical memory








