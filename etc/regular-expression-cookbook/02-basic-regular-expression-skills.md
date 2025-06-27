# Chapter 2: Basic Regular Expression Skills

- when creating a regular expression, you’ll likely need it to match certain text literally, and you’ll need to know which characters to escape.

### 2.1 Match Literal Text

- Any regular expression that does not include any of the dozen characters `$()*+.?[\^{|` simply matches itself.
- The 12 punctuation characters that make regular expressions work their magic are called metacharacters.
- If you want your regex to match them literally, you need to escape them by placing a backslash in front of them.
- Notably absent from the list are the closing square bracket ], the hyphen -, and the closing curly bracket }. The first two become metacharacters only after an unescaped [, and the } only after an unescaped {. There’s no need to ever escape }. Metacharacter rules for the blocks that appear between [ and ] are explained in Recipe2.3.
- By default, regular expressions are case sensitive

### 2.2 Match Nonprintable Characters

- Match a string of the following ASCII control characters: bell, escape, form feed, line feed, carriage return, horizontal tab, vertical tab. These characters have the hexadecimal ASCII codes `07, 1B, 0C, 0A, 0D, 09, 0B`.

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
- A caret `(^)` negates the character class if you place it immediately after the opening bracket. It makes the character class match any character that is not in the list.
- In all the regex flavors discussed in this book, a negated character class matches line break characters, unless you add them to the negated character class. Make sure that you don’t accidentally allow your regex to span across lines.
- A hyphen `(-)` creates a range when it is placed between two characters. The range includes the character before the hyphen, the character after the hyphen, and all characters that lie between them in numerical order
- Reversed ranges, such as ‹[z-a]›, are not permitted.

Shorthands

- Six regex tokens that consist of a backslash and a letter form shorthand character classes: `‹\d›, ‹\D›, ‹\w›, ‹\W›, ‹\s› and ‹\S›`. You can use these both inside and outside character classes.
- `‹\d›` and `‹[\d]›` both match a single digit. ‹\D› matches any character that is not a digit, and is equivalent to ‹[^\d]›.
- `‹\w›` matches a single word character. A word character is a character that can occur as part of a word. That includes `letters, digits, and the underscore`
- `‹\W›` matches any character that is not part of such a propellerhead word.
- `‹\s›` matches any whitespace character. This includes spaces, tabs, and line breaks. ‹\S› matches any character not matched by ‹\s›
- `‹\b›` is not a shorthand character class, but a word boundary.

### 2.4 Match Any Character

Any character except line breaks

- `dot metacharacter`.
- The dot is one of the oldest and simplest regular expression features. Its meaning has always been to match any single character.
- All the other flavors discussed in this book followed suit. `‹.›` thus matches any single character except line break characters.

Any character including line breaks

- If you do want to allow your regular expression to span multiple lines, turn on the “dot matches line breaks” option. 
- `‹\s›` matches any whitespace character, whereas ‹`\S›` matches any character that is not matched by `‹\s›`. Combining these into `‹[\s\S]›` results in a character class that includes all characters, including line breaks. ‹[\d\D]› and ‹[\w\W]› have the same effect.

Dot abuse

- The dot is the most abused regular expression feature
- Inside a character class, the dot is just a literal character.
- Use the dot only when you really want to allow any character. Use a character class or negated character class in any other situation.
- Ruby uses ‹(?m)› to turn on “dot matches line breaks” mode

### 2.5 Match Something at the Start and/or the End of a Line

- The regular expression tokens `‹^›, ‹$›, ‹\A›, ‹\Z›, and ‹\z›` are called `anchors`. They do not match any characters. Instead, they match at certain positions, effectively anchoring the regular expression match at those positions.
- A line is the part of the subject text that lies between the start of the subject and a line break, between two line breaks, or between a line break and the end of the subject
- If there are no line breaks in the subject, then the whole subject is considered to be one line.

Start of the subject

- The anchor `‹\A›` always matches at the very start of the subject text, before the first character
- The anchor `‹^›` is equivalent to `‹\A›`,
- Unless you’re using JavaScript, we recommend that you always use `‹\A›` instead of `‹^›`. The meaning of ‹\A› never changes, avoiding any confusion or mistakes in setting regex options.

End of the subject

- Place `‹\Z›` or `‹\z›` at the end of your regular expression to test whether the subject text ends with the text you want to match.
- The difference between `‹\Z›` and `‹\z›` comes into play when the last character in your subject text is a line break. In that case, `‹\Z›` can match at the very end of the subject text, after the final line break, as well as immediately before that line break. The benefit is that you can search for ‹omega\Z› without having to worry about stripping off a trailing line break at the end of your subject text.
- `‹\z›` matches only at the very end of the subject text, so it will not match text if a trailing line break follows.
- The anchor `‹$›` is equivalent to `‹\Z›`, as long as you do not turn on the “^ and $ match at line breaks” option.
- This option is off by default for all regex flavors except Ruby. Ruby does not offer a way to turn this option off. Just like `‹\Z›`, `‹$›` matches at the very end of the subject text, as well as before the final line break, if any.
- Unless you’re using JavaScript, we recommend that you always use `‹\Z›` instead of `‹$›`. The meaning of `‹\Z›` never changes, avoiding any confusion or mistakes in setting regex options.

Start of a line

- By default, `‹^›` matches only at the start of the subject text, just like `‹\A›`. Only in Ruby does `‹^›` always match at the start of a line. All the other flavors require you to turn on the option to make the caret and dollar sign match at line breaks. This option is typically referred to as “multiline” mode.
- Do not confuse this mode with “single line” mode, which would be better known as “dot matches line breaks” mode. “Multiline” mode affects only the caret and dollar sign; “single line” mode affects only the dot, as Recipe.4 explains. It is perfectly possible to turn on both “single line” and “multiline” mode at the same time. By default, both options are off.
- With the correct option set, `‹^›` will match at the start of each line in the subject text. Strictly speaking, it matches before the very first character in the file, as it always does, and also after each line break character in the subject text. The caret in `‹\n^›` is redundant because `‹^›` always matches after `‹\n›`.

End of a line

- By default, `‹$›` matches only at the end of the subject text or before the final line break, just like `‹\Z›`. Only in Ruby does `‹$›` always match at the end of each line. All the other flavors require you to turn on the “multiline” option to make the caret and dollar match at line breaks.
- With the correct option set, `‹$›` will match at the end of each line in the subject text. (Of course, it also matches after the very last character in the text because that is always the end of a line as well.) The dollar in `‹$\n›` is redundant because `‹$›` always matches before `‹\n›`.

Zero-length matches

- It is perfectly valid for a regular expression to consist of nothing but one or more anchors. Such a regular expression will find a zero-length match at each position where the anchor can match. If you place several anchors together, all of them need to match at the same position for the regex to match.
- You could use such a regular expression in a search-and-replace. Replace `‹\A›` or `‹\Z›` to prepend or append something to the whole subject. Replace `‹^›` or `‹$›`, in “^ and $ match at line breaks” mode, to prepend or append something in each line in the subject text.
- Combine two anchors to test for blank lines or missing input. `‹\A\Z›` matches the empty string, as well as the string that consists of a single newline. `‹\A\z›` matches only the empty string. `‹^$›`, in “^ and $ match at line breaks” mode, matches each empty line in the subject text.

### 2.6 Match Whole Words

Word boundaries

- The regular expression token `‹\b›` is called a `word boundary`. It matches at the start or the end of a word. By itself, it results in a zero-length match. `‹\b›` is an `anchor`, just like the tokens introduced in the previous section.
- Strictly speaking, `‹\b›` matches in these three positions:
  - Before the first character in the subject, if the first character is a word character
  - After the last character in the subject, if the last character is a word character
  - Between two characters in the subject, where one is a word character and the other is not a word character
- To run a “whole words only” search using a regular expression, simply place the word between two word boundaries, as we did with ‹\bcat\b›. The first `‹\b›` requires the ‹c› to occur at the very start of the string, or after a nonword character. The second `‹\b›` requires the ‹t› to occur at the very end of the string, or before a nonword character.
- Line break characters are nonword characters. `‹\b›` will match after a line break if the line break is immediately followed by a word character. It will also match before a line break immediately preceded by a word character. So a word that occupies a whole line by itself will be found by a “whole words only” search. 
- `‹\b›` is unaffected by “multiline” mode or ‹(?m)›, which is one of the reasons why this book refers to “multiline” mode as “^ and $ match at line breaks” mode.
- The tokens before or after the `‹\b›` in your regular expression will determine where `‹\b›` can match. The `‹\b›` in ‹\bx› and ‹!\b› could match only at the start of a word. The `‹\b›` in ‹x\b› and ‹\b!› could match only at the end of a word. ‹x\bx› and ‹!\b!› can never match anywhere.
- If you really want to match only the position before a word or only after a word, you can do so with lookahead and lookbehind. Recipe2.16 explains lookahead and lookbehind
- This method does not work with JavaScript and Ruby 1.8 because these flavors do not support lookbehind.

Nonboundaries

- `‹\B›` matches at every position in the subject text where `‹\b›` does not match. `‹\B›` matches at every position that is not at the start or end of a word.
- Strictly speaking, `‹\B›` matches in these five positions:
  - Before the first character in the subject, if the first character is not a word character
  - After the last character in the subject, if the last character is not a word character
  - Between two word characters
  - Between two nonword characters
  - The empty string
- ‹\Bcat\B› matches cat in staccato, but not in My cat is brown, category, or bobcat.
- To do the opposite of a “whole words only” search (i.e., excluding My cat is brown and including staccato, category, and bobcat), you need to use alternation to combine ‹\Bcat› and ‹cat\B› into ‹\Bcat|cat\B›. ‹\Bcat› matches cat in staccato and bobcat. ‹cat\B› matches cat in category (and staccato if ‹\Bcat› hadn’t already taken care of that). Recipe.8 explains alternation.

Word Characters

- A word character is a character that can occur as part of a word. The subsection Shorthands in Recipe2.3 discussed which characters are included in `‹\w›`, which matches a single word character. Unfortunately, the story is not the same for `‹\b›`.
- Although all the flavors in this book support `‹\b›` and `‹\B›`, they differ in which characters are word characters.
- .NET, JavaScript, PCRE, Perl, Python, and Ruby have `‹\b›` match between two characters where one is matched by `‹\w›` and the other by `‹\W›`. `‹\B›` always matches between two characters where both are matched by `‹\w›` or `‹\W›`.
- JavaScript, PCRE, and Ruby view only ASCII characters as word characters. `‹\w›` is identical to `‹[a-zA-Z0-9_]›`. With these flavors, you can do a “whole words only” search on words in languages that use only the letters A to Z without diacritics, such as English. But these flavors cannot do “whole words only” searches on words in other languages, such as Spanish or Russian.

### 2.7 Unicode Code Points, Categories, Blocks, and Scripts

Unicode code point

- A code point is one entry in the Unicode character database. A code point is not the same as a character, depending on the meaning you give to “character.” What appears as a character on screen is called a grapheme in Unicode.
- The Unicode code point U+2122 represents the “trademark sign” character. You can match this with `‹\u2122›`, `‹\u{2122}›`, or `‹\x{2122}›`, depending on the regex flavor you’re working with.
- The `‹\u›` syntax requires exactly four hexadecimal digits. This means you can only use it for Unicode code points U+0000 through U+FFFF.
- `‹\u{⋯}›` and `‹\x{⋯}›` allow between one and six hexadecimal digits between the braces, supporting all code points U+000000 through U+10FFFF.
- Code points can be used inside and outside character classes.

Unicode category

- Each Unicode code point fits into a single Unicode category. There are 30 Unicode categories, specified with a code consisting of two letters. These are grouped into 7 super-categories that are specified with a single letter.

Unicode block

- The Unicode character database divides all the code points into blocks. Each block consists of a single range of code points. The code points U+0000 through U+FFFF are divided into 156 blocks in version 6.1 of the Unicode standard:
-  Unicode block is a single, contiguous range of code points. Although many blocks have the names of Unicode scripts and Unicode categories, they do not correspond 100% with them. The name of a block only indicates its primary use.

Unicode script

- Each Unicode code point, except unassigned ones, is part of exactly one Unicode script. Unassigned code points are not part of any script. The assigned code points up to U+FFFF are assigned to these 72 scripts in version 6.1 of the Unicode standard:
- A script is a group of code points used by a particular human writing system.

Unicode grapheme

- The difference between code points and characters comes into play when there are combining marks. The Unicode code point U+0061 is “Latin small letter a,” whereas U+00E0 is “Latin small letter a with grave accent.” Both represent what most people would describe as a character.
- U+0300 is the “combining grave accent” combining mark. It can be used sensibly only after a letter. A string consisting of the Unicode code points U+0061 U+0300 will be displayed as à, just like U+00E0. The combining mark U+0300 is displayed on top of the character U+0061.
- What matters to you as a regex user is that all regex flavors discussed in this book operate on code points rather than graphical characters. When we say that the regular expression ‹.› matches a single character, it really matches just a single code point. If your subject text consists of the two code points U+0061 U+0300, which can be represented as the string literal "\u0061\u0300" in a programming language such as Java, the dot will match only the code point U+0061, or a, without the accent U+0300. The regex ‹..› will match both.
- The rules for exactly which combinations of Unicode code points are considered graphemes are quite complicated

### 2.8 Match One of Several Alternatives

- The vertical bar, or pipe symbol, splits the regular expression into multiple alternatives. ‹Mary|Jane|Sue› matches Mary, or Jane, or Sue with each match attempt. Only one name matches each time, but a different name can match each time.
- All regular expression flavors discussed in this book use a regex-directed engine.
- Regex-directed[3] means that all possible permutations of the regular expression are attempted at each character position in the subject text, before the regex is attempted at the next character position.
- The regular expression finds the leftmost match. It scans the text from left to right, tries all alternatives in the regular expression at each step, and stops at the first position in the text where any of the alternatives produces a valid match.
- The order of the alternatives in the regex matters only when two of them can match at the same position in the string. 
  - The regex ‹Jane|Janet› has two alternatives that match at the same position in the text Her name is Janet. There are no word boundaries in the regular expression. The fact that ‹Jane› matches the word Janet in Her name is Janet only partially does not matter.
  - ‹Jane|Janet› matches Jane in Her name is Janet because a regex-directed regular expression engine is eager. In addition to scanning the subject text from left to right, finding the leftmost match in the text, it also scans the alternatives in the regex from left to right. The engine stops as soon as it finds an alternative that matches.
  - When ‹Jane|Janet› reaches the J in Her name is Janet, the first alternative, ‹Jane›, matches. The second alternative is not attempted. If we tell the engine to look for a second match, the t is all that is left of the subject text. Neither alternative matches there.
  - There are two ways to stop Jane from stealing Janet’s limelight. One way is to put the longer alternative first: ‹Janet|Jane›. A more solid solution is to be explicit about what we’re trying to do: we’re looking for names, and names are complete words. Regular expressions don’t deal with words, but they can deal with word boundaries.
  - So ‹\bJane\b|\bJanet\b› and ‹\bJanet\b|\bJane\b› will both match Janet in Her name is Janet. Because of the word boundaries, only one alternative can match. The order of the alternatives is again irrelevant.

### 2.9 Group and Capture Parts of the Match

- The alternation operator, explained in the previous section, has the lowest precedence of all regex operators. If you try ‹\bMary|Jane|Sue\b›, the three alternatives are ‹\bMary›, ‹Jane›, and ‹Sue\b›. This regex matches Jane in Her name is Janet.
- If you want something in your regex to be excluded from the alternation, you have to group the alternatives. Grouping is done with parentheses
- They have the highest precedence of all regex operators, just as in most programming languages. ‹\b(Mary|Jane|Sue)\b› has three alternatives—‹Mary›, ‹Jane›, and ‹Sue›—between two word boundaries. This regex does not match anything in Her name is Janet.
- A pair of parentheses isn’t just a group; it’s a `capturing group`. 
- The regex ‹\b(\d\d\d\d)-(\d\d)-(\d\d)\b› has three capturing groups. Groups are numbered by counting opening parentheses from left to right. ‹(\d\d\d\d)› is group number 1. ‹(\d\d)› is number 2. The second ‹(\d\d)› is group number 3.

Noncapturing groups

- In the regex ‹\b(Mary|Jane|Sue)\b›, we need the parentheses for grouping only. Instead of using a capturing group, we could use a noncapturing group: \b(?:Mary|Jane|Sue)\b
- The noncapturing group provides the same grouping functionality, but does not capture anything.
- Another benefit of noncapturing groups is performance. If you’re not going to use a backreference to a particular group (Recipe2.10), reinsert it into the replacement text (Recipe2.21), or retrieve its match in source code (Recipe3.9), a capturing group adds unnecessary overhead that you can eliminate by using a noncapturing group. In practice, you’ll hardly notice the performance difference, unless you’re using the regex in a tight loop and/or on lots of data.


### 2.10 Match Previously Matched Text Again

- To match previously matched text later in a regex, we first have to capture the previous text. We do that with a capturing group, as shown in Recipe2.9. After that, we can match the same text anywhere in the regex using a backreference. You can reference the first nine capturing groups with a backslash followed by a single digit one through nine. For groups 10 through 99, use ‹\10› to ‹\99›.
  - example: `\b\d\d(\d\d)-\1-\1\b`
- If a capturing group is repeated, either by a quantifier (Recipe2.12) or by backtracking (Recipe2.13), the stored match is overwritten each time the capturing group matches something. A backreference to the group matches only the text that was last captured by the group.
- Because the regex engine proceeds from start to end, you should put the capturing parentheses before the backreference. The regular expressions ‹\b\d\d\1-(\d\d)-\1› and ‹\b\d\d\1-\1-(\d\d)\b› could never match anything. Since the backreference is encountered before the capturing group, it has not captured anything yet. Unless you’re using JavaScript, a backreference always fails if it points to a group that hasn’t already participated in the match attempt.
- JavaScript is the only flavor we know that goes against decades of backreference tradition in regular expressions. In JavaScript, or at least in implementations that follow the JavaScript standard, a backreference to a group that hasn’t participated always succeeds, just like a backreference to a group that captured a zero-length match. So, in JavaScript, ‹\b\d\d\1-\1-(\d\d)\b› can match 12--34.

### 2.11 Capture and Name Parts of the Match

- example: 
  - `\b(?<year>\d\d\d\d)-(?<month>\d\d)-(?<day>\d\d)\b`
  - \b\d\d(?<magic>\d\d)-\k<magic>-\k<magic>\b

Named capture

- Recipes 2.9 and 2.10 illustrate capturing groups and backreferences. To be more precise: these recipes use numbered capturing groups and numbered backreferences. Each group automatically gets a number, which you use for the backreference.
- Modern regex flavors support named capturing groups in addition to numbered groups. The only difference between named and numbered groups is your ability to assign a descriptive name, instead of being stuck with automatic numbers. Named groups make your regular expression more readable and easier to maintain. Inserting a capturing group into an existing regex can change the numbers assigned to all the capturing groups. Names that you assign remain the same.

Named backreferences

- Just as named capturing groups are functionally identical to numbered capturing groups, named backreferences are functionally identical to numbered backreferences. They’re just easier to read and maintain.

### 2.12 Repeat Part of the Regex a Certain Number of Times