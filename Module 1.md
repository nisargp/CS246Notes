# Linux & BASH
==============

##  Special Directories
1. . (dot) : specifies current directory
2. .. (double dot) : specifies parent directory
3. ~ (tilde) : specifies home directory
4. ~userid : userid's home directory

Note: To see all files in current directory, use the ls command.

## Wild Card Matching
* ls can be supplied with a "globbing pattern"
* To list all files ending with .txt:
	```bash
	$> ls *.txt
	```
* The shell identifies all files that match the globbing pattern and replace "*.txt" with a list of those files
* Since the shell handles globbing, it can be used in any command and is processed before the command is executed

## Working with Files
* cat : concatenate
	- used to concatenate files but is mainly used to print the contents of a file
	- cat <FILE_PATH> => prints the contents of FILE_PATH
	
## Redirection
* Each program i sattached to 3 streams
	- Standard input (stdin)
	- Standard error (stderr)
	- Standard output (stdout)
	
	stdin  |---------| ----> stderr
	-----> | program |
		   |---------| ----> stdout