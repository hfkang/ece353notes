## Viruses and More

**trojan horse** is a malicious payload that presents and altered vrsion of a useful program, and tricks the user to run it. 

##### 1) Fake login
unsuspecting user begins login. On windows this is mitigated by requiring a **non-trappable** key sequence. User level programs cannot capture it 

##### 2) Spyware 
Spyware daemon bundled with freeware/shareware programs. hijacks violations of **principle of least privilege**. Either user has unecessary permissions or O/S is too lenient. 

### Buffer OverFlow
Write data larger than buffersize, so that we go beyond allocated space, and overwrite the return address. It can be configured to run code in the buffer. 

_Mitigations_  
Disable execution in stack section (Solaris), or mark pages non-executable (NX in x86-64)

**virus** requires human interaction to spread  
**worm** self replicating program  

## Final Exam

More questions than midterm but same format 
Roughly bellcurved by making the questions hard. He wont bellcurve down?

### Textbook sectionsion

Virtual Memory  
Security  
Page replacement
File systems (*journalling fs*)
Virtualization
Mutex/Synchronization 

Focus on post-midterm, and actual midterm. There will be a question from midterm that's been twisted. 
