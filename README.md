Download Link: https://assignmentchef.com/product/solved-cs107-assignment2-string-me-along
<br>
This assignment is designed to give you practice with C-strings (both raw manipulation and using string library functions) an opportunity to view Unix utility programs from an internal perspective, as implementer not just client

exposure to interacting with the Unix lesystem and shell environment variables

<h1>Overview</h1>

Watch video walkthrough! (https://youtu.be/AAHFpoyk-sM)

This assignment consists of some code-study exercises and a small program to write. Two of the code excerpts come from the C standard library ( atoi and strtok ) and the third introduces you to the opendir and readdir functions.

The program you will write is a version of the Unix which command, a utility used to locate and identify executable programs to run. This is an especially apropos way to be introduced to C and Unix; not only does it continue a thread you began in assign0, but implementing the Unix operating system and its command-line tools were what motivated the creation of the C language in the rst place! Implementing such a utility program is a very natural use of C, and you’ll see how comfortably it ts in this role.

<h1>Get started</h1>

Check out the starter project from your cs107 repo using the command

The starter project contains code.c with the code for the code-reading exercises, C les tokenize.c , scan_token.c , and mywhich.c , and the supporting Makefile , custom_tests ,

and readme.txt les. In the samples subdirectory, you’ll nd our sample solutions.

<h1>1.    Code study: atoi</h1>

In assign0, you used the atoi function to convert command line arguments from strings to integers. The function name comes from <u>a</u>scii <u>to</u> <u>i</u>nteger , and as you found out in assign0, it is not particularly robust — it bails at the rst non-digit character with no indication the conversion failed. atoi has largely been superseded by the more full-featured strtol (used in assign1), but we chose the atoi implementation as the easier one to study.

Below is an implementation of atoi that uses pointer increment to advance through the input string. Although it is probably cleaner to use array indexing, if you read much C code, you will see plenty of pointer arithmetic, so you should get used to reading and understanding it.

In your readme.txt le for assign2, answer the following questions:

<ol>

 <li>How is a single digit character converted to its numeric value?</li>

 <li>If the string begins with a leading minus, at what point does it advance s past that char?</li>

</ol>

(Look closely! How the control ows is subtle and easily overlooked)

<ol>

 <li>The loop builds up the number as a negative value and later negates it. The comment indicates this choice avoids over ow on INT_MIN. Why does INT_MIN necessitate such a special case?</li>

 <li>Below are ve invalid calls to atoi . For each call, work out what is returned and then verify that your understanding is correct by running the program in c . In your readme.txt, indicate what is returned for each call and explain why.</li>

</ol>

<h1>2.    Code study: strtok</h1>

A common string-handling need is to split a string into “tokens” which are separated by one or more delimiting characters. The strtok function is the C standard library function used to split strings. However, unlike functions you may have used in CS106B (e.g., stringSplit ), a single call to strtok does not return a nicely formatted vector of the tokens. Rather, you must call strtok repeatedly, each time receiving a single token, and continue until strtok returns NULL to indicate there are no more tokens.

Start by reading the function’s man page ( man strtok ) to understand its basic operation. Be sure to read the BUGS section of the man page where it critiques the awkward design of this function.

The rst peculiar feature of strtok is that it destructively modi es the input string. Rather than construct a new substring for each token, it overwrites the token’s ending delimiter in the input string with a null byte, thus re-purposing the existing sequence of characters from the input string to become the token substring without copying those characters to new memory.

Another odd design decision is that strtok keeps track of the current state of the tokenization process on behalf of future calls to the function. The rst time you call strtok on a string to tokenize, you pass the input string as the rst argument, but for subsequent calls to strtok you pass NULL as the rst argument. strtok tracks the input being tokenized by internally maintaining a pointer to the beginning of the next token. This variable is declared with the static quali er, the purpose of static will be explored in a question below.

The following is musl (https://www.musl-libc.org)’s implementation of the strtok function:

In your readme.txt le for assign2, answer the following questions:

<ol>

 <li>This is likely the rst time you have seen the static quali er applied to a local variable. The Wikipedia article on static variables (https://en.wikipedia.org/wiki/Static_variable) provides an overview of the static quali er, and the Scope and Example</li>

</ol>

(https://en.wikipedia.org/wiki/Static_variable#Scope) section demonstrates an example of a static local variable and design rationale . Why does strtok declare the local variable p as static ?

<ol>

 <li>Changing the initialization to static char *p = s and re-compiling will produce a compiler error. What is the error message? Use your C reference or web search to research this error message and how it relates to static variables. In readme.txt, explain how static variables are initialized and how it di ers from non-static local variables.</li>

 <li>The rst time strtok is called, the input string is passed as the rst argument. On subsequent calls to continue tokenizing the same string, NULL is passed as the rst argument. Given this context, when will the test in line (5) evaluate to true?</li>

 <li>Read the man page for strspn and explain what happens on line (7) when the remaining part of the input string consists of only delimiter characters.</li>

 <li>Explain what line (12) accomplishes and what this does to the input string.</li>

</ol>

<h1>3.    Write scan_token</h1>

With the goal of making an improved function that performs the same type of tokenization as strtok without its awkward design, you are to write the function scan_token . You will write

and test this function in isolation now, and then will use the function later when writing the

mywhich program. The required prototype for scan_token is:

bool scan_token(const char **p_input, const char *delimiters,                 char buf[], size_t buflen);

The function scans the input string to determine the extent of the token using the delimiters as separators and then writes the token characters to buf , making sure to terminate with a null char. The function returns true if a token was written to buf , and false otherwise.

Speci c details of the function’s operation:

Your implementation of scan_token should take the same general approach as strtok , meaning it can (and should!) use the handy &lt;string.h&gt; functions such as strspn and strcspn , but it should not replicate the bad parts of its design, which is to say no static

variables, no weird use of the input argument to pass information across a sequence of calls, and should not destroy the input string.

The function separates the input into tokens in the same way that strtok does: it scans the input string to nd the rst character <strong>not</strong> contained in delimiters . This is the beginning of the token. It scans from there to nd the rst character contained in delimiters . This delimiter (or the end of the string if no delimiter was found) marks the

end of the token.

Note that the parameter p_input is a char ** . This is a pointer argument that is being passed by reference. The client passes a pointer to the pointer to the rst char of the input string. The function will update the pointer held by p_input to point to the next character following the token that was just scanned.

buf is a xed-length array to store the token and buflen is the length of the bu er. scan_token should not write past the end of buf . If a token does not t in buf , the

function should write buflen – 1 characters into buf , write a null byte in the last slot, and the pointer held by p_input should be updated to point to the next character following the buflen – 1 characters in the token. In other words, the next token scanned will start at the rst character that would have over owed buf .

Consider this sample use of scan_token :

Running the above code produces this output:

Write your implementation of scan_token in the scan_token.c le. You can test it using our provided tokenize.c program. The tokenize program is integrated with sanitycheck.

In addition to teaching you the inner workings of computer systems, CS107 also provides strength+cardio training for your coding skills. A key piece of this training is learning the value of thoughtful and thorough testing–better for you to nd the bugs than our autograder! The default tests supplied with sanitycheck are a start but these basic tests are not comprehensive and should be supplemented with your own tests. In order to encourage you to do the careful testing that we hope you would do anyway, for assign2 we require that you submit your sanitycheck custom_tests le with at least <strong>5 thoughtful and varied tests of your own for the tokenize</strong>

<strong>program</strong>. These tests should cover a variety of cases that validate that scan_token is working properly on ordinary cases as well on on inputs that are unusual or edge conditions.

<h1>4.    Code study: opendir/readdir</h1>

Many Unix utilities read and write from the lesystem. The &lt;dirent.h&gt; header le provides functions used to access to the contents of directories. In particular, you will be using the opendir and readdir functions in your mywhich program to get information about the les in

a given directory.

The &lt;dirent.h&gt; header de nes two important data types: DIR : a type representing a directory stream

struct dirent : a record of information for one le or directory (name, number, type,

and so on). Read the man page for readdir ( man readdir ) to see the struct de nition and its documentation.

The easiest way to see how to gather directory information is through an example:

If you are used to C++, the struct dirent *entry; line might look a bit funny. In C, unless struct s are typedef ‘d, you need to declare a variable using the struct tag. As with C++, structs access their members with dot or arrow notation, as in entry-&gt;d_name in the program above.

You must call the closedir function after you are done with a DIR pointer to release its resources. The opendir call both allocates dynamic memory and uses an entry in the le table. If you forget to close the DIR , those resources cannot be reclaimed. Run the code.c program under valgrind with and without the call to closedir(dp) and see

what Valgrind has to say about this.

In your readme.txt le, answer the following questions:

<ol>

 <li>What does struct dirent de ne to be the maximum lename length?</li>

 <li>How many bytes of memory does Valgrind report are “lost” if a program does an opendir</li>

</ol>

without a matching closedir ?

<h1>Review and comment starter code</h1>

The le mywhich.c is given to you with an incomplete main function that sketches the expected behavior for the case when mywhich is invoked with no arguments. You are to rst read and understand this code, work out how to change/extend it to suit your needs, and nally add comments to document your strategy.

Some questions you might consider for self-test: (do not submit answers)

What is the third argument to main? How do you determine the end of the envp array?

What is PATH_MAX ? What is it used for?

If the user’s environment does not contain a value for MYPATH , what does mywhich use instead?

Do you see anything unexpected or erroneous? We intend for our code to be bug-free; if you nd otherwise, please let us know!

As per usual, the code we provide has been stripped of its comments and it will be your job to provide the missing documentation.

<h1>5.    Implement the mywhich program</h1>

What does the which command do?

The which command searches for a command by name and reports where its matching executable le was found. Read its man page ( man which ) and try it out, e.g. which ls or which make or which vim . The response from which is the full path to the matching

executable le or no output if not found.

It may not be obvious at rst, but this search is intimately related to how commands are executed by the shell. When you run a command such as ls or vim , the shell searches for an executable program that matches that command name and then runs that program.

Where does it search for executables? You might imagine that it looks for an executable le named vim in every directory on the entire lesystem, but such an exhaustive search would be both incredibly ine       cient and dangerously insecure. Instead, it searches only those directories that have been explicitly listed in the user’s search path. The default search path includes directories such as /usr/local/bin/ and /usr/bin/ which house the executable les for the standard unix commands. (The name bin is a nod to the fact that executable les are encoded in <strong>bin</strong>ary).

The user can con gure their search path by changing the value of their PATH environment variable. As you saw in lab2, environment variables track information like username ( USER=zelenski ) and the user’s shell ( SHELL=/bin/bash ). The environment variable of particular interest for which is

PATH=/usr/local/bin:/usr/bin:/bin:/usr/bin/X11:/usr/sbin:/sbin:/usr/games . The value for PATH is a sequence of directories separated by colons; these are the directories searched when looking for an executable. When looking for a command, which searches the directories in the order they are listed in the search path and stops at the rst directory that contains a matching executable. In order to match, the le’s name must be an exact match and the le must be readable and executable by the user.

How does mywhich operate?

The mywhich program you are to write is similar in operation to the standard which with these di erences:

mywhich uses the environment variable MYPATH for the search path. If no such

environment variable exists, it falls back to PATH . (Standard which always uses PATH as the search path)

mywhich invoked with no arguments prints the list of directories searched. (standard which with no arguments does nothing) mywhich treats each command-line argument pre xed with a + as a wildcard match.

(standard which has no option for wildcard match) mywhich does not support any command-line ags. (standard which -a prints all exact

matches)

When invoked with <strong>no arguments</strong>, mywhich prints the directories in the search path, one directory per line. This use case is a testing aid to verify that you are accessing the correct environment variable and can properly tokenize it.

When invoked with <strong>one or more arguments</strong>, mywhich searches the directories in the search path for an exact match for each argument. The sample output below shows invoking mywhich to nd three executables. Two of them were found, but no executable named submit was found in any directory in the user’s MYPATH and thus nothing was printed for it.

Any <strong>argument pre xed with +</strong> is handled as a wildcard search, instead of an exact match. A wildcard search prints all executables that contain that pattern from all directories in the search path. Let’s say you vaguely remember there is a “fun” unix command, so use a wildcard search to nd it:

For testing purposes, you should test having run mywhich with di erent directories in the search path. Rather than muck with your actual PATH (which can create total chaos), we recommend that you change MYPATH , which only a ects mywhich and nothing else. Use env to set the value of MYPATH when running mywhich like this:

Requirements for mywhich

<strong> Usage</strong>. The mywhich program is invoked with zero or more arguments. Any argument pre xed with + is handled as a wildcard search. All other arguments are searched for using exact match. An argument is a non-empty string of one or more characters, i..e. “” or just a single + is invalid.

<strong>                      Assumptions.</strong> You may assume correct usage in all cases and that the user’s MYPATH and

PATH variables are well-formed sequences of one or more paths separated by colons. You do not need to detect or cope with situations where these assumptions do not hold and we will not test on any inputs that violate these assumptions, e.g. no usage of unsupported -a ag, no empty arguments, and no malformed values for MYPATH .

<strong>   Operation</strong>. The user’s MYPATH (or PATH if there is no MYPATH variable in the user’s environment) de nes the search path. The directories are searched in the order they are listed in the search path. For an exact match, the search stops at the rst directory containing a readable, executable le matching the command name. For a wildcard search, it searches all directories and prints all matching executables.

<strong>   Expected output.</strong> For each command name, it prints the full path to the rst matching executable or nothing if no matching executable was found. The matched executables are listed in the order that the command names were speci ed on the command-line. For a wildcard search, it prints the full path for every matching executable in any of the directories in search path. The directories are searched in order of the search path, but the matching les from the directory may be printed in any order.

<strong>   Restrictions</strong>. Your own code should manually search the environment using the envp argument to main . Your code is <strong>prohibited from using facilities such as</strong> getenv , env , and which to do this work on your behalf.

<h1>Advice/FAQ</h1>

Don’t miss out on the good stu in our companion document!

Go to advice/FAQ page (advice.html)


