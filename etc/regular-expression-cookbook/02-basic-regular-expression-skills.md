# Chapter 2: Basic Regular Expression Skills

- when creating a regular expression, you’ll likely need it to match certain text literally, and you’ll need to know which characters to escape.

### 2.1 Match Literal Text

- Any regular expression that does not include any of the dozen characters `$()*+.?[\^{|` simply matches itself.
- The 12 punctuation characters that make regular expressions work their magic are called metacharacters.
- If you want your regex to match them literally, you need to escape them by placing a backslash in front of them.
- Notably absent from the list are the closing square bracket ], the hyphen -, and the closing curly bracket }. The first two become metacharacters only after an unescaped [, and the } only after an unescaped {. There’s no need to ever escape }. Metacharacter rules for the blocks that appear between [ and ] are explained in Recipe2.3.
- By default, regular expressions are case sensitive

### 2.2 Match Nonprintable Characters

- Match a string of the following ASCII control characters: bell, escape, form feed, line feed, carriage return, horizontal tab, vertical tab. These characters have the hexadecimal ASCII codes 07, 1B, 0C, 0A, 0D, 09, 0B.
