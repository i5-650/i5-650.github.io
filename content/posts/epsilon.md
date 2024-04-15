---
title: "How did I infiltrated a scammer group"
date: 2024-02-19
draft: false
tags:
    - Reverse Engineering
    - Hacking
    - Virus
    - Malware
categories:
  - Story
---
# Epsilon: the scammer's that got infiltrated
Before we start, I didn't do all of this alone. So I'd like to thank [Mr. Artemus](https://github.com/mr-artemus/) (we will call him Artemus for the rest of the story) for helping me with this.

I also want to ask you not to go bothering the scammers. They are bad people, but I don't want this blog to draw to much of their attention. I don't want to be the reason they start using better security measures. Plus, this story is now in the hands of the authorities.

## A hacked Discord account
On a saturday evening, Artemus calls me. One of our friends, [Bartho](https://github.com/barthofu) got hacked.

Fortunately, Artemus already asked Bartho about the details and quickly found out that the entry point of the hacker was an infected library. Let's dive into the hacker strategy:
- The hacker create a discord account.
- He joins a servers to find people with a badge on their profile.
- He talk to their target to either:
    - Get them to test their game.
    - Get them to help them on a GitHub project.
- Once the target either run the project or the game, the hacker has access to their computer.

Be cause in our case Bartho ran the project, we will start from that point. Let's start reversing!

## Discord.yt
Because Bartho downloaded the project on GitHub, Artemus got access to it and found the npm page of a library called `discord.yt`. The library was short but long. It was one line but 21699 characters long. And if you wonder what it looks like:

<img src="/epsilon/discord-yt.png" alt="obfuscated javascript" width="100%"/>

> Note: the code was *probably* obfuscated with [javascriptobfuscator.com](https://javascriptobfuscator.com/)
Let's dive into deobfuscating this code.

### What is this code doing?
1. debugger
2. integrity check
3. setInterval

console log pour récupérer les valeurs et les afficher dans la console
premier malware dl UnityLibraryManager.exe et le lance
end up having the json with the code:
line that dl the malware: const WRUz8y = _umZPm + Sn70i4 + euSKdxV + TkpDUF + X1iZMNB + UnzBOi.call(this, 0x25) + wkGJTDJ;
