# DMR-Vbox
Private Voice box (recorder and answering machine) for DMR / Semplice segreteria "radiofonica"

The aim of the project is to have a system that connects to the DMR network and records messages directed to your ID, and that the messages can then be listened to by choice by sending them as an attachment to an email, listed on a php page or directly through the radio.

Compared to the code of [narspt](https://github.com/narspt/DMRVMsg), I introduced the reading of a configuration file instead of having to insert all the variables on a command line, reading and writing from/ for mysql server, sending messages as an attachment to an email and finally listening back to the recorded messages by the recipient using only the radio.

The program can be compiled on both Raspberry Pi and Debian on PC. I write Debian because it is the distribution I used for development. Other distributions may be suitable but the name of the required packages or libraries may likely change.

Before starting, you must already have a working mysql/mariadb server. Obviously if you only want to use email, mysql is not needed, but you need to modify the source so that there are no references to sql.
However, I will limit myself to the explanations for the “Full” version.<br>
On mysql create a database and a table as shown below.<br>
For the subsequent compilation of dmrvbox you need to have the libmariadb-dev library installed

Another server, this time to be compiled, is md280-emu which is used to convert the frames from ambe (DMR) to pcm and vice versa and therefore write and read the .wav files
You can find the source here:
usual git clone, then go in, type make and once the compilation is finished you get the server that you can put wherever you want.<br> I opted to keep each executable inside separate folders, all under /opt.<br>
The command line for the server is ./md380-emu -s 4000<br>
On my repository, you can find an example of a service file to feed to systemd and thus have both md380-emu and dmrvbox managed as a service by the operating system.
Please note that md380-emu must be compiled on raspberry or possibly cross-compiled on PC, therefore to make it work from PC it must be called via the qemu arm emulator.
So if you run everything on Raspberry, compile and you're good to go, otherwise on PC after compiling with the cross-compiler you have to install qemu-user-static and qemu-system-arm

The library that manages the email can be found here: , download the zip and extract it where you do the development.
To be able to compile it, you must necessarily install libcurl4-gnutls-dev
Then just enter the libquickmail folder, type make and if all goes well make install.
And as far as the library is concerned, here we are.<br> However, it is necessary for the quickmail.h file to be visible after the compiler, <br>I therefore chose to copy it from the libquickmail folder into /usr/include/email/quickmail.h

We move to the folder cloned from github (git clone https://github.com/Anubi61/DMR-Vbox)

The command line to compile everything is
gcc -o dmrvbox dmrvbox.c ini.c $(mariadb_config --include --libs) -lquickmail

Note: I suggest copying the executable to a folder under /opt.
The important thing is that inside this folder there are the messages and voices folders, the executable file and the getids.sh file (used to download the DMR ID database)

For example, my prototype under test is here:

/opt/DMRVbox-qtf<br>
-rw-r--r-- 1 root root 585 Jun 2 09:08 config.ini<br>
-rw-r--r-- 1 root root 5517119 May 31 20:36 DMRIds.dat<br>
-rwxr-xr-x 1 root root 68800 Jun 2 09:03 dmrvbox<br>
-rw-r--r-- 1 root root 111 May 7 09:09 getids.sh<br>
drwxr-xr-x 2 root root 4096 Jun 3 07:27 messages<br>
drwxr-xr-x 2 root root 4096 Jun 2 09:00 voices<br>

Go on. We have the executable and now let's move on to the config.ini (it can also have another name)

The config.ini file is understandable by reading it, just open it and read.
Please note that the [opts] block enables or disables compiled functions.
That is, I only want to have access to sql and I'm not interested in email or radio, so I'll set:<br>
[opts]<br>
use_email = "false"<br>
use_sql = "true"<br>
use_radio = "false"<br>

Or, I just want to listen to it on the radio and I'm not interested in sending it by email:<br>
[opts]<br>
use_email = "false"<br>
use_sql = "true" # irrelevant, use_radio sets use_sql independently<br>
use_radio = "true"<br>

In the [dmr] block, pay attention that the dmrid key is not your ID but the id + two numbers to identify it on the DMR network as if it were a hotspot. <br>For example my id is:
2230026 and the dmrid of the voicebox is 223002604
The dmrtg key instead contains your 7-digit ID.
Note: To use the voicebox in tg mode dmrtg = your ID, while in cp mode (private call) dmrtg = 0

Finally, in the voices folder there are voice messages generated online. I am both English and Italian.<br>
For the user who listens the recorded messages via radio<br>
err = file message does not exist <br>
n1 = a new message<br>
n2 = new messages<br>
nx = end of messages<br>
nn = no new messages<br>
<br>
For the correspondent who records them.<br>
tx0 = welcome<br>
tx1 = message recorded and 73<br>
<br>
Provided you have everything working, here's how to use it:<br>

Corresponding User sets our "news" and presses the ptt.<br>
If the transmission time is less than 3 seconds, the tx0 file is sent, otherwise the tx1 file and its transmission was recorded in the messages folder.<br>
If we have chosen to use email and configured the parameters in the ini file, the messages arrive as attachments.<br>
If instead we have opted for the radio, we set our news on our radio and press PTT (briefly). After a few seconds depending on the condition, the system can transmit nn or n1 or n2.<br>
In case of n1 or n2, to listen to the next message, briefly press ptt again to advance listening until ending with nx.<br>
At this link, a practical demonstration of use.<br>

That's all for now.<br>

73 of IU4QTF (formerly IK4DRV)
