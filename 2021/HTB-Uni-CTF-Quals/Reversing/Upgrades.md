# Upgrades

## tl;dr
Given a macro-enabled PowerPoint presentation:
- Extract the macro using *olevba*
- Optional: re-write the decryption function in another language, e.g. *Python*
- Run the given strings (ciphers) from the macro source code through the decryption function in order to get the flag

## Description
We received this strange advertisement via pneumatic tube, and it claims to be able to do amazing things! But we there's suspect something strange in it, can you uncover the truth?

## Attack Narrative

### Getting Started
We are given a *.pptm* file to download as part of the challengem, which is basically a 'Macro-Enabled' PowerPoint Presentation. Upon opening the file and navigating to the Macros tab, we can see that a macro with the name *'OnSlideShowPageChange'* is present:

![Macros](https://i.imgur.com/zx9iOos.png)

However, upon trying to edit the macro, we are asked for a password:

![VBAProject_Password](https://i.imgur.com/f4ilOeV.png)

Since I couldn't see the actual code I decided to put the presentation into Slide Show mode and traverse through the slides (see macro name) a few times to see if I could notice anything. The only thing I could notice was that the title of the 2nd slide was changing (in a random order) each time I went back to that slide. 

### Google time
It was time for me to consult my good old friend, Google! Upon searching for ways to extract macros from PowerPoint files I quickly came across a suite of tools called [oletools](https://github.com/decalage2/oletools) which contains *olevba*, a tool used *"to extract and analyze VBA Macro source code from MS Office documents (OLE and OpenXML)."*.

I installed the suite using *pip*:
```
pip install oletools
```

### Flag Hunting
We can now use olevba to extract the VBA code for the macro:
```
olevba -c Upgrades.pptm
```
and got the following VBA code:
```VB
Private Function q(g) As String
q = ""
For Each I In g
q = q & Chr((I * 59 - 54) And 255)
Next I
End Function
Sub OnSlideShowPageChange()
j = Array(q(Array(245, 46, 46, 162, 245, 162, 254, 250, 33, 185, 33)), _
q(Array(215, 120, 237, 94, 33, 162, 241, 107, 33, 20, 81, 198, 162, 219, 159, 172, 94, 33, 172, 94)), _
q(Array(245, 46, 46, 162, 89, 159, 120, 33, 162, 254, 63, 206, 63)), _
q(Array(89, 159, 120, 33, 162, 11, 198, 237, 46, 33, 107)), _
q(Array(232, 33, 94, 94, 33, 120, 162, 254, 237, 94, 198, 33)))
g = Int((UBound(j) + 1) * Rnd)
With ActivePresentation.Slides(2).Shapes(2).TextFrame
.TextRange.Text = j(g)
End With
If StrComp(Environ$(q(Array(81, 107, 33, 120, 172, 85, 185, 33))), q(Array(154, 254, 232, 3, 171, 171, 16, 29, 111, 228, 232, 245, 111, 89, 158, 219, 24, 210, 111, 171, 172, 219, 210, 46, 197, 76, 167, 233)), vbBinaryCompare) = 0 Then
VBA.CreateObject(q(Array(215, 11, 59, 120, 237, 146, 94, 236, 11, 250, 33, 198, 198))).Run (q(Array(59, 185, 46, 236, 33, 42, 33, 162, 223, 219, 162, 107, 250, 81, 94, 46, 159, 55, 172, 162, 223, 11)))
End If
End Sub
```

First thing I tried was commenting out the if statement and using the macro in a dummy presentation I made - guess what? It would trigger a shutdown in less than a minute after running it - thanks HTB! (Yes, I know you can use *shutdown -a* to abort, but still). It was kind of funny though.

Anyway, from the above script we can see that there's a decoding function *q* which takes in an array, let's translate this code to python:
```Python
def decode(array):
    decoded = ''
    for elem in array:
        decoded += chr((elem * 59 - 54) & 255)
    return decoded
```
The entries in the *j* array seem to be the strings that rotate on slide 2, so we ignore them for now (we'd come back in case of a dead end). Usually what's interesting when reversing are the strings in *if statement conditions* and the ones in the *if statement body*. So let's try decoding those:

First we'll try to decode 
```
(81, 107, 33, 120, 172, 85, 185, 33)
```

![first_string](https://i.imgur.com/Hy8gvHs.png)

then we try decoding
```
(154, 254, 232, 3, 171, 171, 16, 29, 111, 228, 232, 245, 111, 89, 158, 219, 24, 210, 111, 171, 172, 219, 210, 46, 197, 76, 167, 233)
```

![flag](https://i.imgur.com/fyw0LSS.png)

BINGO! There's our flag!

P.S.: decrypting the ciphers within the if statement body:

![F_for_HTB](https://i.imgur.com/z2P2zoI.png)

Now it makes sense why my PC kept insisting to shutdown...

## Flag
Here the flag in text:
```
HTB{33zy_VBA_M4CR0_3nC0d1NG}
```


