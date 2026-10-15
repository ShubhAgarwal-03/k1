Here are the exact steps for Programs 3, 4, and 5, extracted from your lab manual, organized so you can practice them directly.

## Program 3: Command Injection (DVWA on Metasploitable)

**Setup**
1. Download `Metasploitable2-Linux.zip` from sourceforge.net/projects/metasploitable/
2. Extract the ZIP — you'll get `Metasploitable.vmdk` and `Metasploitable.vmx`
3. In VirtualBox, click **New** → name it "Metasploitable" → leave ISO empty → Type: Linux, Version: Ubuntu (32-bit) or Other Linux (32-bit)
4. Expand **Specify Virtual Hard Disk** → choose **Use an Existing Virtual Hard Disk File** → browse to `Metasploitable.vmdk` → **Finish**
5. Configure networking so Kali and Metasploitable can talk:
   - Select Metasploitable → Settings → Network → Adapter 1 → set to **Host-only Adapter** (or Bridged, matching Kali)
   - Select Kali → Settings → Network → Adapter 1 → set to the **same** adapter type

**Running the attack**
6. Start Metasploitable → login: `msfadmin` / password: `msfadmin`
7. Run `ifconfig` → note the `eth0` IP address (e.g., `192.168.56.102`)
8. Start Kali → open terminal → `ping 192.168.56.102` to confirm connectivity
9. Open Firefox in Kali → go to `http://<Metasploitable-IP>/dvwa`
10. Login: username `admin`, password `password` (click Create/Reset Database first if prompted)
11. Go to **DVWA Security** (left menu) → set level to **Low** → Submit
12. Click **Command Execution** in the left menu
13. Test normal use: enter `127.0.0.1` → Submit → should show ping output
14. Test injection — enter these one at a time and click Submit:
   - `127.0.0.1; ls`
   - `127.0.0.1; pwd`
   - `127.0.0.1; whoami`
   - `127.0.0.1; ls -la`
   - `127.0.0.1; ifconfig`
15. Observe: each command appends extra output beyond the ping — showing the app executes unsanitized input as shell commands.

## Program 4: Buffer Overflow using GCC

1. Open terminal in Kali/Ubuntu → `nano buffer.c`
2. Type this program:
```c
#include <stdio.h>
#include <string.h>

int main()
{
   char buff[15];
   int pass = 0;

   printf("\nEnter the password:\n");

   fgets(buff, sizeof(buff), stdin);
   buff[strcspn(buff, "\n")] = '\0';   // Remove newline

   if(strcmp(buff, "thegeekstuff"))
   {
      printf("\nWrong Password\n");
   }
   else
   {
      printf("\nCorrect Password\n");
      pass = 1;
   }

   if(pass)
   {
      printf("\nRoot privileges given to the user\n");
   }

   return 0;
}
```
3. Save: `Ctrl+O` → Enter → `Ctrl+X`
4. Compile: `gcc -g -fno-stack-protector -z execstack -o buffer buffer.c`
   - `-g`: debugging symbols
   - `-fno-stack-protector`: disables stack protection
   - `-z execstack`: makes stack executable (for demo only)
5. Run: `./buffer`

**Three test runs:**
- **Run 1 (correct password):** enter `thegeekstuff` → Output: "Correct Password" + "Root privileges given to the user"
- **Run 2 (wrong password):** enter `hello` → Output: "Wrong Password", no root privileges
- **Run 3 (overflow attempt):** run `./buffer` again, enter a long string of characters (e.g., many repeated letters, well beyond 15 chars) → observe behavior: may crash (segmentation fault), show unexpected output, or behave abnormally — demonstrating memory corruption from exceeding the buffer.

## Program 5: File Comparison using Compare It! (on Windows)

1. Open **Notepad** → type:
```
Student Name: Rahul
USN: 1DS22CS001
Department: CSE
Semester: 7
College: DSCE
```
Save as `File1.txt` (e.g., on Desktop).

2. Open Notepad again → type a slightly modified version:
```
Student Name: Rahul Kumar
USN: 1DS22CS001
Department: Computer Science
Semester: 7
College: DSCE Bangalore
```
Save as `File2.txt` in the **same folder** as File1.txt.

3. Download and install **Compare It!** from `grigsoft.com/wincmp3.htm` → run installer → follow wizard → Finish.
4. Launch Compare It! from Start Menu/Desktop shortcut.
5. Click **Browse/Open** on the left panel → select `File1.txt` → Open.
6. Click **Browse/Open** on the right panel → select `File2.txt` → Open.
7. The tool auto-compares and shows both files side by side, highlighting differences (commonly: Yellow = modified, Green = added, Red = deleted).
8. Analyze: note the changed name, modified department, and added words in college name.
9. Use toolbar/keyboard shortcuts to jump between **next/previous difference**.
10. (Optional) To sync: select a difference → click **Copy Left→Right** or **Copy Right→Left**.
11. Generate a report: **File → Print or Print Preview** → save/print the differences report for documentation.
12. Review results, save reports, and close Compare It!.

One thing worth double-checking before your exam: confirm with your instructor whether security level should be **Low** for SQL injection too (Program 2 used Low), since Command Injection in your manual also sets it to Low — consistency matters if you're asked to repeat across modules.
