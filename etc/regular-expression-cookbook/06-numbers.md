# Chapter 6: Numbers

### 6.1. Integer Numbers

- You want to find various kinds of integer decimal numbers in a larger body of text, or check whether a string variable holds an integer decimal number.
- `\b[0-9]+\b`
- We prefer to use the explicit range ‹[0-9]› instead of the shorthand ‹\d›. In .NET and Perl, ‹\d› matches any digit in any script, but ‹[0-9]› always just matches the 10 digits in the ASCII table. If you know your subject text doesn’t include any non-ASCII digits, you can save a few keystrokes and use ‹\d› instead of ‹[0-9]›.
- If you don’t know whether your subject will include digits outside the ASCII table, you need to think about what you want to do with the regex matches and what the user’s expectations are in order to decide whether you should use ‹\d› or ‹[0-9]›

### 6.2. Hexadecimal Numbers

- You want to find hexadecimal numbers in a larger body of text, or check whether a string variable holds a hexadecimal number.
- `\b[0-9A-F]+\b`
- `\b[0-9A-Fa-f]+\b`
- `^[0-9A-F]+$`
- `\b0x[0-9A-F]+\b`
- `&H[0-9A-F]+\b`

### 6.3. Binary Numbers

- You want to find binary numbers in a larger body of text, or check whether a string variable holds a binary number.
- `\b[01]+\b`
- `^[01]+$`
- `\b0b[01]+\b`
- `\b[01]+B\b`
- `\b[01]{8}\b`
- `\b[01]{16}\b`

### 6.4. Octal Numbers

- You want to find octal numbers in a larger body of text, or check whether a string variable holds an octal number. An octal number is a number that consists of the digits 0 to 7. The number must either have at least one leading zero, or it must be prefixed with 0o.
- `\b0[0-7]*\b`
- `^0[0-7]*$`
- `\b0o[0-7]+\b`

### 6.5. Decimal Numbers

- You want to find various kinds of integer decimal numbers in a larger body of text, or check whether a string variable holds an integer decimal number. The number must not have a leading zero, as only octal numbers can have leading zeros. But the number zero itself is a valid decimal number.
- `\b(0|[1-9][0-9]*)\b`
- `^(0|[1-9][0-9]*)$`

### 6.6. Strip Leading Zeros

- You want to match an integer number, and either return the number without any leading zeros or delete the leading zeros.
- Regex
  - `\b0*([1-9][0-9]*|0)\b`
- Replacement
  - `$1`
- We use a capturing group to separate a number from its leading zeros. Before the group, ‹0*› matches the leading zeros, if any. Within the group, ‹[1-9][0-9]*› matches a number that consists of one or more digits, with the first digit being nonzero. The number can begin with a zero only if the number is zero itself. The word boundaries make sure we don’t match partial numbers,

### 6.7. Numbers Within a Certain Range

- You want to match an integer number within a certain range of numbers. You want the regular expression to specify the range accurately, rather than just limiting the number of digits.
- 1 to 12 (hour or month):
  - `^(1[0-2]|[1-9])$`
- 1 to 24 (hour):
  - `^(2[0-4]|1[0-9]|[1-9])$`
- 1 to 31 (day of the month):
  - `^(3[01]|[12][0-9]|[1-9])$`
- 1 to 53 (week of the year):
  - `^(5[0-3]|[1-4][0-9]|[1-9])$`
- 0 to 59 (minute or second):
  - `^[1-5]?[0-9]$`
- 0 to 100 (percentage):
  - `^(100|[1-9]?[0-9])$`
- 1 to 100:
  - `^(100|[1-9][0-9]?)$`
- 32 to 126 (printable ASCII codes):
  - `^(12[0-6]|1[01][0-9]|[4-9][0-9]|3[2-9])$`
- 0 to 127 (nonnegative signed byte):
  - `^(12[0-7]|1[01][0-9]|[1-9]?[0-9])$`
- –128 to 127 (signed byte):
  - `^(12[0-7]|1[01][0-9]|[1-9]?[0-9]|-(12[0-8]|1[01][0-9]|[1-9]?[0-9]))$`
- 0 to 255 (unsigned byte):
  - `^(25[0-5]|2[0-4][0-9]|1[0-9]{2}|[1-9]?[0-9])$`
- 1 to 366 (day of the year):
  - `^(36[0-6]|3[0-5][0-9]|[12][0-9]{2}|[1-9][0-9]?)$`
- 1900 to 2099 (year):
  - `^(19|20)[0-9]{2}$`
- 0 to 32767 (nonnegative signed word):
  - `^(3276[0-7]|327[0-5][0-9]|32[0-6][0-9]{2}|3[01][0-9]{3}|[12][0-9]{4}|[1-9][0-9]{1,3}|[0-9])$`
- –32768 to 32767 (signed word):
  - `^(3276[0-7]|327[0-5][0-9]|32[0-6][0-9]{2}|3[01][0-9]{3}|[12][0-9]{4}|[1-9][0-9]{1,3}|[0-9]|-(3276[0-8]|327[0-5][0-9]|32[0-6][0-9]{2}|3[01][0-9]{3}|[12][0-9]{4}|[1-9][0-9]{1,3}|[0-9]))$`
- 0 to 65535 (unsigned word):
  - `^(6553[0-5]|655[0-2][0-9]|65[0-4][0-9]{2}|6[0-4][0-9]{3}|[1-5][0-9]{4}|[1-9][0-9]{1,3}|[0-9])$`
- Regular expressions work character by character. If we want to match a number that consists of more than one digit, we have to spell out all the possible combinations for the digits.
- We listed the range with two digits before the one with a single digit. This is intentional because the regular expression engine is eager. It scans the alternatives from left to right, and stops as soon as one matches
- If your subject text is 12, then ‹1[0-2|[1-9]› matches 12, whereas ‹[1-9]|1[0-2]› matches just ‹1›. The first alternative, ‹[1-9]›, is tried first. Since that alternative is happy to match just 1, the regex engine never tries to check whether ‹1[0-2]› might offer a “better” solution.
- POSIX-compliant regex engines and DFA regex engines do not follow this rule. They try all alternatives, and return the one that finds the longest match. All the flavors discussed in this book, however, are NFA engines, which don’t do the extra work required by POSIX. They will all tell you that ‹[1-9]|1[0-2]› matches 1 in 12.
- That’s all there really is to matching numeric ranges with regular expressions: simply split up the range until you have ranges with a fixed number of positions with independent digits. This way, you’ll always get a correct regular expression that is easy to read and maintain, even if it may get a bit long-winded.

### 6.8. Hexadecimal Numbers Within a Certain Range

- You want to match a hexadecimal number within a certain range of numbers. You want the regular expression to specify the range accurately, rather than just limiting the number of digits.
- 1 to C (1 to 12: hour or month):
  - `^[1-9a-c]$`
- 1 to 18 (1 to 24: hour):
  - `^(1[0-8]|[1-9a-f])$`
- 1 to 1F (1 to 31: day of the month):
  - `^(1[0-9a-f]|[1-9a-f])$`
- 1 to 35 (1 to 53: week of the year):
  - `^(3[0-5]|[12][0-9a-f]|[1-9a-f])$`
- 0 to 3B (0 to 59: minute or second):
  - `^(3[0-9a-b]|[12]?[0-9a-f])$`
- 0 to 64 (0 to 100: percentage):
  - `^(6[0-4]|[1-5]?[0-9a-f])$`
- 1 to 64 (1 to 100):
  - `^(6[0-4]|[1-5][0-9a-f]|[1-9a-f])$`
- 20 to 7E (32 to 126: printable ASCII codes)
  - `^(7[0-9a-e]|[2-6][0-9a-f])$`
- 0 to 7F (0 to 127: 7-bit number):
  - `^[1-7]?[0-9a-f]$`
- 0 to FF (0 to 255: 8-bit number):
  - `^[1-9a-f]?[0-9a-f]$`
- 1 to 16E (1 to 366: day of the year):
  - `^(16[0-9a-e]|1[0-5][0-9a-f]|[1-9a-f][0-9a-f]?)$`
- 76C to 833 (1900 to 2099: year):
  - `^(83[0-3]|8[0-2][0-9a-f]|7[7-9a-f][0-9a-f]|76[c-f])$`
- 0 to 7FFF: (0 to 32767: 15-bit number):
  - `^([1-7][0-9a-f]{3}|[1-9a-f][0-9a-f]{1,2}|[0-9a-f])$`
- 0 to FFFF: (0 to 65535: 16-bit number):
  - `^([1-9a-f][0-9a-f]{1,3}|[0-9a-f])$`
- As the previous recipe explains, split the range into multiple ranges, until each range has a fixed number of positions with independent hexadecimal digits. Then it’s just a matter of using a character class for each position, and combining the ranges with alternation.
- Since letters and digits occupy separate areas in the ASCII and Unicode character tables, you cannot use the character class ‹[0-F]› to match any of the 16 hexadecimal digits. Though this character class will actually do that, it will also match the punctuation symbols that sit between the digits and the letters in the ASCII table. Instead, place two character ranges in the character class: [0-9A-F].

### 6.9. Integer Numbers with Separators

- You want to find various kinds of integer numbers in a larger body of text, or check whether a string variable holds an integer number. Underscores are allowed as separators between groups of numbers, to make the integers easier to read. Numbers may not begin or end with an underscore. You want to allow decimal, octal, hexadecimal, and binary numbers. Hexadecimal and binary numbers must be prefixed with 0x and 0b.
- 0b0111_1111_1111_1111_1111_1111_1111_1111, 0177_7777_7777, 2_147_483_647, and 0x7fff_ffff are examples of valid numbers.
- Find any decimal or octal integer with optional underscores in a larger body of text:
  - `\b[0-9]+(_+[0-9]+)*\b`
- Find any hexadecimal integer with optional underscores in a larger body of text:
  - `\b0x[0-9A-F]+(_+[0-9A-F]+)*\b`
- Find any binary integer with optional underscores in a larger body of text:
  - `\b0b[01]+(_+[01]+)*\b`
- Find any decimal, octal, hexadecimal, or binary integer with optional underscores in a larger body of text:
  - `\b([0-9]+(_+[0-9]+)*|0x[0-9A-F]+(_+[0-9A-F]+)*|0b[01]+(_+[01]+)*)\b`
- Check whether a text string holds just a decimal, octal, hexadecimal, or binary integer with optional underscores:
  - `\A([0-9]+(_+[0-9]+)*|0x[0-9A-F]+(_+[0-9A-F]+)*|0b[01]+(_+[01]+)*)\Z`
- Our solution ‹[0-9]+(_+[0-9]+)*› uses ‹[0-9]+› to match the initial digit or digits as before. We add ‹(_+[0-9]+)*› to allow the digits to be followed by one or more underscores, as long as those underscores are followed by more digits. ‹_+› allows any number of sequential underscores. ‹[0-9]+› allows any number of digits after the underscores. We put those two inside a group that we repeat zero or more times with a asterisk. This allows any number of nonsequential underscores with digits in between them and after them, while also allowing numbers with no underscores at all.

### 6.10.Floating-Point Numbers

- You want to match a floating-point number and specify whether the sign, integer, fraction and exponent parts of the number are required, optional, or disallowed. You don’t want to use the regular expression to restrict the numbers to a specific range, and instead leave that to procedural code,
- Mandatory sign, integer, fraction, and exponent:
  - `^[-+][0-9]+\.[0-9]+[eE][-+]?[0-9]+$`
- Mandatory sign, integer, and fraction, but no exponent:
  - `^[-+][0-9]+\.[0-9]+$`
- Optional sign, mandatory integer and fraction, and no exponent:
  - `^[-+]?[0-9]+\.[0-9]+$`
- Optional sign and integer, mandatory fraction, and no exponent:
  - `^[-+]?[0-9]*\.[0-9]+$`
- Optional sign, integer, and fraction. If the integer part is omitted, the fraction is mandatory. If the fraction is omitted, the decimal dot must be omitted, too. No exponent
  - `^[-+]?([0-9]+(\.[0-9]+)?|\.[0-9]+)$`
- Optional sign, integer, and fraction. If the integer part is omitted, the fraction is mandatory. If the fraction is omitted, the decimal dot is optional. No exponent.
  - `^[-+]?([0-9]+(\.[0-9]*)?|\.[0-9]+)$`
- Optional sign, integer, and fraction. If the integer part is omitted, the fraction is mandatory. If the fraction is omitted, the decimal dot must be omitted, too. Optional exponent.
  - `^[-+]?([0-9]+(\.[0-9]+)?|\.[0-9]+)([eE][-+]?[0-9]+)?$`
- Optional sign, integer, and fraction. If the integer part is omitted, the fraction is mandatory. If the fraction is omitted, the decimal dot is optional. Optional exponent.
  - `^[-+]?([0-9]+(\.[0-9]*)?|\.[0-9]+)([eE][-+]?[0-9]+)?$`
- The preceding regex, edited to find the number in a larger body of text:
  - `[-+]?(\b[0-9]+(\.[0-9]*)?|\.[0-9]+)([eE][-+]?[0-9]+\b)?`
- All regular expressions are wrapped between anchors (Recipe2.5) to make sure we check whether the whole input is a floating-point number, as opposed to a floating-point number occurring in a larger string. You could use word boundaries or lookaround as explained in Recipe6.1 if you want to find floating-point numbers in a larger body of text.

### 6.11.Numbers with Thousand Separators

- You want to match numbers that use the comma as the thousand separator and the dot as the decimal separator.
- Mandatory integer and fraction:
  - `^[0-9]{1,3}(,[0-9]{3})*\.[0-9]+$`
- Mandatory integer and optional fraction. Decimal dot must be omitted if the fraction is omitted.
  - `^[0-9]{1,3}(,[0-9]{3})*(\.[0-9]+)?$`
- Optional integer and optional fraction. Decimal dot must be omitted if the fraction is omitted.
  - `^([0-9]{1,3}(,[0-9]{3})*(\.[0-9]+)?|\.[0-9]+)$`
- The preceding regex, edited to find the number in a larger body of text:
  - `\b[0-9]{1,3}(,[0-9]{3})*(\.[0-9]+)?\b|\.[0-9]+\b`

### 6.12. Add Thousand Separators to Numbers

- You want to add commas as the thousand separator to numbers with four or more digits. You want to do this both for individual numbers and for any numbers in a string or file.
- 7000000000 => 7,000,000,000
- Regex: `[0-9](?=(?:[0-9]{3})+(?![0-9]))`
- Replacement: `$&,`
- These replacement strings all put the matched number back using backreference zero (the entire match, which in this case is a single digit), followed by a comma.
- The first part is the leading character class ‹[0-9]› that matches any single digit. The second part is the positive lookahead ‹(?=(?:[0-9]{3})+(?![0-9]))› that causes the match attempt to fail unless it’s at a position followed by digits in exact sets of three. In other words, the lookahead ensures that the regex matches only the digits that should be followed by a comma
- The ‹(?:[0-9]{3})+› within the lookahead matches digits in sets of three. The negative lookahead ‹(?![0-9])› that follows is there to ensure that no digits come immediately after the digits we matched in sets of three. Otherwise, the outer positive lookahead would be satisfied by any number of following digits, so long as there were at least three.
- Match separator positions only, using lookbehind
  - Regex: `(?<=[0-9])(?=(?:[0-9]{3})+(?![0-9]))`
  - Replacement: `,`
- This adaptation of the previous regex doesn’t match any digits at all. Instead, it matches only the positions where we want to insert commas within numbers. These positions are wherever there are digits on the right in exact sets of three, and at least one digit on the left.
- The lookahead used to search for sets of exactly three digits on the right is the same as in the last regex. The difference here is that, instead of starting the regex with ‹[0-9]› to match a digit, we instead assert that there is at least one digit to the left by using the positive lookbehind ‹(?<=[0-9])›. Without the lookbehind, the regex would match the position to the left of 123 and therefore the search-and-replace would convert it to ,100.
- Don’t add commas after a decimal point
  - The preceding regexes add commas to any sequence of four or more digits. A rather glaring issue with this basic approach is that it can add commas to digits that come after a dot as the decimal separator, so long as there are at least four digits after the dot

### 6.13.Roman Numerals

- You want to match Roman numerals such as IV, XIII, and MVIII.
- Roman numerals without validation:
  - `^[MDCLXVI]+$`
- Modern Roman numerals, strict:
  - `^(?=[MDCLXVI])M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$`
- Modern Roman numerals, flexible:
  - `^(?=[MDCLXVI])M*(C[MD]|D?C*)(X[CL]|L?X*)(I[XV]|V?I*)$`
- Simple Roman numerals:
  - `^(?=[MDCLXVI])M*D?C{0,4}L?X{0,4}V?I{0,4}$`
- 