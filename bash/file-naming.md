---
title: UNIX Filename Best Practices
layout: page
permalink: /bash/filenames.html
parent: Bash and the Command Line
nav_enabled: true
nav_order: 4
---
# Filename Best Practices on UNIX Filesystems

The following directives can prevent unintended bugs or crashes when your files and folders are ingested by scripts or software.

1. Don't use spaces in filenames, ever.
2. Don't begin filenames with `-` (used for options) or `.` (reserved for hidden files).
3. Stick with letters, numbers, `.` (period or 'full stop'), `-` (dash) and `_` (underscore).
4. Use at most one `.` in a filename, preferably to indicate the extension.
5. Avoid non-ASCII characters like `ï` or `好` or in filenames.