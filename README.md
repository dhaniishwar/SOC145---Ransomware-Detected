**Platform:** LetsDefend

**Lab Name:** SOC145 - Ransomware Detected

**Difficulty:** Medium

**Date Completed:** Jul 23, 2022

**Type:** Malware / Ransomware Detection

**Target:** MarkPRD (172.16.17.88), user account "MarkGuna"

---
<h3 align =center> GOAL </h3>

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Investigate a ransomware deteion alert, confirm whether the file ab.exe is malicious, determine if C2 Communication occurred, verify containment and close the case with the correct verdict.
<br>
<br>

---
<h3 align =center> Walkthrough </h3>

***Alert Review:***

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Opening the alert in the Monitoring tab under the Investigation Channel reveals all the critical details. 
<br>
<br>

<img width="1012" height="320" alt="1" src="https://github.com/user-attachments/assets/3546a494-5bc3-47f1-874b-6c32c938a6ad" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp;  The sigle most important field in this whole table is Device Action: Allowed. This means the file executed on the endpoint without being stopped.  It also contains hash of the file and the actual file with the password. let's analyze this alert deeper by creating playbook.
<br>
<br>

---
***Starting the Playbook:***

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Once you've reviewed the alert, head to Case Management and start the playbook. Think of the playbook as your structured checklist — it makes sure you don't skip steps under pressure.
<br>
<br>

<img width="582" height="237" alt="2" src="https://github.com/user-attachments/assets/199d347b-2ebd-48ae-9ccc-9c43bb868955" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; in this case , the file had to be reached and pulled down from an external location before it even landed on MarkPRD, so the fitting classification is "Unknown or unexpected outgoing internet traffic". This was the way the ransomware payload always arrive into targat system.
<br>
<br>

<img width="588" height="235" alt="3" src="https://github.com/user-attachments/assets/57697238-8d22-4575-be58-d6c7f356087a" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Let's head to log Management and Endpoint security, To see what happened exactly.
<br>
<br>

---
***Log Management and Endpoint Investigation:***

<img width="631" height="287" alt="4" src="https://github.com/user-attachments/assets/9db2d490-abb1-4265-a372-a7dd7b61fb8d" />

<img width="632" height="300" alt="5" src="https://github.com/user-attachments/assets/61c7ae25-26e1-4ea8-b57b-cfce2c81f59e" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; If we see closly in the log we can notice that this log nearly seven weeks (April 4, 2021) before the ransomware alert (May 23, 2021) and the process responsible is chrome.exe, not ab.exe. that means this is unrelated browers activity and the log has nothing to do with the current incident. Move on to Endpoint 
<br>
<br>

<img width="752" height="525" alt="6" src="https://github.com/user-attachments/assets/6ddc7691-bed5-4153-9d7e-3b436b17e604" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; we can see that the Last Login was in Aug 29, 2020. That was way before the alert but this does not matter much. what matters here is the processes tab, which lists 16 running processes, including ab.exe itself. By this and the Device Action: Allowed form the alert, we confirm this malware is "Not Quarantined".
<br>
<br>

---
***Malware Analyze:***

<img width="576" height="266" alt="7" src="https://github.com/user-attachments/assets/5706f3ff-30e2-420a-8746-747fe5bd8600" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; let's copy the hash from the alert and run the hash on VirusTotal to analyze.
<br>
<br>

<img width="1265" height="683" alt="8" src="https://github.com/user-attachments/assets/2c10df29-39a6-4e27-825c-d6b8c900b646" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Searching the hash returns a detection ratio of 58 out of 69 security vendors flagging the file as maliciousInterestingly, VirusTotal shows this exact hash under the filename taskhost.exe rather than ab.exe. Attackers frequently rename payloads to blend in and naming a malicious file "taskhost.exe" is a deliberate attempt to look like a legitimate Windows system process. If you want go to Relations tab, under Name section we can see the file name ab.exe and othe file name related to this hash. This confroms it was "Malicious" but still let's head over to AnyRun to find what the filr exactly doing. 
<br>
<br>

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Let's get the URL for the file from the alert by right click the Download button and copy the link. Run the link on AnyRun to analyze.
<br>
<br>

<img width="1280" height="727" alt="9" src="https://github.com/user-attachments/assets/09323845-8e4a-4f46-92d0-728a543ef865" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; [Note: when you download the file, the name of the file will be ab.bin. we have to change the name into ab.exe to run on the windows]
<br>
<br>

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; The task is tagged ransomware and avaddon and the banner immediately flags "Malicious activity". The malicious ab.exe process carries a 100/100 malicious score and its command line shows it running out of a temporary WinRAR extraction folder under the user's AppData path. We can also notice, that nothing in the visible connections stands out as a distinct C2 server. based on this analysis, we again confirm the file is "Malicious".
<br>
<br>

---
***Contamination:***

<img width="591" height="293" alt="10" src="https://github.com/user-attachments/assets/1ebdbd72-00c6-4a59-af9a-8856b3e92ece" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Given that AnyRun network activity showed nothing but legitimate traffic and the log we found earlier was unrelated browser activity from weeks before the infection alert. There's no evidence in the available logs that this specific ransomware sample ever reached out to a C2 server. The correct answer is "Not Accessed".
<br>
<br>

<img width="813" height="514" alt="11" src="https://github.com/user-attachments/assets/66154f1a-3d14-4c78-89e4-032f0a1a672c" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; We have to isolate the target host to prevent from spread across shared drives or reach other systems. For that we hve to turn ON the "Host Contained".
<br>
<br>

---
***Documentation:***

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; This was the area, where we log our IOCs so they can be used for future detection and threat hunting across the environment. We add three artifacts: Target Host IP, Hash of the file and URL from were the file was doownloaded.
<br>
<br>

<img width="579" height="326" alt="12" src="https://github.com/user-attachments/assets/4422b526-aa1a-4a52-9bc9-3a553bf051c3" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; The Analyst Note is where I write the full story in plain language, as if explaining it to someone who wasn't there for any of the investigation.
<br>
<br>

<img width="574" height="312" alt="13" src="https://github.com/user-attachments/assets/b84d2b75-4d4b-49b0-98bc-04913171e87b" />

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; The case is closed as a "True Positive".
<br>
<br>

---
<h3 align =center> Summary </h3>

<br>
<br>
&nbsp;&nbsp;&nbsp;&nbsp; Afile called ab.exe landed on a computer named MarkPRD and it turned out to be real ransomware. It ran on the machine and started deleting the built-in Windows Backups (called shawdow copies) which is exactly what ransomeware daes to stop you from recovering your files without paying. Two separate tools, VirusTotal and AnyRun both confirmed the file was dangerous, so there was no doubt about the verdict. This alert is a confirmed True Positive.
<br>
<br>
