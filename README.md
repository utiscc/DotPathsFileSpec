# DotPathsFileSpec

Specification for the ".paths" file extension

This is a proposal for using the `.paths` file extension for particular file content, sharable between a wide range of programs that process a bunch of files (and folders etc.).

This document is currently WIP.

## Semantics (Protocol)

If an app supports opening .paths files, then it shall treat them as a set of files for _browsing_, as opposed to _opening_ them.

This is useful for any program that provides a file browser. If the app can also open certain documents for purpose of viewing or editing them, then, by default, it would always do so with any file it's asked to open.

So, if the user had a list of files, e.g. collected in Finder, and would then drag them to the app, or double click them, the app would open each file as a document, usually.

But what if the user wants to have the app's browser show the list of files, instead? Currently, there is no protocol for this – and that's what this protocol is about:

If the app is asked to open a .paths file, it should default to showing the items in its browser instead of opening them as invidual items or as a text document.

For instance, a text editor like BBEDit, which can both edit text files and also show a file browser for a set of files, opening a .paths file should give the user the option to either edit the file as a text document or list the referenced files in its file browser. How BBEdit provides this choice to the user is not part of the proposal, though.

For programs that cannot view/edit text files, opening a .paths file is even clearer: It's meant to read the file paths from it and then process them in a way that makes sense for that program. For instance, a file renamer would then take the list and process them all by renaming them as the user wishes.

Alongside this, it's also a good idea that such programs do not only respond to the "open document" event by processing the paths from the file but also by accepting .paths files be dropped into any windows of the app as far as it makes sense.

Example: Find Any File (FAF) finds files and shows them in a browser window, similar to the Finder. If you use the Save command, it'll write a .paths file containing the paths of the found files. And if you double click the .paths file, it'll open a new browser window, listing these files again. But a browser window also accepts drops of .paths files - and if the user does this, the files referenced by the droppged .paths file will be _added_ to the window, thereby _extending_ the browser's contents.

## Format specs

Roughly, the .paths format is a plain text (UTF-8, with optional [BOM](https://en.wikipedia.org/wiki/Byte_order_mark)) file with each line following these rules:

- empty -> ignore
- starts with "/" -> interpret as POSIX path
- starts with "~" -> interpret as POSIX path, replacing the leading ~ with the user's home folder
- starts with "#" -> handle as user-readable comment
- starts with "{" ->  interpret as JSON string. Content is currently private, so if you use this, add an identifier that allows you to detect whether you understand its contents. And the entire JSON string needs to be in a single line!

Ignore anything else (i.e. don't complain to the user about lines in an unexpected format).

Also, if the file contains a 00-byte, then assume that the lines are separated by 00-bytes (as from output of many cmdline tools with the "-0" option), otherwise assume LF as line separator (note that, at least on macOS, CRs are allowed in file names, such as for the infamous "Icon␍" file names).

That's about it. The format is therefore suited to be generated from the output of many cmdline tools that list file paths, and is easily user-readable as well.

Pretty obvious, really, but the rules above (handling of 00 bytes, optional comments and support for "~", and silently ignoring unknown lines) make sure that it's suitable for a lot of applications. For instance, FindAnyFile only writes the full paths to the file, whereas Scherlokk also writes some info about the search parameters into the first line, in JSON form. That way, Scherlokk uses this format to Save a result, and if the user later reopens the file, he'll not only get the results back but also gets the search parameters filled into the UI again.

## UTI

The UTI would conform to `public.utf8-plain-text`

For the UTI name itself, I like to come up with something that's not bound to anyone's own domain. Ideally, we'd register one with Apple under "public." but that's unlikely to happen.

So, here are some suggestions:

* public.posix-paths (this would be rogue because Apple reserves the right to name "public." for themselves only)
* common.posix-paths (there is no "common" domain, but so it would be safe)

Also, once decided, we should add this to https://en.wikipedia.org/wiki/Uniform_Type_Identifier

And this document should provide the exact plist declarations for exporting the types so that everyone adopting it gets it right.

## Programs that currently support this file format (in alphabetical order)

- [Find Any File](http://findanyfile.app/) (macOS), since v2.3
- [Scherlokk](https://naarakstudio.com/scherlokk/) (macOS)

## Program proposals (i.e. I/we contacted them or plan to do so)

- [BBEdit](https://www.barebones.com/products/bbedit/) (macOS)
- [FileBuddy](https://www.skytag.com) (macOS)
