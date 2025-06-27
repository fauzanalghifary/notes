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

Fixed repetition

- The quantifier `‹{n}›`, where n is a nonnegative integer, repeats the preceding regex token n number of times. The ‹\d{100}› in ‹\b\d{100}\b› matches a string of 100 digits. You could achieve the same by typing ‹\d› 100 times.
- ‹{1}› repeats the preceding token once, as it would without any quantifier. ‹ab{1}c› is the same regex as ‹abc›.
- ‹{0}› repeats the preceding token zero times, essentially deleting it from the regular expression. ‹ab{0}c› is the same regex as ‹ac›.

Variable repetition

- For variable repetition, we use the quantifier `‹{n,m}›`, where n is a nonnegative integer and m is greater than n. ‹\b[a-f0-9]{1,8}\b› matches a hexadecimal number with one to eight digits.
- If n and m are equal, we have fixed repetition. ‹\b\d{100,100}\b› is the same regex as ‹\b\d{100}\b›.

Infinite repetition

- The quantifier `‹{n,}›`, where n is a nonnegative integer, allows for infinite repetition. Essentially, infinite repetition is variable repetition without an upper limit.
- `‹\d{1,}›` matches one or more digits, and `‹\d+›` does the same. A plus after a regex token that’s not a quantifier means “one or more.” Recipe2.13 shows the meaning of a plus after a quantifier.
- `‹\d{0,}›` matches zero or more digits, and `‹\d*›` does the same. The asterisk always means “zero or more.” In addition to allowing infinite repetition, ‹{0,}› and the asterisk also make the preceding token optional.

Making something optional

- If we use variable repetition with n set to zero, we’re effectively making the token that precedes the quantifier optional. 
- `‹h{0,1}›` matches the ‹h› once or not at all. If there is no h, ‹h{0,1}› results in a zero-length match. If you use ‹h{0,1}› as a regular expression all by itself, it will find a zero-length match before each character in the subject text that is not an h. Each h will result in a match of one character (the h).
- `‹h?›` does the same as `‹h{0,1}›`. 

Repeating groups

- If you place a quantifier after the closing parenthesis of a group, the whole group is repeated. 
  - ‹(?:abc){3}› is the same as ‹abcabcabc›.
- Quantifiers can be nested.
  - ‹(e\d+)?› matches an e followed by one or more digits, or a zero-length match.
- Capturing groups can be repeated.
  - As explained in Recipe2.9, the group’s match is captured each time the engine exits the group, overwriting any text previously matched by the group
  - ‹(\d\d){1,3}› matches a string of two, four, or six digits. The engine exits the group three times. When this regex matches 123456, the capturing group will hold 56, because 56 was stored by the last iteration of the group. The other two matches by the group, 12 and 34, cannot be retrieved.
  - ‹(\d\d){3}› captures the same text as ‹\d\d\d\d(\d\d)›
  - If you want the capturing group to capture all two, four, or six digits rather than just the last two, you have to place the capturing group around the quantifier instead of repeating the capturing group: ‹((?:\d\d){1,3})›. Here we used a noncapturing group to take over the grouping function from the capturing group.
  - We also could have used two capturing groups: ‹((\d\d){1,3})›. When this last regex matches 123456, ‹\1› holds 123456 and ‹\2› holds 56.

### 2.13 Choose Minimal or Maximal Repetition

- All the quantifiers discussed in Recipe2.12 are greedy, meaning they try to repeat as many times as possible, giving back only when required to allow the remainder of the regular expression to match.
- Regex: `<p>.*</p>`
  - The asterisk repeats it zero or more times. The asterisk is greedy, and so `‹.*›` matches everything all the way to the end of the subject text. 
  - Let me say that again: `‹.*›` eats up your whole XHTML file, starting with the first paragraph.
  - When the ‹.*› has its belly full, the engine attempts to match the ‹<› at the end of the subject text. That fails. But it’s not the end of the story: the regex engine backtracks.
  - The asterisk prefers to grab as much text as possible, but it’s also perfectly satisfied to match nothing at all (zero repetitions). With each repetition of a quantifier beyond the quantifier’s minimum, the regular expression stores a backtracking position. Those are positions the engine can go back to, in case the part of the regex following the quantifier fails.
  - When ‹<› fails, the engine backtracks by making the `‹.*›` give up one character of its match. Then ‹<› is attempted again, at the last character in the file. If it fails again, the engine backtracks once more, attempting ‹<› at the second-to-last character in the file. This process continues until ‹<› succeeds. If ‹<› never succeeds, the `‹.*›` eventually runs out of backtracking positions and the overall match attempt fails.
  - If ‹<› does match at some point during all that backtracking, ‹/› is attempted. If ‹/› fails, the engine backtracks again. This repeats until ‹</p>› can be matched entirely.
  - So what’s the problem? Because the asterisk is greedy, the incorrect regular expression matches everything from the first <p> in the XHTML file to the last </p>. But to correctly match an XHTML paragraph, we need to match the first <p> with the first </p> that follows it.
- That’s where lazy quantifiers come in. You can make any quantifier lazy by placing a question mark after it: `‹*?›, ‹+?›, ‹??›, and ‹{7,42}?›` are all `lazy quantifiers`.
  - Lazy quantifiers backtrack too, but the other way around. A lazy quantifier repeats as few times as it has to, stores one backtracking position, and allows the regex to continue. If the remainder of the regex fails and the engine backtracks, the lazy quantifier repeats once more. If the regex keeps backtracking, the quantifier will expand until its maximum number of repetitions, or until the regex token it repeats fails to match.
- `‹<p>.*?</p>›`
  - `‹<p>.*?</p>›` uses a lazy quantifier to correctly match an XHTML paragraph. When ‹<p>› matches, the `‹.*?›`, lazy as it is, initially does nothing but procrastinate. If ‹</p>› immediately occurs after <p>, an empty paragraph is matched. If not, the engine backtracks to `‹.*?›`, which matches one character. If ‹</p>› still fails, `‹.*?›` matches the next character. This continues until either ‹</p>› succeeds or `‹.*?›` fails to expand. Since the dot matches everything, failure won’t occur until the `‹.*?›` has matched everything up to the end of the XHTML file.
- The quantifiers `‹*›` and `‹*?›` allow all the same regular expression matches. The only difference is the order in which the possible matches are tried. The greedy quantifier will find the longest possible match. The lazy quantifier will find the shortest possible match.
- If possible, the best solution is to make sure there is only one possible match.
- It may help to understand the operation of greedy and lazy repetition by comparing how `‹\d+\b›` and `‹\d+?\b›` act on a couple of different subject texts. The greedy and lazy versions produce the same results, but test the subject text in a different order.
  - If we use ‹\d+\b› on 1234, ‹\d+› will match all the digits. ‹\b› then matches, and an overall match is found. If we use ‹\d+?\b›, ‹\d+?› first matches only 1. ‹\b› fails between 1 and 2. ‹\d+?› expands to 12, and ‹\b› still fails. This continues until ‹\d+?› matches 1234, and ‹\b› succeeds.
  - If our subject text is 1234X, the first regex, ‹\d+\b›, still has ‹\d+› match 1234. But then ‹\b› fails. ‹\d+› backtracks to 123. ‹\b› still fails. This continues until ‹\d+› has backtracked to its minimum 1, and ‹\b› still fails. Then the whole match attempt fails.
  - If we use ‹\d+?\b› on 1234X, ‹\d+?› first matches only 1. ‹\b› fails between 1 and 2. ‹\d+?› expands to 12. ‹\b› still fails. This continues until ‹\d+?› matches 1234, and ‹\b› still fails. The regex engine attempts to expand ‹\d+?› once more, but ‹\d› does not match X. The overall match attempt fails.

### 2.14 Eliminate Needless Backtracking 

- The previous recipe explains the difference between greedy and lazy quantifiers, and how they backtrack. In some situations, this backtracking is unnecessary.
- ‹\b\d+\b› uses a greedy quantifier, and ‹\b\d+?\b› uses a lazy quantifier. They both match the same thing: an integer. Given the same subject text, both will find the exact same matches. Any backtracking that is done is unnecessary.
- The easiest solution is to use a possessive quantifier
  - `\b\d++\b`
- An atomic group provides exactly the same functionality, using a slightly less readable syntax. Support for atomic grouping is more widespread than support for possessive quantifiers.
  - `\b(?>\d+)\b`
- JavaScript and Python do not support possessive quantifiers or atomic grouping. There is no way to eliminate needless backtracking with these two regex flavors
- A possessive quantifier is similar to a greedy quantifier: it tries to repeat as many times as possible. The difference is that a possessive quantifier will never give back, not even when giving back is the only way that the remainder of the regular expression could match. Possessive quantifiers do not keep backtracking positions.
  - You can make any quantifier possessive by placing a plus sign after it. For example, `‹*+›, ‹++›, ‹?+›, and ‹{7,42}+›` are all `possessive quantifiers`.
- Wrapping a greedy quantifier inside an atomic group has the exact same effect as using a possessive quantifier. When the regex engine exits the atomic group, all backtracking positions remembered by quantifiers and alternation inside the group are thrown away. 
  - The syntax is `‹(?>⋯)›`, where ‹⋯› is any regular expression. An atomic group is essentially a noncapturing group, with the extra job of refusing to backtrack. The question mark is not a quantifier; the opening bracket simply consists of the three characters ‹(?>›.
- We describe the possessive quantifier as failing to remember backtracking positions, and the atomic group as throwing them away. This makes it easier to understand the matching process, but don’t get hung up on the difference, as it may not even exist in the regex flavor you’re working with. In many flavors, ‹x++› is merely syntactic sugar for ‹(?>x+)›, and both are implemented in exactly the same way. Whether the engine never remembers backtracking positions or throws them away later is irrelevant for the final outcome of the match attempt.
- Where possessive quantifiers and atomic grouping differ is that a possessive quantifier applies only to a single regular expression token, whereas an atomic group can wrap a whole regular expression.
- Within the atomic group, backtracking occurs normally. Backtracking positions are thrown away only when the engine exits the whole group.
- Possessive quantifiers and atomic grouping don’t just optimize regular expressions. They can alter the matches found by a regular expression by eliminating those that would be reached through backtracking.
- This recipe shows how to use possessive quantifiers and atomic grouping to make minor optimizations, which may not even show any difference in benchmarks. The next recipe will showcase how atomic grouping can make a dramatic difference.

### 2.15 Prevent Runaway Repetition

- Example use case: Use a single regular expression to match a complete HTML file, checking for properly nested html, head, title, and body tags. The regular expression must fail efficiently on HTML files that do not have the proper tags.
  - `<html>(?>.*?<head>)(?>.*?<title>)(?>.*?</title>)↵
(?>.*?</head>)(?>.*?<body[^>]*>)(?>.*?</body>).*?</html>`
- JavaScript and Python do not support atomic grouping. There is no way to eliminate needless backtracking with these two regex flavors. 
  - When programming in JavaScript or Python, you can solve this problem by doing a literal text search for each of the tags one by one, searching for the next tag through the remainder of the subject text after the one last found.
    - `<html>.*?<head>.*?<title>.*?</title>↵
  .*?</head>.*?<body[^>]*>.*?</body>.*?</html>`
- We call this catastrophic backtracking. So much backtracking occurs that the regex either takes forever or crashes your application. Some regex implementations are clever and will abort runaway match attempts early, but even then the regex will still kill your application’s performance.
  - Catastrophic backtracking is an instance of a phenomenon known as a combinatorial explosion, in which several orthogonal conditions intersect and all combinations have to be tried. You could also say that the regex is a Cartesian product of the various repetition operators.
- The solution is to use atomic grouping to prevent needless backtracking

### 2.16 Test for a Match Without Adding It to the Overall Match

- Example use case: Find any word that occurs between a pair of HTML bold tags, without including the tags in the regex match. For instance, if the subject is My <b>cat</b> is furry, the only valid match should be cat.
  - `(?<=<b>)\w+(?=</b>)`
- JavaScript and Ruby 1.8 support the lookahead `‹(?=</b>)›`, but not the lookbehind `‹(?<=<b>)›`.
  - WAIT, JS SUPPORT LOOKBEHIND SINCE ES2018
  - THIS BOOK IS FROM 2012!!!

Lookaround

- The four kinds of lookaround groups supported by modern regex flavors have the special property of giving up the text matched by the part of the regex inside the lookaround. Essentially, lookaround checks whether certain text can be matched without actually matching it.
- Lookaround that looks backward is called `lookbehind`. This is the only regular expression construct that will traverse the text from right to left instead of from left to right.
  - The syntax for positive lookbehind is `‹(?<=⋯)›`. The four characters `‹(?<=›` form the opening bracket
  - What you can put inside the lookbehind, here represented by ‹⋯›, varies among regular expression flavors. But simple literal text, such as ‹(?<=<b>)›, always works.
  - Lookbehind checks to see whether the text inside the lookbehind occurs immediately to the left of the position that the regular expression engine has reached
  - If you match `‹(?<=<b>)›` against `My <b>cat</b> is furry`, the lookbehind will fail to match until the regular expression starts the match attempt at the letter c in the subject. The regex engine then enters the lookbehind group, telling it to look to the left. ‹<b>› matches to the left of c. The engine exits the lookbehind at this point, and discards any text matched by the lookbehind from the match attempt.
  - In other words, the match-in-progress is back at where it was when the engine entered the lookbehind. In this case, the match-in-progress is the zero-length match before the c in the subject string. The lookbehind only tests or asserts that ‹<b>› can be matched; it does not actually match it. Lookaround constructs are therefore called zero-length assertions.
- lookaround can match something but can never consume anything.
- Lookaround that looks forward, in the same direction that the regular expression normally traverses the text, is called `lookahead`.
  - Lookahead is equally supported by all regex flavors in this book
  - The syntax for positive lookahead is `‹(?=⋯)›`. The three characters ‹(?=› form the opening bracket of the group
  - Everything you can use in a regular expression can be used inside lookahead, here represented by ‹⋯›.
  - When the ‹\w+› in ‹(?<=<b>)\w+(?=</b>)› has matched cat in My <b>cat</b> is furry, the regex engine enters the lookahead. The only special behavior for the lookahead at this point is that the regex engine remembers which part of the text it has matched so far, associating it with the lookahead. ‹</b>› is then matched normally. Now the regex engine exits the lookahead. The regex inside the lookahead matches, so the lookahead itself matches. The regex engine discards the text matched by the lookahead, by restoring the match-in-progress it remembered when entering the lookahead. Our overall match-in-progress is back at cat. Since this is also the end of our regular expression, cat becomes the final match result.

Negative Lookaround

- `‹(?!⋯)›`, with an exclamation point instead of an equals sign, is negative lookahead. Negative lookahead works just like positive lookahead, except that whereas positive lookahead matches when the regex inside the lookahead matches, negative lookahead matches when the regex inside the lookahead fails to match.
- Similarly, `(?<!⋯)`is negative lookbehind. Negative lookbehind matches when none of the alternatives inside the lookbehind can be found looking backward from the position the regex has reached in the subject text.

Different levels of lookbehind

- Lookahead is easy. All regex flavors discussed in this book allow you to put a complete regular expression inside the lookahead. Everything you can use in a regular expression can be used inside lookahead. You can even nest other lookahead and lookbehind groups inside lookahead. Your brain might get into a twist, but the regex engine will handle everything nicely.
- Lookbehind is a different story. Regular expression software has always been designed to search the text from left to right only. Searching backward is often implemented as a bit of a hack: the regex engine determines how many characters you put inside the lookbehind, jumps back that many characters, and then compares the text in the lookbehind with the text in the subject from left to right.
- If all this sounds rather inefficient, it is. Lookbehind is very convenient, but it won’t break any speed records. Later, we present a solution for JavaScript and Ruby 1.8, which don’t support lookbehind at all. This solution is actually far more efficient than using lookbehind.

Matching the same text twice

- If you use lookbehind at the start of the regex or lookahead at the end of the regex, the net effect is that you’re requiring something to appear before or after the regex match, without including it in the match. If you use lookaround in the middle of your regular expression, you can apply multiple tests to the same text.

Lookaround is atomic

- When the regular expression engine exits a lookaround group, it discards the text matched by the lookaround. Because the text is discarded, any backtracking positions remembered by alternation or quantifiers inside the lookaround are also discarded. This effectively makes lookahead and lookbehind atomic
- In most situations, the atomic nature of lookaround is irrelevant. A lookaround is merely an assertion to check whether the regex inside the lookaround matches or fails. How many different ways it can match is irrelevant, as it does not consume any part of the subject text.
- The atomic nature comes into play only when you use capturing groups inside lookahead (and lookbehind, if your regex flavor allows you to). While the lookahead does not consume any text, the regex engine will remember which part of the text was matched by any capturing groups inside the lookahead. If the lookahead is at the end of the regex, you will indeed end up with capturing groups that match text not matched by the regular expression itself. If the lookahead is in the middle of the regex, you can end up with capturing groups that match overlapping parts of the subject text.
- The only situation in which the atomic nature of lookaround can alter the overall regex match is when you use a backreference outside the lookaround to a capturing group created inside the lookaround.
  - `(?=(\d+))\w+\1`
  - it wouldn't match `123x12`

Alternative to Lookbehind

- `<b>\K\w+(?=</b>)`
  - for: PCRE 7.2, Perl 5.10

Solution Without Lookbehind

- There’s no way to solve the problem as stated with these regex flavors, but you can work around the need for lookbehind by using capturing groups.
  - `(<b>)(\w+)(?=</b>)`

### 2.17 Match One of Two Alternatives Based on a Condition

- Example use case: 
  - Create a regular expression that matches a comma-delimited list of the words one, two, and three. Each word can occur any number of times in the list, and the words can occur in any order, but each word must appear at least once.
  - `\b(?:(?:(one)|(two)|(three))(?:,|\b)){3,}(?(1)|(?!))(?(2)|(?!))(?(3)|(?!))`
  - Java, JavaScript, and Ruby do not support conditionals. When programming in these languages (or any other language), you can use the regular expression without the conditionals, and write some extra code to check if each of the three capturing groups matched something.
    - `\b(?:(?:(one)|(two)|(three))(?:,|\b)){3,}`
- .NET, PCRE, Perl, and Python support conditionals using numbered capturing groups
- `‹(?(1)then|else)›` is a conditional that checks whether the first capturing group has already matched something. If it has, the regex engine attempts to match ‹then›. If the capturing group has not participated in the match attempt thus far, the ‹else› part is attempted.
- The parentheses, question mark, and vertical bar are all part of the syntax for the conditional. They don’t have their usual meaning. You can use any kind of regular expression for the ‹then› and ‹else› parts. The only restriction is that if you want to use alternation for one of the parts, you have to use a group to keep it together. Only one vertical bar is permitted directly in the conditional.
- To better understand how conditionals work, let’s examine the regular expression `‹(a)?b(?(1)c|d)›`. This is essentially a complicated way of writing `‹abc|bd›`.
  - In English, `‹(a)?b(?(1)c|d)›` either matches `ab followed by c`, or matches `b followed by d`.

### 2.18 Add Comments to a Regular Expression

Free-spacing mode

- Regular expressions can quickly become complicated and difficult to understand. Just as you should comment source code, you should comment all but the most trivial regular expressions.
- All regular expression flavors in this book, except JavaScript, offer an alternative regular expression syntax that makes it very easy to clearly comment your regular expressions. You can enable this syntax by turning on the free-spacing option. It has different names in various programming languages.

### 2.19 Insert Literal Text into the Replacement Text

- Example: Search and replace any regular expression match literally with the eight characters `$%\*$1\1`.
  - `$%\*$$1\1` (.NET, JavaScript)
  - `$%\*$1\\1` (Python, Ruby)

When and how to escape characters in replacement text

- This recipe shows you the different escape rules used by the various replacement text flavors. The only two characters you may ever need to escape in the replacement text are the dollar sign and the backslash. The escape characters are also the dollar sign and the backslash.
- The fact that this problem has five different solutions for seven replacement text flavors demonstrates that there really is no standard for replacement text syntax.

.NET and JavaScript

- .NET and JavaScript always treat a backslash as a literal character. Do not escape it with another backslash, or you’ll end up with two backslashes in the replacement.
- A lone dollar sign is a literal character. Dollar signs need to be escaped only when they are followed by a digit, ampersand, backtick, straight quote, underscore, plus sign, or another dollar sign. To escape a dollar sign, precede it with another dollar sign. You can double up all dollar signs if you feel that makes your replacement text more readable. This solution is equally valid:

Python and Ruby

- The dollar sign has no special meaning in the replacement text in Python and Ruby. Backslashes need to be escaped with another backslash when followed by a character that gives the backslash a special meaning.
- For Ruby, you need to escape a backslash followed by a digit, ampersand, backtick, straight quote, or plus sign.
- In both languages, a backslash also escapes another backslash. Thus, you need to write «\\\\» to include two literal backslashes in replacement text. All other backslashes are treated as literal backslashes.

More escape rules for string literals

- Remember that in this chapter, we deal only with the regular expressions and replacement text themselves. The next chapter covers programming languages and string literals.
- In other words, if your application provides a text box for the user to type in the replacement text, these solutions show what the user would have to type in order for the search-and-replace to work as intended. If you test your search-and-replace commands with RegexBuddy or another regex tester, the replacement texts included in this recipe will show the expected results.
- String literals in programming languages have their own escape rules, and you need to follow those rules on top of the replacement text escape rules. You may indeed end up with a mess of backslashes.