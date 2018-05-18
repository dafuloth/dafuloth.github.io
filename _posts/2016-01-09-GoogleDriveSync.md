---
published: true
title: "Google Drive misbehaves when sync folder is on a NAS"
author: "David Thai"
categories: [tech]
tags: [cloud,problem and solution,google,google drive,onedrive]
comments: true
---
#### Problem:
If the Google Drive sync folder is on a NAS, Google Drive sync will complain that the folder is missing when computer resumes from Sleep or Hibernation.

#### Causes:
* Probably because my NAS enters power saving state when there is no disk activity so it takes a moment for the computer to re-establish connections to network drives as it has to wait for NAS drives to spin up.

* Assuming this is the case, I would speculate that this issue would also occur if the Google Drive location was on any other network drive or device which wasn't continuously available.
 
#### Symptoms:
* Unfortunately, although Google Drive is happy to have its sync folder on a network drive it also expects that location to continuously be available, regardless of location or computer Sleep/Hibernation.
 
  That would be OK if Google Drive bothered to wait for the location to become  available or automatically tried again after short interval. Instead it just complains that "Your Google Drive folder is missing" and stays that way.

   <blockquote style="font-size:12px; font-style: oblique;">Whatever other faults OneDrive has, it does actually recover gracefully from temporarily not being able to locate its folder. However, it does also require a silly workaround involving a subfolder and symlink to be able to store its contents on a network drive.</blockquote>
   

#### Solutions:
1. Of course, you could use a local folder as the Google Drive sync folder but that's not a solution to the problem. That's just avoiding the problem.

2. Simplest/easiest/tedious solutions:
    +   *Simplest*: Restart your computer and defeat the point of Sleep/Hibernation.
    
    +   *Easiest*: Kill **`googledrivesync.exe`** with  
        **`taskkill /F /IM googledrivesync.exe`** at the command line or with the Run command and then start Google Drive again.
    
    +  *Tedious*:   Click the Google Drive tray icon, click on the error, click Locate Folder and remind Google Drive where the folder is.
    
3. A better way. It does need a little setup but in my opinion it beats all the above. Set it up once and never worry about it again:  
    1. Use the Task Scheduler to automatically kill **`googledrivesync.exe`** whenever the system is going to Sleep or Hibernate.
    
        We want this task to trigger **"On an event"** and the trigger should be as follows:
        
        - **Log: `System`**  
        - **Source: `Kernel-Power`**  
        - **Event ID: `42`**
<br /><br />

       That Event ID 42 indicates that *"The system is entering sleep."*
        
        As the action, choose **"Start a program"** and put **`taskkill`** as the program and **`/F /IM googledrivesync.exe`** as arguments.  

    2. We want to have more control over how and when the sync client runs because if it starts too soon after resuming from Sleep or Hibernation, the network location won't be ready and it will complain that its folder is missing.  Open up Preferences in Google Drive and untick **"Start Google Drive on system start-up".**  

    3. Use Task Scheduler to have **`googledrivesync.exe`** start after a delay, when you log in to Windows.
    
        This task should also trigger **"On an event"** and the trigger should be as follows:

        - **Log: `Security`**  
        - **Source: `Microsoft Windows security auditing.`**  
        - **Event ID: `4624`**
<br /><br />

       Where the Event ID 4624 indicates that *"An account was successfully logged on."* The important part here is to tick **"Delay task for:"** under **Advanced settings** and changing the value to something sensible. You want the delay to be long enough so that the network location becomes available but short enough so that Google Drive doesn't take forever to start up. Maybe "1 minute".

        As the action, choose **"Start a program"** and put the path to your **`googledrivesync.exe`**.
        
        Even after this task has done its job, its status remains "Running". I haven't noticed any ill effects as a result of this. Actually it might be a good thing. I've observed that Event ID 4624 can intermittently reoccur even while I'm logged in and actively using the PC. This results in this task running again and trying to launch the Google Drive client again, and because the client is already running it just opens your Google Drive folder which can be annoying. For good measure, select the option **`Do not start a new instance`** under **If the task is already running...**
        
#### Comments:
I realise that there are a lot of people this issue won't affect but I'm also fairly sure that are people who, like me, prefer to Hibernate their PC so that they can always pick up where they left off - as opposed to Sleep which still uses power or a full shutdown which means opening up browsers and directories from scratch.

If you're wondering where the trigger criteria comes from, I put the machine to Sleep or Hibernate and looked in the Event Viewer. There's really so much in the event logs but it's so helpful when looking for something specific.


