# Knowledge Repository

**Category: Miscellaneous**

**Points system: Dynamic reducing based on number of solves**

**Points collected: 396**&#x20;

<figure><img src="../../.gitbook/assets/image (265).png" alt=""><figcaption></figcaption></figure>

The challenge gives us a zip file. I unzipped it and saw an EML file in there. Since I had never encountered an EML in a CTF file before, I had to read up about it. [https://www.adobe.com/uk/acrobat/resources/document-files/text-files/eml.html](https://www.adobe.com/uk/acrobat/resources/document-files/text-files/eml.html)

It is just a plain text file where an E-mail conversation is stored. The contents of the E-mail were base64 encoded.

<figure><img src="../../.gitbook/assets/image (266).png" alt=""><figcaption></figcaption></figure>

Upon inspecting the file, I saw that there was an attachment too.

<figure><img src="../../.gitbook/assets/image (267).png" alt=""><figcaption></figcaption></figure>

After decoding the message contents, I saw this message:

<figure><img src="../../.gitbook/assets/image (268).png" alt=""><figcaption></figcaption></figure>

I used an online EML viewer to directly download the attachment. This was a Git bundle.

<figure><img src="../../.gitbook/assets/image (269).png" alt=""><figcaption></figcaption></figure>

I then created a new git repo and verified the bundle

<figure><img src="../../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

I cloned the repository then.

<figure><img src="../../.gitbook/assets/image (272).png" alt=""><figcaption></figcaption></figure>

Upon visiting the recently cloned repository, I saw an audio file.

<figure><img src="../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

Upon listening to this audio file, it felt like a simple frequency varying beeps file. The beeps were quickly changing and had only 2 distinguishable tones. This indicated it was probably a morse code.



{% file src="../../.gitbook/assets/data.wav" %}

Upon deciphering it on an online tool [https://morsecode.world/international/decoder/audio-decoder-adaptive.html](https://morsecode.world/international/decoder/audio-decoder-adaptive.html), I noticed some text come out. This initially didn't make sense.&#x20;

<figure><img src="../../.gitbook/assets/image (276).png" alt=""><figcaption></figcaption></figure>

Upon analyzing this in dcode.fr, I observeed that it is NATO encoded text.

<figure><img src="../../.gitbook/assets/image (277).png" alt=""><figcaption></figcaption></figure>

Further, upon deciphering it, I noticed that it was english for "="

<figure><img src="../../.gitbook/assets/image (278).png" alt=""><figcaption></figcaption></figure>

Since, this is not at all a flag, there has to be more to it. Further, it can be speculated that "=" is the start or end of a base64 encoded string.

I examined the github repo a bit more and found out that a total of 3000+ commits were there.

`git rev-list --count --all`

<figure><img src="../../.gitbook/assets/image (279).png" alt=""><figcaption></figcaption></figure>

Upon inspecting git log and a few of the commits (reverting repo to the commit), I observed that in each commit, data file was of different size. So, the data file must have different character/word which would be later combined together to give us the next clue.





