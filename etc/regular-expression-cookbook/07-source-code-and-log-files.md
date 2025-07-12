# Chapter 7: Source Code and Log Files

### 7.1. Keywords

- You are working with a file format for forms in a software application. The words “end,” “in,” “inline,” “inherited,” “item,” and “object” are reserved keywords in this format.[9] You want a regular expression that matches any of these keywords.
- `\b(?:end|in|inline|inherited|item|object)\b`
- Matching a word from a list of words is very easy with a regular expression. We simply use alternation to match any one of the keywords. 
- The word boundaries at the start and the end of the regex make sure we only match entire words. The regex should match inline rather than in when the file contains inline, and it should fail to match when the file contains interesting. 
- Because alternation has the lowest precedence of all regex operators, we have to put the list of keywords inside a group. Here we used a noncapturing group for efficiency. When using this regex as part of a larger regular expression, you may want to use a capturing group instead, so you can determine whether the regex matched a keyword or something else.

### 7.2. Identifiers

- You need a regular expression that matches any identifier in your source code. Your programming language requires identifiers to start with an underscore or an ASCII letter. The following characters can be underscores or ASCII letters or digits. Identifiers can be between 1 and 32 characters long.
- `\b[a-z_][0-9a-z_]{0,31}\b`

### 7.3. Numeric Constant

- You need a regular expression that matches a decimal integer without a leading zero, an octal integer with a leading zero, a hexadecimal integer prefixed with 0x, or a binary integer prefixed with 0b. The integer may have the suffix L to denote it is a long rather than an int.
- `\b(?:([1-9][0-9]*)|(0[0-7]*)|0x([0-9A-F]+)|0b([01]+))(L)?\b`
- This regular expression is essentially the combination of the solutions presented in Recipe6.5 (decimal), Recipe6.4 (octal), Recipe6.2 (hexadecimal), and Recipe6.3 (binary)

### 7.4. Operators

- You are developing a syntax coloring scheme for your favorite text editor. You need a regular expression that matches any of the characters that can be used as operators in the programming language for which you’re creating the scheme: -, +, *, /, =, <, >, %, &, ^, |, !, ~, and ?. The regex doesn’t need to check whether the combination of characters forms a valid operator. That is not a job for a syntax coloring scheme; instead, it should simply highlight all operator characters as such.
- `[-+*/=<>%&^|!~?]`

### 7.5. Single-Line Comments

- You want to match a comment that starts with // and runs until the end of the line.
- `//.*`
- The forward slash has no special meaning in regular expressions, so we can easily match the start of the comment with ‹//›. Some programming languages use forward slashes to delimit regular expressions. When you use this regular expression in your code, you may need to escape the forward slashes as explained in Recipe3.1.
- ‹.*› simply matches everything up to the end of the line. We don’t need to add anything to the regular expression to make it stop at the end of a line

### 7.6. Multiline Comments

- You want to match a comment that starts with /* and ends with `*/.` Nested comments are not permitted. Any /* between /* and */ is simply part of the comment. Comments can span across lines.
- `/\*.*?\*/`
- `/\*[\s\S]*?\*/`
- The forward slash has no special meaning in regular expressions, but the asterisk does. We need to escape the asterisk with a backslash. This gives ‹/\*› and ‹\*/› to match /* and */.
- We use ‹.*?› to match anything between the two delimiters of the comment. The option “dot matches line breaks” that most regex engines have allows this to span multiple lines. We need to use a lazy quantifier to make sure that the comment stops at the first `*/` after the `/*`, rather than at the last */ in the file.
- JavaScript is the only regex flavor in this book that does not have an option to make the dot match line breaks. If you’re using JavaScript without the XRegExp library, you can use ‹[\s\S]› to accomplish the same. Although you could use ‹[\s\S]› with the other regex flavors too, we do not recommend it, as regex engines generally have optimized code to handle the dot, which is one of the most elementary features of regular expressions.

### 7.7. All Comments

- You want a regex that matches both single-line and multiline comments, as described in the "Problem" sections of the preceding two recipes.
- `//.*|/\*[\s\S]*?\*/`

### 7.8. Strings 

- You need a regex that matches a string, which is a sequence of zero or more characters enclosed by double quotes. A string with nothing between the quotes is an empty string. Two sequential double quotes in a character string denote a single character, a double quote. Strings cannot include line breaks. Backslashes or other characters have no special meaning in strings.
- `"[^"\r\n]*(?:""[^"\r\n]*)*"`

### 7.9. Strings with Escapes

- You need a regex that matches a string, which is a sequence of zero or more characters enclosed by double quotes. A string with nothing between the quotes is an empty string. A double quote can be included in the string by escaping it with a backslash, and backslashes can also be used to escape other characters in the string. Strings cannot include line breaks, and line breaks cannot be escaped with backslashes.
- `"[^"\\\r\n]*(?:\\.[^"\\\r\n]*)"`

### 7.10. Regex Literals

- You need a regular expression that matches regular expression literals in your source code files so you can easily find them in your text editor or with a grep tool. Your programming language uses forward slashes to delimit regular expressions. Forward slashes in the regex must be escaped with a backslash.
- `[=:(,](?:\s*!)?+\s*(/[^/\\\r\n]*(?:\\.[^/\\\r\n]*)*/)`
- A literal regular expression really is just a string quoted with forward slashes that can contain forward slashes if escaped with a backslash.

### 7.11. Here Documents

- You need a regex that matches here documents in source files for a scripting language in which a here document can be started with << followed by a word. The word may have single or double quotes around it. The here document ends when that word appears at the very start of a line, without any quotes, using the same case.
- `<<(["']?)([A-Za-z]+)\b\1[\s\S]*?^\2\b`
- This regex may look a bit cryptic, but it is very straightforward. ‹<<› simply matches <<. ‹(["']?)›, then matches an optional single or double quote. The parentheses form a capturing group to store the quote, or the lack thereof. It is important that the quantifier ‹?› is inside the group rather than outside of it, so that the group always participates in the match. If we made the group itself optional, the group would not participate in the match when no quote can be matched, and a backreference to that group would fail to match.
- The capturing group with character class ‹([A-Za-z]+)› matches a word and stores it into the second backreference. The word boundary ‹\b› makes sure we match the entire word after ‹<<›. If we were to omit the word boundary, the regex engine would backtrack. It would try to match the word partially if the backreference ‹\2› cannot be matched. We do not need a word boundary before the word, because ‹<<(["']?)› already makes sure there is a nonword character before the word.
- ‹\1› is a backreference to the first capturing group. This group will hold the quote if we matched one; otherwise, the group holds the empty string. Thus ‹\1› matches the same quote matched by the capturing group. ‹\1› has no effect if the capturing group holds the empty string.
- ‹.*?› matches any amount of text. We turned on the option “dot matches line breaks” to allow it to span multiple lines. JavaScript does not have that option, and so for JavaScript we use ‹[\s\S]*?› to match the text. Either way, the question mark makes the asterisk lazy, telling it to match as few characters as possible. The here document should end at the first occurrence of the terminating word rather than the last occurrence. The file may have multiple here documents using the same terminating word, and the lazy quantifier makes sure we match each here document separately.
- ‹\2› is a backreference to the second capturing group. This group holds the word we matched at the start of the here document. Because the here document syntax of our scripting language is case sensitive, our regex needs to be case sensitive too. That’s why we used ‹[A-Za-z]+› to match the word rather than using ‹[a-z]+› or ‹[A-Z]+› and turning on case insensitivity. Backreferences also become case insensitive when the case insensitivity option is turned on.

### 7.12. Common Log Format

- You need a regular expression that matches each line in the log files produced by a web server that uses the Common Log Format.[11] For example:
  - `127.0.0.1 - jg [27/Apr/2012:11:27:36 +0700] "GET /regexcookbook.html HTTP/1.1" 200 2326`
- The regular expression should have a capturing group for each field, to allow the application using the regular expression to easily process the fields of each entry in the log.
- `^(?<client>\S+)●\S+●(?<userid>\S+)●\[(?<datetime>[^\]]+)\]●"(?<method>[A-Z]+)●(?<request>[^●"]+)?●HTTP/[0-9.]+"●(?<status>[0-9]{3})●(?<size>[0-9]+|-)`
- `^(\S+)●\S+●(\S+)●\[([^\]]+)\]●"([A-Z]+)●([^●"]+)?●HTTP/[0-9.]+"●([0-9]{3})●([0-9]+|-)●"([^"]*)"●"([^"]*)"`

### 7.13. Combined Log Format

- You need a regular expression that matches each line in the log files produced by a web server that uses the Combined Log Format.[14] For example:
- `^(?<client>\S+)●\S+●(?<userid>\S+)●\[(?<datetime>[^\]]+)\]●"(?<method>[A-Z]+)●(?<request>[^●"]+)?●HTTP/[0-9.]+"●(?<status>[0-9]{3})●(?<size>[0-9]+|-)●"(?<referrer>[^"]*)"●"(?<useragent>[^"]*)"`
- `^(\S+)●\S+●(\S+)●\[([^\]]+)\]●"([A-Z]+)●([^●"]+)?●HTTP/[0-9.]+"●([0-9]{3})●([0-9]+|-)●"([^"]*)"●"([^"]*)"●"([^"]*)"●"([^"]*)"`

### 7.14. Broken Links Reported in Web Logs

- You have a log for your website in the Combined Log Format. You want to check the log for any errors caused by broken links on your own website.
- `"(?:GET|POST)●(?<file>[^#?●"]+)(?:[#?][^●"]*)?●HTTP/[0-9.]+"●404●(?:[0-9]+|-)●"(?<referrer>http://www\.yoursite\.com[^"]*)"`
- `"(?:GET|POST)●([^#?●"]+)(?:[#?][^●"]*)?●HTTP/[0-9.]+"●404●(?:[0-9]+|-)●"(http://www\.yoursite\.com[^"]*)"`