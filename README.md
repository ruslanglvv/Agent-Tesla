


### Introduction

Agent Tesla is a remote access trojan written in .NET that has been actively targeting users with Microsoft Windows OS-based systems since 2014. It is a versatile malware with a wide range of capabilities, including sensitive information stealing, keylogging and screenshot capture.

In this report i will analyze sample of this malware that was uploaded to MalwareBazaar.

### Analysis


![[Pasted image 20260211123452.png]]



After downloading the sample, an initial inspection of the file revealed the presence of an obfuscated command string, indicating potentially malicious behavior.

![[Annotation 2026-02-09 134552 1.png]]

Using the decoding function identified in the code, the obfuscated command string was successfully decrypted, resulting in the following output:

![[Annotation 2026-02-09 142440.png]]

After manual refactoring and cleanup, the code was transformed into a readable and structured form suitable for further analysis.

![[Annotation 2026-02-09 142635 1.png]]

The analysis revealed that execution of the program results in the download of a file named **DT9.txt** from a remote source.

To further investigate its contents, the file was retrieved manually using the `curl` utility and saved locally for examination.

Upon reviewing the downloaded file, multiple obfuscated variables were identified, indicating that the file likely contains an additional encoded payload.

![[Annotation 2026-02-09 144124.png]]

The variable `r0t0r` underwent additional processing, where all trailing occurrences of the character “W” were replaced with “00”, effectively restoring the original binary byte representation.
   
   ![[Annotation 2026-02-09 151113.png]]

 The identified function performs byte-level reconstruction, effectively decoding the data from its obfuscated representation.

The variables `y74gh00rffd` and `TOgr` were also processed in a similar manner: the `">_"` sequence at the end of the strings was replaced with `"0x"`, which represents the standard hexadecimal prefix. This technique was used to conceal the actual hexadecimal values within the code.
   ![[Annotation 2026-02-09 153520.png]] ![[Annotation 2026-02-09 153544.png]]
The extracted files were saved in `.BIN` format to preserve their original binary structure.  
Subsequent analysis of the files’ properties was performed using Detect It Easy (DIE) to determine file type and characteristics.
![[Annotation 2026-02-10 091308.png]]
![[Annotation 2026-02-10 091330.png]]

The first file was identified as a DLL, while the second was an executable (EXE); both were developed using the .NET framework.  

The EXE was packed and protected using obfuscation techniques, preventing successful static decompilation.

![[photo_2026-02-11_13-42-42.jpg]]

After unpacking the GZIP archive, dynamic analysis of the sample was conducted. As the managed assembly was loaded directly into memory at runtime, the MegaDumper tool was employed to extract it. 
The sample was executed within an isolated virtual environment, allowing MegaDumper to successfully dump all loaded .NET assemblies from the process memory.

![[Annotation 2026-02-11 104741.png]]

![[Annotation 2026-02-11 104844.png]]

The extracted file was successfully decompiled using dnSpy.  

Code analysis revealed that the malware operates as an information stealer and keylogger, exfiltrating collected data via the SMTP protocol.

![[Annotation 2026-02-11 113718 1.png]]

The malware collects system information, including operating system details, processor specifications, and stored user credentials, enabling the exfiltration of sensitive data.


![[Annotation 2026-02-11 105153.png]]

Hardcoded SMTP authentication credentials, including an email address and password, were found in the malware code.

### Conclusion

The analyzed sample was identified as a multi-stage .NET malware belonging to the Agent Tesla family.  
The executable employs code obfuscation and reflective loading to evade static analysis.  
Its core capabilities include credential harvesting, keylogging, and data exfiltration via SMTP.  
Memory dumping using MegaDumper allowed extraction of the managed assembly.  
The dumped assembly was successfully decompiled in dnSpy, enabling identification of the data collection routines and exfiltration mechanisms.  
This analysis highlights the effectiveness of combining static and dynamic approaches for examining protected .NET malware samples.
### IOCs

- filename: doc20241402070611.img 
	sha256: 8FF5AE82390178A13604AA06329C05295BA6631BFACC4FAB7004B56EFEE885D6


- filename: DT9.txt 
	sha256: EA70E6C8908AC145D3AE65214BBF7DFA0509EE0044BEF2141733D09002AE0280

- filename: file2.exe
	sha256: 968BF956EB1730D73B90695D96ABA0C3589EDCF967FADC596A32DB78D6BFB6CD

- hxxps://delp-heizungsbau.de/DT9.txt

- SmtpServer:  "sslout.de"  

- service@cosmedicus[.]de

- service2@cosmedicus[.]de

### Sources
- https://bazaar.abuse.ch/sample/8ff5ae82390178a13604aa06329c05295ba6631bfacc4fab7004b56efee885d6/#file_info
