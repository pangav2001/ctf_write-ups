# Peel Back the Layers

## tl;dr
- Download the given Docker Image
- Save it into a tar file
- Extract the contents
- Use *strings* to get the flag

## Description
An unknown maintainer managed to push an update to one of our public docker images. Our SOC team reported suspicious traffic coming from some of our steam factories ever since. The update got retracted making us unable to investigate further. We are concerned that this might refer to a supply-chain attack. Could you investigate? Docker Image: steammaintainer/gearrepairimage

## Attack Narrative

### Getting Started
First of all, we'll need to have Docker installed which can be done by typing either:
```
apt install docker.io
```
OR
```
snap install docker
```
But, please, for the love of God, don't use *snap*...

By the description we are prompted to download a docker image to work on:
```
docker pull steammaintainer/gearrepairimage
```
We can then save that image into a tar file to see its contents:
```
docker save steammaintainer/gearrepairimage > gearrepair.tar
```

### Flag hunting begins
The first thing I always try when I have some sort of artifact(s) for a challenge is to use the *strings* command so I did the following:

![strings_command](https://i.imgur.com/DtkRXqf.png)

Ha! This tells us that the beginning (and probably the rest of the flag) can be found in plaintext format somewhere within our *.tar* file. So let's make a directory to extract it to and the extract it:
```
mkdir gearrepair
tar -xf gearrepair.tar -C ./gearrepair
```
Upon performing the extraction, we are greeted with the following contents:

![tar_contents](https://i.imgur.com/OFLPLl2.png)

Now let's use *grep* to see where the flag, or part of, is located at:

![grep](https://i.imgur.com/TsfA5lw.png) 

So, now we know it's in the following archive 
```
0aec9568b70f59cc149be9de4d303bc0caf0ed940cd5266671300b2d01e47922/layer.tar
```
We can either examine the contents of the archive or just try our luck again with the *strings* command since we know that, at least, the start of the flag is in plaitext format. I've decided to go with the latter for a start; so we type:
```
strings 0aec9568b70f59cc149be9de4d303bc0caf0ed940cd5266671300b2d01e47922/layer.tar
```
And... BINGO!

![flag](https://i.imgur.com/jjxwvyf.png)

We've found our flag!

## Flag
Let's type that in one line to get our flag:
```
HTB{1_r3H4lly_l1kH3_st34mpHunk_r0b0Hts!!!}
```
Upon submitting this, and the flag not getting accepted, I realized there was an extra *'H'* on each line break. So let's remove that to get the actual flag:
```
HTB{1_r34lly_l1k3_st34mpunk_r0b0ts!!!}
```