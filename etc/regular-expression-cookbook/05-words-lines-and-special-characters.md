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