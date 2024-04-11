+++
title = 'Reinstalling a MacOS Sierra Mac - No bag entry'
date = 2024-04-11T11:58:15+03:00
draft = false
+++
Some MacOS Sierra Mac's don't let you reinstall (Option-Shift-Command-R default MacOS version or Option-Command-R new version) MacOS. I got these handy instructions from [Mr. Macintosh](https://www.youtube.com/@Mr.Macintosh).
In Terminal do:  
`curl -L https://archive.org/download/sierraurl/sierra.txt`
 - Open new Terminal, Check date & time to make sure it is correct  
`date`
 - If not correct make sure you are connected to wifi or ethernet and run this command  
`ntpdate -u time.apple.com`
 - Change directory to Macintosh HD  
`cd /Volumes/Macintosh\ HD`
 - Download the macOS Sierra 10.12.6 InstallOS.dmg from Apple.com using curl. Note this is a 4.7GB file and might take a while or fail. If it fails run it again.  
`curl http://updates-http.cdn-apple.com/2019/cert/061-39476-20191023-48f365f4-0015-4c41-9f44-39d3d2aca067/InstallOS.dmg -o InstallOS.dmg`
 - We need to mount the InstallOS.dmg  
`hdiutil attach InstallOS.dmg`
 - Now we need to install the "Install macOS Sierra.app" from the InstallOS.dmg to Macintosh HD volume  
`installer -pkg /Volumes/Install\ macOS/InstallOS.pkg  -target /Volumes/Macintosh\ HD`
 - The Install macOS Sierra.app is now in the /Volumes/Macintosh\ HD/Applications folder. We can now start the Sierra installer!  
`/Volumes/Macintosh\ HD/Applications/Install\ macOS\ Sierra.app/Contents/MacOS/Installassistant_springboard`

In a few seconds the Sierra installer will start BEHIND the terminal window. If you don't see it make the terminal window but DON'T close it as it will stop the installer. Now step through the installer you are good to go now!

**Optional Create macOS Sierra USB installer while we are here. Take a picture of these instructions with your phone as we need to close terminal.**

 - Plug in your USB Drive and make sure to backup all data as we are going to ERASE the dive in disk utility 
 - When erasing the USB drive, make sure you click "view" and click "Show all devices"
 - Make sure to use Format the USB Drive with macOS Extended Journaled and GUID Partition map. Leave Untitled as the default drive name
 - Go back to terminal

Even though Apple reissued the Sierra installer to fix the Certificate issue in 2019 it has another problem.
The Info.plist CFBundleShortVersionString has an incorrect entry of 12.6.06 it needs to be 12.6.03
 - To fix it we can run this command to edit the new entry in. (Credit MacRumors forum user: EricFromCanada)  
`plutil -replace CFBundleShortVersionString -string "12.6.03" /Volumes/Macintosh\ HD/Applications/Install\ macOS\ Sierra.app/Contents/Info.plist`
 - Now we can create the macOS Sierra USB Installer!  
`/Volumes/Macintosh\ HD/Applications/Install\ macOS\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --applicationpath /Volumes/Macintosh\ HD/Applications/Install\ macOS\ Sierra.app`

I hope these instructions helped you get your Mac back up and running.
Mr. Macintosh
