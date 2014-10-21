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
	1. `...`
	2. $(...) <-- allows for nesting
* Example: 

	```bash
	$> echo I am `whoami`, and it is `date`
	```
	