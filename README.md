# Zettelkasten Tools

This is a repository for [Keyboard Maestro](https://www.keyboardmaestro.com/) macros which aid in managing plain text notes following wiki and Zettelkasten conventions.

The macros are intended as companions to note-taking and -management software such as [nvUltra](https://nvultra.com/) and [The Archive](https://zettelkasten.de/the-archive/). Other such macros are listed in [this Zettelkasten.de forum thread](https://forum.zettelkasten.de/discussion/213/the-archive-keyboard-maestro-alfred-macros).

The shell scripts invoked by these macros (which appear below) may be of some use even to those who don't use Keyboard Maestro. They are intended to be POSIX-compatible and as portable as possible. Variables beginning with `$KMVAR_` are Keyboard Maestro variables which are expanded before the script is passed to the shell.

## Page Contents

* [Assumptions](#assumptions)
* [How to Use the Macros](#how-to-use-the-macros)
* [Relevent Shell Scripting Resources](#relevant-shell-scripting-resources)
* [Macros](#macros)

| Macro | Function | v. | Updated |
| :---- | -------- | :- | :------------ |
| [Back Up Notes](#back-up-notes) | Copies all notes to a timestamped backup directory. | 1.00 | 2020-07-02 |
| [Find and Replace](#find-and-replace) | Performs a find and replace operation on the content but not the titles of all notes. | 1.00 | 2020-07-02 |
| [Rename and Update Wikilinks](#rename-and-update-wikilinks) | Renames a specified note and updates `[[wikilinks]]` to it. | 1.00 | 2020-07-02 |

## Assumptions

The tools assume:

* 	Your notes lie flat in a single directory (no subdirectories).

*	Your notes are `[[wikilinked]]` using full filenames rather than explicit UIDs.

	By "full filenames" I mean full basenames, where `[[Another Note]]` links to `Another Note.txt`, without regard to the file extension.

	An explicit UID (unique identifier) is a part of the filename of a note which follows a consistent syntax (usually a timestamp prefix like `202007011200`) and is supposed never to change, like a primary key for a record in a database. The wikilink `[[202007011200]]` would point to `202007011200 Another Note.txt`, and continue to point to the same note even if that note is renamed `202007011200 Edited Note.txt`. This robustness is one of the chief benefits of using explicit UIDs. Obvious drawbacks include the interruption of the flow of text with a long, usually non-semantic string of characters, and the difficulty of specifying the UID of a note one has in mind (made trivially easy by a wikilink autocompletion macro, such as by Zettelkasten.de forum users [kaidoh](https://forum.zettelkasten.de/discussion/176/quick-insertion-of-links-to-other-zettels-with-type-ahead-search-using-keyboard-maestro) or [Will](https://forum.zettelkasten.de/discussion/comment/2516/#Comment_2516)).

	The Archive [recognises UIDs](https://zettelkasten.de/the-archive/help/#how-do-i-create-links-between-notes). nvUltra [does not](https://twitter.com/nvUltraApp/status/1268882547066515458).

*	You prefer that automated operations affecting multiple notes leave the modification times of those notes unchanged.
	
	An example of a case in which this behaviour is desireable is if you use a 'find and replace' macro to correct a spelling mistake made consistently across many notes. If you sort your notes by time of last modification, you probably want to see a list of the notes you've been working on recently, rather than the notes which contained that spelling mistake.

*	You trust that the notes you're working with don't include malicious command injections. I've taken precautions where I know to, but I'm new to shell scripting and this hasn't been front of mind.

## How to Use the Macros

1.	Download and open the desired macro (`.kmmacros`).

2.	Edit any variables which should be customised, in the "Set Variables" action(s) highlighted yellow:

	*	Notes Directory: The directory containing your plain text notes. A folder can be dragged into the action window from Finder.

	*	Backup Directory: The directory to which all notes to be edited by a macro will be backed up. The notes will be copied into a subdirectory with a descriptive title, like `2020-07-01, 12.00 - Rename "this" to "that"`. If you prefer, a version of each macro which doesn't take backups is included.

3.	Assign a suitable trigger to the macro.

4.	Enable the macro.

## Relevant Shell Scripting Resources

If you are also new to shell scripting and want to modify the scripts above, these pages may be useful:

* Keyboard Maestro
	* [Wiki: Execute a Shell Script](https://wiki.keyboardmaestro.com/action/Execute_a_Shell_Script)
	* [Forum: Bash Shell Script Needs a Variable Passed with Characters “Escaped”](https://forum.keyboardmaestro.com/t/bash-shell-script-needs-a-variable-passed-with-characters-escaped/14130)
* [Greg's Wiki](https://mywiki.wooledge.org/EnglishFrontPage)
	* [Bash Guide: Practices](http://mywiki.wooledge.org/BashGuide/Practices)
	* [Using `find`](http://mywiki.wooledge.org/UsingFind)
	* [How can I rename all my \*.foo files to \*.bar, or convert spaces to underscores, or convert upper-case file names to lower case?](https://mywiki.wooledge.org/BashFAQ/030)
* File path string manipulation (via [this SO answer](https://stackoverflow.com/a/2664746))
	* [`bash` String Manipulations](http://tldp.org/LDP/LG/issue18/bash.html)
	* [Shell Command Language: 2.6.2 Parameter Expansion](https://pubs.opengroup.org/)

# Macros

## Back Up Notes

Copies all notes to a timestamped backup directory.

### Links

* [Direct link to `.kmmacros`](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Back%20Up.kmmacros) (right click and 'save link/target')

### Invoked Shell Script

```sh
cp -a "$KMVAR_Notes_Directory/" "$KMVAR_Backup_Directory/$(date "+%Y-%m-%d, %H.%M") - Manual Backup/"
```

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.00 | 2020-07-02 | Initial commit |

## Find and Replace

Performs a find and replace operation on the content but not the titles of all notes. Leaves modification times unchanged.

### Links

*	Direct links to `.kmmacros`: (right click and 'save link/target')
	* [with backups](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Find%20and%20Replace%2C%20with%20Backups.kmmacros)
	* [without backups](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Find%20and%20Replace%2C%20without%20Backups.kmmacros)

### Invoked Shell Script

```sh
cd "$KMVAR_Notes_Directory"

# Create a backup directory.
backup_directory="$KMVAR_Backup_Directory/$(date "+%Y-%m-%d, %H.%M") - Replace \"$KMVAR_Find\" with \"$KMVAR_Replace\""
mkdir "$backup_directory"

# Identify notes containing the string to be replaced.
grep -rl "$KMVAR_Find" . | while read -r f ; do

	# Back up the notes.
	cp -p "$f" "$backup_directory/$f"

	# Perform the replacement.
	# Save the modification timestamp against an empty temporary file. An alternative method would be to use `stat` to save the modification time to a variable.
	touch "$f.temp"
	touch -r "$f" "$f.temp"
	sed -i "" "s|$KMVAR_Find|$KMVAR_Replace|g" "$f"
	touch -r "$f.temp" "$f"
	rm "$f.temp"

done
```

### To Be Improved

* Escape pipes (`|`), which currently break the call to `sed`.

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.00 | 2020-07-02 | Initial commit |

## Rename and Update Wikilinks

Renames a specified note and updates `[[wikilinks]]` to it.

As per the general assumptions above, this macro assumes that wikilinks always use the full filename of the target note.

If you use UID-only wikilinks, do not use this macro. A principal benefit of a UID scheme is that it should obviate the need for a macro like this.

### Links

*	Direct links to `.kmmacros`: (right click and 'save link/target')
	* [with backups](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Rename%20and%20Update%20Wikilinks%2C%20with%20Backups.kmmacros)
	* [without backups](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Rename%20and%20Update%20Wikilinks%2C%20without%20Backups.kmmacros)

*	[Zettelkasten.de forum post for this macro](https://forum.zettelkasten.de/discussion/1230/km-macro-rename-note-and-update-wikilinks-to-it/)

### Steps in the Macro

1. The user is prompted for the note to be renamed, in an autocomplete list search.
2. The user is prompted for the new title.
3. The note to be renamed is backed up and renamed. Its modification time is updated.
4. All wikilinks to `[[Old Title]]` are replaced with `[[New Title]]`. The modification time of notes other than the one renamed are not updated.

### To Be Improved

* The macro should check that a note with the new title doesn't already exist; if does, it should abort.

* The shell script compiling a list of all notes should use `find` rather than `ls`, for the reasons described on Greg's Wiki: [Bash Guide: Practices](http://mywiki.wooledge.org/BashGuide/Practices) (under "5. Don't Ever Do These") and [Why you shouldn't parse the output of ls(1)](http://mywiki.wooledge.org/ParsingLs).

* Escape pipes (`|`), which currently break the call to `sed`.

### Invoked Shell Scripts

Compile a list of all notes:

```sh
cd "$KMVAR_Notes_Directory"
ls -t | sed -e 's/.[^.]*$//'
```

Rename the note and update `[[wikilinks]]` to it, taking backups:

```sh
cd "$KMVAR_Notes_Directory"

# Create a backup directory.
backup_subdirectory="$KMVAR_Backup_Directory/$(date "+%Y-%m-%d, %H.%M") - Rename \"$KMVAR_Old_Title\" to \"$KMVAR_New_Title\""
mkdir "$backup_subdirectory"

f=$(find . -name "$KMVAR_Old_Title.*")

# Back up the note to be renamed.
cp -p "$f" "$backup_subdirectory/$f"

mv "$f" "${f//$KMVAR_Old_Title/$KMVAR_New_Title}"

# Identify notes containing wikilinks to the renamed note.
grep -rl "[[$KMVAR_Old_Title]]" . | while read -r f ; do

	# Back up the notes.
	# Exclude the just-renamed note, which may include a wikilink to itself (such as in a YAML header).
	f_basename=${f##*/}
	f_basename=${f_basename%.*}
	if [ "$f_basename" != "$KMVAR_New_Title" ]; then
		cp -p "$f" "$backup_subdirectory/$f"
	fi

	# Update the wikilinks.
	# Save the modification timestamp against an empty temporary file. An alternative method would be to use `stat` to save the modification time to a variable.
	touch "$f.temp"
	touch -r "$f" "$f.temp"
	sed -i "" "s|\[\[$KMVAR_Old_Title\]\]|\[\[$KMVAR_New_Title\]\]|g" "$f"
	touch -r "$f.temp" "$f"
	rm "$f.temp"

done
```

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.00 | 2020-07-02 | Initial commit |