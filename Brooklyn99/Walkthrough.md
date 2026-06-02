# THM Brooklyn 99 Walkthrough

Author: DarkRose Security
Date: 6/2/2026
Platform: Tryhackme

**Tools Used**
- Nmap
- Hydra
- Stegseek
- Ftp
- SSH

**Enumeration**
After deploying the machine and obtaining the target IP (`10.146.146.136`), I started an nmap scan to see what was exposed:

![[nmap.png| 600]]

After the scan finished, we can see that ports 21, 22, and 80 are all open. We also see that FTP allows anonymous login access, so that immediately became my first target to see if I could find anything useful. I logged in using the username: `anonymous` & password: `anonymous`: 

![[ftp.png]]

Looking at the FTP server, we can see a single file called `note_to_jake.txt`. After using the `get` command in FTP to transfer the file to my own machine, I’m able to read it and see the following message: "From Amy, Jake please change your password. It is too weak and Holt will be mad if someone hacks into the nine nine". After finding this, my first thought was that we might have to brute force a password for SSH. Before trying that though, we still have port 80 open, so I decided to take a look at the website first:

![[CTF Writeups/THM Brooklyn 99/Photos/website.png| 700]]

At first glance, the website doesn’t really reveal anything useful, so I decided to check the source code to see if there was anything hidden in the comments or somewhere in the page itself:

![[source_code.png| 900]]

**Steganography** 
Sure enough, inside the source code we find a comment saying: "Have you heard of steganography?" After thinking about it for a minute, I figured the background image was probably hiding something. I downloaded it using `curl -O http://IP ADDRESS/brooklyn99.jpg`, and then ran Stegseek against the image to see if there was anything hidden inside it:

![[steg.png| 600]]

Sure enough, this ended up giving us Holt’s password from the hidden data inside the image.

At this point we now have two different ways to get into the machine. We can SSH in as Holt using the password we just found, or we can try to brute force Jake’s password for SSH. First, I’m going to take the Holt route, and after that I’ll also explore the Jake path as well.

**Holts Route**
So the first thing we do now is ssh into the target machine using the credentials we found in the steganography step to login to Holt's account. After logging in I ran `ls -la` to see what files Holt has access to and found the user.txt flag right away: 

![[Holt_ssh.png| 600]]

Now with the first flag in hand now we need to get the second flag also known as the root flag. The root flag usually is in the `/root` directory but as Holt we don't have access to that so we have to do some privilege escalation. Since we have the password to Holt's account the first type of priv esc i like to try is `sudo -l`: 

![[Holt_sudo.png]]

After running `sudo -l` we can see that as Holt we are able to run nano. Using this info i headed over to gtfo bins and searched for nano and luckily it had it. To get a root shell using nano you just run `sudo /bin/nano` or whatever your path is to nano then CTRL R & CTRL X which will allow you to run a command inside of nano. Once you get to that point just type `reset; sh 1>&0 2>&0` then hit enter. After giving it a little to load hit enter a few more time and you should have a root shell and are able to get the root flag:

![[Holt_root.png| 700]]

Congrats!! You just completed the Brooklyn99 THM challenge as Holt. This was just one way to complete it now lets go over how to do it if you want to complete this challenge as Jake.

**Jake Route**
For this we have to go back a little bit to ftp and the note we found on the ftp server. If you remember the note said "From Amy, Jake please change your password. It is too weak and Holt will be mad if someone hacks into the nine nine" so lets try to brute force his password by using hydra and the rockyou.txt wordlist:

![[Jake_pass.png]]
After letting it run for a little bit we were able to get a match on the password. We are now able to login though ssh as Jake this time. After logging in i ran the same `ls -la` i did before but this time we didnt get the user.txt right away so i went looking for it. The first place i went was `/home` to see if there are any other users accounts i have access to. After running `ls -la` in the home directory i discovered i have access to all three accounts (Jake, Holt, Amy) on the machine: 

![[Jake_ssh.png| 500]]

At first i entered Amy's directory and ran `ls -la` but wasnt able to find anything interesting. The next account i went into is Holt and I was able to find the user.txt in his directory. Now with the user.txt flag secured is time to go get the root flag. Since we have a password that we brute forced we are able to run `sudo -l`:

![[Jake_sudo.png]]

After running it we see that we are able to run the `less` command as sudo. With this I know that you can use the less command to be able to read files. Knowing this and combining it with the fact we can run it as sudo i ran the command `sudo less /root/root.txt` and was able to read the root flag as Jake: 

![[Jake_root.png]]

**Conclusion**
Overall this was a really fun beginner-friendly box that covered a little bit of everything including enumeration, steganography, brute forcing, SSH access, and privilege escalation. What I liked most about this room is that it gives you multiple intended paths to complete it instead of forcing you down one single route. The Holt path focused more on web enumeration and steganography, while the Jake route leaned more into password attacks and credential access. Both paths eventually lead to privilege escalation but in completely different ways which made the room a lot more interesting to work through. One thing this box reinforced for me is how important enumeration really is. Almost every piece of information found early on ended up becoming useful later, whether it was the FTP note, the website source code, or checking sudo permissions after getting access to the machine. Overall I think this is a great room for beginners who want practice with basic Linux enumeration, SSH access, privilege escalation, and thinking through multiple attack paths instead of just following one exploit chain from start to finish.
