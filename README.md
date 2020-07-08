# Zettelkasten Tools

This is a repository for [Keyboard Maestro](https://www.keyboardmaestro.com/) macros which aid in managing plain text notes following wiki and Zettelkasten conventions.

The macros are intended as companions to note-taking and -management software such as [nvUltra](https://nvultra.com/) and [The Archive](https://zettelkasten.de/the-archive/). Other such macros are listed in [this Zettelkasten.de forum thread](https://forum.zettelkasten.de/discussion/213/the-archive-keyboard-maestro-alfred-macros).

The shell scripts invoked by these macros (which appear below) may be of some use even to those who don't use Keyboard Maestro. They are intended to be POSIX-compatible and as portable as possible. Variables beginning with `$KMVAR_` are Keyboard Maestro variables which are expanded before the script is passed to the shell.

## Page Contents

*	[Assumptions](#assumptions)
*	[How to Use the Macros](#how-to-use-the-macros)
*	[Relevent Shell Scripting Resources](#relevant-shell-scripting-resources)
*	[Macros](#macros)
	* 	[Areas for Improvement](#areas-for-improvement)

| Macro | Function | v. | Updated |
| :---- | -------- | :- | :------------ |
| [Append UIDs to Filenames](#append-uids-to-filenames) | Appends UIDs in the pattern ` (YYYYMMddHHmm)` to the names of the files selected in Finder. | 1.00 | 2020-07-08 |
| [Back Up Notes](#back-up-notes) | Copies all notes to a timestamped backup directory. | 1.01 | 2020-07-07 |
| [Insert UID](#insert-uid) | Inserts a UID in the pattern `YYYYMMddHHmm` at the cursor. | 1.00 | 2020-07-08 |
| [Find and Replace](#find-and-replace) | Performs a find and replace operation on the content but not the titles of all notes. | 1.02 | 2020-07-08 |
| [Open File by UID](#open-file-by-uid) | Opens a file outside the Zettelkasten using a UID. | 1.00 | 2020-07-07 |
| [Rename and Update Wikilinks](#rename-and-update-wikilinks) | Renames a specified note and updates `[[wikilinks]]` to it. | 1.02 | 2020-07-08 |

## Assumptions

In general, the tools assume:

* 	Your notes lie flat in a single directory (no subdirectories).

*	Your notes are `[[wikilinked]]` using full filenames rather than explicit UIDs.

	By "full filenames" I really mean full basenames, where `[[Another Note]]` links to `Another Note.txt`, without regard to the file extension.

	An explicit UID (unique identifier) is a part of the filename of a note which follows a consistent syntax (usually a timestamp prefix like `202007011200`) and is supposed never to change, like a primary key for a record in a database. The wikilink `[[202007011200]]` would point to `202007011200 Another Note.txt`, and continue to point to the same note even if that note is renamed `202007011200 Edited Note.txt`. This robustness is one of the chief benefits of using explicit UIDs. Obvious drawbacks include the interruption of the flow of text with a long, usually non-semantic string of characters, and the difficulty of specifying the UID of a note one has in mind (made trivially easy by a wikilink autocompletion macro, such as by Zettelkasten.de forum users [kaidoh](https://forum.zettelkasten.de/discussion/176/quick-insertion-of-links-to-other-zettels-with-type-ahead-search-using-keyboard-maestro) or [Will](https://forum.zettelkasten.de/discussion/comment/2516/#Comment_2516)).

	The Archive [recognises UIDs](https://zettelkasten.de/the-archive/help/#how-do-i-create-links-between-notes). nvUltra [does not](https://twitter.com/nvUltraApp/status/1268882547066515458).

*	You trust that the notes you're working with don't include malicious command injections. I've taken precautions where I know to, but I'm new to shell scripting and this hasn't been front of mind.

## How to Use the Macros

1.	Download and open the desired macro (`.kmmacros`).

2.	Edit any variables which should be customised, in the "Set Variables" action(s) highlighted yellow, such as:

	*	Notes Directory: The directory containing your plain text notes. A folder can be dragged into the action window from Finder.

	*	Backup Directory: The directory to which all notes to be edited by a macro will be backed up. The notes will be copied into a subdirectory with a descriptive title, like `2020-07-01, 12.00 - Rename "this" to "that"`. If you prefer, a version of each macro which doesn't take backups is included.

3.	Assign a suitable trigger to the macro.

4.	Enable the macro.

## Relevant Shell Scripting Resources

If you are also new to shell scripting and want to modify the scripts above, these pages may be useful:

*	Keyboard Maestro
	*	[Wiki: Execute a Shell Script](https://wiki.keyboardmaestro.com/action/Execute_a_Shell_Script)
	*	[Forum: Bash Shell Script Needs a Variable Passed with Characters “Escaped”](https://forum.keyboardmaestro.com/t/bash-shell-script-needs-a-variable-passed-with-characters-escaped/14130)
*	[Greg's Wiki](https://mywiki.wooledge.org/EnglishFrontPage)
	*	[Bash Guide: Practices](http://mywiki.wooledge.org/BashGuide/Practices)
	*	[Using `find`](http://mywiki.wooledge.org/UsingFind)
	*	[How can I rename all my \*.foo files to \*.bar, or convert spaces to underscores, or convert upper-case file names to lower case?](https://mywiki.wooledge.org/BashFAQ/030)
*	File path string manipulation (via [this SO answer](https://stackoverflow.com/a/2664746))
	*	[`bash` String Manipulations](http://tldp.org/LDP/LG/issue18/bash.html)
	*	[Shell Command Language: 2.6.2 Parameter Expansion](https://pubs.opengroup.org/)

## Areas for Improvement

These are general to most or all macros. Points specific to individual macros are noted below.

*	Preserve original creation times, preferably without the use of `SetFile`, which requires that the user have the Command Line Tools for Xcode installed.
	
	The creation time of a file can be saved as a string in syntax preferred by `touch` with:

	```sh
	birthtime=$(date -j -f "%b  %d %T %Y" "$(stat -f "%SB" "$f")" +"%Y%m%d%H%M")
	```

# Macros

## Append UIDs to Filenames

Appends UIDs in the pattern ` (YYYYMMddHHmm)` to the names of the files selected in Finder. The macro shares a global variable `Last UID` with [Insert UID](#insert-uid) so that UIDs assigned in rapid succession (that is, more than one in a given clock minute) don't overlap.

### Links

*	[Direct link to `.kmmacros`](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Append%20UIDs%20to%20Filenames.kmmacros) (right click and 'save link/target')

### Invoked Shell Script

```sh
extension="${KMVAR_Instance_Files##*.}"
basename="${KMVAR_Instance_Files%.*}"
mv "$KMVAR_Instance_Files" "${basename} ($KMVAR_Instance_UID_to_Assign).${extension}"
```

### Areas for Improvement

*	The shell script which performs house-keeping on the global variable `Last UID` should delete any UIDs corresponding to times in the past.

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.00 | 2020-07-08 | Initial commit |

## Back Up Notes

Copies all notes to a timestamped backup directory.

### Links

*	[Direct link to `.kmmacros`](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Back%20Up.kmmacros) (right click and 'save link/target')

### Invoked Shell Script

```sh
cp -a "$KMVAR_Instance_Notes_Directory/" "$KMVAR_Instance_Backup_Directory/$(date "+%Y-%m-%d, %H.%M") - Manual Backup/"
```

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.01 | 2020-07-07 | Uses [instance](https://wiki.keyboardmaestro.com/manual/Variables#Instance_Variables_v8) rather than global variables |
| 1.00 | 2020-07-02 | Initial commit |

## Insert UID

Inserts a UID in the pattern `YYYYMMddHHmm` at the cursor. The macro shares a global variable `Last UID` with [Append UIDs to Filenames](#append-uids-to-filenames) so that UIDs assigned in rapid succession (that is, more than one in a given clock minute) don't overlap.

### Links

*	[Direct link to `.kmmacros`](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Insert%20UID.kmmacros) (right click and 'save link/target')

### Areas for Improvement

*	The shell script which performs house-keeping on the global variable `Last UID` should delete any UIDs corresponding to times in the past.

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.00 | 2020-07-08 | Initial commit |

## Find and Replace

Performs a find and replace operation on the content but not the titles of all notes.

To leave the modification times of edited files as they were, uncomment the relevant lines in the invoked shell script. This might be desirable if, in your note-taking software, you sort your notes by modification time and want to see the notes you've been working on recently at the top, rather than the notes updated by this macro. Note that if The Archive is open while this macro is invoked, it won't recognise that the edited files have changed, and you may as a result end up overwriting what this macro has changed.

### Links

*	Direct links to `.kmmacros`: (right click and 'save link/target')
	* [with backups](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Find%20and%20Replace%2C%20with%20Backups.kmmacros)
	* [without backups](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Find%20and%20Replace%2C%20without%20Backups.kmmacros)

### Invoked Shell Script

```sh
cd "$KMVAR_Instance_Notes_Directory"

# Create a backup directory.
backup_directory="$KMVAR_Instance_Backup_Directory/$(date "+%Y-%m-%d, %H.%M") - Replace \"$KMVAR_Instance_Find\" with \"$KMVAR_Instance_Replace\""
mkdir "$backup_directory"

# Identify notes containing the string to be replaced.
grep -rl "$KMVAR_Instance_Find" . | while read -r f ; do

	# Back up the notes.
	cp -p "$f" "$backup_directory/$f"

	# Perform the replacement.
	# Uncomment the commented lines in the following block if you prefer that the modification times of notes containing wikilinks are *not* updated. These lines save the modification timestamp against an empty temporary file. An alternative method would be to use `stat` to save the modification time to a variable.
	# touch "$f.temp"
	# touch -r "$f" "$f.temp"
	sed -i "" "s|$KMVAR_Instance_Find|$KMVAR_Instance_Replace|g" "$f"
	# touch -r "$f.temp" "$f"
	# rm "$f.temp"

done
```

### Areas for Improvement

*	Escape pipes (`|`), which currently break the call to `sed`.

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.02 | 2020-07-08 | Update modification times by default |
| 1.01 | 2020-07-07 | Uses [instance](https://wiki.keyboardmaestro.com/manual/Variables#Instance_Variables_v8) rather than global variables |
| 1.00 | 2020-07-02 | Initial commit |

## Open File by UID

Opens the file in a specified directory with a filename containing the currently selected text. The intended use case is a UID scheme for documents outside the Zettelkasten; as such, the macro expects that only one file will match the search string.

As an example, a daily/diary note might contain a line like:

> *	[[Project Name]]: The table of sources has grown too large and unwieldy for Markdown. Moved it into a spreadsheet (202007071949).

Double-clicking the spreadsheet's UID and triggering the macro will open the spreadsheet in the default application for its filetype.

### Links

*	[Direct link to `.kmmacros`](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Open%20File%20by%20UID.kmmacros) (right click and 'save link/target')
*	[Zettelkasten.de forum post for this macro](https://forum.zettelkasten.de/discussion/1235/km-macro-open-a-file-outside-the-zettelkasten-identified-by-a-uid)

### Invoked Shell Script

```sh
open "$(find "$KMVAR_Instance_Documents_Directory" -type f -name "*$KMVAR_Instance_Document_UID*" | head -n1)"
```

### Areas for Improvement

*	If no match is found, the macro should abort and inform the user.

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.00 | 2020-07-07 | Initial commit |

## Rename and Update Wikilinks

Renames a specified note and updates `[[wikilinks]]` to it.

As per the general assumptions above, this macro assumes that wikilinks always use the full filename of the target note.

If you use UID-only wikilinks, you don't want to use this. A principal benefit of a UID scheme is that it should obviate the need for a macro like this.

To leave the modification times of edited files as they were, uncomment the relevant lines in the invoked shell script. This might be desirable if, in your note-taking software, you sort your notes by modification time and want to see the notes you've been working on recently at the top, rather than the notes updated by this macro. Note that if The Archive is open while this macro is invoked, it won't recognise that the edited files have changed, and you may as a result end up overwriting what this macro has changed.

### Links

*	Direct links to `.kmmacros`: (right click and 'save link/target')
	*	[with backups](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Rename%20and%20Update%20Wikilinks%2C%20with%20Backups.kmmacros)
	*	[without backups](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Rename%20and%20Update%20Wikilinks%2C%20without%20Backups.kmmacros)
*	[Zettelkasten.de forum post for this macro](https://forum.zettelkasten.de/discussion/1230/km-macro-rename-note-and-update-wikilinks-to-it/)

### Steps in the Macro

1.	The user is prompted for the note to be renamed, in an autocomplete list search.
2.	The user is prompted for the new title.
3.	The note to be renamed is backed up and renamed. Its modification time is updated.
4.	All wikilinks to `[[Old Title]]` are replaced with `[[New Title]]`. The modification time of notes other than the one renamed are not updated.

### Areas for Improvement

*	The macro should check that a note with the new title doesn't already exist; if does, it should abort.

*	The shell script compiling a list of all notes should use `find` rather than `ls`, for the reasons described on Greg's Wiki: [Bash Guide: Practices](http://mywiki.wooledge.org/BashGuide/Practices) (under "5. Don't Ever Do These") and [Why you shouldn't parse the output of ls(1)](http://mywiki.wooledge.org/ParsingLs).

*	Escape pipes (`|`), which currently break the call to `sed`.

### Invoked Shell Scripts

Compile a list of all notes:

```sh
cd "$KMVAR_Instance_Notes_Directory"
ls -t | sed -e 's/.[^.]*$//'
```

Rename the note and update `[[wikilinks]]` to it, taking backups:

```sh
cd "$KMVAR_Instance_Notes_Directory"

# Create a backup directory.
backup_directory="$KMVAR_Instance_Backup_Directory/$(date "+%Y-%m-%d, %H.%M") - Rename \"$KMVAR_Instance_Old_Title\" to \"$KMVAR_Instance_New_Title\""
mkdir "$backup_directory"

f=$(find . -name "$KMVAR_Instance_Old_Title.*")

# Back up the note to be renamed.
cp -p "$f" "$backup_directory/$f"

touch "$f"
mv "$f" "${f//$KMVAR_Instance_Old_Title/$KMVAR_Instance_New_Title}"

# Identify notes containing wikilinks to the renamed note.
grep -rl "[[$KMVAR_Instance_Old_Title]]" . | while read -r f ; do

	# Back up the notes.
	# Exclude the just-renamed note, which may include a wikilink to itself (such as in a YAML header).
	f_basename=${f##*/}
	f_basename=${f_basename%.*}
	if [ "$f_basename" != "$KMVAR_Instance_New_Title" ]; then
		cp -p "$f" "$backup_directory/$f"
	fi

	# Update the wikilinks.
	# Uncomment the commented lines in the following block if you prefer that the modification times of notes containing wikilinks are *not* updated. These lines save the modification timestamp against an empty temporary file. An alternative method would be to use `stat` to save the modification time to a variable.
	# touch "$f.temp"
	# touch -r "$f" "$f.temp"
	sed -i "" "s|\[\[$KMVAR_Instance_Old_Title\]\]|\[\[$KMVAR_Instance_New_Title\]\]|g" "$f"
	# touch -r "$f.temp" "$f"
	# rm "$f.temp"

done
```

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.02 | 2020-07-08 | Update modification times by default |
| 1.01 | 2020-07-07 | Uses [instance](https://wiki.keyboardmaestro.com/manual/Variables#Instance_Variables_v8) rather than global variables |
| 1.00 | 2020-07-02 | Initial commit |