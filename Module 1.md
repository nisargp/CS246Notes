# Linux & BASH
==============

###  Special Directories
1. . (dot) : specifies current directory
2. .. (double dot) : specifies parent directory
3. ~ (tilde) : specifies home directory
4. ~userid : userid's home directory

Note: To see all files in current directory, use the ls command.

### Wild Card Matching
* ls can be supplied with a "globbing pattern"
* To list all files ending with .txt:
	
	```bash
	$> ls *.txt
	```
* The shell identifies all files that match the globbing pattern and replace "*.txt" with a list of those files
* Since the shell handles globbing, it can be used in any command and is processed before the command is executed

### Working with Files
* cat : concatenate
	- used to concatenate files but is mainly used to print the contents of a file
	- cat <FILE_PATH> => prints the contents of FILE_PATH
	
### Redirection
* Each program i sattached to 3 streams
	- Standard input (stdin)
	- Standard error (stderr)
	- Standard output (stdout)
* When a program needs input, it looks at stdin (whatever that is at the time)
	- Defaults to keyboard
* A program outputs to the screen by default
* You can change the input, and output (both normal and error) using > and <
* For example, the following commands read from a file and outputs to another file
	
	```bash
	$> cat > out.txt < in.txt 2> error.txt
	```
 * Note that you must specify the stderr with *2>*
 
 ### Pipes
 * Used to connect streams together
 * Example: How many words are there in the first 20 lines of sample.txt?
	
	```bash
	$> cat sample.txt | head -20 | wc -w
	```
* Example: Given that w1.txt and w2.txt both contain a list of words (1 per line), gve a duplicate free list of all the words in w1.txt

	```bash
	cat w1.txt w2.txt | sort | uniq
	```
	
* If we want to give the output of a program as a parameter to another program we can use either:
	1. \`...\`
	2. $(...) <-- allows for nesting
* Example: 

	```bash
	$> echo I am `whoami`, and it is `date`
	```
	
### Pattern Matching in Text Files
* egrep = "extended global regular expression pattern"
	- Equivalent to grep with the -E option
* Usage :
```bash
$> egrep <PATTERN> <FILE_NAME>
``` - Outputs lines that contain the pattern

#### Useful Regex
* .  : match anything
* ^  : starts with
* $  : ends with
* (exp1|exp2)  : match exp1 or exp2
* [...] : match anything in [] (can take ranges)
* [^...] : match anything apart from stuff in [] (can take ranges)
* +  : occues one or more times
* *  : occures zero or more times
* {x,y}  : occurs x to y times

### File Permissions
* See file listing with permissions using 
```bash
$> ls -l
```
* Permission strings start with one bit to indicate the type of file (file or directory)
* The rest are three groups of 3 bits (owner, group, other)
	- First bit is the read bit (either r or -)
	- Second bit is the write bit (either w or -)
	- Third bit is the execute bit (either x or -)

Permission | File | Directory
--- | --- | ---
r | See contents (cat) | read contents (ls)
w | modify contents (vim) | add/remove files (rm, mkdir, touch, etc.)
x | execute if a program | enter directory (cd)

#### Changing Permissions
* Owner is the only person with ability to change permissions
* Use the chmod command
* Usage:
```bash
$> chmod <MODE> <FILE>
```
* Example: add the write permission to foo.txt for all users

```bash
$> chmod a+w foo.txt
```
* Use +, -, = to add, remove, or set permissions respectfully
* a, g, o represent all, group, and owner respectfully
* Note: copying files copies their permissions

### Shell Variables
* Example: set a variable and print it's contents

```bash
$> x=1 #notice that there are no spaces
$> echo $x #the "$" is required or it will just ouput "x"
```
* It is good practice to not just use $, and to wrape variables in ${...}
* The above example sets x to be a string, not an integer (there are commands to do integer operations)
* Global variables
	- PATH is the most common one
	- PATH points to directories to locate dependencies, or programs
		- This is so you can use them without specifying a path every time

### Scripts
* File containing Linux commands that can be executed as a program
* Example: A script that prints date, current user, and current directory

```bash
#!/bin/bash #required for every bash program
date
whoami
uid
```
* To execute a bash program you either add it to PATH or give it's full directory
	- Make sure the executable permission is set properly

#### Command Line Arguments
* Denoted as 0, 1, 2,...
	- 0 is a special case; it is the name of the program
	- User inputted arguments start at 1
* ${#} outputs the number of command line arguments
* Example: Check if an arg is a good password. A good password is a word that is not in words.txt.

```bash
#!/bin/bash
egrep "^${1}$" /usr/share/dict/words.txt > /dev/null
if [ ${?} -eq 0 ]; then
	echo "Not a good password"
else
	echo "Maybe a good password"
fi
```
* This program hilights some important things
	- /dev/null essentially throws away output that we don't want to display
	- ? contains the status code of the last program to run (in this case it is egrep)
		- 0 => success
		- Not 0 => failure
	- The spaces used in the if statements are *not optional*

#### While Loops
* Example: Print the numbers from 1 to $1

```bash
#!/bin/bash
x=1
while [ ${x} -le ${1} ]; do
	echo "${x}"
	x=$((x+1))
done
```
* Some things to note:
	- You can evaluate arithmatic expressions using $((expr))
		- Just doing x=$x+1 would set x to be the string "1+1"
* We can improve this by checking that $1 is greater than 1 and returning an error status if not

```bash
#!/bin/bash
x=1
if [ ${1} -lt 1 ]; then
	echo "You are not using this properly :("
	exit 1 #non-zero indicates some form of error
fi
while [ ${x} -lt 1 ]; do
	echo "${x}"
	x=$((x+1))
done
```

#### For Loops
* Example: Rename all .cpp files to .cc

```bash
for name in *.cpp; do
	mv ${name} ${name%.cpp}.cc
done
```
* The above code creates a variable "name"
* *.cpp is a globbing pattern and is thus replaced by a list of files before the script is executed
* ${<VARIABLE>%<STRING>} strips <STRING> from <VARIABLE> and outputs result

* Example: Count how many times a word ($1) occurs in a file ($2)

```bash
x=0
for word in `cat ${2}`; do
	if [ ${word} = ${1} ]; then
		x=$((x+1))
	fi
done
echo x
```
* Note that we can't just count the number of lines produced by egrep because lines can have multiple occurences of words

### Testing
* Know how to create a test suite program. Here is an example.

```bash
#!/bin/bash

usage() {
        echo "Usage: $0 suite-file program"
        exit 1
}

if [ "${#}" -ne 2 ]; then
        echo "Expected 2 arguments, given ${#}" 1>&2
        usage
fi

while read test; do
    if [ ! -r "${test}".in ]; then #checks if "${test}".in exists and is readable
        echo "${test}.in does not exist or is not readable" 1>&2
        usage
    fi

	if [ ! -r "${test}".out ]; then #checks if "${test}".out exists and is readable
		echo "${test}.out does not exist or is not readable" 1>&2
		usage
	fi

	tmp_file=`mktemp /tmp/output.XXX #creates a temp file with XXX replaced by random numbers
	
	if [ -r "${test}".args ]; then #checks if "${test}".args exists and is readable
    cat "${test}".in | "${2}" `cat "${test}".args` > "${tmp_file}"
	else
		cat "${test}".in | "${2}" > "${tmp_file}"
	fi
	
	diff "${tmp_file}" "${test}".out > /dev/null

	if [ "${?}" -ne 0 ]; then #diff failed (the two files don't match exactly)
		echo "Test failed: ${test}"
		echo "Input:"
		cat "${test}".in
		echo ""
		echo "Expected:"
		cat "${test}".out
		echo ""
		echo "Actual:"
		cat "${tmp_file}"
		echo""
		echo""
	fi
	rm "${tmp_file}"
done < "${1}" #sets the stdin for the while loop to be our test suite
exit 0 #sueccess!
```
