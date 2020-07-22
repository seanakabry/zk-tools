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
| [Append UIDs to Filenames](#append-uids-to-filenames) | Appends UIDs in the pattern ` (YYYYMMddHHmm)` to the names of the files selected in Finder. | 1.02 | 2020-07-14 |
| [Back Up Notes](#back-up-notes) | Copies all notes to a timestamped backup directory. | 1.01 | 2020-07-07 |
| [Clip Highlighted Text in Firefox](#clip-highlighted-text-in-firefox) | Copies the currently selected text in Firefox to a text file, with metadata, optionally with an annotation. | 1.00 | 2020-07-14 |
| [Insert UID](#insert-uid) | Inserts a UID in the pattern `YYYYMMddHHmm` at the cursor. | 1.02 | 2020-07-17 |
| [Find and Replace](#find-and-replace) | Performs a find and replace operation on the content but not the titles of all notes. | 1.02 | 2020-07-08 |
| [Open File or Folder by UID](#open-file-or-folder-by-uid) | Opens a file or folder outside the Zettelkasten using a UID. | 1.02 | 2020-07-22 |
| [Rename and Update Wikilinks](#rename-and-update-wikilinks) | Renames a specified note and updates `[[wikilinks]]` to it. | 1.03 | 2020-07-09 |

## Assumptions

In general, the tools assume:

* 	Your notes lie flat in a single directory (no subdirectories).

*	Your notes are `[[wikilinked]]` using full filenames rather than explicit UIDs. (A number of the macros assume that explicit UIDs are used to refer to files outside the notes directory.)

	By "full filenames" I really mean full basenames, where `[[Another Note]]` links to `Another Note.txt`, without regard to the file extension.

	An explicit UID (unique identifier) is a part of the filename of a note which follows a consistent syntax (usually a timestamp prefix like `202007011200`) and is supposed never to change, like a primary key for a record in a database. The wikilink `[[202007011200]]` would point to `202007011200 Another Note.txt`, and continue to point to the same note even if that note is renamed `202007011200 Edited Note.txt`. This robustness is one of the chief benefits of using explicit UIDs. Obvious drawbacks include the interruption of the flow of text with a long, usually non-semantic string of characters, and the difficulty of specifying the UID of a note one has in mind (made trivially easy by a wikilink auto-completion macro, such as by Zettelkasten.de forum users [kaidoh](https://forum.zettelkasten.de/discussion/176/quick-insertion-of-links-to-other-zettels-with-type-ahead-search-using-keyboard-maestro) or [Will](https://forum.zettelkasten.de/discussion/comment/2516/#Comment_2516)).

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

If you are also new to shell scripting and want to modify the scripts below, these pages may be useful:

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

*	Better account for special characters such as square brackets in find-replace strings.

# Macros

## Append UIDs to Filenames

Appends UIDs in the pattern ` (YYYYMMddHHmm)` to the names of the files selected in Finder.

This macro uses a global Keyboard Maestro variable, `Last UID`, shared with a number of other macros on this page so that UIDs assigned in rapid succession (more than one in a given clock minute) don't clash.

### Links

*	[Direct link to `.kmmacros`](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Append%20UIDs%20to%20Filenames.kmmacros) (right click and 'save link/target')

### Invoked Shell Script

```sh
extension="${KMVAR_Instance_Files##*.}"
basename="${KMVAR_Instance_Files%.*}"
mv "$KMVAR_Instance_Files" "${basename} ($KMVAR_Instance_UID_to_Assign).${extension}"
```

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.02 | 2020-07-14 | Save only most recent UID to `Last UID` |
| 1.01 | 2020-07-13 | Remove unnecessary braces |
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
| 1.01 | 2020-07-07 | Use [instance](https://wiki.keyboardmaestro.com/manual/Variables#Instance_Variables_v8) rather than global variables |
| 1.00 | 2020-07-02 | Initial commit |

## Clip Highlighted Text in Firefox

Copies the currently selected text in Firefox to a text file, with metadata, optionally with an annotation.

This macro uses a global Keyboard Maestro variable, `Last UID`, shared with a number of other macros on this page so that UIDs assigned in rapid succession (more than one in a given clock minute) don't clash.

Keyboard Maestro has [more convenient ways](https://wiki.keyboardmaestro.com/Tokens) of fetching the title and URL of the active page from Safari and Chrome, which would obviate steps 3 to 6 below.

### Links

*	Direct link to `.kmmacros` (right click and 'save link/target')
	*	[clip only](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Clip%20Highlighted%20Text%20in%20Firefox.kmmacros)
	*	[clip and annotate](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Clip%20Highlighted%20Text%20in%20Firefox,%20with%20an%20Annotation.kmmacros)

### Steps in the Macro

1.	(Prompt the user for a note to be appended to the clipped text.)
2.	Copy the selected text and save it to a variable.
3.	Simulate the keystroke `⌘L` to select the active page's URL, then copy and saved it to a variable.
4.	Simulate the keystroke `⌘⌥K` to open Firefox's web console. Pause for the console to become active.
5.	Execute the expression `copy(document.title)` to copy the active page's title, then save it to a variable. (If `copy(document.title)` appears anywhere in the resulting text file, try lengthening the pause preceding this step.)
6.	Close the developer tools.
7.	Assign a UID, checking for clashes.
8.	Create the text file with the UID as its filename.
9.	Open the text file for the user to check.

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.00 | 2020-07-14 | Initial commit |

## Insert UID

Inserts a UID in the pattern `YYYYMMddHHmm` at the cursor.

This macro uses a global Keyboard Maestro variable, `Last UID`, shared with a number of other macros on this page so that UIDs assigned in rapid succession (more than one in a given clock minute) don't clash.

### Links

*	[Direct link to `.kmmacros`](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Insert%20UID.kmmacros) (right click and 'save link/target')

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.02 | 2020-07-17 | Use "Delete Past Clipboard" |
| 1.01 | 2020-07-14 | Save only most recent UID to `Last UID` |
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
| 1.01 | 2020-07-07 | Use [instance](https://wiki.keyboardmaestro.com/manual/Variables#Instance_Variables_v8) rather than global variables |
| 1.00 | 2020-07-02 | Initial commit |

## Open File or Folder by UID

Opens the file or folder in a specified directory with a name containing the currently selected text. The intended use case is a UID scheme for documents and folders outside the Zettelkasten; as such, the macro expects that only one file or folder will match the search string.

As an example, a daily/diary note might contain a line like:

> *	[[Project Name]]: The table of sources has grown too large and unwieldy for Markdown. Moved it into a spreadsheet (202007071949).

Double-clicking the spreadsheet's UID and triggering the macro will open the spreadsheet in the default application for its filetype.

### Links

*	[Direct link to `.kmmacros`](https://raw.githubusercontent.com/seanakabry/zk-tools/master/kmmacros/Open%20File%20or%20Folder%20by%20UID.kmmacros) (right click and 'save link/target')
*	[Zettelkasten.de forum post for this macro](https://forum.zettelkasten.de/discussion/1235/km-macro-open-a-file-outside-the-zettelkasten-identified-by-a-uid)

### Invoked Shell Script

```sh
open "$(find "$KMVAR_Instance_Documents_Directory" -name "*$KMVAR_Instance_Document_UID*" | head -n1)"
```

### Areas for Improvement

*	If no match is found, the macro should abort and inform the user.

### Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.02 | 2020-07-22 | Allow folders as well as files to be opened |
| 1.01 | 2020-07-17 | Use "Delete Past Clipboard" |
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

1.	Prompt the user for the note to be renamed, in an auto-complete list search.
2.	Prompt the user for the new title.
3.	Back up and rename the subject note.
4.	Back up all notes containing `[[Old Title]]`, and replace all instances of `[[Old Title]]` with `[[New Title]]`.

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
grep -rl "\[\[$KMVAR_Instance_Old_Title\]\]" . | while read -r f ; do

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
| 1.03 | 2020-07-09 | Add escapes to `grep` calls |
| 1.02 | 2020-07-08 | Update modification times by default |
| 1.01 | 2020-07-07 | Use [instance](https://wiki.keyboardmaestro.com/manual/Variables#Instance_Variables_v8) rather than global variables |
| 1.00 | 2020-07-02 | Initial commit |