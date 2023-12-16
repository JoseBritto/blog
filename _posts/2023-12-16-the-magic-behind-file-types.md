---
layout: post
title: The magic behind file types
date: 2023-12-16 1:30 -0500
image: /assets/img/wizard.png
categories: ['Exploration', 'How stuff works']
tags: ['behind the scenes']
---

We all know that file extensions are supposed to identify file types. But how many of you know that in most cases they
are mostly redundant? &#129300; 

In today's side quest we explore the magic numbers that help our computers run everyday.

## Don't judge a file by its cover

Literally, just **don't**! Haha. I'll explain. 

First I'm gonna need you to grab your closest png file laying around.

Now open that file. What happens? Well, it opens normally... duh &#128580;

Ok, now I want you to rename the file extension to say **.bunny** instead of **.png**  
_(windows users might have to tick the show file extensions option in file explorer to do this)_

Now try to open. Chances are it won't for the most of you. 

Why is that?  
Well, here the OS (windows) doesn't know what program to use to open this file. 
It doesn't know if its an image, a video, an excel sheet etc... It uses the file extension as a hint to what
type of file it is dealing with an open it in a supported program.

Now I want you to right click and open with image viewer. It opens, right? And you can see the image. You might say
this is why we need file extension for and why we **should** judge a file by its name.

You are correct...partially.

Did you notice that I said that the file won't open for most of you (instead of all of you) earlier:
> Now try to open. Chances are it won’t for the most of you.

There is a reason for that. If you are using linux, you probably were able to open the file correctly.
You usually would be able to open the image even without any file extension. 
Whereas windows freaks out if you do this.
This is because there are other ways to detect a file type.

## The magic numbers &#x1FA84;
I want you to open your friendly neighbourhood terminal and `cd` to the directory where the image you just opened is.  
Now execute this command:
```bash
hexdump imagename.png | head
```

Your output would probably look something like this:
```
0000000 5089 474e 0a0d 0a1a 0000 0d00 4849 5244
0000010 0000 7202 0000 7202 0608 0000 4000 2d2e
0000020 0095 0000 7009 5948 0073 0b00 0013 0b00
0000030 0113 9a00 189c 0200 0f49 4449 5441 9c78
0000040 ddec 9877 4764 f075 6fff dd55 79d4 6cf2
0000050 6ad4 ed25 472a b394 4250 0501 2040 2440
0000060 5e13 81b2 c61f 26d8 98c8 d8d7 18c6 12f3
0000070 324c b008 c6c9 0464 2402 8ca1 ce72 b569
0000080 2779 874f aa9b f7ea 9dc7 cd19 4aee 95da
0000090 bbb4 40d2 9e7d 1fa7 6f69 dba7 ba77 9efb
```

Here we can ignore the first set of 7 digit numbers on every line. They basically function as line numbers.
The rest are the binary data contained in the file represented as hexadecimals
_(technically the first 10 lines of hexdump. see: [head](https://man7.org/linux/man-pages/man1/head.1.html))_

The way most file formats work is that the first few bytes serve as a unique identifier for that file type.
In our case we opened a png file. No matter how many png files you open, you will always have the first sequence of bytes
as this: `5089 474e 0a0d 0a1a` _(the numbers maybe reversed due to the endianness of your system)_.
This is because those bits serve as the unique identifier for a png file. There is another set for jpegs, a different one
for mp4s and so on.

> The above commands might not work on windows systems.
> Use this powershell command instead `Format-Hex -Path .\imagename.png -Count 48`.
> See [here](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/format-hex?view=powershell-7.4) for more explanation.
{: .prompt-tip }

So, you may ask, why would we need this information if we can know the file type from the extension.
Well, there are other use cases.

The obvious one being that users can easily modify file names whereas they typically can't modify the binary data.
And so, if we use the binary data to determine the file type, it adds an extra layer of validation and accuracy to the identification process.
This is the reason why image viewer programs typically disregard the file extension and use the magic numbers to determine
what kind of image its dealing with and how to go about rendering them, or even check if its an unsupported format.

The magic numbers can also be used as a quick way of detecting corrupted or unexpected files, if we know what type of files they are supposed to be.
_(This is obviously not the best way to detect partial data corruption. See [this](https://en.wikipedia.org/wiki/Error_detection_and_correction))._

Most antivirus software also typically use this to categorise files instead of just the file names.

## The security implications
Since the mechanism of how file types are determined are one of the most misunderstood and exploitable part of a computer system,
it has given rise to various exploits and deceptions to be used by nefarious actors against unsuspecting victims.
This is made worse what windows does with file extensions.

First off, windows hides the file extension by default. Although this might look prettier to see, it is a horrible thing
to do. Now the only thing an attacker have to do is to rename their `malware.exe` to `documents.zip.exe` and windows will only
show `documents.zip`. This is already bad and is enough to trick most people into running it. This is made even worse by the
fact that you can have custom icons on a file. If an attacker puts the default icon for a zip file as the custom icon for this exe,
then they can probably fool 99% of the user base.

To make this situation even worse, there are other ways the bad guys can trick the users, even if they can see the whole path.
The [unitrix](https://reasonlabs.com/research/revenge-of-the-unitrix) exploit was one of them. This expoit uses a special unicode
character to reverse the order characters are displayed on screen to make it appear like the malicious file is not an executable.
So potentially our disguised malware can look something like `documents_exe.zip` to the human eye and `documents_piz.exe` to the computer.

So, what is the best way to defend from these attacks? On windows systems, right click and select properties and check the displayed
file type to see if it is what you expect it to be. For linux and mac systems, you can run the `file` command to check the file type.
Even with that said, the best method I would say it to avoid opening or even downloading untrusted files.

## Conclusion
So, to sum it all up, file extensions, though commonly used for file identification, can be manipulated.
Magic numbers, unique byte sequences in file headers, offer a more reliable identification method.
Unlike extensions, users can't easily alter magic numbers, adding a layer of validation.
And beware of those pesky hackers! Windows hides extensions, creating the perfect disguise.
Stay on guard, peek into file types, and be cautious with unfamiliar files.

With all that said, it's time to bid adieu, adventurer. 

![Until we meet again gif](https://media1.tenor.com/m/aWZ6PaC5x5EAAAAC/skeletor-until-we-meet-again.gif)
