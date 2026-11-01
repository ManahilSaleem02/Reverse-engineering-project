# DIVA Android Reverse Engineering
# DAA Assignment 1: Reverse Engineering a Vulnerable Android Application
 
![Platform](https://img.shields.io/badge/platform-Android-green)
![Target](https://img.shields.io/badge/target-DIVA%20(training%20app)-orange)
![Purpose](https://img.shields.io/badge/purpose-educational-blue)
 
Security analysis and controlled modification of **DIVA (Damn Insecure and Vulnerable App)**, a deliberately vulnerable Android application provided by the instructor for training.
 
| | |
| --- | --- |
| **Course** | Secure Software Design and Development ( theory)  |
| **Program** | BSCB, Department of Cybersecurity |
| **Student** | Manahil Saleem (74408) |
 
---
 
## Disclaimer
 
This project was carried out **only** on an instructor-provided, intentionally vulnerable training application, on a personally owned test device. No third-party or production software was analysed or modified. Do not apply these techniques to applications you are not authorised to test.
 
---
 
## Objectives
 
1. Install and observe the original application's normal behaviour.
2. Reverse engineer the APK (manifest, resources, code) and trace one feature.
3. Identify and evaluate at least one security weakness.
4. Apply one approved modification to show how resources and code determine runtime behaviour.
5. Document everything in a report, with screenshots, a screen recording and a verifiable Git history.
---
 
## Application Details
 
| Item | Value |
| --- | --- |
| Name | DIVA (Damn Insecure and Vulnerable App) |
| Package | `jakhar.aseem.diva` |
| File | `original/diva-beta.apk` |
| Test device | REALME C75, Android RMX3941_16.0.5.1300(EX01) (physical device) |
 
---
 
## Tools Used
 
| Tool | Purpose |
| --- | --- |
| JADX-GUI | Static analysis and decompilation |
| Apktool | Decoding and rebuilding the APK |
| ADB (Platform-Tools) | Install, logcat, device interaction |
| zipalign, apksigner (Build-Tools) | Alignment and signing |
| JDK / keytool | Signing key generation |
| scrcpy / OBS (optional) | Screen recording |
| Git / GitHub | Version control and proof of work |
 
---
 
 
## Methodology
 
### 1. Setup and baseline
Installed the original APK over ADB on a physical phone, explored the main menu and recorded the normal behaviour of the Insecure Logging module.
 
### 2. Static analysis
Opened the APK in JADX-GUI and reviewed `AndroidManifest.xml` (permissions, activities, exported components, debug and backup flags). Traced the Insecure Logging feature from the UI to the code path that handles input.
 
### 3. Vulnerability identification
Located the weakness in `jakhar.aseem.diva.LogActivity`, method `checkout(View)`, and confirmed it dynamically with `adb logcat -s diva-log`.
 
### 4. Modification
Decoded the APK with Apktool, edited a visible label in `res/values/strings.xml`, rebuilt, aligned, signed and verified the package, then reinstalled it on the device.
 
---
 
## Findings
 
### Insecure logging of sensitive data
 
| | |
| --- | --- |
| **Type** | CWE-532, Insertion of Sensitive Information into Log File |
| **Reference** | OWASP MASVS-STORAGE / Mobile Top 10 M9 |
| **Location** | `LogActivity.checkout()` |
| **Description** | When checkout fails, the full credit card number entered by the user is written to the system log with `Log.e("diva-log", ...)`, while the user only sees a generic error message. |
| **Impact** | Card data is readable from logcat via ADB (and by other apps on legacy Android versions), and may be captured in bug reports and diagnostics. |
| **Risk rating** | Medium to High |
| **Remediation** | Never log user-supplied sensitive values. Log identifiers or error codes only, mask values if unavoidable, and strip debug logging from release builds (R8/ProGuard or `BuildConfig.DEBUG` checks). |
 
Other observations from the manifest review are listed in the full report.
 
---
 
## Modification Performed
 
| | |
| --- | --- |
| **File** | `decoded/res/values/strings.xml` |
| **Change** | Main menu label `1. Insecure Logging` changed to `1. Hacked by MANAHIL SALEEM` |
| **Result** | Visible label changed with no change to application logic |
| **Note** | This patch changes the label only. The logging vulnerability itself is unchanged. |
 
---
 
## Reproducing the Work
 
Prerequisites: JDK, Git, Android Platform-Tools, Android Build-Tools, Apktool, a phone with USB debugging enabled.
 
```bash
# verify device and install the original
adb devices
adb install original/diva-beta.apk
 
# observe the leak
adb logcat -s diva-log
 
# decode
java -jar apktool.jar d original/diva-beta.apk -o decoded -f
 
# rebuild after editing decoded/res/values/strings.xml
java -jar apktool.jar b decoded -o diva-unsigned.apk
 
# create a signing key (do not commit it)
keytool -genkeypair -v -keystore my-release-key.keystore -alias divakey -keyalg RSA -keysize 2048 -validity 10000
 
# align, sign, verify (adjust the Build-Tools path)
zipalign -p -f 4 diva-unsigned.apk diva-aligned.apk
apksigner sign --ks my-release-key.keystore --ks-key-alias divakey --out diva-final-submission.apk diva-aligned.apk
apksigner verify --verbose diva-final-submission.apk
 
# reinstall (signature differs, so uninstall first)
adb uninstall jakhar.aseem.diva
adb install diva-final-submission.apk
```
 
---
 
 
## Commit History (Proof of Work)
 
| Step | Commit message |
| --- | --- |
| 1 | Initial setup: add original diva-beta.apk, README, .gitignore |
| 2 | Static analysis: identified insecure logging in `LogActivity.checkout` |
| 3 | Decompile `diva-beta.apk` with Apktool into `decoded/` |
| 4 | Patch `strings.xml`: change main menu label |
| 5 | Rebuild, align, sign and verify modified APK; add final assets |
 
Run `git log --oneline` to see the full chronological history.
 
---
 
## Deliverables
 
- [x] `DAA_Assignment_Report.pdf`
- [x] `practical_walkthrough.mp4` (submitted separately)
- [x] `diva-final-submission.apk`
---
 
## References
 
- [OWASP Mobile Application Security Testing Guide (MASTG)](https://mas.owasp.org/MASTG/)
- [OWASP MASVS](https://mas.owasp.org/MASVS/)
- [CWE-532: Insertion of Sensitive Information into Log File](https://cwe.mitre.org/data/definitions/532.html)
- [Apktool](https://apktool.org/)
- [JADX](https://github.com/skylot/jadx)
- DIVA Android by Payatu / Aseem Jakhar
---
 
## License and Use
 
Submitted for academic assessment. The DIVA application belongs to its original authors and is used here solely for educational analysis.
 