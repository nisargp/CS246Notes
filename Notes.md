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
``` 

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
#!/bin/bash
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



# C++
=====

Hello world in C++:

```C++
#include <iostream>
using namespace std;

int main() {
	cout << "Hello, world!!" << endl;
	return 0; //optional (will happen automatically)
}
```
* Some things to note:
	- Wile you can use printf by including stdio, this is not allowed in the course
	- If we don't inlcude the namespace, then we would have to refer to cout as std::cout, and endl as std::endl

### Compiling
* We use the g++ compiler
* Common Usage: 
```bash
$> g++ <FILE_TO_COMPILE> -o <DESIRED_NAME>
```
* You can compile larger programs in parts. This will be covered later.

### I/O
* C++ provides 3 I/O objects:
	1. cout : pritns to stdout
	2. cin : reds from stdin
	3. cerr : prints to stderr
* C++ has 2 I/O operators
	1. << : output operator (put to)
	2. >> : input operator (get from)

* Example: Get two numbers from stdin, add them, print result to stdout

```C++
#include <iostream>
using namespace std;

int main() {
	int x, y;
	cin >> x >> y;
	cout << x + y << endl;
}
```
* This code works in perfect conditions but how can we tell if the read failed?
	- If the read fails for any reason, then cin.fail() returns true
	- If the read fails because of EOF, then cin.eof() returns true
	- Note: Use cin.clear() to reset these flags

* Example: Read all integers from stdin and print to stdout 1 per line. Terminate for bad reads (not int or EOF)

```C++
#include <iostream>
using namespace std;
int x;
while(cin >> x) {
	cout << x << endl;
}
```
* We are able to put cin in the conditional because cin >> x evaluates to false when cin.fail() is true
	- This is because of an implicit conversion to a void *, as well as the fact that >> returns cin with it's newest state (since a reference is returned)

* Example: Rerad all ints and echo them . Ignore non-ints. Terminate at EOF

```C++
#include <iostream>
using namespace std;

int main() {
	int x;
	while(true) {
		cin >> x;
		if(cin.fail()) {
			if(cin.eof()) {
				cin.clear(); //clears error flags
				cin.ignore(); //skips the current thing in stream
			} else {
				break; //exits out of the loop
			}
		} else {
			cout << x << endl;
		}
	}
}
```

### Streams
* In C++ we have a string data type
* Example: Read in a string and print it out
```C++
#include <iostream>
#include <string> //in order to use strings
using namespace std;

int main() {
	string s;
	cin >> s;
	cout << s << endl;
}
```
* There is a flaw with this program. If you input "Hello world", it will only output "Hello" because cin reads until the next whitespace.
* To read an entire line, use getline(cin, s), where cin is the cin object and s is a string.
	- Reads until a newline

#### Formatting Output
* Use something called I/O manipulators
	- #include <iomanip> to get access to a lot of manipulators
* Example: Print an integer in hexadecimal, then print it again in base 10

```C++
int i = 95;
cout << hex << i << endl; //switch to hex mode
cout << dec << i << endl; //switch back to base 10
```
* The stream abstraction applies to other sources of input such as files and even strings
* Using <fstream> we get access to:
	- ifstream : similar to cin
	- ofstream : similar to cout
* Example: Read from a file and write to stdout

```C++
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

int main() {
	string s;
	ifsream f("in.txt");
	string s;
	while(f >> s) {
		cout << s << endl;
	}
}
```
* Anything you can do with cin, you can do with another stream type

* As I mentioned previously, we can also attach a stream to a string using <sstring>
* This gives us access to:
	- istringstream : read
	- ostringstream : write
* Example: Create a string using strings and possibly integers
	- I am going to start omitting boilerplate code
```C++
ostringstream ss;
ss << "Enter a number between "
ss << low << "and" << high;
string s = ss.str(); //gets the actual string from the object
cout << s << endl;
```

* istringstream is very useful for converting a string to an integer:

```C++
int n;
string s;
while(cin >> s) { //breaks automatically on cin.eof()
	cout << "Enter a number" << endl;
	cin >> s;
	istringstream ss(s);
	if(ss >> n) break; //we would probably do something with n here in real life
	cout << "Try again..." << endl;
}
```
* Note: 
	- ss is stack allocated and pops off the stack at every iteration. This means that if there is a bad read, the entire object will be discareded so we don't need to worry about resetting any flags.
	- The only error case for cin is if there is an EOF as a string can be anything else (unlike when you read to an integer)
	
### Assorted Goodies/Review
#### Default Arguments
* Consider the following code:

```C++ 
void printSuiteFile(string name = "suite.txt") {
	ifstream f(name.c_string());
	string s;
	while(f >> s) {
		cout << s << endl;
	}
}
```
* Note that we have to instantiate f with name.c_string() and not just name. This is a design flaw of istringstream which requires to be initialized with a c style string which type string is not.
* The above function can be called in two ways:
	1. printSuiteFile()
		- This will set name to be "suite.txt" when the function runs
	2. printSuiteFile(FILE_NAME)
		- The default parameter is ignored, and FILE_NAME is used instead

* Some rules for default params:
	1. Optional params *must* appear last.
	2. Not allowed to follow a default argument with a non-default argument.
	3. You can only specifty default arguments as long as you have specified prior arguments. (Follows from rule 2)

#### Overloading Functions

```C++
int negate(int a { return -a; }
int negate(bool b) {return !b;}
```
* In C this won't compile becaues the C compiler only looks at the names and sees that there are multiple "negates"
* In C++, the compiler considers the name, number of arguments, and the type of arguments
	- It is not enough for the return types to differ

#### Declaration Before Use
* In C/C++ you must declare something before you use it.
	- Can cause issues with mutual recursion.
* You are able to declare something without a definition which acts as a promise to the compiler that there will be a definition eventually.
* You do this by just putting the function header (followed by a ;)

#### Pointers

```C++
int n = 5;
int * p = &n;
cout << p; //prints address of n
cout << *p; //prints the value of n
int **pp; //pointer to a pointer to an int
pp = &p; //pp points to address of the pointer p
**pp = 10; //sets the value of n to 10
```

#### Arrays

```C++
int a[] = {1,2,3,4,8}
```
* Above code requests continous piece of memory to fit given elements.
* Arrays are NOT pointers
* Source of confusion: Name of array is shorthand for address of the first element of the array.
	- a is equivalent to a[0]
	- *(a+1) is equivalent to a[1] <-- pointer arithmatic

#### Structs

```C++
struct Node {
	int data;
	struct Node *next; //you can ommit the "struct" here
}; //semi-colon required for reasons not discussed in class
```

* Be catious of the following:

```C++
struct Node {
	int data;
	Node next;
};
```
* The above won't compile because the compiler can't determine the size of next (infinite loop). If next is a pointer then the compiler knows it is the same size as an integer.

#### Constants
* A constant *must* be initialized at definition.
* Constants never change

```C++
const Node n1= {5,0};
n1.data = 5; //INVALID

int n = 5;
const int *np = &n; //pointer to an integer that is constant (can change pointer, can't change value of n)
int * const p = &n; //const pointer to an int (can't change where pointer points to, can change n)
const int * const p = &n; //read only (can't change anything)
```

#### Parameter Passing

```C++
void inc(int n) { n += 1; }
int x = 5;
inc(x);
cout << x; //prints 5, not 6
```
* x is passed by value, a copy is made, and then it is popped off the stack. The original x is never changed.
* We want to pass by reference (send address of x)

```C++
void inc(int *n) { *n += 1; }
```
* The above code will work as originally intended (but someone using this function has to explicitly pass in the address of x)

### References
* Take cin >> i as an example. How does this work? Why don't we need to say cin >> &i?
	- This is because C++ provides another pointer-like type called a reference
* Creating a reference:

```C++
int y = 10;
int &z = y; //z is a reference to y
```
* Do *NOT* read &z as the address of z (this is wrong!)

* A reference is like a constant pointer
	- z points to y and always points to y
* References also give us automatic dereferencing
	- z = 12; is legal and sets y to 12
* Consider the following:
```C++
int *p = &z;
```
* In the above example, p is set to the address of y, not z (z has no identity on it's own it's only purpose is to represent y)
* Some properties of references:
	1. Cannot leave a reference uninitialized
	2. Must be initialized to something that has an address (called l-values)
	3. Cannot create a reference to a pointer
	4. Cannot create a reference to a reference
	5. Cannot create an array of references
* Key use of references : Passing by reference into a function
	- Using a pointer: 
	
	```C++
	void inc(int *n) { *n += 1; }
	inc(&x);
	```
	
	- Using a reference
	
	```C++
	void inc(int &n) { n += 1; }
	inc(x);
	```
* Note: you are not able to do inc(x+y); you must pass in an l-value
	- There is an exception when the function takes a *constant* reference

* Now we can understand how cin >> i works
	- The operator is overloaded

	```C++
	istream &operator>>(istream &in, int &data) {...}
	```
	- Note: we return an istream object reference (actually, the *same* one as passed in) so that cascading works
* Constant references
	- *Always* pass by a constant reference where possible
	- This allows us to pass in non l-values as the reference is read only
		- This means we can pass in things like 5, x+x, etc.

### Dynamic Memory
* In C we did something like this:

```C
int *n = malloc(size * sizeof(int));
//do stuff here...
free(n);
```
* C++ does support malloc and free but there is a better way now
* new : allocate memory like malloc
* delete : deallocate memory like free
* Example:

```C++
struct Node {
	int data;
	Node *next;
};
Node *np = new Node; //don't need to do size calculations (they are done automatically)
//do stuff...
delete np;
```
* new is type aware as you don't have to do size calculations
	- Results in new being a lot less error prone

#### Dynamic Arrays

```C++ 
cin >> n;
int *arr = new int[n];
delete [] arr; //slightly differenty syntac for arrays
```
* If you use just delete on a dynamic array, the behaviour is not defined

#### Memory
* Memory associated wth a program contains 3 things:
	1. Program
	2. Stack
	3. Heap
* Variables can be allocated on the stack or the heap
	- Local variables are allocated on the stack
		- Reclaimed after going out of scope
	- Dynamic memory is allocated from the heap
		- Not reclaimed after going out of scope
* Observe this common mistake...

```C++
Node *getMeANode() {
	Node n;
	return &n; //REALLY BAD!!
}
```
* This code returns a pointer to a _stack allocated_ Node. The pointer returned will point at reclaimed memory since the Node is reclaimed after going out of scope.
* Do the following instead:

```C++ 
Node *getMeANode() {
	Node *n = new Node; //don't forget to make n a pointer
	return n;
}
```

### Operator Overloading
* Recall function overloading:

```C++
int negate(int n) { return -n; }
bool negate(bool n) { return !n; }
```
* We have already seen that >> and << can be overloaded
* Actually, any operator can be overloaded (+, -, etc.)
* Consider the following struct:

```C++ 
struct Vec {
	int x, y;
};
```
* We can now define vector addition:

```C++
Vec operator+(const &v1, const &v2) {
	Vec ret;
	ret.x = v1.x + v2.x;
	ret.y = v1.y + v2.y;
	return ret;
}
```
* Now lets define scalar multiplication:

```C++
Vec operator*(const &v, int k) {
	Vec ret;
	ret.x = v.x * k;
	ret.y = v.y * k;
	return ret;
}
```
* Note that we have only defined multiplication for when the scalar is on the RHS
* Let's leverage our current code to define it for the LHS too:

```C++
Vec operator*(int k, const &v) { return v * k; }
```

* Now consider the following struct:

```C++
struct Grade {
	int theGrade;
};
```
* It seems silly to have a struct with only one filed, but this allows us to overload operators specifically for values that are supposed to be grades.
* Here is an example of overloading the output:

```C++
ostream &operator<<(osream &out, const Grade &g) {
	out << g.theGrade << "%";
	return out; //for cascading to work
} 
```
* Now the input:

```C++
istream &operator>>(istream &in, Grade &g) {
	in >> g.theGrade;
	if(g.theGrade > 100) g.theGrade = 100;
	if(g.theGrade < 0) g.theGrade = 0;
	return in;
}
```

### Preprocessor
* #include<FILENAME> is an example of a preprocessor derective
	- Copies and pastes the contents of FILENAME
* #define is another derective
	- #define VAR VALUE
	- Creates a preprocessor variable with a given value (empty string by default)
	- Simply just search and replace
	- Predates constant variables
* Note: The output of the preprocessor won't necessarily be valid C++ code if you aren't careful
* An interesting example:

```C++
#define ever ;;
for(ever) {...} //infinite loop
```

#### Actual Uses
* Defined constants for conditional compilation
	- UNIX executes int main() {...}
	- Windows executes int WinMain() {...}
* This is a possible way to implement conditional compilation:

```C++
#define UNIX 1
#define WINDOWS 2
#define OS UNIX //or WINDOWS when you want to compile on Windows (just change this line)
#if OS == UNIX
int main() {
#elif OS == WINDOWS
int WinMain() {
#endif
	//main code here
}
```
* In this example we have to go into the file to change which environment we want to compile for. We can change this so that this can be done through the command line instead.
* Compile time arguments:
	- Sets preprocessor variable
	- In our example:
	
	```bash
	$> g++ -DOS=UNIX <FILES>
	```
* Conditional compilation is useful for debugging (choose to compile loggin statements)
	- Look at debug.cc in the SVN repo for an example
* Note: You can make large block comments using preprocessor blocks that never get executred:

```C++
#if 0
look at me! I am a big comment!!!
I can even put invalid code in here! int l =; grape = new Banana[4];
#endif
```
* The above code never reaches the compiler

### Seperate Compilation
* We want to modularize our code
* Code can be broken up into .cc and .h files
	- .h : interface (contains declarations)
	- .cc : implementations

* To compile project with multiple .cc files:
	1. 
	```bash
	$> g++ *.cc #if all needed files in directory
	```
	2. 
	```bash
	g++ <FILE_1>...<FILE_N>
	```
* Note: Never compile a .h file
* Another Note: Never #include a .cc file

* Seperate compilationis th eability to compile induvidual pieces of a program and then combine them
	- Sometimes, compiling large code bases can take hours, but compiling one part may take a minute
* It is common to get linker errors when compiling files sepeartly (ld:)
* By default g++ does compilation, linking, and produces an executable all at once
* To seperatly compile (just compile), use the -c option
	- Creates .o files (object file)
	- Contains compiled code as well as information about dependencies
* After creating .o files, g++ all of them to link them

#### Global Variables
* Lets say we have a file abc.h with the following code: 
```C++
int globalVar;
```
* Every file that includes abc.h will have it's own versin of globalVar so the linker will complain
* Instead, we need to do the definition in abc.cc
* This solves the linke error but now globalVar isn't global
* We can solve this by changing abc.h to: 
```C++
extern int globalVar;
```
* extern is strictly just a declaration and not a definition

#### Inlcude Guard
* We have to be careful that we don't double include things (will cause linker errors)
* For our Vector example:

```C++
#ifndef __VECTOR_H__
#define __VECTOR_H__
//put contents of .h here
#endif
```
* The variable name is arbitrary, but it has to be unique
* ALWAYS USE AN INCLUDE GUARD
* Another piece of advice: Never define a namespace in a .h file, instead explicitly use the :: operator

### C++ Classes
* Essentially you can put functions inside of a struct
* Consider the example Student:

```C++
struct Student {
	int assns, mid, final;
	float grade() {
		return assns * 0.4 +
			   mt * 0.2    +
			   final * 0.4;
	}
};
```
* The above example is a class
* Now we can create a student: 
```C++
Student s = {80,55,70};
```
* Note: There is a class keyword that will be talked about later (not much of a difference)

#### Functions vs. Methods
* Methods always have a hidden parameter called "this"
* "this" is a pointer to the object on which the method was called
* If we call s.grade() then this is a pointer to s
	 - this == &s
	 - *this == s
* The C-style initialization {...} is limited because the passed in values must be compile time constants

#### Constructors (ctors)
* Same name as the class
* A ctor for the Student class:

```C++
struct Student {
	Student(int assns, int mid, int final) {
		this->assns = assns;
		this->mid = mid;
		this->final = final;
	}
};
```
* Note that we need to use the "->" syntax as this is a pointer (not a value)
* We can now use the Student ctor:

```C++
Student billy(60,30,40); //valid
Student bob = Student(40,34,23); //also valid (equivalent to above code)
```
* Both of the above examples are stack allocated, we can also create a student on the heap:

```C++
Student billy = new Student(50,60,80);
//do stuff
delete billy;
```

* Advantages of a ctor:
	1. Arbitrary computation
	2. Default arguments
	3. Overloading
* Note: When calling the empty ctor, do *not* type "Student s()" because this looks like a function declaration to the compiler
	- Doing "Student s = new Student()" is OK
* Default ctors are only available before you define one
	- Also, you can't use C-style initialization after making a ctor (but who uses that shit anyway?)
* Default ctors call default ctors of fileds that are objects and leave base types uninitialized

#### Member Initialization Lists
* Suppose we have the following code:

```C++
struct MyStruct {
	const int myConst;
	int &myRef;
};
```
* The above code won't compile because constants (inlcuding references) need to be initialized at declaration
* The problem is, we can't initialize fields
* The steps of creating an object are:
	1. Space is allocated (stack/heap)
	2. Fields are initialized : default ctors run
	3. Constructor body runs
* We need to hijack step 2; this is where MILs (Member Initialization Lists) come in
* The following is how we would us an MIL in this case:

```C++
struct MyStruct {
	const int myConst;
	int &mtRef;
	MyStruct(int c, int &r) : myConst(c), myRef(r) {...}
};
```
* Note: MILs are *only* available for ctors

* You can use MILs for other cases. Consider this case with the Student class:

```C++ 
struct Student {
	int assns, mid, final;
	Student(int assns, int mid, int final)
		: assns(assns), mid(mid), final(final) {}
};
```
* Note: We don't have to use this because of the way MILs work. The scoping just works out nicely.
* Advantages of MIL:
	1. Only way to initialize constants
	2. Same name for fileds/parameters
	3. Can be more efficient

#### Copy Constructors
* Copy ctors run on the following cases:
	1. Initializing object to existing object
	2. Pass by value
	3. Return by value
