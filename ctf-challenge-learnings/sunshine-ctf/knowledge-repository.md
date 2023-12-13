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

<figure><img src="../../.gitbook/assets/image (280).png" alt=""><figcaption></figcaption></figure>

Now upon checking different commits, I see different data files. I inspect a few data files and turns out each one of then had a single character (Alfa, Bravo, Delta etc.)

<figure><img src="../../.gitbook/assets/image (282).png" alt=""><figcaption></figcaption></figure>

All of these files are encoded in NATO phonetic encoding and it goes like A,B,C for Alfa, Bravo, Charlie etc.

<figure><img src="../../.gitbook/assets/image (281).png" alt=""><figcaption></figcaption></figure>

To extract the message, we might need to combine all of these together. Now, doing this for 3016 commits is cumbersome. So, a programmatic approach was essential to save some time.

I found a script online that would extract all the files in a repository here: [https://gist.github.com/magnetikonline/5faab765cf0775ea70cd2aa38bd70432](https://gist.github.com/magnetikonline/5faab765cf0775ea70cd2aa38bd70432)

<figure><img src="../../.gitbook/assets/image (283).png" alt=""><figcaption></figcaption></figure>

But since the script was only producing output related to the commits name, I had to modify the script so that it is numbered (in order to view from first to last commit)

Script used was:

```
#!/bin/bash -e

function getRepoSHA1List {
        pushd "$1" >/dev/null
        git rev-list --all
        popd >/dev/null
}

function exportCommits {
        local commitSHA1
        local count=1

        local IFS=$'\n'
        for commitSHA1 in $(getRepoSHA1List "$1"); do
                # build export directory for commit and create
                local exportDir="${2%%/}/$(printf "%04d" $count)"
                echo "Export $commitSHA1 -> $exportDir"
                mkdir --parents "$exportDir"

                # create archive from commit then unpack to export directory
                git \
                        --git-dir "$1/.git" \
                        archive \
                        --format tar \
                        "$commitSHA1" | \
                                tar \
                                        --directory "$exportDir" \
                                        --extract
                count=$((count + 1))
        done
}


# verify arguments
if [[ (! -d $1) || (! -d $2) ]]; then
        echo "Usage: $(basename "$0") GIT_DIR OUTPUT_DIR"
        exit 1
fi

if [[ ! -d "$1/.git" ]]; then
        echo "Error: it seems [$1] is not a Git repository?" >&2
        exit 1
fi

exportCommits "$1" "$2"

```

Upon running the script we see the data was being renamed as per the numbers

<figure><img src="../../.gitbook/assets/image (284).png" alt=""><figcaption></figcaption></figure>

Now, upon inspecting the data files in these folders, I see some of them had the same sizees. For example, files with size 4444 corresponded to the alphabet "6"

<figure><img src="../../.gitbook/assets/image (285).png" alt=""><figcaption></figcaption></figure>

Now, an algorithm is needed to rename these files based on the size to alphanumeric conversion. Since there are 26 english alphabets and 10 numerals, I need to classify 36 files of different sizes and put them in a dictionary for conversion. I wrote a simple Python script to rename size 4444 to 6 first and tested it out. Before that, I ran simple CLI commands to ensure how many files of size 4444 are there.

<figure><img src="../../.gitbook/assets/image (286).png" alt=""><figcaption></figcaption></figure>

Putting this through wc would give me how many files have 4444 size.

<figure><img src="../../.gitbook/assets/image (287).png" alt=""><figcaption></figcaption></figure>

```
import os

size_to_code = {
    4444: "6",
}

def rename_files_in_directory(directory):
    count = 0
    for foldername, subfolders, filenames in os.walk(directory):
        for filename in filenames:
            filepath = os.path.join(foldername, filename)
            filesize = os.path.getsize(filepath)

            if filesize in size_to_code:
                new_filename = size_to_code[filesize] + os.path.splitext(filename)[1]
                new_filepath = os.path.join(foldername, new_filename)
                os.rename(filepath, new_filepath)
                count += 1
                print(f'{count}. Renamed {filename} to {new_filename}')

output_directory = '/home/kali/ctf/sunshine/knowledge_repo/_the_ai_repository_of_knowledge/output'
rename_files_in_directory(output_directory)
```

<figure><img src="../../.gitbook/assets/image (288).png" alt=""><figcaption></figcaption></figure>

I saw that a toal 94 files were renamed. I confirmed this in CLI too.

<figure><img src="../../.gitbook/assets/image (289).png" alt=""><figcaption></figcaption></figure>

So the algorithm to solve and extract the message is as follows:

1. Classify 36 files (decode audio morse) manually and based on their sizes, add them to the dictionary "size\_to\_code" in Python script. To obtain the directory names of these 36 different files, write a simple bash script.
2. Rename the files from NATO phonetic to their respective plaintext conversion.
3. Open folders one by one using Python and read the file name. Keep appending the filename in a text file.
4. Reverse the obtained string.
5. Decode base64 to plaintext and save in a file.
6. Extract the message.

I wrote a small script to find out the folders with data files of similar sizes:

```
import os
from collections import defaultdict

def find_files_with_same_size(directory):
    size_to_directories = defaultdict(list)
    for foldername, subfolders, filenames in os.walk(directory):
        for filename in filenames:
            if filename != "data":
                continue
            filepath = os.path.join(foldername, filename)
            filesize = os.path.getsize(filepath)
            folder = os.path.basename(foldername)
            size_to_directories[filesize].append(folder)

    with open('sizes_and_files.txt', 'w') as f:
        for size, folders in size_to_directories.items():
            if len(folders) > 1:
                f.write(f"Size: {size}\nFiles: {', '.join(folders)}\n\n")

output_directory = '/home/kali/ctf/sunshine/knowledge_repo/_the_ai_repository_of_knowledge/output'
find_files_with_same_size(output_directory)
```

The script classifies a size then tells which folders have "data" file of the same sizes.

<figure><img src="../../.gitbook/assets/image (290).png" alt=""><figcaption></figcaption></figure>

This shortened my looking up time in audio files. Now I can manually go ahead and decode them, then add the decoded value in my solution script's dictionary.

<figure><img src="../../.gitbook/assets/image (291).png" alt=""><figcaption></figcaption></figure>

Also, since the audio files showed no case sensitivity, it is safe to assume the message could also be base32 where all letters are capital. The final dictionary becomes:

```
size_to_code = {
    4444: "6",
    11549: "E",
    35098: "F",
    14364: "2",
    18804: "A",
    22464: "N",
    10004: "3",
    32442: "Q",
    24124: "D",
    16614: "V",
    17094: "L",
    12439: "G",
    66378: "Y",
    76892: "I",
    50701: "Z",
    4986: "7",
    394044: "K",
    19998: "X",
    26684: "P",
    98164: "H",
    31644: "W",
    29616: "C",
    73508: "O",
    120294: "J",
    10404: "4",
    6036: "5",
    14576: "T",
    12830: "S",
    13006: "M",
    17643: "R",
    34298: "U",
    278894: "=",
}
```

Upon running the modified solution script, we can see that whole 3016 files were renamed to their respective mappings.

<figure><img src="../../.gitbook/assets/image (292).png" alt=""><figcaption></figcaption></figure>

Finally, a script was created to read the audio file names in respective folder, append those filenames in a file called output.txt

```
import os

def read_filenames_and_append_to_file(directory, output_file):
    characters = []
    for folder in sorted(os.listdir(directory)):
        folder_path = os.path.join(directory, folder)
        if not os.path.isdir(folder_path):
            continue
        files = os.listdir(folder_path)
        if files:
            characters.append(files[0])

    with open(output_file, 'w') as f:
        f.write(''.join(characters))

output_directory = '/home/kali/ctf/sunshine/knowledge_repo/_the_ai_repository_of_knowledge/output'
output_txt = '/home/kali/ctf/sunshine/knowledge_repo/_the_ai_repository_of_knowledge/output.txt'

read_filenames_and_append_to_file(output_directory, output_txt)
```

<figure><img src="../../.gitbook/assets/image (293).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (294).png" alt=""><figcaption></figcaption></figure>

I then finally reversed this string and saved the output in a file called final. This turned out to be a gz file. Then finally, I used gunzip to extract a text file out of this archive and obtained the flag!

<figure><img src="../../.gitbook/assets/image (296).png" alt=""><figcaption></figcaption></figure>

Alternate: One can also use python based decoders like morse2ascii to do this. I tried it using that but a lot of the small length audio files were not being decoded correctly. Also, the same algorithm I devised can be done using md5 checksums of the files.

