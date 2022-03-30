# HTB_Paper_writeup
Write-up of the machine "Paper" on HackTheBox. This machine include some LFI(I guess). This machine took me two days to complete and I learned some things interesting.

## Score:
**Points: 20**

## Solution:

## WEB
When we starts the machine it give us an IP address (in my case 10.10.11.143). It's a default page of an apache server. We can ran nmap, nikto to see if there's any suspicious stuff.
[nmap.log]
[nikto.log]

Looking at `nikto.log` we can see it capture an "Uncommon header 'x-backend-server' found, with contents: office.paper". We could find this 'x-backend-server' using `whatweb`
[whatweb.log]

Good, now we need to add the value of that header in our /etc/hosts file. So that we can access the site http://office.paper.
[host file]

Acessing this site, we come across to an Wordpress blog. Goingo straight to the point, in the post "Feeling Alone!" we can see that nick has commented to Michael(the owner) that he sould "remove the secret content from the drafts" since they are not secure as he thinks. Using Wappalyzer extension (or just going to page source), it's possible to see that the version of wordpress is 5.2.3, which is vulnerable.

[searchsploit.png]

The exploit says that if we insert `?static=1` to a wordpress URL, we can leak its secret content. Trying this...
[static.png]

It give us a secret link to registration to a chat system. Copy and paste the link, it take us to a registration page where we can simply create an account an enter to the chat. 
[rocket_notloged.chat]
[rocket_loged]

Here we are introduced to a bot(recyclops) who can retrieve some informations to us, but we are going to use only two: "file" and "list" (`cat` and `ls` command respectively). We can use the bot by just going to chat in direct message and type "recyclops list", and it will list the files on the present directory. But there's a vulnerability here, if we type "recyclops file ../../../etc/passwd" it'll retrieve to us the file passwd. Great, now we just need to "leverage"!.
[etc_passwd_bot.png]

In this part I didn't know what to do after the LFI, so just googling i found a site which tells how to do it. The file /proc/self/environ has some informations of the environment of the process. Just type "recyclops ../../../proc/self/environ". We got it! Now we have the credentials of the user "dwight".
[proc_self]

Now we can use SSH to connect to dwight terminal. We in! 

## Privilege escalation

