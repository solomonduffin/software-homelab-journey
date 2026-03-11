I'll update this file as I learn and do more, I just wanted to put some basics in and test scripting out.
I've never created scripts for the shell before, and it's pretty interesting.

So I had a folder of audiobooks with these titles:

'The Expanse 01 Leviathan Wakes'  "The Expanse 02 Caliban's War"   'The Expanse 04 Cibola Burn'        'The Expanse 05 Nemesis Games'
'The Expanse 00 The Churn'  'The Expanse 02.5 Gods of Risk'   "The Expanse 03 Abaddon's gate"  'The Expanse 05.5 The Vital Abyss'  'The Expanse 06 Babylons Ashes'

I wanted to change them to fit Audiobookshelf's naming conventions, so 'Book 0 - The Churn', 'Book 1 - Leviathan Wakes', etc...
This is pretty easy with this small amount of files, but this was more for fun.

So first of all, when you run a command like 'ls', shell is actually starting a process by running a file in /bin/ls. I've
learned some of the details here before that I'm forgetting now, but it's pretty interesting and I'd like to learn more about
how it works.

But anyway, so if we want to run a command, we have to import the `subprocess` module which is built-in in Python.

```
import subprocess

# Run the "ls" command
result = subprocess.run(["ls", "-l"])
```
Saving and running this file will run ls -l in the CWD. result here is an object with some information that can be printed.

You can also capture the output instead of printing it like this:
```
import subprocess

result = subprocess.run(["ls", "-1"], capture_output=True, text=True)

print("STDOUT:")
print(result.stdout)

print("STDERR:")
print(result.stderr)
```

Here's an example of renaming files using subprocess:
```
import os
import subprocess

# Get a list of files in the current directory
files = os.listdir(".")

for f in files:
    new_name = "renamed_" + f
    subprocess.run(["mv", f, new_name])
```


But it's also possible to use os, which is a Python built-in function, supposedly is faster and works cross-platform:
```
import os

for f in os.listdir("."):
    new_name = "renamed_" + f
    os.rename(f, new_name)
```

So here was my final successful script to turn 'The Expanse 00 The Churn' into 'Book 0 - The Churn':
```
import os
import re

files = os.listdir(".")

for f in files:
        if f.startswith("The Expanse"):
                num_title = f.split("se 0")[1]
                num_title = "Book " + num_title

                # Find all sequences of digits in the name
                digits_in_num_title = list(re.finditer(r"\d+", num_title))

                if digits_in_num_title:
                        last_match = digits_in_num_title[-1]

                        # Split the title into before/after the last digit
                        start = num_title[:last_match.end()]
                        end = num_title[last_match.end():]
                        num_dash_title = start + " -" + end

                os.rename(f, num_dash_title)
```




