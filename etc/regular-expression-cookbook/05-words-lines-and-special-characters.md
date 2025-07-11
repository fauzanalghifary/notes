# Chapter 5: Words, Lines, and Special Characters

### 5.0. Intro

- This chapter contains recipes that deal with finding and manipulating text in a variety of contexts
- The central theme of this chapter is showing a variety of regular expression constructs and techniques in action. 
- Reading through it is like a workout for a large number of regular expression syntax features, and will help you apply regular expressions generally to the problems you encounter. In many cases, what we search for is simple, but the templates we provide in the solutions allow you to customize them for the specific problems you’re facing.

### 5.1. Find a Specific Word

- You’re given the simple task of finding all occurrences of the word cat, case insensitively. The catch is that it must appear as a complete word. You don’t want to find pieces of longer words, such as hellcat, application, or Catwoman.
- `\bcat\b`
- the `word boundaries` require that cat is set apart from other text by the beginning or end of the string, whitespace, punctuation, or other nonword characters.
- Regular expression engines consider letters, numbers, and underscores to all be word characters. Recipe2.6 is where we first talked about word boundaries, and covers them in greater detail.
- A problem can occur when working with international text in JavaScript, PCRE, and Ruby, since those regular expression flavors only consider letters in the ASCII table to create a word boundary
- This prevents ‹\b› from being useful for a “whole word only” search within text that contains accented letters or words that use non-Latin scripts. For example, in JavaScript, PCRE, and Ruby, ‹\büber\b› will find a match within darüber, but not within dar über. In most cases, this is the exact opposite of what you would want.
- You can deal with this problem by using lookahead and lookbehind (collectively, lookaround—see Recipe 2.16) instead of word boundaries

### 5.2. Find Any of Multiple Words

- You want to find any one out of a list of words, without having to search through the subject string multiple times.
- `\b(?:one|two|three)\b`
- There are three parts to this regular expression: the word boundaries on both ends, the noncapturing group, and the list of words (each separated by the ‹|› alternation operator)
- Since the words are surrounded on both sides by word boundaries, they can appear in any order. Without the word boundaries, however, it might be important to put longer words first; otherwise, you’d never find “awesome” when searching for ‹awe|awesome›
- Because the regex engine attempts to match each word in the list from left to right, you might see a very slight performance gain by placing the words that are most likely to be found in the subject text near the beginning of the list.
- Note that this regular expression is meant to generically demonstrate matching one out of a list of words. Because both the ‹two› and ‹three› in this example start with the same letter, you can more efficiently guide the regular expression engine by rewriting the regex as ‹\b(?:one|t(?:wo|hree))\b›. Don’t go crazy with such hand-tuning, though. Most regex engines try to perform this optimization for you automatically, at least in simple cases

### 5.3. Find Similar Words

- You have several problems in this case:
  - You want to find all occurrences of both color and colour in a string.
  - You want to find any of three words that end with “at”: bat, cat, or rat.
  - You want to find any word ending with phobia.
  - You want to find common variations on the name “Steven”: Steve, Steven, and Stephen.
  - You want to match any common form of the term “regular expression.”
- `\bcolou?r\b`
- `\b[bcr]at\b`
- `\b\w*phobia\b`
- `\bSte(?:ven?|phen)\b`
- `\breg(?:ular●expressions?|ex(?:ps?|e[sn])?)\b`
- You could do the same thing using ‹\b(?:b|c|r)at\b›, ‹\b(?:bat|cat|rat)\b›, or ‹\bbat\b|\bcat\b|\brat\b›. However, any time the difference between allowed matches is a choice from one of a list of characters, you’re better off using a character class.
  - Not only do character classes provide a more compact and readable syntax (thanks to being able to drop all the vertical bars and use ranges such as A–Z), most regex engines also provide far superior optimization for character classes. Alternation using vertical bars requires the engine to use the computationally expensive backtracking algorithm, whereas character classes use a simpler search approach.
  - Character classes are only capable of matching one character at a time from the characters specified within them—no exceptions.
  - Mistake:
    - Putting words in character classes 
    - Trying to use the alternation operator within character classes
- Here we add alternation to the mix as yet another means for regex variation. A noncapturing group, written as ‹(?:⋯)›, limits the reach of the ‹|› alternation operator. The ‹?› quantifier used inside the group’s first alternation option makes the preceding ‹n› character optional. This improves efficiency (and brevity) versus the equivalent ‹\bSte(?:ve|ven|phen)\b›. The same principle explains why the literal string ‹Ste› appears at the front of the regular expression, rather than being repeated three times as with ‹\b(?:Steve|Steven|Stephen)\b› or ‹\bSteve\b|\bSteven\b|\bStephen\b›

### 5.4.Find All Except a Specific Word

- You want to use a regular expression to match any complete word except cat. Catwoman, vindicate, and other words that merely contain the letters “cat” should be matched—just not cat.
- `\b(?!cat\b)\w+`
- Although a negated character class (written as ‹[^⋯]›) makes it easy to match anything except a specific character, you can’t just write ‹[^cat]› to match anything except the word cat. ‹[^cat]› is a valid regex, but it matches any character except c, a, or t. Hence, although ‹\b[^cat]+\b› would avoid matching the word cat, it wouldn’t match the word time either, because it contains the forbidden letter t. The regular expression ‹\b[^c][^a][^t]\w*› is no good either, because it would reject any word with c as its first letter, a as its second letter, or t as its third. Furthermore, that doesn’t restrict the first three letters to word characters, and it only matches words with at least three characters since none of the negated character classes are optional.
```markdown
\b     # Assert position at a word boundary.
(?!    # Not followed by:
  cat  #   Match "cat".
  \b   #   Assert position at a word boundary.
)      # End the negative lookahead.
\w+    # Match one or more word characters.
```
- The negative lookahead disallows the sequence cat followed by a word boundary, without preventing the use of those letters when they do not appear in that exact sequence, or when they appear as part of a longer or shorter word
- When applied to the subject string categorically match any word except cat, the regex will find five matches: `categorically, match, any, word, and except`.
- Find words that don’t contain another word
  - `\b(?:(?!cat)\w)+\b`
  - the word boundary at the beginning of the regular expression provided a convenient anchor that allowed us to simply place the negative lookahead at the beginning of the word. The solution used here is not as efficient, but it’s nevertheless a commonly used construct that allows you to match something other than a particular word or pattern. It does this by repeating a group containing a negative lookahead and a single word character. Before matching each character, the regex engine makes sure that the word cat cannot be matched starting at the current position.
  - Unlike the previous regular expression, this one requires a terminating word boundary. Otherwise, it could match just the first part of a word, up to where cat appears within it.
  - When applied to the subject string categorically match any word except cat, the regex will find four matches: match, any, word, and except.

### 5.5. Find Any Word Not Followed by a Specific Word

- You want to match any word that is not immediately followed by the word cat, ignoring any whitespace, punctuation, or other nonword characters that appear in between.
- `\b\w+\b(?!\W+cat\b)`
- As with many other recipes in this chapter, word boundaries (‹\b›) and the word character token (‹\w›) work together to match a complete word. You can find in-depth descriptions of these features in Recipe2.6.
- Lookahead tells the regex engine to temporarily step forward in the string, to check whether the pattern inside the lookahead can be matched just ahead of the current position. It does not consume any of the characters matched inside the lookahead. Instead, it merely asserts whether a match is possible
- As for the pattern inside the lookahead, the ‹\W+› matches one or more nonword characters, such as whitespace and punctuation, that appear before ‹cat›. The word boundary at the end of the lookahead ensures that we skip only words not followed by cat as a complete word, rather than just any word starting with cat.

### 5.6. Find Any Word Not Preceded by a Specific Word

- You want to match any word that is not immediately preceded by the word cat, ignoring any whitespace, punctuation, or other nonword characters that come between.
- Lookbehind lets you check if text appears before a given position. It works by instructing the regex engine to temporarily step backward in the string, checking whether something can be found ending at the position where you placed the lookbehind.
- `(?<!\bcat\W+)\b\w+`
- `(?<!\bcat\W)\b\w+`
- `(?<!\Wcat\W)(?<!^cat\W)\b\w+`

### 5.7. Find Words Near Each Other

- You want to emulate a NEAR search using a regular expression. For readers unfamiliar with the term, some search tools that use Boolean operators such as NOT and OR also have a special operator called NEAR. Searching for “word1 NEAR word2” finds word1 and word2 in any order, as long as they occur within a certain distance of each other.
- If you’re searching for just two different words, you can combine two regular expressions—one that matches word1 before word2, and another that flips the order of the words
- `\b(?:word1\W+(?:\w+\W+){0,5}?word2|word2\W+(?:\w+\W+){0,5}?word1)\b`
```markdown
\b(?:
  word1                 # first term
  \W+ (?:\w+\W+){0,5}?  # up to five words
  word2                 # second term
|                       #   or, the same pattern in reverse:
  word2                 # second term
  \W+ (?:\w+\W+){0,5}?  # up to five words
  word1                 # first term
)\b
```
- The first subpattern matches word1, followed by between zero and five words, and then word2. The second subpattern matches the same thing, with the order of word1 and word2 reversed.

### 5.8. Find Repeated Words

- You’re editing a document and would like to check it for any incorrectly repeated words. You want to find these doubled words despite capitalization differences, such as with “The the.” You also want to allow differing amounts of whitespace between words, even if this causes the words to extend across more than one line. Any separating punctuation, however, should cause the words to no longer be treated as if they are repeating.
- `\b([A-Z]+)\s+\1\b`
- There are two things needed to match something that was previously matched: a capturing group and a backreference
- Place the thing you want to match more than once inside a capturing group, and then match it again using a backreference.
- Finally, the word boundaries at the beginning and end of the regular expression ensure that it doesn’t match within other words ( e.g., with “this thistle”).

### 5.9. Remove Duplicate Lines

- You have a log file, database query output, or some other type of file or string with duplicate lines. You need to remove all but one of each duplicate line using a text editor or other similar tool.
- When you’re programming, options two and three should be avoided since they are inefficient compared to other available approaches, such as using a hash object to keep track of unique lines. However, the first option (which requires that you sort the lines in advance, unless you only want to remove adjacent duplicates) may be an acceptable approach since it’s quick and easy.
- Option 1: Sort lines and remove adjacent duplicates
  - After sorting the lines, use the following regex and replacement string to get rid of the duplicates:
  - `^(.*)(?:(?:\r?\n|\r)\1)+$`
  - Replace with: `$1`
- Option 2: Keep the last occurrence of each duplicate line in an unsorted file
  - `^([^\r\n]*)(?:\r?\n|\r)(?=.*^\1$)`
  - `^(.*)(?:\r?\n|\r)(?=[\s\S]*^\1$)`
- Option 3: Keep the first occurrence of each duplicate line in an unsorted file
  - `^([^\r\n]*)$(.*?)(?:(?:\r?\n|\r)\1$)+`

### 5.10. Match Complete Lines That Contain a Word

- You want to match all lines that contain the word error anywhere within them.
- `^.*\berror\b.*$`
- To expand the regex to match a complete line, add ‹.*› at both ends. The dot-asterisk sequences match zero or more characters within the current line. The asterisk quantifiers are greedy, so they will match as much text as possible. The first dot-asterisk matches until the last occurrence of “error” on the line, and the second dot-asterisk matches any non-line-break characters that occur after it.
- Finally, place caret and dollar sign anchors at the beginning and end of the regular expression, respectively, to ensure that matches contain a complete line. Strictly speaking, the dollar sign anchor at the end is redundant since the dot and greedy asterisk will always match until the end of the line. However, it doesn’t hurt to add it, and makes the regular expression a little more self-explanatory.
- Remember that the three key metacharacters used to restrict matches to a single line (the ‹^› and ‹$› anchors, and the dot) do not have fixed meanings. To make them all line-oriented, you have to enable the option to let ^ and $ match at line breaks, and make sure that the option to let the dot match line breaks is not enabled. Recipe3.4 shows how to apply these options in code. If you’re using JavaScript or Ruby, there is one less option to worry about, because JavaScript does not have an option to let dot match line breaks, and Ruby’s caret and dollar sign anchors always match at line breaks.

### 5.11. Match Complete Lines That Do Not Contain a Word

- You want to match complete lines that do not contain the word error.
- `^(?:(?!\berror\b).)*$`
- In order to match a line that does not contain something, use negative lookahead
- Notice that in this regular expression, a negative lookahead and a dot are repeated together using a noncapturing group. This is necessary to ensure that the regex ‹\berror\b› fails at every position in the line.
- Testing a negative lookahead against every position in a line or string is rather inefficient. This solution is intended to be used in situations where one regular expression is all that can be used, such as when using an application that can’t be programmed. When programming, it is more efficient to search through text line by line

### 5.12. Trim Leading and Trailing Whitespace

- You want to remove leading and trailing whitespace from a string. For instance, you might need to do this to clean up data submitted by users
- To keep things simple and fast, the best all-around solution is to use two substitutions—one to remove leading whitespace, and another to remove trailing whitespace.
  - Leading whitespace: `^\s+`
  - Trailing whitespace: `\s+$`
- Simply replace matches found using one of the “leading whitespace” regexes and one of the “trailing whitespace” regexes with the empty string

### 5.13. Replace Repeated Whitespace with a Single Space

- As part of a cleanup routine for user input or other data, you want to replace repeated whitespace characters with a single space. Any tabs, line breaks, or other whitespace should also be replaced with a space.
- Clean any whitespace characters
  - `\s+`
- Clean horizontal whitespace characters
  - `[●\t\xA0]+`

### 5.14. Escape Regular Expression Metacharacters

- You want to use a literal string provided by a user or from some other source as part of a regular expression. However, you want to escape all regular expression metacharacters within the string before embedding it in your regex, to avoid any unintended consequences.
- By adding a backslash before any characters that potentially have special meaning within a regular expression, you can safely use the resulting pattern to match a literal sequence of characters. Of the programming languages covered by this book, all except JavaScript have a built-in function or method to perform this task (listed in Table5-3). However, for the sake of completeness, we’ll show how to pull this off using your own regex, even in the languages that have a ready-made solution.
- `[[\]{}()*+?.\\|^$\-,&#\s]`