
# Table of Contents

1.  [Abstract](#orgdc011a6)
2.  [Some kind of license?](#orga297ae9)
3.  [Preamble](#orgf4e9188)
4.  [Use](#orgec584ca)
5.  [Code](#orgd58d19b)
    1.  [prehelp](#org1170b9b)
        1.  [if](#org3b0e66f)
        2.  [fi](#org1cf2e05)
    2.  [help](#org6231f56)
        1.  [if](#org8ad90fa)
        2.  [show help](#org9255990)
        3.  [show options](#org58b6800)
        4.  [warning](#org184dad9)
        5.  [fi](#org8662b4b)
    3.  [identify file to be manipulated](#orgf4f7909)
        1.  [if](#orgf01530d)
        2.  [store file info](#orgda025d3)
        3.  [fi](#org513cc52)
    4.  [overwrite](#org982e42a)
        1.  [if](#orgf66d8de)
        2.  [heads up](#orga6a4c24)
        3.  [backup file and start `tmuxp freeze` process](#org65f6ad0)
        4.  [done check](#org4af87df)
        5.  [fi](#orgebc0a11)
    5.  [recover](#org09371bf)
        1.  [if](#org0013ecc)
        2.  [mv file~ to file](#org4c665d8)
        3.  [fi](#org23d94a9)
    6.  [depython](#orgf6f46bd)
        1.  [if](#org9945649)
        2.  [heads up](#org3756a23)
        3.  [the actual thing](#org9b698c8)
        4.  [done check](#org42066f3)
        5.  [fi](#orgdf5302e)
    7.  [debash](#org722d3f0)
        1.  [case](#org869795b)
        2.  [code](#org1ad667f)
    8.  [exit](#orgdf889b8)
6.  [Installation](#orgef4733c)



<a id="orgdc011a6"></a>

# Abstract

Tmuxp doesn&rsquo;t currently allow you to overwrite existing tmuxp files
generated by typing `tmuxp freeze`. This bash script emulates that
behavior.


<a id="orga297ae9"></a>

# Some kind of license?

Do with this code what you deem appropriate, just remember to use at
your own risk. I&rsquo;d recommend you to check it first since it&rsquo;s not that
complicated anyway, just a bunch of conveniently placed bash
commands. Happy coding.


<a id="orgf4e9188"></a>

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


<a id="orgec584ca"></a>

# Use

Warning: there are a couple of things you have to make sure before you
execute this command.

-   You have to be <span class="underline">attached</span> to the tmux session you want to `tmuxp
      freeze`.

-   Your tmux session must be the only active session in the tmux
    server. This is a thing with tmuxp itself which I&rsquo;m not gonna
    investigate further. I just know that&rsquo;s how it can work without any
    problems.


<a id="orgd58d19b"></a>

# Code


<a id="org1170b9b"></a>

## prehelp


<a id="org3b0e66f"></a>

### if

    if [[ -z "$1"  ]]
    then

    echo 'tpoverwrite is a bash macro to overwrite tmuxp sessions'
    echo 'type '\''tpoverwrite --help'\'' for more info'
    exit


<a id="org1cf2e05"></a>

### fi

    fi


<a id="org6231f56"></a>

## help


<a id="org8ad90fa"></a>

### if

    if [[ $1 == '--help' ]]
    then


<a id="org9255990"></a>

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


<a id="org58b6800"></a>

### show options

    echo 'OPTIONS'
    echo '-o'
    echo '--overwrite'
    echo 'overwrite your EXISTING tmuxp frozen file'
    echo ''
    echo '--depython'
    echo 'remove that '\''python3'\'' line from the frozen file'
    echo '--debash'
    echo 'remove that '\''bash'\'' line from the frozen file'
    echo 'remove that '\''shell_command: bash'\'' line from the frozen file'


<a id="org184dad9"></a>

### warning

    echo 'for the moment it only works with .yaml files since I don'\''t know yet'
    echo 'how to implement TAB completion from within a bash script :D'
    echo 'Happy coding'
    exit


<a id="org8662b4b"></a>

### fi

    fi


<a id="orgf4f7909"></a>

## identify file to be manipulated

If you give it not `--help` option then you want to actually use it.

By the way, `tmuxp` is more flexible in the search of the file, this
script for the moment is not that flexible; it looks only in
~/.tmuxp/. If I need it to be more flexible in the future I will
update it, but that&rsquo;s not guaranteed since for the moment it&rsquo;s just
what I need.


<a id="orgf01530d"></a>

### if

    if [[ "$1" != "--help" ]] || [[ -z $1 ]]
    then


<a id="orgda025d3"></a>

### store file info

    FILE_LOCATION=~/.tmuxp
    SESSION_NAME=$(tmux display-message -p '#S')
    EXTENSION=yaml
    YOUR_FILE=$FILE_LOCATION/$SESSION_NAME.$EXTENSION


<a id="org513cc52"></a>

### fi

    fi


<a id="org982e42a"></a>

## overwrite

backup and remove existing frozen file

Get tmux session name from bash:
<https://superuser.com/questions/410017/how-do-i-know-current-tmux-session-name-by-running-tmux-command>


<a id="orgf66d8de"></a>

### if

    if [[ "$1" == "-o" ]] || [[ "$1" == "--overwrite" ]]
    then


<a id="orga6a4c24"></a>

### heads up

There&rsquo;s a catch I don&rsquo;t know how to solve:

If for any reason you abort the `freeze` process, you&rsquo;ll end up with
no `freeze` file, which will cause errors the next time you&rsquo;ll want to overwrite.

Gotta check that.

If you abort the freezing, you can recover.

    echo "Overwriting your tmuxp session:"


<a id="org65f6ad0"></a>

### backup file and start `tmuxp freeze` process

    cat $YOUR_FILE > $YOUR_FILE~ # for backup
    rm $YOUR_FILE
    tmuxp freeze 


<a id="org4af87df"></a>

### done check

    echo 'tmuxp session backed up and overwriten in:'
    echo $YOUR_FILE


<a id="orgebc0a11"></a>

### fi

    fi


<a id="org09371bf"></a>

## recover


<a id="org0013ecc"></a>

### if

    if [[ "$1" == "-r" ]] || [[ "$1" == "--recover" ]]
    then


<a id="org4c665d8"></a>

### mv file~ to file

    if mv "$YOUR_FILE~" "$YOUR_FILE"
    then
    echo 'renamed:'
    echo \'$YOUR_FILE~\'
    echo 'to'
    echo \'$YOUR_FILE\'
    fi


<a id="org23d94a9"></a>

### fi

    fi


<a id="orgf6f46bd"></a>

## depython

remove `python3` line from generated `tmuxp freeze` file.
Once the file is generated, it should contain a line that says

    ~shell_command: python3~

, the next code gets rid of that line.


<a id="org9945649"></a>

### if

    if [[ "$1" == "--depython" ]]
    then


<a id="org3756a23"></a>

### heads up

    echo 'Warning: if you actually are running anything that contains the word'
    echo 'python3 and gets saved into the session, it could mess with your'
    echo 'tmuxp freeze file.'
    echo ''
    echo 'A backup of your file will be stored in'
    echo '/tmp/tmuxp/frozenPreCleaning/'
    echo 'in case anything goes wrong.'


<a id="org9b698c8"></a>

### the actual thing

    AUX_FILE=$FILE_LOCATION/auxFile
    cat $YOUR_FILE > ~/.tmuxp/depython/$SESSION_NAME.$EXTENSION
    sed '/python3/d' $YOUR_FILE > $AUX_FILE
    cat $AUX_FILE > $YOUR_FILE


<a id="org42066f3"></a>

### done check

    echo 'done: file depythoned.'


<a id="orgdf5302e"></a>

### fi

    fi


<a id="org722d3f0"></a>

## debash


<a id="org869795b"></a>

### case

When you `freeze` a session that has at least one window with the bare
command line in it, `tmuxp freeze` actually remembers that too and,
much like the python3 thing, also does it with `bash`. That is less of
a problem that the `python3` case, but still when you `tmuxp load`
you&rsquo;ll end up with unnecessary shell nesting. Try it out yourself:
type 

    echo $SHLVL

and see the output.

-   When you open a terminal it&rsquo;s 1
-   When you open tmux inside a terminal it&rsquo;s 2
-   When you `tmuxp load` it&rsquo;s 3. However when I created that session to
    be frozen it was 2. One shell nesting level extra.

The next script does the exact same thing that the `python3` case but
now for the `bash` command being written in the frozen file.


<a id="org1ad667f"></a>

### code

1.  if

        if [[ "$1" == "--debash" ]]
        then

2.  heads up

        echo 'Warning: if you actually are running anything that contains the word'
        echo ''\''bash'\'' and gets saved into the session, it could mess with your'
        echo 'tmuxp freeze file.'
        echo ''
        echo 'A backup of your file will be stored in'
        echo '/tmp/tmuxp/frozenPreCleaning/'
        echo 'in case anything goes wrong.'

3.  the actual thing

        AUX_FILE=$FILE_LOCATION/auxFile
        cat $YOUR_FILE > ~/.tmuxp/depython/$SESSION_NAME.$EXTENSION
        sed '/shell_command:\ bash/d' $YOUR_FILE > $AUX_FILE
        cat $AUX_FILE > $YOUR_FILE

4.  done check

        echo 'done: file debashed.'

5.  fi

        fi


<a id="orgdf889b8"></a>

## exit

    exit


<a id="orgef4733c"></a>

# Installation

It&rsquo;s just a bash script, you can `git clone` this repo, move it into
your `~/bin/` and start using it (remember to load your `bin` folder in
your $PATH).

    git clone git@github.com:Ma-Nu-El/tmuxpoverwrite.git 

