# Zettelkasten Tools

This is a repository for [Keyboard Maestro](https://www.keyboardmaestro.com/) macros which aid in managing plain text notes following wiki and Zettelkasten conventions. They are intended as companions to note-taking and -management software such as [nvUltra](https://nvultra.com/) and [The Archive](https://zettelkasten.de/the-archive/). Other such macros can be found in [this Zettelkasten.de forum thread](https://forum.zettelkasten.de/discussion/213/the-archive-keyboard-maestro-alfred-macros).

The macros are described in the repository's [wiki](https://github.com/seanakabry/zk-tools/wiki).

This repository may be most useful as a reference for building your own macros and shell scripts, as most are likely to require some customisation. To this end, the shell scripts invoked by the macros are documented in the wiki and heavily commented. They are intended to be POSIX-compatible and as portable where practical, but the default context is assumed to be `zsh` in macOS. For instance, some scripts make use of `zsh`'s [glob qualifiers](http://zsh.sourceforge.net/Doc/Release/Expansion.html#Glob-Qualifiers), and [the macOS-specific `mdfind` is used in place of `find`](https://forum.zettelkasten.de/discussion/comment/7203/#Comment_7203).

## Assumptions

* 	Your notes lie flat in a single directory (no subdirectories).

*	As above, the shell scripts invoked by the macros are intended to be portable where practical, but the default context is assumed to be macOS.

*	You trust that the notes you're working with don't include malicious command injections. I've taken precautions where I know to, but I'm new to shell scripting and this hasn't been front of mind.

## How to Use the Macros

1.	Download and open the desired macro (`.kmmacros`).

2.	Edit any variables which should be customised, in the "Set Variables" action(s) highlighted yellow, such as:

	*	Notes Directory: The directory containing your plain text notes. A folder can be dragged into the action window from Finder.

	*	Backup Directory: The directory to which all notes to be edited by a macro will be backed up. The notes will be copied into a subdirectory with a descriptive title, like `2020-07-01, 12.00 - Rename "this" to "that"`. If you prefer, a version of each macro which doesn't take backups is included.

3.	Assign a suitable trigger to the macro.

4.	Enable the macro.

## Areas for Improvement

The following are general to most or all macros. Items specific to individual macros are noted on their respective wiki pages.

*	List searches (Keyboard Maestro's "Prompt With List from Variable" action) ought to ignore diacritics (so that a search for "tokyo" will match "Tōkyō").

*	Preserve original creation times, preferably without the use of `SetFile`, which requires that the user have the Command Line Tools for Xcode installed.
	
	The creation time of a file can be saved as a string in the syntax preferred by `touch` with:

	```zsh
	birthtime=$(date -j -f "%b  %d %T %Y" "$(stat -f "%SB" "$f")" +"%Y%m%d%H%M")
	```