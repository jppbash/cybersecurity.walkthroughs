# TShark II: Directory
### Walkthrough
This is the second part of using the TShark tool from the command line which in my personal opinion, I found it fun, entertaining and very useful for a Cyber Security Analyst related role. 

For this you must get access to the TryHackMe VM because the information is within it. In addition, they suggest you to also follow the courses "[TShark: The Basics](https://tryhackme.com/room/tsharkthebasics)" and "[TShark: CLI Wireshark Features](https://tryhackme.com/room/tsharkcliwiresharkfeatures)". I agree with the staff and strongly, suggest you to follow those courses and practice a little prior to completing this box. It is totally worth it.

For this exercises your best friends are TShark and Virustotal. So have your toys ready and let's begin the exercise.

As a Security Analyst, you have been notified about human error due to their curiosity and poor file index. You have been assigned a packet capture file (**aka .pcap**) located in the ```~/Desktop/exercise-files``` directory. Your job is to analyse it with the tools mentiioned above and confirm the alert is indeed a true positive.

## Investigate the DNS queries
Investigate the domains by using VirusTotal.

According to VirusTotal, there is a domain marked as malicious/suspicious.

#### **What is the name of the malicious/suspicious domain?**
For this you must filter basically domain names from the pcap file. There are multiple options. You can use the `dns.qry.name` filter to extract the DNS query name. Using the command `tshark -r directory-curiosity.pcap -Y "dns.qry.name"` filters exculsively the DNS queries done but it is possible to filter data through the commands `awk, sed, etc` and also with the `-T fields` argument in order to just lookup for potential URLs to be investigated.

After playing a little bit, the  most appropriate command in my opinion is the following"
```
tshark -r directory-curiosity.pcap -T fields -e dns.qry.name | awk NF | sort -r | uniq -c | sort -r
```

This command does the following:

- Filters DNS queries from URLs to addresses getting only the domain names
- Eliminates duplicate answers
- Sort them alphabetically in reverse order

![](/TryHackMe/TShark%20II:%20Directory/Figures/1.jpg)

There are 7 domains worth to investigate. After reviewing them in Virustotal it was found that the malicious one is **jx2-bavuong.com**. Defang it and we have the first answer.

#### What is the total number of HTTP requests sent to the malicious domain?
Do not be afraid to read documentation from multiple sources or even ask a generative AI. What matters is the fact that you approch as many solutions as possible, practice on the command line and get hands-on experience.

For this question, I used perplexity and read some documentation and found two approaches. Using the filters `http.host` and `http.request.full_uri`. I am showing both to see the results. This filters made the solution very easy. Typing the commands

```
tshark -r directory-curiosity.pcap -Y "http.host matches 'jx2'"
```

and

```
tshark -r directory-curiosity.pcap -Y "http.request.full_uri contains 'jx2'"
```

both throw the same output as shown below.

![](/TryHackMe/TShark%20II:%20Directory/Figures/2.jpg)

Which is a total of **14 requests**. But just in case you do not feel confident, you can use the `wc -l` command which counts every line which is a request.

#### What is the IP address associated with the malicious domain? Enter your answer in a defanged format.

We can take the previous command without the line counting and only display the destination IP, which is the one that corresponds to the malicious domain. Since the IP address is 141.164.41.174 and there are 14 requests to that IP, the `uniq` command is useful to only print it once at the terminal.

```
tshark -r directory-curiosity.pcap -Y "http.request.full_uri contains "jx2"" -T fields -e ip.dst | uniq -c
```

Defang the IP address that that is the answer.

#### What is the server info of the suspicious domain?
We can use TShark to search for keywords. In this case, the word "server", I find it important for this query. Let's practice a little bit. COnsidering that HTTP requests are based on the TCP 3-way handshake, we can user a filter starting from TCP. The `tcp contains "server"` filter sounds promising. In addition there is a filter named `http.server` which is going to be useful because since we are looking for the server info, we need the header information to identify the web server software currently running. Furthermore, at this time, we know the IP address of the server which is 141.164.41.174. This is a great opportunity to play around with different filters and develop muscle memory. I used the command

```
    tshark -r directory-curiosity.pcap -Y "ip.addr==141.164.41.174" -T fields -e http.server | awk NF | sort -r | uniq -c | sort -r
```

The next command provides the same output. We can see the results on the next picture.

```
    tshark -r directory-curiosity.pcap -Y "tcp contains 'server'" -T fields -e http.server | awk NF | sort -r | uniq -c | sort -r
```

![](/TryHackMe/TShark%20II:%20Directory/Figures/3.jpg)

The drawn rectangle is the answer. 

#### Follow the "first TCP stream" in "ASCII". Investigate the output carefully. What is the number of listed files?
This is something that requires you to pay attention to details. In Wireshark, you normally right-click on a packet and then use the option ***Follow TCP Stream***. In TShark, you use the filter `-z follow,tcp,ascii,0 -q` which in WireShark is the same as `tcp.stream eq 0`. Now, when we review this, pay attention to the content of the First paragraph of the HTML code. We find three files: **123.php, vlauto.exe and vlauto.php**. Therefore the answer is 3.

![](/TryHackMe/TShark%20II:%20Directory/Figures/4.jpg)

#### What is the number of listed files?
Kind of straight forward. The first file found is 123.php. Defang it to get the answer.

#### Export all HTTP traffic objects. What is the name of the downloaded executable file? Enter your answer in a defanged format.
For this it is important to understand the argument `--export-objects`. It allows the user to extract and save files from a capture file based on specific protocols. This functionality enables the exportation of objects, such as files, transmitted over a network via protocols including, but not limited to, HTTP, SMB, and TFTP.

Protocol specification is requisite, wherein the user designates the protocol for which object exportation is desired. For instance, `http` for HTTP files or `smb` for SMB files.
Additionally, a directory must be specified, wherein the exported files will be saved.

The syntax of this argument is `--export-objects protocol,output_folder`.

After this research, and understanding that I am technically downloading a file from a packet capture. I used the following command:


```
    tshark -r directory-curiosity.pcap --export-objects http,$HOME/Desktop/exercise-files -q
```

The command does the following, we are reading the pcap file, extract and save all HTTP objects and store them in the exercise-files directory which is under /home/ubuntu/Desktop. the `-q` option is to run TShark in quiet mode. Listing we found a file named **"vlauto.exe"**. Defang it to get the answer.

#### What is the SHA256 value of the malicious file?
This is easily obtained with the `sha256sum` command.

#### Search the SHA256 value of the file on VirtusTotal. What is the "PEiD packer" value?
Once the has value is inserted, go to the Details Tab and search. It is a .net executable.

#### Search the SHA256 value of the file on VirtusTotal. What does the "Lastline Sandbox" flag this as?
Let's go to the behaviour tab. You might encounter some sandboxes. Uncheck all of them except Zenbox which is the last one. Going to Analysis Sandbox Detection section, we can appreciate this executable is catalogued as **MALWARE TROJAN**. That is your final answer.

### Thank you for reading!!
### Hopefully, this write-up has been useful and gave you great insights and inspired you to get your hands dirty to get skills in cyber security.
### This is my first time writing it so, I appreciate every feedback using the discord app or through the TryHackMe platform, add me and let's get some great content to expand the cyber security community.

``` HTML
    <iframe src="https://tryhackme.com/api/v2/badges/public-profile?userPublicId=2570404" style='border:none;'></iframe>
```

## **Keep learning, stay safe and have fun learning!!**