## Bash in practice

### 1. Shells and modes
#### Interactive mode
Here you can enter a variety of Unix commands, such as `ls`, `grep`, `cat`, `pwd`, etc.

#### Non-interactive mode
In non-interactive mode, the shell reads commands from a file or a pipe and executes them. When the interpreter reaches the end of the file, the shell process terminates the session and returns to the parent process.

Use the following commands for running the shell in non-interactive mode:

```bash
sh /path/to/script.sh
bash /path/to/script.sh
```

**Invoking the script by making it an executable file using the `chmod` command:**

```bash
chmod +x /path/to/script.sh
```

Additionally, the first line in the script must indicate which program it should use to run the script.

```bash
#!/bin/bash
echo "Hello, World!"
```

If you prefer to use `sh` instead of `bash`, change `#!/bin/bash` to `#!/bin/sh`.

The use of `#!/bin/bash` would result in using the original bash, while `#!/usr/bin/env bash` would make use of the newer version.


### 2. Exit codes
Every command returns an exit code. A successful command always returns 0, and a command that fails returns a non-zero exit code. Failure codes must be positive integers between 1 and 255.

Another handy command we can use when writing a script is `exit`. This command terminates the script and returns an exit code to the shell.

```bash
exit 0
```

When a program terminates, the shell assigns its exit code to the variable `$?`. The `$?` variable is how we usually test whether a script has succeeded or not in its execution.

### 3. Comments
Comments are special statements ignored by the `shell` interpreter. They begin with `#` and go to the end of the line.

```bash
#!/bin/bash
# This script will print your username.
whoami
```

### 4. Variables
Variables can contain only numbers, a string of one or more characters.

#### Local variables
Local variables are variables that exist only within a single script. They are inaccessible to other programs and scripts.

A local variable can be declared using `=` (should not have spaces before and after the `=` sign)

```bash
username="hello"  # declare variable
echo $username          # display value
unset username          # delete variable
```

We can also declare a variable local to a single function using the `local` keyword.

```bash
local local_var="local value"
```

#### Environment variables
Environment variables are variables that are available to all processes running in a shell. They are defined using the `export` keyword.

```bash
export VAR="value"
```

There are a lot of global variables in bash.
| Variable | Description |
|----------|-------------|
| $HOME    | User's home directory |
| $PATH    | Command search path |
| $PWD     | Current working directory |
| $SHELL   | Path to current shell |


#### Positional parameters
Positional parameters are variables that contain the values of the arguments passed to the script. 

| Parameter      | Description             |
| :------------- | :-----------------------|
| `$0`           | script name             |
| `$1 … $9`      | first to ninth argument |
| `${10} … ${N}` | tenth and further arguments |
| `$*` or `$@`   | all arguments except `$0` |
| `$#`           | number of arguments except `$0` |
| `$FUNCNAME`    | function name (only in function) |

In the example below, the positional parameters will be `$0=./script.sh`, `$1=foo`, `$2=bar`.

```bash
./script.sh foo bar
```

### Shell expansions

#### Brace expansion
Brace expansion is a way to generate arbitrary strings.

```bash
echo beg{i,a,u}n # begin began begun
```

Also brace expansions may be used for creating ranges, which are iterated over in loops.

```bash
echo {0..5} # 0 1 2 3 4 5
echo {0..10..3} # 0 3 6 9
```

#### Command substitution
Command substitution is a way to execute a command and substitute its output into the current command. Command substitution is performed when a command is enclosed by  ``` `` ``` or `$()` .

```bash
now=`date +%T`
# or
now=$(date +%T)

echo $now 
```

#### Arithmetic expansion
Arithmetic expansion is a way to evaluate an arithmetic expression.

```bash
echo $((1 + 1)) # 2
```

#### Double and single quotes
Inside double quotes variables or command substitutions are expanded. Inside single quotes they are not.

```bash
echo "Today's date is $now" # Today's date is 12:00:00
echo 'Today's date is $now' # Today's date is $now
```

Another example:

```bash
FILE="Favorite Things.txt"
cat $FILE   # attempts to print 2 files: `Favorite` and `Things.txt`
cat "$FILE" # prints 1 file: `Favorite Things.txt`
```

### 5. Arrays
IFS, or Input Field Separator, is the character that separates elements in an array. The default value is an empty space IFS=```' '```.

```bash
array=(1 2 3 4 5)
echo ${array[0]} # 1
echo ${array[@]} # 1 2 3 4 5
```

#### Array declaration
In bash, we can declare an array by simply assigning a list of values to a variable.

```bash
fruits[0]=Apple
fruits[1]=Pear
fruits[2]=Plum
```

Array variables can also be created using compound assignments such as:

```bash
fruits=(Apple Pear Plum)
```

#### Array expansion
Array expansion is a way to expand an array into a list of values.

```bash
echo ${fruits[1]} # Pear
echo ${fruits[@]} # Apple Pear Plum
echo ${fruits[*]} # Apple Pear Plum
```

We want to print each element of the array on a separate line, so we try to use the printf builtin:

```bash
printf "+ %s\n" ${fruits[*]}
# + Apple
# + Desert
# + fig
# + Plum
```

#### Array slice
we can extract a slice of array using the slice operators:

```bash
echo ${fruits[@]:0:2} # Apple Desert fig
```

#### Adding elements into an array

```bash
fruits=(Orange "${fruits[@]}" Banana Cherry)
echo ${fruits[@]} # Orange Apple Desert fig Plum Banana Cherry
```

`${fruits[@]}` expands to the entire contents of the array and substitutes it into the compound assignment, then assigns the new value into the fruits array mutating its original value.

#### Removing elements from an array

```bash
unset fruits[0]
echo ${fruits[@]} # Apple Desert fig Plum Banana Cherry
```

### 6. Streams, pipes and lists
- Using streams we can send the output of a program into another program or file and thereby write logs or whatever we want.
- Pipes give us opportunity to create conveyors and control the execution of commands.

#### Streams
Bash receives input and sends output as sequences or streams of characters. These streams may be redirected into files or other processes.
There are three descriptors:
- `0: stdin`: standard input
- `1: stdout`: standard output
- `2: stderr`: standard error

#### Redirection
Redirection is the process of sending the output of a command into a file or another command.

For redirecting streams, we use the following syntax:

```bash
command > file # redirect stdout to file
command &> file # redirect stdout and stderr to file
command &>> file # append stdout and stderr to file
command < file # redirect stdin from file
```
Example:

```bash
# output of ls will be written to list.txt
ls -l > list.txt

# append output to list.txt
ls -a >> list.txt

# all errors will be written to errors.txt
grep da * 2> errors.txt

# read from errors.txt
less < errors.txt
```

#### Pipes
We could redirect standard streams not only in files, but also to other programs. Pipes let us use the output of a program as the input of another.

In the example below, command1 sends its output to command2, which then passes it on to the input of command3:

```bash
command1 | command2 | command3
```

Example:

In practice, this can be used to process data through several programs. For example, here the output of `ls -l` is sent to the `grep` program, which prints only files with a `.md` extension, and this output is finally sent to the `less` program:
```bash
ls -l | grep .md$ | less
```

#### Lists
A list of commands is a sequence of one or more pipelines separated by `;`, `&`, `&&` or `||` operator.

```bash
# command2 will be executed after command1
command1 ; command2

# which is the same as
command1
command2

# command2 will be executed only if command1 exits with 0 exit code
command1 && command2

# command2 will be executed only if command1 exits with non-zero exit code (当且仅当command1执行失败（返回错误码）时，command2才会执行)
command1 || command2
```

### 7. Conditionals
Bash conditionals let us decide to perform an action or not. The result is determined by evaluating an expression, which should be enclosed in `[[ ]]` or `[ ]`.
Conditional expression may contain && and || operators, which are AND and OR accordingly. 
There are two different conditional statements: `if` statement and `case` statement.

#### Primary and combining expressions

Working with the file system:

```bash
[-e file] # True if FILE exists.
[-f file] # True if FILE exists and is a regular file.
[-d file] # True if FILE exists and is a directory.
[-r file] # True if FILE exists and is readable.
[-w file] # True if FILE exists and is writable.
[-x file] # True if FILE exists and is executable.
[-s file] # True if FILE exists and has a size greater than zero.
[ file1 -nt file2 ] # True if file1 is newer than file2.
[ file1 -ot file2 ] # True if file1 is older than file2.
```

Working with strings:
```bash
[ -z STR ] # STR is empty (the length is zero).
[ -n STR ] # STR is not empty (the length is not zero).
[ STR1 = STR2 ] # STR1 is equal to STR2.
[ STR1 != STR2 ] # STR1 is not equal to STR2.
[ STR1 < STR2 ] # STR1 is less than STR2.
[ STR1 > STR2 ] # STR1 is greater than STR2.
```

Working with Arithmetic binary operators:
```bash
[ ARG1 -eq ARG2 ] # ARG1 is equal to ARG2.
[ ARG1 -ne ARG2 ] # ARG1 is not equal to ARG2.
[ ARG1 -gt ARG2 ] # ARG1 is greater than ARG2.
[ ARG1 -ge ARG2 ] # ARG1 is greater than or equal to ARG2.
[ ARG1 -lt ARG2 ] # ARG1 is less than ARG2.
[ ARG1 -le ARG2 ] # ARG1 is less than or equal to ARG2.
```

Working with combining expressions:

```bash
[ ! EXPR ] # True if EXPR is false.
[ (EXPR) ] # Returns the value of EXPR
[ EXPR1 -a EXPR2 ] # True if EXPR1 and EXPR2 are true. Logical AND.
[ EXPR1 -o EXPR2 ] # True if EXPR1 or EXPR2 is true. Logical OR.
```

#### If statement
If the expression within the `[[ ]]` is true, the commands between `then` and `fi` are executed. `fi` is the end of the if statement.

```bash
# Single-line
if [[ 1 -eq 1 ]]; then echo "true"; fi

# Multi-line
if [[ 1 -eq 1 ]]; then
  echo "true"
fi
```

```bash
# Single-line
if [[ 2 -ne 1 ]]; then echo "true"; else echo "false"; fi

# Multi-line
if [[ 2 -ne 1 ]]; then
  echo "true"
else
  echo "false"
fi
```

```bash
if [[ `uname` == "Adam" ]]; then
  echo "Do not eat an apple!"
elif [[ `uname` == "Eva" ]]; then
  echo "Do not take an apple!"
else
  echo "Apples are delicious!"
fi
```

#### Case statement
The `case` statement is used to match a value against a list of patterns.

```bash
case "$extension" in    # Start case statement, checking the value of $extension
  "jpg"|"jpeg")        # First pattern: matches either "jpg" OR "jpeg"
    echo "It's an image with a jpeg extension."
    ;;                 # Double semicolon marks the end of this case
  "png")              # Second pattern: matches "png"
    echo "It's an image with a png extension."
    ;;
  "gif")              # Third pattern: matches "gif"
    echo "Oh, it's a gif!"
    ;;
  *)                  # Default pattern: matches anything not matched above
    echo "Woops! It's not an image!"
    ;;
esac                  # End of case statement
```

The `|` sign is used for separating multiple patterns, and the `)` operator terminates a pattern list. The commands for the first match are executed. `*` is the pattern for anything else that doesn't match the defined patterns. Each block of commands should be divided with the `;;` operator.

### 8. Loops
There are four types of loops in Bash: `for`, `while`, `until` and `select`.

#### For loop
The `for` loop is used to iterate over a list of items.

```bash
for i in {1..5}; do
  echo $i
done

# in one line
for i in {1..5}; do echo $i; done
```

`for` is handy when we want to do the same operation over each file in a directory. For example, if we need to move all `.bash` files into the `script` folder and then give them execute permissions, we can do this with one `for` loop:

```bash
for file in $HOME/*.bash; do
  mv "$file" $HOME/script/
  chmod +x $HOME/script/"$file"
done
```

#### While loop
The `while` loop is used to execute a command repeatedly until a condition is met.

```bash
while [[ condition ]]
do
  # statements
done
```

For example:
```bash
#!/bin/bash

# square of numbers from 1 to 5
x=0
while [[ $x -le 5 ]]; do
  echo $((x*x))
  x=$((x+1))
done
```

#### Until loop
The `until` loop is the exact opposite of the while loop. Like a `while` it checks a test condition, but it keeps looping as long as this condition is **false**:

```bash
until [[ condition ]]
do
  # statements
done
```

#### Select loop
The `select` loop is used to create a menu from which users can select items.

```bash
select answer in elem1 elem2 ... elemN
do
  # statements
done
```

For example:
This script will create a menu from which users can select items and install them using the corresponding package manager.

```bash
#!/bin/bash

PS3="Choose the package manager: "
select ITEM in npm gem pip
do
    echo -n "Enter the package name: " && read PACKAGE
    case $ITEM in
        npm) npm install $PACKAGE ;;
        gem) gem install $PACKAGE ;;
        pip) pip install $PACKAGE ;;
    esac
    break # exit the loop
done
```

#### Loops control

- `break`: exit the loop
- `continue`: skip the rest of the current iteration
- `exit`: exit the script
- `return`: exit the function

For example:
This script will print all odd numbers from 0 to 9:

```bash
for (( i = 0; i < 10; i++ )); do
  if [[ $(( i % 2 )) -eq 0 ]]; then continue; fi
  echo $i
done
```

### 9. Functions
Functions are a way to group commands together. They allow us to reuse code and make our scripts more modular. You just write the name and the function will be invoked.

```bash
my_func () {
  # statements
}
my_func # call my_func
```

For example:

```bash
# function with params
greeting () {
  if [[ -n $1 ]]; then
    echo "Hello, $1!"
  else
    echo "Hello, unknown!"
  fi
  return 0
}

greeting Denys  # Hello, Denys!
greeting        # Hello, unknown!
```

### 10.Debugging

#### Debugging commands
If we want to run a script in debug mode, we use a special option in our script's shebang:```#!/bin/bash options```

- `#!/bin/bash -x`: print commands and their arguments as they are executed.
- `#!/bin/bash -v`: print shell input lines as they are read.
- `#!/bin/bash -n`: read commands but do not execute them.
- `#!/bin/bash -i`: interactive mode.
- `#!/bin/bash -t`: exit after first command.

`set -x` will print commands and their arguments as they are executed.
`set -v` will print shell input lines as they are read.
`set -n` will read commands but do not execute them.
`set -i` will enter interactive mode.
`set -t` will exit after first command.

### 11. Reading user input
The user can enter data into shell variables using `read` commands.
This command reads input from stdin into variables.

```bash
read -p "Enter your name: " name
echo "Hello, $name!"
```

- `-a`: read into an array variable, not a single variable.
- `-p`: prompt for the input.
- `-t`: timeout.
- `-n`: read only when n characters or delimiter is read
- `-s`: read without echo.
- `-u`: read input from file descriptor specified



















