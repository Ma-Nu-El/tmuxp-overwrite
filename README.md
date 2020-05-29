
# Table of Contents

1.  [Abstract](#orgec16632)
2.  [Some kind of license?](#orgf259c9c)
3.  [Preamble](#org3477966)
4.  [Use](#orge086735)
5.  [Code](#org14d5334)
    1.  [prehelp](#org0b99c7e)
        1.  [if](#org46706bf)
        2.  [fi](#org29a4f7c)
    2.  [help](#org87b7342)
        1.  [if](#orgae8d312)
        2.  [show help](#org7245403)
        3.  [show options](#orga94b060)
        4.  [warning](#org4f195db)
        5.  [fi](#org967bc1d)
    3.  [identify file to be manipulated](#orgb1e58b9)
        1.  [if](#org0131e87)
        2.  [store file info](#org6a4c3ae)
        3.  [fi](#orgd343b2e)
    4.  [overwrite](#orgbe6e661)
        1.  [if](#orgcfc1e93)
        2.  [heads up](#org3846048)
        3.  [backup file and start `tmuxp freeze` process](#orgcbdee0a)
        4.  [done check](#org7c93031)
        5.  [fi](#org48a04f6)
    5.  [recover](#org7c48a11)
        1.  [if](#org9e5313f)
        2.  [mv file~ to file](#orge1413f4)
        3.  [fi](#org2205795)
    6.  [depython](#org10ace87)
        1.  [if](#org05539e2)
        2.  [heads up](#org9c6fec0)
        3.  [the actual thing](#org8b6585b)
        4.  [done check](#orge00f9d7)
        5.  [fi](#orga7d0926)
    7.  [exit](#org72ab081)
6.  [Installation](#org42e9881)



<a id="orgec16632"></a>

# Abstract

Tmuxp doesn't currently allow you to overwrite existing tmuxp files
generated by typing `tmuxp freeze`. This bash script emulates that
behavior.


<a id="orgf259c9c"></a>

# Some kind of license?

Do with this code what you deem appropriate, just remember to use at
your own risk. I'd recommend you to check it first since it's not that
complicated anyway, just a bunch of conveniently placed bash
commands. Happy coding.


<a id="org3477966"></a>

# Preamble

Be aware: this is just a momentary solution until tmuxp adopts session
overwriting natively. And my bash scripting capabilities are quite
humble, still I want to be able to programmatically overwrite a `tmuxp
freeze` file.

Here you can see that the issue has not been addressed in a while (at
least at the time of this writing):
<https://github.com/tmux-python/tmuxp/issues/64>

Also, `tmuxp freeze` at the moment of `freezing` a session adds the
next line to the generated file:

    shell_command: python3

This line makes `tmuxp load yourFrozenSession` to automatically start
a `python3` process every time you load your session. At least for me,
this behavior is unwanted. This code removes that line from the
generated file too.

I opened (and quickly closed) an issue about this which you can read
about here:

<https://github.com/tmux-python/tmuxp/issues/588>


<a id="orge086735"></a>

# Use

Warning: there are a couple of things you have to make sure before you
execute this command.

-   You have to be <span class="underline">attached</span> to the tmux session you want to `tmuxp
      freeze`.

-   Your tmux session must be the only active session in the tmux
    server. This is a thing with tmuxp itself which I'm not gonna
    investigate further. I just know that's how it can work without any
    problems.


<a id="org14d5334"></a>

# Code


<a id="org0b99c7e"></a>

## prehelp


<a id="org46706bf"></a>

### if

    if [[ -z "$1"  ]]
    then

    echo 'tpoverwrite is a bash macro to overwrite tmuxp sessions'
    echo 'type '\''tpoverwrite --help'\'' for more info'
    exit


<a id="org29a4f7c"></a>

### fi

    fi


<a id="org87b7342"></a>

## help


<a id="orgae8d312"></a>

### if

    if [[ $1 == '--help' ]]
    then


<a id="org7245403"></a>

### show help

    echo 'tpoverwrite - tmuxp overwrite'
    echo ''
    echo 'tpoverwrite is a bash macro that lets you quickly overwrite an'
    echo 'EXISTING tmuxp session (it won'\''t create a new one from scratch).'
    echo ''
    echo 'source code: '
    echo 'USAGE'
    echo ''\''tpoverwrite OPTION'\'''
    echo ''


<a id="orga94b060"></a>

### show options

    echo 'OPTIONS'
    echo '--overwrite (or -o)'
    echo 'overwrite your EXISTING tmuxp frozen file'
    echo ''
    echo '--depython (or -d)'
    echo 'remove that '\''python3'\'' line from the frozen file'


<a id="org4f195db"></a>

### warning

    echo 'for the moment it only works with .yaml files since I don'\''t know yet'
    echo 'how to implement TAB completion from within a bash script :D'
    echo 'Happy coding'
    exit


<a id="org967bc1d"></a>

### fi

    fi


<a id="orgb1e58b9"></a>

## identify file to be manipulated

If you give it not `--help` option then you want to actually use it.

By the way, `tmuxp` is more flexible in the search of the file, this
script for the moment is not that flexible; it looks only in
~/.tmuxp/. If I need it to be more flexible in the future I will
update it, but that's not guaranteed since for the moment it's just
what I need.


<a id="org0131e87"></a>

### if

    if [[ "$1" != "--help" ]] || [[ -z $1 ]]
    then


<a id="org6a4c3ae"></a>

### store file info

    FILE_LOCATION=~/.tmuxp
    SESSION_NAME=$(tmux display-message -p '#S')
    EXTENSION=yaml
    YOUR_FILE=$FILE_LOCATION/$SESSION_NAME.$EXTENSION


<a id="orgd343b2e"></a>

### fi

    fi


<a id="orgbe6e661"></a>

## overwrite

backup and remove existing frozen file

Get tmux session name from bash:
<https://superuser.com/questions/410017/how-do-i-know-current-tmux-session-name-by-running-tmux-command>


<a id="orgcfc1e93"></a>

### if

    if [[ "$1" == "-o" ]] || [[ "$1" == "--overwrite" ]]
    then


<a id="org3846048"></a>

### heads up

There's a catch I don't know how to solve:

If for any reason you abort the `freeze` process, you'll end up with
no `freeze` file, which will cause errors the next time you'll want to overwrite.

Gotta check that.

If you abort the freezing, you can recover.

    echo "Overwriting your tmuxp session:"


<a id="orgcbdee0a"></a>

### backup file and start `tmuxp freeze` process

    cat $YOUR_FILE > $YOUR_FILE~ # for backup
    rm $YOUR_FILE
    tmuxp freeze 


<a id="org7c93031"></a>

### done check

    echo 'tmuxp session backed up and overwriten in:'
    echo $YOUR_FILE


<a id="org48a04f6"></a>

### fi

    fi


<a id="org7c48a11"></a>

## recover


<a id="org9e5313f"></a>

### if

    if [[ "$1" == "-r" ]] || [[ "$1" == "--recover" ]]
    then


<a id="orge1413f4"></a>

### mv file~ to file

    if mv "$YOUR_FILE~" "$YOUR_FILE"
    then
    echo 'renamed:'
    echo \'$YOUR_FILE~\'
    echo 'to'
    echo \'$YOUR_FILE\'
    fi


<a id="org2205795"></a>

### fi

    fi


<a id="org10ace87"></a>

## depython

remove `python3` line from generated `tmuxp freeze` file.
Once the file is generated, it should contain a line that says

    ~shell_command: python3~

, the next code gets rid of that line.


<a id="org05539e2"></a>

### if

    if [[ "$1" == "-d" ]] || [[ "$1" == "--depython" ]]
    then


<a id="org9c6fec0"></a>

### heads up

    echo 'Warning: if you actually are running anything that contains the word'
    echo 'python3 and gets saved into the session, it could mess with your'
    echo 'tmuxp freeze file.'
    echo ''
    echo 'A backup of your file will be stored in'
    echo '/tmp/tmuxp/frozenPreCleaning/'
    echo 'in case anything goes wrong.'


<a id="org8b6585b"></a>

### the actual thing

    AUX_FILE=$FILE_LOCATION/auxFile
    cat $YOUR_FILE > ~/.tmuxp/depython/$SESSION_NAME.$EXTENSION
    sed '/python3/d' $YOUR_FILE > $AUX_FILE
    cat $AUX_FILE > $YOUR_FILE


<a id="orge00f9d7"></a>

### done check

    echo 'file depythoned.'


<a id="orga7d0926"></a>

### fi

    fi


<a id="org72ab081"></a>

## exit

    exit


<a id="org42e9881"></a>

# Installation

It's just a bash script, you can `git clone` this repo, move it into
your `~/bin/` and start using it (remember to load your `bin` folder in
your $PATH).

    git clone git@github.com:Ma-Nu-El/tmuxpoverwrite.git 

