# linux-bash-notes

Main takeaways from the amazing [Complete intro to linux and cli course](https://btholt.github.io/complete-intro-to-linux-and-the-cli/) by Brian Holt that I would like to have easy access in the future, should I need them.

### Concepts

Terminals (iTerm, Terminal app, etc) are emulators running shells (it's possible to change which shell a given terminal runs).

Shells are computer programs that expose OS services to humans and other programs.

Bash, zsh, Fish, PowerShell, cmd, Xonsh, etc are all shells.

Bash is actually a programming language that is executed interactively (one line at a time) - REPL (read, evaluate, print and loop). Via bash script (or shell script) it's possible to run entire applications with bash.

*Linux is very file system oriented (in a nutshell :grin: everything is a file on linux)

Basic anatomy os commands:

`{command} {arguments / (parameters)} {flags}`

`ls home --help`

_Some programs do not care about the arguments / flags order, but some do and it's down to how they were written_

_By convention, single dash is for long flags (--help) and double dash is for shot flags (-h). However it's not enforced and down to the programs implementation._


### Shortcuts

- CTRL + A = Move cursor to the beginning
- CTRL + E = Move cursor to the end
- CTRL + K = "yank" everything after the cursor
- CTRL + U = "yank" everything before the cursor
- CTRL + Y = past what has just been yanked

_Yank operations where especially useful before there was clipboard and copy and paste_

*Bang bang*

Double exclamation marks (or bang bang) re-executes the last command. Especially useful when rerunning something that needs sudo `sudo !!`.


### Signals

- CTRL + C = sigint (signal interruption) - `requests` the program to terminate "nicely" - allows for time for the program to finish nicely
- CTRL + D = sigquit (signal quit) - `commands` the program to terminate - does not allow for time to finish

When a program is interrupted with control signals, the terminal prints `^C` (for sigint). The caret (circumflex) symbol means control.

However not all programs implement signals as they should.

- sigkill (signal kill) - `kills` the program without any grace period

`kill -9 {pid}` (`-9` is a shotcut for `-SIGKILL`)

It is possible to send SIGINT and SIGQUIT to a program running in the background `kill -3 {pid} or kill -SIGINT {pid}`. For a full list run `kill -l`

- sigterm (signal terminate) - it is usually issued by the OS to programs when it's going to shutdown (users can emit, it'll work but shouldn't)


### Nano

To exit, just enter `CTRL (^ circumflex) + x` - `^x`. 

### VI / VIM

It has many modes. The most commons are insert mode (for edditing files) and command mode (executing commands like save file, exit, delete line etc).


### Interacting with files (most commonly used programs)

- less
- cat (standard output)
- head / tail
- mkdir
- rouch
- rm 
- cp
- move
- tar (grouping files, not necessary compressing but supported)


### Wildcard and replacements

#### Expansions

Bash has a conception of words expansion. It's particularly useful because whenever an expansion is used, bash does apply it first and then passes the generated words to the program, which means that the programs do not have to support it natively.

Example of words expasion:

`echo {1,2,3,abc,"test with space"}` # prints 1 2 3 abc test with space
`touch file-{1,2,3}.txt` # creates files file-1.txt file-2.txt and file-3.txt

Ranges and skips:

`echo {0..10..2}` # prints 0 2 4 6 8 10 
`echo {a..g..3}` # prints a d g



Wildcard expands words when they match files / directories:: 

_For the next examples consider a directory with 4 files `file-1.txt file-2.txt file-3.txt file_a.txt`_

`echo *` # prints file-1.txt file-2.txt file-3.txt file_a.txt
`echo file_*` # prints file_a.txt
`echo file-[1,2]*` # prints file-1.txt file-2.txt
`echo file?[1-9,a-z]*` # prints file-1.txt file-2.txt file-3.txt file_a.txt

`*` means any number of characters (including none)
`?` means exactly one characted
`[^1-5]` means not matching characters 1 to 5


Ps: in case nothing matching is found, it prints the provided string. Example `echo files[1-5].txt` prints `files[1-5].txt` as nothing matches.

_Ps2: be careful between the curly braces for expanding and the brackets for matching and their semantics._


### Streams and Pipes

Linux has a concept that all inputs and outputs are streams of data. There are basically 3 types:

- STIN (standard in)
- STOUT (standard out)
- STERR (standard error)

Outputs of programs go either to STOUT or STERR, depending on the execution result. When outputs are not defined and redirected, they end up going to the default stout - the terminal screen.

Redirecting outputs:

`cat {file} 1> stout.txt 2> sterror.txt`
`cat {file} > both_output.txt`

_Double angle brackets (>>) appends to the file. Single overrides the whole file_

When the output (or outputs) are not important, they can be redirected to `/dev/null` - that is like a blackhole.

`{program} 1> /dev/null`
`{program} 1> file.txt 2> /dev/null`
`{program} > /dev/null`


Standard in:

It is possible to provide data to programs via the STIN - *however not all programs read from STIN*.

`cat < {file}` is the same as `cat {file}`. The first one provides the file via stin stream, the second via argument. In this example they are effectively the same.


When providing `stin` and `outputs` the order doesn't matter:

`cat < {file} 1> success.txt 2> error.txt` or `cat 2> error.txt < {file} 1> success.txt`
`cat > result.txt < {file}` or `cat < {file} > result.txt`


#### Piping

It's possible pipe (or direct) the output of one program into the input of another. It's especially powerful as it's possible chain many programs one relying on the output of the other:

`cat < {file} | grep "abc" 1> result.txt `

*Careful with programs that do not read from STIN, as it breaks the piping*


##### Answering yes automatically

There is a program that all of its existence is to answer yes automatically. Imagine deleting thousands of files interactively and answering yes for all of them. It is possible to pipe the output of the yes program into the interactive rm:

```
touch file-{1..9}.txt
yes n | rm -i file-[1-9].txt
```

This example is silly as it could've been executed without iteractive mode, but it showcases how useful it can be (in this cas the answer is actually no, making it even more sillie but still).

### Exit codes

Every program exists with an exit code. 0 (zero) means the program completed successfully and any other number with an error.

Even though there are standardisation regarding the error codes, it's not enforced by bash and down to the developers to follow (thus it's a bit inconsistent). Suffice to know that 0 is success and anything else error.


In order to check the exit code of the last executed program run `echo $?`


### Sequences

There are 3 ways to run one command after another (not piping):

- && (double ampersand) runs the next program if the current one finishes successfully
- || (double pipes) runs the next program if the current one finished with an error
- ; (semi colon) runs the next program independently of the result of the current program

It is possible to mix and match in order to create more complex execution logic:

`{program 1} && {program 2} || {program 3} ; {program 4}` - runs program 1 and if it finishes successfully runs program 2, otherwise program 3. Then run program 4 independenty of anything else.

_It is important to be careful when executing sequences with both cases: success and error. If the error case is placed before the success case it could end up executing all programs: initial program errors out, thus the error case is executed. If the error case completes successfully, the success case also end up executins. Eg: {program 1} || {error program} && {success program}_


### Subcommands

Subcommand are used when a command needs to be called from within another command (and the result provided to the outer program). It is possible to nest subcommands indefinitely.

`echo hello $(whoami), how are you?` # attention to brackets
`echo hello $(echo $(whoami) $(date)), how are you?`

Backticks are the older version of the dollar brackets approach.


### Processes - JOBS

When a long program is launched on the foreground (attached), it's possible to send it to the background executing `CTRL + Z` - it however stops the execution.

To resume the execution, just run `bg {job id}`. 

To bring a program execution to the foreground / reattach, just run `fg {job id}`

To see all the running programs on the background and get their {job id} run `jobs -l`

It is possible to start a program in the backgrand straighaway by appending a ampersand (&) in the end of the command (sends to jobs straightaway). The downside of this approach is that if the outputs are not redirected, they end up being printed on the terminal even if they are running in the background.

Examples:

```
yes
CTRL + Z
jobs -l
bg {yes job id}
fg {yes job id}
CTRL + Z
bg {yes job id}
fg {yes job id}
```

```
yes 1> /dev/null &
jobs -l
fg {yes job id}
CTRL + Z
bg {yes job id}
fg {yes job id}
```

### SSH - Secure shell

To enable remote access with ssh key, just copy the public key from the local computer into the `authorized_keys` file in the remote machine under the .ssh directory of the desired user. Then just run the command `ssh {destination_user_name}@ip`.

Remote computer:
```
user name: b
~/.ssh/authorized_keys # add user c public key here
```

Local computer:
```
user name: c
ssh b@remote_ip
```

#### Managing users

To create a new user: `sudo useradd -s /bin/bash -m -g {whatever_group} {user_name}` (-s is the login shell, -m create home directory, -g group)

To change the passowrd: `sudo passwd {user_name}`.

To change files and directories permissions: `chmod u=rw,g=rw,o=r {file/directory}`, `chmod +x {file}`. Directories must have execute permission.


### SFTP

When SSH is enabled, SFTP is also enabled out of the box (though they can be independently disabled).

To connect via sftp just run `sftp {user}@ip` - similarly to ssh.

The main concept to bear in mind here is that you are working in 2 folders / directories at the same time: the remote and local one. The commands are similar to bash's and they all have a version prefixed with l (stands for local):

- pwd (remote)
- lpwd (local)
- ls
- lls
- !{command} (allows to run a command on the local computer) - example: `!echo "hello there" >> file.txt`
- {command} (runs a command on the remote compute)

To copy a file from the local computer to the remote one: `put {file}` or `put {file} {remote_name}` - this will copy the file from the current local directory to the current remote directory.

To download a file from the remote compute to the local one: `get {file}` - the same apply to remote and local directories.


### WGET x cURL

Main differences:

- wget does not pipe (do not write result to stout) while curl does
- wget can do recursive download (download an entire website) while curl doesn't (both can follow redirects)


### Package management

Each linux distro comes with a package manager: Ubuntu comes with APT (advanced packaging manager) and DPKG, Debian with DPKG, Redhat RPM (Redhat package manager) and YUM, Alpine APK, etc.

_don't forget, Ubuntu is a downstram from Debian_

*APT VS APT-GET*

APT is a convenience abstraction on top of APT-GET (APT-REMOVE, etc) to make it simpler for users.

APTITUDE is a nice frontend for APT to allow package management (like remove orphans, upgrades, etc) in an easier way.


*Upgrade vs Update*

Upgrade updates all upgradable packages.

Update updates the list of available packages and versions.


Commonly used commands:

- apt install {package}
- apt search {package}
- apt show {package}
- apt autoremove (remove packages not referenced by anyone else thus not used anumore)
- apt update
- apt upgrade
- apt upgrade {package}


#### Snaps

Snaps is an initiative to try and make downloading packages/programs safer - due to how they are packaged, downloaded, installed and executed (in a sandbox style). Even more, it is an initiative to try and get new program versions approved faster (e.g version node on apt repository is way older than the one on snaps repository).


Snaps are packaged differently from Debs.

Snaps have forced autoupdate feature (nice for local desktops but not for servers).

Snapd daemon is necessary to run snaps (but easily installable is not available in the distro) - everything that ends with d "initd", "containerd" means daemon running in the bg.

Example:

```
sudo snap install node --channel=14/stable --classic
```

_careful with --classic flag. It means out of the sandbox_

_channel flag is necessary to avoid major version updates and end up breaking servers due to compatibility_


The cool thing about snaps is that it is independent of distro and even if it is not shipped out of the box, it can be installed.


### Ways of executing scripts

- . {file}
- ./{file}
- source {file}
- bash {file}


It's possible to run scripts as though they were native. There are a couple of ways of accomplishing this:

1. create a folder, add the script there and add execution permission. Change you environmental path to also include that folder (`EXPORT PATH=~/whatever:$PATH`)
2. Move the script into `/bin` directory (all users would have access)

When executing a script with `./` it tells bash: do not go looking anywhere else in the path, this is the actual program I want to run.

_Running the script with bash (providing the file as stin or argument) spawns a subcommand execution, which effectively preserves the current users directory in case a cd command is executed. The other approaches actually run the script as the main process, which in turn changes the current directory of the user_


### Scripts

#### Hash bang / Shebang

When creating a script file, always start with `#! /bin/bash` - or the absolute / qualified path to the program (interpreter) that is used by that script. It gives bash a clue how to run the script.

It helps when sharing scripts with colleagues as it hints bash / shells on how to execute the script - favours compatibility.

Examples:

- mybash.sh:
```
#! /bin/bash

echo "hi from bash!"
```
chmod +x mybash.sh

`./mybash.sh`


- mypython.sh:
```
#! /usr/bin/python3

print("hello from python")
```

chmod +x mypython.sh

`./mypython.sh`


- mynode.sh:
```
#! /snap/bin/node

console.log("hello from node!")
```

chmod +x mynode.sh

`./mynode.sh`


#### Variables

Variables in scripts are pretty much the same as environmental variables.

Definition and assignment:

```
VAR_NAME=~/temp-folder # as thee is no spaces, quotes are optional
ANOTHER_VAR="~/temp-folder" 
```

Usage:

```
mkdir -p $VAR_NAME 
cd $VAR_NAME

# in some cases, it might be necessary to wrap the variables in curly braces as it wouldn't be possible to determine where the var finishes:

mkdir -p ${ANOTHER_VAR}-amazing
```

_Bash tends to follow screaming case instead of pascal and others_

#### Arguments

$1, $2, $3, ... -> the first, second, third, etc argument passed to the command ($0 is the script file name)

```
ABC=$1 # assigns the first argument to variable ABC
```

*Interactive user's inputs*

```
read -p "Hello, whats is your name?" NAME # -p is the prompt to the user and NAME is the variable where the input will be assigned
```

:point_up: it's possible to pipe the output of another program into the previous one :wink:


#### Conditionals (if statements)

```bash

if [ -z $MY_VAR ]; then
    echo "My var is empty, defaulting to 123"
    MY_VAR=123
fi
```

The if statements on bash script are actually rearranged into `test` program and relies on the exit code to determine if a condition is true or false (a bit confusing as 0 means true).

The same if condition above can be directly executed on the interative shell by running `test -z ${MY_VAR}` and then `echo $?` or appending `&& echo true || echo false`.

A full list for all the supported flag and comparisons can be obtain on the manual `man test`.

```bash
if [ $1 -gt 10 ]; then
    echo greater than 10
elif [ $1 -lt 10]; then
    echo less than 10
else
    echo it's ten
fi
```

Switch / case

```bash
case $1 in
    "smile") # this closing brackets signifies this is the case
        echo ":)"
        ;; # end case
    "sad")
        echo ":("
        ;;
    *) # default case
        echo "I don't know this one"
        ;;
esac

```

#### Loops & Arrays

```bash
#! /bin/bash


FAMILY=(don bob "gabe ced" mary)

echo hello ${FAMILY[0]}!

echo array size ${#FAMILY[*]}

for f in ${FAMILY[*]}
do
    echo Hi $f
    echo Hi again ${f}
done

COUNTER=0
while [ $COUNTER -lt ${#FAMILY[*]} ]
do
    echo Inside while ${FAMILY[$COUNTER]}
    COUNTER=$(($COUNTER + 1)) 
done
```

_To perform maths operations, it is necessary to wrap the operation in double brackets `$(())`. It passes the operation to `let` program. `echo $((3 % 2))`_



### Automation

To create create a job (or scheduled task) simply run `crontab -e`. To run with a different user, like root, just add the flag `-u {user}` - careful with running as root due to security, preferably it should be run with a user that only possess the necessary privilege to do job.

[crontab.guru](crontab.guru) is a good resource when more advanced and wild cron expressions are necessary.

Besides the traditional expression, there are also some special strings:

- @reboot ** deserves highlight
- @daily
- @weekly
- @monthly
- @yearly
- @annually


*Ubuntu small handy-hack (applicable to some distros, not all):*

Scripts added in the following directories will run according to their periodicity:

- /etc/cron.daily
- /etc/cron.hourly
- /etc/cron.monthly
- /etc/cron.weekly


### Customisation

Particularly speaking, I don't tend to customise OSs and IDEs as I like to easily transition between computers. But it's possible to customise your shell - even more, there are really good ready customisations by other people.


*Prompt*

The prompt (bit shown on every new line on the right side of the cursor) is defined by the environmental variable `PS1`.

`echo $PS1`

`export $PS1="Abc 123"`

To make the change permanent, just place the change in the `~/.bashrc` script.


*Known customisations:*

[Powerline](https://github.com/riobard/bash-powerline) for bash and (Spaceship)[https://github.com/spaceship-prompt/spaceship-prompt] for zsh - cool stuff related to git added to the prompt.


[Awesome bash](https://github.com/awesome-lists/awesome-bash) is a nice list with plenty of useful frameworks, toolkits, customisations, etc for bash.


*Colours:*

`Echo` program supports colouring the text. Example `echo -e "This is how you make text \e[32mgreen"`. However for something more sofisticated better tools like [chalk](https://github.com/chalk/chalk) for node or even a specific program for bash is desirable.
