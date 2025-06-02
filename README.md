# osxmetadata

[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/python/black)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![All Contributors](https://img.shields.io/badge/all_contributors-5-orange.svg?style=flat)](#contributors-)

## What is osxmetadata?

osxmetadata provides a simple interface to access various metadata about MacOS / OS X files.  Currently supported metadata attributes include tags/keywords, Finder comments, authors, etc.

## Why osxmetadata?

Apple provides rich support for file metadata through the [MDItem](https://developer.apple.com/documentation/coreservices/file_metadata/mditem) class and the [NSURL getResourceValue:forKey:error:](https://developer.apple.com/documentation/foundation/nsurl/1408874-getresourcevalue?language=objc) method. However, Apple does not provide a way to easily set much of the metadata. For example, while there is a documented [MDItem MDItemCopyAttribute](https://developer.apple.com/documentation/coreservices/1427080-mditemcopyattribute?language=objc) to copy metadata attributes such as [kMDItemAuthors](https://developer.apple.com/documentation/coreservices/kmditemauthors?language=objc), Apple does not provide a public interface to set this data.  Other data, such as Finder comments, can only be set through sending AppleScript commands to the Finder and others, like Finder tags can be retrieved but cannot be set through the public API.

osxmetadata provides a unified interface to get and set most of the metadata available on your Mac from python.  It uses a combination of documented and undocumented APIs to access the metadata.  It also provides a simple interface to set Finder tags and Finder comments.

MacOS provides some tools to view these various metadata attributes.  For example, `mdls` lists the MDItem Spotlight metadata associated with a file but doesn't let you edit the data. osxmetadata makes it easy to to both view and manipulate the macOS metadata attributes, either programmatically or through an included `osxmetadata` command line tool.

## Supported operating systems

Only works on MacOS. Tested on macOS 13.5.1 (Ventura); should work on all versions of macOS 10.15 and later. Supported on Python 3.10+.

## Installation instructions

### Installation using pipx

If you aren't familiar with installing python applications, I recommend you install `osxmetadata` with [pipx](https://github.com/pipxproject/pipx). If you use `pipx`, you will not need to create a virtual environment as `pipx` takes care of this. The easiest way to do this on a Mac is to use [homebrew](https://brew.sh/):

- Open `Terminal` (search for `Terminal` in Spotlight or look in `Applications/Utilities`)
- Install `homebrew` according to instructions at [https://brew.sh/](https://brew.sh/)
- Type the following into Terminal: `brew install pipx`
- Then type this: `pipx install osxmetadata`
- Now you should be able to run `osxmetadata` by typing: `osxmetadata`

Once you've installed osxmetadata with pipx, to upgrade to the latest version:

    pipx upgrade osxmetadata

### Installation using pip

You can also install directly from [pypi](https://pypi.org/project/osxmetadata/):

    pip install osxmetadata

Once you've installed osxmetadata with pip, to upgrade to the latest version:

    pip install --upgrade osxmetadata

### Installation from git repository

OSXMetaData uses setuptools, thus simply run:

    git clone https://github.com/RhetTbull/osxmetadata.git
    cd osxmetadata
    pip install poetry
    poetry install

I recommend you create a [virtual environment](https://docs.python.org/3/tutorial/venv.html) before installing osxmetadata.

## Using the API

```pycon
>>> import datetime
>>> import pathlib
>>> from osxmetadata import *
>>> pathlib.Path("test_file.txt").touch()
>>> md = OSXMetaData("test_file.txt")
>>> md.set(kMDItemAuthors, ["Jane Smith", "John Doe"])
>>> md.get(kMDItemAuthors)
['Jane Smith', 'John Doe']
>>> md.kMDItemFinderComment = "This is my comment"
>>> md.kMDItemFinderComment
'This is my comment'
>>> md.tags = [Tag("Test", FINDER_COLOR_NONE), Tag("ToDo", FINDER_COLOR_RED)]
>>> md.tags
[Tag(name='Test', color=0), Tag(name='ToDo', color=6)]
>>> md[kMDItemDueDate] = datetime.datetime(2022,10,1)
>>> md[kMDItemDueDate]
datetime.datetime(2022, 10, 1, 0, 0)
>>>
```

Somewhat contrary to the [Zen of Python](https://peps.python.org/pep-0020/), osxmetadata provides more than one way to access metadata attributes.  You can get and set metadata attributes using the `get()`/`set()` getter/setter methods, using the attribute name as a dictionary key on the OSXMetaData object, or using the attribute name as an attribute on the `OSXMetaData()` object.  For example, the following are all equivalent:

- `OSXMetaData.get(attribute)` - get the value of the metadata attribute `attribute`
- `OSXMetaData[attribute]` - get the value of the metadata attribute `attribute`
- `OSXMetaData.attribute` - get the value of the metadata attribute `attribute`

As are the following:

- `OSXMetaData.set(attribute, value)` - set the value of the metadata attribute `attribute` to `value`
- `OSXMetaData[attribute] = value` - set the value of the metadata attribute `attribute` to `value`
- `OSXMetaData.attribute = value` - set the value of the metadata attribute `attribute` to `value`

This allows you to use osxmetadata in accordance with your own code style preferences.

Supported attribute names include all attributes defined for [MDItem](https://developer.apple.com/documentation/coreservices/file_metadata/mditem) and all resource keys defined for [NSURL](https://developer.apple.com/documentation/foundation/nsurl?language=objc). Additionally, the metadata constants defined in the [MDImporter](https://developer.apple.com/documentation/coreservices/file_metadata/mdimporter?language=objc) are supported as well as the following additional attributes:

- `_kMDItemUserTags` - list of Finder tags
- `kMDItemDownloadedDate` - list of datetime objects for when the file was downloaded

Additionally, osxmetadata defines a "shortcut name" attribute for each MDItem attribute that can be used as a shortcut `OSXMetaData` class attribute.  The shortcut name is the lowercase value of text following `kMDItem` for each attribute. For example, `kMDItemAuthors` has a short name of `authors` so you can set the authors like this:

```pycon
>>> from osxmetadata import *
>>> md = OSXMetaData("test_file.txt")
>>> md.authors = ["Jane Smith", "John Doe"]
>>> md.authors
['Jane Smith', 'John Doe']
>>>
```

and `kMDItemDueDate` would have a short name of `duedate`:

```pycon
>>> from osxmetadata import *
>>> import datetime
>>> md = OSXMetaData("test_file.txt")
>>> md.duedate = datetime.datetime(2022, 10, 1)
>>> md.duedate
datetime.datetime(2022, 10, 1, 0, 0)
>>>
```

The names of all supported attributes are available in the `osxmetadata.ALL_ATTRIBUTES` set:

```pycon
>>> from osxmetadata import ALL_ATTRIBUTES
>>> "kMDItemDueDate" in ALL_ATTRIBUTES
True
>>> "NSURLTagNamesKey" in ALL_ATTRIBUTES
True
>>> "findercomment" in ALL_ATTRIBUTES
True
>>>
```

The class attributes are handled dynamically which, unfortunately, means that IDEs like PyCharm and Visual Studio Code cannot provide tab-completion for them.

> [!NOTE]
> When writing or updating metadata with OSXMetaData, the OS will take some time to update the metadata on disk; in my testing, this can be as short as 100ms or as long as 3s. This means that if you read the metadata immediately after writing it, you may not see the updated metadata. If your use case requires the use of immediate read after write, you may need to implement a delay in your code to allow the OS time to update the metadata on disk. This appears to be an OS limitation and not something that can be controlled by osxmetadata.

```pycon

## Finder Tags

Unlike other attributes, which are mapped to native Python types appropriate for the source Objective C type, Finder tags (`_kMDItemUserTags` or `tags`) have two components: a name (str) and a color ID (unsigned int in range 0 to 7) representing a color tag in the Finder.  Reading tags returns a list of `Tag` namedtuples and setting tags requires a list of `Tag` namedtuples.

```pycon
>>> from osxmetadata import *
>>> md = OSXMetaData("test_file.txt")
>>> md.tags = [Tag("Test", FINDER_COLOR_NONE), Tag("ToDo", FINDER_COLOR_RED)]
>>> md.tags
[Tag(name='Test', color=0), Tag(name='ToDo', color=6)]
>>> md.get("_kMDItemUserTags")
[Tag(name='Test', color=0), Tag(name='ToDo', color=6)]
>>>
```

Tag names (but not colors) can also be accessed through the [NSURLTagNamesKey](https://developer.apple.com/documentation/foundation/nsurltagnameskey) resource key and the label color ID is accessible through `NSURLLabelNumberKey`; the localized label color name is accessible through `NSURLLocalizedLabelKey` though these latter two resource keys only return a single color whereas a file may have more than one color tag. For most purposes, I recommend using the `tags` attribute as it is more convenient and provides access to both the name and color ID of the tag.

```pycon
>>> from osxmetadata import *
>>> md = OSXMetaData("test_file.txt")
>>> md.tags = [Tag("Test", FINDER_COLOR_NONE), Tag("ToDo", FINDER_COLOR_RED)]
>>> md.tags
[Tag(name='Test', color=0), Tag(name='ToDo', color=6)]
>>> md.NSURLTagNamesKey
(
    Test,
    ToDo
)
>>> md.NSURLLabelNumberKey
6
>>> md.NSURLLocalizedLabelKey
'Red'
>>> md.NSURLTagNamesKey = ["NewTag"]
>>> md.NSURLTagNamesKey
(
    NewTag
)
>>> md.tags
[Tag(name='NewTag', color=0)]
>>>
```

### Create a Tag namedtuple

`Tag(name, color)`

- `name`: tag name (str)
- `color`: color ID for Finder color label associated with tag (int)

Valid color constants (exported by osxmetadata):

- `FINDER_COLOR_NONE` = 0
- `FINDER_COLOR_GRAY` = 1
- `FINDER_COLOR_GREEN` = 2
- `FINDER_COLOR_PURPLE` = 3
- `FINDER_COLOR_BLUE` = 4
- `FINDER_COLOR_YELLOW` = 5
- `FINDER_COLOR_RED` = 6
- `FINDER_COLOR_ORANGE` = 7

## Finder Comments

Finder comments can be access via the `kMDItemFinderComment` attribute or the `findercomment` shortcut attribute. Apple provides a public API for getting Finder comments but does not provide a programmatic method for setting Finder comments and I have not been able to find a private API for doing so. osxmetadata works around this by send AppleScript events to the Finder to set the Finder comment. This means that setting Finder comments is slower than setting other attributes and may not work in all circumstances. The first time you set a Finder comment, your terminal app may need to prompt you to allow AppleScript to control the Finder. If you include osxmetadata in a standalone app, for example, one created with [py2app](https://py2app.readthedocs.io/en/latest/), you will need to include the `com.apple.security.automation.apple-events` entitlement and the `NSAppleEventsUsageDescription` key in your app's `Info.plist` file. See the [Apple Developer Documentation](https://developer.apple.com/documentation/bundleresources/information_property_list/nsappleeventsusagedescription?language=objc) for more information.

```pycon
>>> from osxmetadata import *
>>> md = OSXMetaData("test_file.txt")
>>> md.kMDItemFinderComment = "Hello World!"
>>> md.kMDItemFinderComment
'Hello World!'
>>> md.findercomment
'Hello World!'
>>>
```

## Dates/Times

Metadata attributes which return date/times such as `kMDItemDueDate` or `kMDItemDownloadedDate` return a `datetime.datetime` object.  The `datetime.datetime` object is timezone-naive (does not contain timezone) and returns the time in the local timezone.  Internally, Apple appears to store these as [CFDate](https://developer.apple.com/documentation/corefoundation/cfdate?language=objc) objects in the UTC timezone but when retrieved, they are returned in the local time.  You may pass a timezone-aware datetime object to set these attributes and it will be converted appropriately.

```pycon
>>> from osxmetadata import *
>>> md = OSXMetaData("test_file.txt")
>>> import datetime
>>> md.kMDItemDueDate = datetime.datetime(2022, 10, 1)
>>> md.kMDItemDueDate
datetime.datetime(2022, 10, 1, 0, 0)
>>> md.kMDItemDownloadedDate = datetime.datetime(2022, 10, 1, tzinfo=datetime.timezone.utc)
>>>
```

## Extended Attributes

In addition to `MDItem` and `NSURL` metadata attributes, osxmetadata can also read & write metadata saved in extended attributes. For many MDItem attributes, Apple stores the same data in both the MDItem and extended attribute (with name `com.apple.metadata:AttributeName`).  For example, the `kMDItemWhereFroms` attribute can be accessed both via MDItemCopyAttribute (exposed via osxmetadata's `get()` method) and via the `com.apple.metadata:kMDItemWhereFroms` extended attribute.  The extended attribute is a binary plist (BPLIST) and can be read using the `xattr` command line tool.  The `get_xattr()` method will return the value of the extended attribute and the `set_xattr()` method will set it.  Extended attributes can be removed with the `remove_xattr()` method.  `get_xattr()` provides for an optional callable argument, `decode`, which will be called on the returned value.  `set_xattr()` provides an optional callable argument `encode`. This is useful for encoding/decoding binary plist data.  For example, to decode the `com.apple.metadata:kMDItemWhereFroms` extended attribute, you can use the `plistlib.loads()` function:

```pycon
>>> from osxmetadata import *
>>> import plistlib
>>> from plistlib import FMT_BINARY
>>> from functools import partial
>>> md = OSXMetaData("test_file.txt")
>>> md.kMDItemWhereFroms = ["apple.com"]
>>> md.kMDItemWhereFroms
['apple.com']
>>> decode = partial(plistlib.loads, fmt=FMT_BINARY)
>>> encode = partial(plistlib.dumps, fmt=FMT_BINARY)
>>> md.get_xattr("com.apple.metadata:kMDItemWhereFroms")
b'bplist00\xa1\x01Yapple.com\x08\n\x00\x00\x00\x00\x00\x00\x01\x01\x00\x00\x00\x00\x00\x00\x00\x02\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x14'
>>> md.get_xattr("com.apple.metadata:kMDItemWhereFroms", decode=decode)
['apple.com']
>>> md.set_xattr("com.apple.metadata:kMDItemWhereFroms", ["google.com"], encode=encode)
>>> md.get_xattr("com.apple.metadata:kMDItemWhereFroms", decode=decode)
['google.com']
>>> md.remove_xattr("com.apple.metadata:kMDItemWhereFroms")
>>>
```

For most use cases, it is recommended you do not directly access the Apple metadata related extended attributes and instead use the getter/setter methods provided by osxmetadata.

## Finder Info

The Finder keeps some legacy Finder info data about files in a bitstring stored in the `com.apple.FinderInfo` extended attribute. osxmetadata provides some attributes for working with this data.

- `stationerypad`: True if the file is a stationery pad (file template) otherwise False; setting this attribute has the same effect as setting the `Stationery pad` checkbox in the Finder's `Get Info` window.
- `findercolor`: The color of the file as an integer; setting this attribute has the same effect as applying a color label in the Finder's `Get Info` window. osxmetadata will set this attribute automatically when setting user tags; it is recommended you do not set this attribute directly.
- `finderinfo`: The raw Finder info data as a bytes object; you should only manipulate this attribute if you know what you are doing.

## Temporary Files

Spotlight does not appear to index temporary files (those in `/tmp` or `/private/var/tmp`). Setting metadata using osxmetadata on temporary files in these locations will not fail but but it appears the metadata will not be indexed and a subsequent read will return the default value as if the metadata had not been written. This is not a limitation of osxmetadata but rather a limitation of Spotlight. If you need to set metadata on temporary files, you should use a different location.

## Command Line Usage

Installs command line tool called `osxmetadata` which provides a simple interface to view/edit metadata supported by osxmetadata.

If you only care about the command line tool, I recommend installing with [pipx](https://github.com/pipxproject/pipx)

The command line tool can also be run via `python -m osxmetadata`.  Running it with no arguments or with --help option will print a help message:

<!-- [[[cog
import cog
from osxmetadata.__main__ import cli
from click.testing import CliRunner
runner = CliRunner()
result = runner.invoke(cli, ["--help"])
help = result.output.replace("Usage: cli", "Usage: osxmetadata")
cog.out(
    "```\n{}\n```".format(help)
)
]]] -->
```
Usage: osxmetadata [OPTIONS] FILE

  Read/write metadata from file(s).

Options:
  -v, --version                   Show the version and exit.
  -w, --walk                      Walk directory tree, processing each file in
                                  the tree.
  -j, --json                      Print output in JSON format, for use with
                                  --list and --get.
  -X, --wipe                      Wipe all metadata attributes from FILE.
  -s, --set ATTRIBUTE VALUE       Set ATTRIBUTE to VALUE. If ATTRIBUTE is a
                                  multi-value attribute, such as keywords
                                  (kMDItemKeywords), you may specify --set
                                  multiple times to add to the array of values:
                                  '--set keywords foo --set keywords bar' will
                                  set keywords to ['foo', 'bar']. Not that this
                                  will overwrite any existing values for the
                                  attribute; see also --append.
  -l, --list                      List all metadata attributes for FILE.
  -c, --clear ATTRIBUTE           Remove attribute from FILE.
  -a, --append ATTRIBUTE VALUE    Append VALUE to ATTRIBUTE; for multi-valued
                                  attributes, appends only if VALUE is not
                                  already present. May be used in combination
                                  with --set to add to an existing value: '--set
                                  keywords foo --append keywords bar' will set
                                  keywords to ['foo', 'bar'], overwriting any
                                  existing values for the attribute.
  -g, --get ATTRIBUTE             Get value of ATTRIBUTE.
  -r, --remove ATTRIBUTE VALUE    Remove VALUE from ATTRIBUTE; only applies to
                                  multi-valued attributes.
  -m, --mirror ATTRIBUTE1 ATTRIBUTE2
                                  Mirror values between ATTRIBUTE1 and
                                  ATTRIBUTE2 so that ATTRIBUTE1 = ATTRIBUTE2;
                                  for multi-valued attributes, merges values;
                                  for string attributes, sets ATTRIBUTE1 =
                                  ATTRIBUTE2 overwriting any value in
                                  ATTRIBUTE1.  For example: '--mirror keywords
                                  tags' sets tags and keywords to same values.
  -B, --backup                    Backup FILE attributes.  Backup file
                                  '.osxmetadata.json' will be created in same
                                  folder as FILE. Only backs up attributes known
                                  to osxmetadata unless used with --all.
  -R, --restore                   Restore FILE attributes from backup file.
                                  Restore will look for backup file
                                  '.osxmetadata.json' in same folder as FILE.
                                  Only restores attributes known to osxmetadata
                                  unless used with --all.
  -V, --verbose                   Print verbose output.
  -f, --copyfrom SOURCE_FILE      Copy attributes from file SOURCE_FILE (only
                                  updates destination attributes that a