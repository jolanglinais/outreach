# Shell CLI Readability

![Bash Shell Logo Header Image][headerImg]

This configuration helps make my command line interface a bit more readable and increases the ease of use. It prevents from needing some consistent chores:

- Running `pwd` to see where you are
- Running `git status` to see which branch you are in
- Investigating _when_ you ran that one command a while ago

## Zsh

### Context

My setup is currently running macOS Ventura, using [iTerm2][iTerm2] with [Zsh][zsh].

My command line appears as:

![My Zsh Shell Command Prompt][zshImg]

This is showing the current date and which directory I am in, as well as the git branch (if this is a git repository):

```sh
<DATE>@<TIME>[<WORKING_DIRECTORY>][<GIT_BRANCH>]:
```

### Configuration

This customization can be done by editing an existing `~/.zshrc` file or creating one. My current `~/.zshrc`:

```sh
autoload -Uz vcs_info
precmd_vcs_info() { vcs_info }
precmd_functions+=( precmd_vcs_info )
setopt prompt_subst
zstyle ':vcs_info:git:*' formats '%b'

PROMPT='%F{#FE9A0A}%D%f%F{#09CAFF}@%f%T[%F{#0165F5}%~%f[%F{#78D236}${vcs_info_msg_0_}%f]: '
```

First section is for loading the version control system (git in this case) and displaying the branch.

### Prompt Options

Within the prompt:

- `%F{}...%f`
    - This formatting indicates the text in the `...` area will be of the color within the `{}`
- `%F{#FE9A0A}%D%f`
    - `%D` indicates a date format of `YY-MM-DD`
- `%F{#09CAFF}@%f`
    - This is only the character `@`
- `%T`
    - Indicates the 24 hour time in format `HH:MM`
- `[`
    - This is only the character `[`
- `%F{#0165F5}%~%f`
- `][`
    - This is only the characters `][`
- `%F{#78D236}${vcs_info_msg_0_}%f`
    - `${vcs_info_msg_0_}` will display the current git branch
- `]:`
    - This is only the characters `]:`

---

## Bash

### Context

My setup is currently running macOS Catalina, using [iTerm2][iTerm2] with [Bash][Bash].

My command line appears as:

![My Bash Shell Command Prompt][exampleImg]

This is showing the current date and which directory I am in, as well as the git branch (if this is a git repository):

```sh
<DATE>@<TIME>[<WORKING_DIRECTORY>][<GIT_BRANCH>]:
```

### Configuration

This customization can be done by editing an existing `~/.bashrc` or `~/.bash_profile` file, or creating one. This file needs to export `PS1`, which is the primary prompt. My current `~/.bash_profile`:


```sh
# get current branch in git repo
function parse_git_branch() {
	BRANCH=`git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/\1/'`
	if [ ! "${BRANCH}" == "" ]
	then
		STAT=`parse_git_dirty`
		echo "[${BRANCH}${STAT}]"
	else
		echo ""
	fi
}

# get current status of git repo
function parse_git_dirty {
	status=`git status 2>&1 | tee`
	dirty=`echo -n "${status}" 2> /dev/null | grep "modified:" &> /dev/null; echo "$?"`
	untracked=`echo -n "${status}" 2> /dev/null | grep "Untracked files" &> /dev/null; echo "$?"`
	ahead=`echo -n "${status}" 2> /dev/null | grep "Your branch is ahead of" &> /dev/null; echo "$?"`
	newfile=`echo -n "${status}" 2> /dev/null | grep "new file:" &> /dev/null; echo "$?"`
	renamed=`echo -n "${status}" 2> /dev/null | grep "renamed:" &> /dev/null; echo "$?"`
	deleted=`echo -n "${status}" 2> /dev/null | grep "deleted:" &> /dev/null; echo "$?"`
	bits=''
	if [ "${renamed}" == "0" ]; then
		bits=">${bits}"
	fi
	if [ "${ahead}" == "0" ]; then
		bits="*${bits}"
	fi
	if [ "${newfile}" == "0" ]; then
		bits="+${bits}"
	fi
	if [ "${untracked}" == "0" ]; then
		bits="?${bits}"
	fi
	if [ "${deleted}" == "0" ]; then
		bits="x${bits}"
	fi
	if [ "${dirty}" == "0" ]; then
		bits="!${bits}"
	fi
	if [ ! "${bits}" == "" ]; then
		echo " ${bits}"
	else
		echo ""
	fi
}

export PS1="\[\e[38;5;214m\]\D{%d.%m.%Y}\[\e[36m\]@\[\e[m\]\A\[\e[m\][\[\e[38;5;63m\]\W\[\e[m\]]\[\e[38;5;82m\]\`parse_git_branch\`\[\e[m\]: "
```

This `parse_git_dirty` function will cause the `...[git-branch-name]` section of my prompt to include the following optional characters:

- `*`: This branch is ahead of `main`
- `+`: A new file exists in this branch
- `?`: Untracked files exist in this branch
- `x`: Files have been deleted from this branch
- `!`: Files have been modified in this branch

### Prompt Options

- `\d`: Date in “Weekday Month Date” format (e.g., `Tue May 26`)
- `\D{format}`: Date in format passed in
- `\j`: Number of jobs currently managed by the shell
- `\l`: Basename of the shells terminal device
- `\n`: Newline
- `\s`: Name of the shell
- `\t`: Current time in 24-hour `HH:MM:SS` format
- `\T`: Current time in 12-hour `HH:MM:SS` format
- `\@`: Current time in 12-hour `am/pm` format
- `\A`: Current time in 24-hour `HH:MM` format
- `\u`: Username of the current user
- `\w`: Current working directory (`$HOME` is represented by `~`)
- `\W`: Basename of the working directory (`$HOME` is represented by `~`)
- `\!`: History number of this command
- `\#`: Command number of this command
- `\$`: Specifies whether the user is root (`#`) or otherwise (`$`)
- `\\`: Backslash
- `\[`: Start a sequence of non-displayed characters (useful if you want to add a command or instruction set to the prompt)
- `\]`: Close or end a sequence of non-displayed characters

Note that my `PS1` includes the following sections:

```sh
# Orange
\[\e[38;5;214m\]
# Date, in format
\D{%d.%m.%Y}
# Teal
\[\e[36m\]
# @ symbol
@
# Plain color
\[\e[m\]
# Time, 24hr format
\A
# Plain color
\[\e[m\]
# Open bracket symbol
[
# Blue
\[\e[38;5;63m\]
# Current directory, not path
\W
# Plain color
\[\e[m\]
# Close bracket symbol
]
# Green
\[\e[38;5;82m\]
# Function to generate git branch
\`parse_git_branch\`
# Plain color
\[\e[m\]
# Colon symbol
: 
```

[headerImg]: ../images/bashHeader.png
[exampleImg]: ../images/bashExample.png
[zshImg]: ../images/zshPic.png


[iTerm2]: https://iterm2.com/
[Bash]: https://en.wikipedia.org/wiki/Bash_(Unix_shell)
[zsh]: https://en.wikipedia.org/wiki/Z_shell