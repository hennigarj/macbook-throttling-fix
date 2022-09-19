# macbook-throttling-fix

howdy, if you're here i feel very bad for you because you are having an issue that is nearly impossible to solve without doing an absolutely excrutiating deep-dive into hackintosh forums/superuser/stackoverflow where information is rather vague and unclear. you have a macbook pro that is throttling itself to death and unusability. i am sorry. but, with the complete lack of info online, i figured i would throw this together for anyone who is deep in google and finds it lol

lo and behold:

FIXING A THROTTLED MACBOOK PRO 16" (2019) 

REMOVING IOPLUGINPLATFORMFAMILY.KEXT
1. this is a doozy so get ready
2. enter recovery mode and go to the terminal. enter 

        csrutil disable && csrutil authenticated-root disable
        
3. restart back into macos and open terminal and in bash:

        diskutil list
        
and then find your macintosh hd on the list (probably disk3s5). in bash:

        diskutil mount disk3s5
        
now 

        sudo mount -uw /Volumes/Mactintosh\ HD\ 1
        sudo mkdir /Volumes/Macintosh\HD \ 1/KextBackup

4. cd to /Volumes/Macintosh\ HD\ 1/System/Library/Extensions and, in bash:

        sudo mv IOPlatformPluginFamily.kext Volumes/Mactinosh\HD \1 /KextBackup
		    sudo mv AppleSMCLMU.kext Volumes/Mactinosh\HD \1 /KextBackup

5. now create a snapshot and tag it for the next reboot: in bash,

        sudo /System/Library/Filesystems/apfs.fs/Contents/Resources/apfs_systemsnapshot -s "SnapshotName" -v /Volumes/Macintosh\ HD\ 1
        sudo /System/Library/Filesystems/apfs.fs/Contents/Resources/apfs_systemsnapshot -r "SnapshotName" -v /Volumes/Macintosh\ HD\ 1
	
6. reboot and enjoy your very slow boot time resulting from the removal of the throttling kexts

DEALING WITH BD_PROCHOT, OVERHEATING, UNDERVOLTING & POWER LIMITS
1. DOWNLOAD VOLTAGESHIFT 
2. DOWNLOAD INTEL POWER GADGET ('IPG')
3. OPEN INTEL POWER GADGET AND KEEP NOTE OF TEMPS, FREQ, POWER
3. run these scripts in bash from VoltageShift directory:

        sudo chown -r root:wheel VoltageShift.kext && sudo chown -r root:wheel voltageshift

4. if kernal_task is throttling, or you're locked at ~1.0ghz, you must write to MSR:
	1. to disable bd_prochot you must determine values specific to your model. the result will be binary, convert to hexadecimal. in bash:

		    ./voltageshift read 0x1FC. 

	disable bd_prochot, in bash:

		    ./voltageshift write 0x17FC 0x64005E' (note if you are no longer throttled)

2. to disable turbo boost, if required to prevent overheating/crash, in bash:

        ./voltageshift turbo 0' (note clock frequency return to base speeds in IPG)

3. to set your clock frequency above the throttled ~1.0ghz, find your models base clock
	frequency; convert it to hexadecimal; in my case, "1700" or 2.6ghz. then in bash:

        ./voltageshift write 0x199 1700 (note clock frequency in IPG)


5. determine your currently set CPU power limit:

        ./voltageshift mon (note PL1 & PL2)

6. set a lower power limit (do this wisely through trial and error, start with small decrease and  use Intel Power Gadget to monitor power usage, clock frequency, and temperatures in response). running a benchmark during this could be useful to determine stability. in bash:

        ./voltageshift power <PL1> <PL2> (PL1 & PL2 from earlier)

7. now you should find stability under higher cpu usage, though performance is limited due to the nature of the lower power limit and base clock frequency. its better than 0.8ghz tho. 

SETTING VOLTAGESHIFT ON REBOOT
1. voltageshift writes to msr, meaning whenever your computer restarts or sleeps it reverts to default. it does have a built in tool to set a launchagent, but you cannot write to msr with it. so,
2. create a shell script that runs the commands you did earlier, for example:

	        #!/bin/bash
	        /Users/username/Desktop/voltageshift power 18 18 && echo SUCCESS || echo FAIL
	        /Users/username/Desktop/voltageshift write 0x1FC 64005E && echo SUCCESS || echo FAIL
	        /Users/username/Desktop/voltageshift write turbo 0 && echo SUCCESS || echo FAIL
          /Users/username/Desktop/voltageshift write 0x199 0x1700 && echo SUCCESS || echo FAIL
	
3. now create an app to run the script on boot:
	open script editor
	write something along the lines of,

        tell application "Terminal"
               activate
			         do script "<script directiory/script>" in window 1
		    end tell
	save it as an app from the save as menu.
	in bash give it proper permissions and make it executable:
		
        sudo chmod -R 755 <app> 
        sudo chmod a+x <app>
        sudo chown -R root:wheel <app>

4. create a launchagent that will run your app to execute the script when you log in automatically. i suggest downloading launchcontrol as a gui for launchctl, but otherwise, open textedit and paste this with modifications to fit your context:
	
        <?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
        <plist version="1.0">
        <dict>
        <key>Label</key>
        <string>com.<username>.<launchagentname></string>
        <key>Program</key>
        <string><full path of the script app you just made></string>
        <key>RunAtLoad</key>
        <true/>
        <key>StandardErrorPath</key>
        <string>/tmp/com.<username>.<launchagentname>.stderr</string>
        <key>StandardOutPath</key>
        <string>/tmp/com.<username>.<launchagentname>.stdout</string>
        </dict>
        </plist>

5. save the file as the same value you gave the label in .plist format. correct the permissions:
	
        sudo chmod 755 com.<username>.<launchagentname>.plist
        sudo chown root:wheel <username>.<launchagentname>.plist

6. restart and hopefully the script will run in a terminal window telling you if each command in the script succeeded or failed.

gl lol

		
