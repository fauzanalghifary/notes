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

### 2.3 Match One of Many Characters

- a character class.
- `c[ae]l[ae]nd[ae]r`
- `[a-fA-F0-9]`
- The notation using square brackets is called a character class. A character class matches a single character out of a list of possible characters
- Inside a character class, only four characters have a special function: `\, ^, -, and ]`
- A backslash always escapes the character that follows it, just as it does outside character classes
- The other four metacharacters get their special meanings only when they’re placed in a certain position. It is possible to include them as literal characters in a character class without escaping them, by positioning them in a way that they don’t get their special meaning.
- But we recommend that you always escape these metacharacters
- Escaping the metacharacters makes your regular expression easier to understand.
- A caret (^) negates the character class if you place it immediately after the opening bracket. It makes the character class match any character that is not in the list.
- In all the regex flavors discussed in this book, a negated character class matches line break characters, unless you add them to the negated character class. Make sure that you don’t accidentally allow your regex to span across lines.
- A hyphen (-) creates a range when it is placed between two characters. The range includes the character before the hyphen, the character after the hyphen, and all characters that lie between them in numerical order
- Reversed ranges, such as ‹[z-a]›, are not permitted.
- Six regex tokens that consist of a backslash and a letter form shorthand character classes: ‹\d›, ‹\D›, ‹\w›, ‹\W›, ‹\s› and ‹\S›. You can use these both inside and outside character classes.
- ‹\d› and ‹[\d]› both match a single digit. ‹\D› matches any character that is not a digit, and is equivalent to ‹[^\d]›.
- ‹\w› matches a single word character. A word character is a character that can occur as part of a word. That includes letters, digits, and the underscore
- ‹\W› matches any character that is not part of such a propellerhead word.
- ‹\s› matches any whitespace character. This includes spaces, tabs, and line breaks. ‹\S› matches any character not matched by ‹\s›
- ‹\b› is not a shorthand character class, but a word boundary.

### 2.4 Match Any Character

- dot metacharacter.
- The dot is one of the oldest and simplest regular expression features. Its meaning has always been to match any single character.
- All the other flavors discussed in this book followed suit. ‹.› thus matches any single character except line break characters.
- If you do want to allow your regular expression to span multiple lines, turn on the “dot matches line breaks” option. 
- ‹\s› matches any whitespace character, whereas ‹\S› matches any character that is not matched by ‹\s›. Combining these into ‹[\s\S]› results in a character class that includes all characters, including line breaks. ‹[\d\D]› and ‹[\w\W]› have the same effect.
- The dot is the most abused regular expression feature
- Inside a character class, the dot is just a literal character.
- Use the dot only when you really want to allow any character. Use a character class or negated character class in any other situation.
- Ruby uses ‹(?m)› to turn on “dot matches line breaks” mode

### 2.5 Match Something at the Start and/or the End of a Line