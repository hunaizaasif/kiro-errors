# CCR Service Start Problems - Solution Guide
**Roman Urdu Mein**

## Error-1: Port Already in Use

### Problem

### Windows Terminal Error

Agar CCR Ubuntu terminal mein chal raha ho aur aap usko stop nahi karte to Windows terminal `CMD` mein error aata hai.

```bash
ccr status
```
- **Run karne ke bhaad:**

### Error Screenshot

<img width="440" height="199" alt="image" src="https://github.com/user-attachments/assets/e092e436-1935-4685-8a0f-12f6bbff60e0" />



## **Solution of Error-1**:

Matlab port busy hai. Pehle WSL2 (Ubuntu terminal) mein Claude Code Router stop karo run:

```bash
ccr stop
```

Phir `ccr start` aur `ccr status` run karo. Agar Claude Code Router running ho to fine hai, nahi to restart karo:

```bash
ccr restart
```

Phir status check karo:

```bash
ccr status
```

------------------------- Solution  No.1 Ended --------------------------------------

## Error-2: Not properly  adding a content in config.json file!

### Error Screenshot
<img width="768" height="235" alt="image" src="https://github.com/user-attachments/assets/31fbc08c-27d2-4db1-9ebe-2514b2a04dde" />

Solution of Error-2**:

### Run this command that file exist or not:

> Check by running this command:

```bash
cat ~/.claude-code-router/config.json
```

If file exists it shows some properties in the terminal like:


```bash
agentive-solution@DESKTOP-PNAKLV9:~$ cat ~/.claude-code-router/config.json
{
  "PORT": 3456,
  "Providers": [],
  "Router": {}
}
```

Or if the file is not exist it shows:

```bash
cat: /home/agentive-solution/.claude-code-router/config.json: No such file or directory
```

In both cases paste this configuration in the Ubuntu terminal:

```bash
cat > ~/.claude-code-router/config.json << 'EOF'
{
  "LOG": true,
  "LOG_LEVEL": "debug",
  "Providers": [
    {
      "name": "kiro",
      "api_base_url": "http://localhost:8000/v1/chat/completions",
      "api_key": "my-super-secret-password-123",
      "models": [
        "claude-sonnet-4-5",
        "claude-haiku-4-5",
        "claude-opus-4-5"
      ],
      "transformer": {
        "use": ["openrouter"]
      }
    }
  ],
  "Router": {
    "default": "kiro,claude-sonnet-4-5",
    "think": "kiro,claude-sonnet-4-5",
    "background": "kiro,claude-sonnet-4-5",
    "longContext": "kiro,claude-sonnet-4-5",
    "webSearch": "kiro,claude-sonnet-4-5"
  }
}
EOF
```

### CCR Start Karo

```bash
ccr start
```
### CCR restart Karo


### Status Check Karo

```bash
ccr status
```

### Claude Code Start Karo

```bash
ccr code
```

### Result

Message `Hi` bhejne ke baad aap notice karoge ki previous error `400-Error-Missing-Model-In-Request-Body` resolve ho gaya hai. \

> **Note**: Lekin aapko ek aur error milega `500`.

----------------------------------- Solution Ended of Error No.2 -----------------------------------

## Error-3: Connection Failed (500 Error)

### Error Screenshot
<img width="760" height="292" alt="image" src="https://github.com/user-attachments/assets/cec4b79b-e498-4223-b48c-83ae1e0be5ac" />


<div align="center">

## **There are two steps to solve this error**

</div>

## Step-1:

### Open the directory `kiro-openai-gateway`

Root directory `kiro-openai-gateway` (jahan actual kiro repository clone ki thi) terminal mai phir:

- **Run this command to start the server:**

```bash
python main.py
```

Server start hojaegha.

- **Run this command to restart the claude-code-router:**

```bash
ccr restart
```

- **Run this command to start the claude code CLI:**

```bash
ccr code
```

## Step-2:

```text
WSL apna localhost use kar raha ha system ka nhi.
WSL ke Localhost ko Windows IP Address se replace karo config.json file mein Ubuntu terminal se.
```

### **Ubuntu terminal mein ye commands run karo:**

#### 1. Connection Diagnose Karo

```bash
curl http://localhost:8000
```
Failed - confirmed localhost doesn't work

#### 2. Windows Host IP Find Karo WSL Se

```bash
ip route show | grep -i default | awk '{ print $3}'
```
Output kuch aisa milega: `172.x.x.x`

#### 3. Verify Karo Kiro Gateway Reachable Hai

```bash
curl http://172.x.x.x:8000
```
Success: `{"status":"ok","message":"Kiro Gateway is running"}`

#### 4. CCR Config File Locate Karo

```bash
find ~ -name ".claude-code-router" -type d
```
Found: `/home/agentive-solution/.claude-code-router/`

#### 5. Configuration Update Karo

```bash
nano ~/.claude-code-router/config.json
```

#### 6. Save Aur Exit Karo

```
Ctrl + o → Save
Enter → Confirm
Ctrl + x → Exit
```

#### 8. CCR Restart Karo

```bash
ccr restart
```
#### 9. Run claude code.

```bash
ccr code
```

✅ Successfully Claude-Code_Router is  running with kiro.

---

**End of Guide**


📌 PROBLEM SOLVE KARNE KE STEPS
--------------------------------------------------------------------------------


## Why Error-1: Port Already in Use Occurs? Details Guide:

Jab `ccr start` command chalaya to service start nahi hui. Error message mein sirf "Loaded JSON config" dikhaya aur phir exit code 1 ke saath fail ho gaya.

`ccr status` check kiya to "Not Running" dikh raha tha, lekin `ccr start` karne pe bhi service shuru nahi ho rahi thi.

### Problem Ki Wajah (Reason)

Port 3456 pe ek purana process "wslrelay.exe" chal raha tha.

Yeh isliye hua kyunki:
- Pehle ccr ko WSL Ubuntu terminal se start kiya gaya tha
- WSLRelay.exe ek helper process hai jo Windows aur WSL ke beech network traffic forward karta hai
- Jab ccr band hua ya crash hua, wslrelay ne port 3456 release nahi kiya
- Is wajah se naya ccr instance us port ko use nahi kar saka

**Simple words mein:** Darwaza (port 3456) pehle se kisi aur ne rok rakha tha, isliye ccr andar nahi ja saka.

------------------------------------------------------------------------------------------

First run this `ccr start` then check:

Step 1: Check karo port 3456 kaun use kar raha hai
        Command: netstat -ano | findstr :3456

Step 2: Process ID (PID) note karo jo port use kar raha hai
        Example output: TCP 127.0.0.1:3456 ... LISTENING 13348
        (Yahan 13348 PID hai)

Step 3: Check karo yeh kaun sa process hai
        Command: tasklist | findstr 13348

Step 4: Us process ko kill karo
        Command (CMD mein): taskkill /PID 13348 /F

Step 5: Ab ccr start karo
        Command: ccr start

Step 6: Verify karo service chal rahi hai
        Command: ccr status


📊 Quick Reference Table
TaskWindows (CMD)Ubuntu/WSL (Bash)Port checknetstat -ano | findstr :3456sudo lsof -i :3456Process listtasklist | findstr <PID>ps aux | grep <PID>Kill processtaskkill /PID <PID> /Fkill -9 <PID>CCR startccr startccr startCCR statusccr statusccr status



### Maintained by Shahzain Ali | [LinkedIn Profile](https://www.linkedin.com/in/shahzain-ali-518b862ba/)
