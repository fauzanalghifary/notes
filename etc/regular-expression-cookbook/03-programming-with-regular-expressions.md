# Chapter 3: Programming with Regular Expressions

### 3.0 Intro

- This chapter explains how to implement regular expressions with your programming language of choice.
- Because of the level of detail in this chapter, reading it from start to finish may get a bit tedious. If you’re reading Regular Expression Cookbook for the first time, we recommend you skim this chapter to get an idea of what can or needs to be done. Later, when you want to implement one of the regular expressions from the following chapters, come back here to learn exactly how to integrate the regexes with your programming language of choice.
- Chapters 4 through 9 use regular expressions to solve real-world problems. Those chapters focus on the regular expressions themselves, and many recipes in those chapters don’t show any source code at all. To make the regular expressions you find in those chapters work, simply plug them into one of the code snippets in this chapter.
- Because the other chapters focus on regular expressions, they present their solutions for specific regular expression flavors, rather than for specific programming languages. Regex flavors do not correspond one-on-one with programming languages. Scripting languages tend to have their own regular expression flavor built-in, and other programming languages rely on libraries for regex support. Some libraries are available for multiple languages, while certain languages have multiple libraries available for them.

### 3.1 Literal Regular Expressions in Source Code

- You have been given the regular expression `‹[$"'\n\d/\\]›` as the solution to a problem. This regular expression consists of a single character class that matches a dollar sign, a double quote, a single quote, a line feed, any digit between 0 and 9, a forward slash, or a backslash. You want to hardcode this regular expression into your source code as a string constant or regular expression operator.
- JavaScript: `/[$"'\n\d\/\\]/`
- Ruby: `/[$"'\n\d\/\\]/` or `%r![$"'\n\d/\\]!`
- Carelessly copying and pasting regular expressions from a regular expression tester into your source code—or vice versa—will often leave you scratching your head as to why the regular expression works in your tool but not in your source code, or why the tester fails on a regex you’ve copied from somebody else’s code.
- All programming languages discussed in this book require literal regular expressions to be delimited in a certain way, with some languages requiring strings and some requiring a special regex constant. If your regex includes the language’s delimiters or certain other characters with special meanings in the language, you have to escape them.
- The backslash is the most commonly used escape character. That’s why most of the solutions to this problem have far more backslashes in them than the four in the original regular expression.
- In JavaScript, regular expressions are best created by using the special syntax for declaring literal regular expressions. Simply place your regular expression between two forward slashes. If any forward slashes occur within the regular expression itself, escape those with a backslash.
  - Although it is possible to create a RegExp object from a string, it makes little sense to use the string notation for literal regular expressions in your code. You would have to escape quotes and backslashes, which generally leads to a forest of backslashes.
- In Ruby, regular expressions are best created by using the special syntax for declaring literal regular expressions. Simply place your regular expression between two forward slashes. If any forward slashes occur within the regular expression itself, escape those with a backslash.
  - If you don’t want to escape forward slashes in your regex, you can prefix your regular expression with %r and then use any punctuation character of your choice as the delimiter.
  - Although it is possible to create a Regexp object from a string, it makes little sense to use the string notation for literal regular expressions in your code. You then would have to escape quotes and backslashes, which generally leads to a forest of backslashes.
- Ruby is very similar to JavaScript in this respect, except that the name of the class is Regexp as one word in Ruby, whereas it is RegExp with camel caps in JavaScript.

### 3.2 Import the Regular Expression Library

- To be able to use regular expressions in your application, you want to import the regular expression library or namespace into your source code.
- Some programming languages have regular expressions built-in. For these languages, you don’t need to do anything to enable regular expression support. Other languages provide regular expression functionality through a library that needs to be imported with an import statement in your source code. Some languages don’t have regex support at all. For those, you’ll have to compile and link in the regular expression support yourself.
- JavaScript
  - JavaScript’s regular expression support is built-in and always available.
- Ruby
  - Ruby’s regular expression support is built-in and always available.

### 3.3 Create Regular Expression Objects

- You want to instantiate a regular expression object or otherwise compile a regular expression so you can use it efficiently throughout your application.
- JavaScript
  - `var myregexp = /regex pattern/;`
  - `var myregexp = new RegExp(userinput);`
- Ruby 
  - `myregexp = /regex pattern/;`
  - `myregexp = Regexp.new(userinput);`
- Before the regular expression engine can match a regular expression to a string, the regular expression has to be compiled. This compilation happens while your application is running. The regular expression constructor or compile function parses the string that holds your regular expression and converts it into a tree structure or state machine. The function that does the actual pattern matching will traverse this tree or state machine as it scans the string. Programming languages that support literal regular expressions do the compilation when execution reaches the regular expression operator.
- JavaScript
  - If you have a regular expression stored in a string variable (e.g., because you asked the user to type in a regular expression), use the RegExp() constructor to compile the regular expression. Notice that the regular expression inside the string is not delimited by forward slashes. Those slashes are part of JavaScript’s notation for literal RegExp objects, rather than part of the regular expression itself.
  - Since assigning a literal regex to a variable is trivial, most of the JavaScript solutions in this chapter omit this line of code and use the literal regular expression directly. In your own code, when using the same regex more than once, you should assign the regex to a variable and use that variable instead of pasting the same literal regex multiple times into your code. This increases performance and makes your code easier to maintain.
- Ruby
  - basically same as JavaScript

### 3.4 Set Regular Expression Options

- You want to compile a regular expression with all of the available matching modes: free-spacing, case insensitive, dot matches line breaks, and “^ and $ match at line breaks.”
- JavaScript
  - `var myregexp = /regex pattern/im;`
  - `var myregexp = new RegExp(userinput, "im");`
- Ruby
  - `myregexp = /regex pattern/xim;`
  - `myregexp = Regexp.new(userinput,
    Regexp::EXTENDED or Regexp::IGNORECASE or
    Regexp::MULTILINE);`
- Many of the regular expressions in this book, and those that you find elsewhere, are written to be used with certain regex matching modes. There are four basic modes that nearly all modern regex flavors support. Unfortunately, some flavors use inconsistent and confusing names for the options that implement the modes. Using the wrong modes usually breaks the regular expression.
- All the solutions in this recipe use flags or options provided by the programming language or regular expression class to set the modes. Another way to set modes is to use mode modifiers within the regular expression. Mode modifiers within the regex always override options or flags set outside the regular expression.
- JavaScript
  - In JavaScript, you can specify options by appending one or more single-letter flags to the RegExp literal, after the forward slash that terminates the regular expression.
  - When talking about these flags in documentation, they are usually written as /i and /m, even though the flag itself is only one letter. No additional slashes are added to specify regex mode flags.
  - When using the RegExp() constructor to compile a string into a regular expression, you can pass an optional second parameter with flags to the constructor. The second parameter should be a string with the letters of the options you want to set. Do not put any slashes into the string.
  - Free-spacing: Not supported by JavaScript.
  - Case insensitive: /i 
  - Dot matches line breaks: Not supported by JavaScript. 
  - ^ and $ match at line breaks: /m
  - If you want to apply a regular expression repeatedly to the same string (e.g., to iterate over all matches or to search and replace all matches instead of just the first) specify the /g or “global” flag.
- Ruby
  - In Ruby, you can specify options by appending one or more single-letter flags to the Regexp literal, after the forward slash that terminates the regular expression. When talking about these flags in documentation, they are usually written as /i and /m, even though the flag itself is only one letter. No additional slashes are added to specify regex mode flags.
  - When using the Regexp.new() factory to compile a string into a regular expression, you can pass an optional second parameter with flags to the constructor. The second parameter should be either nil to turn off all options, or a combination of constants from the Regexp class combined with the or operator.
  - Free-spacing: /r or Regexp::EXTENDED 
  - Case insensitive: /i or Regexp::IGNORECASE 
  - Dot matches line breaks: /m or Regexp::MULTILINE. Ruby indeed uses “m” and “multiline” here, whereas all the other flavors use “s” or “singleline” for “dot matches line breaks.” 
  - ^ and $ match at line breaks: The caret and dollar always match at line breaks in Ruby. You cannot turn this off. Use ‹\A› and ‹\Z› to match at the start or end of the subject string.
  - The Regexp.new() factory takes an optional third parameter to select the string encoding your regular expression supports. If you do not specify an encoding for your regular expression, it will use the same encoding as your source file. Most of the time, using the source file’s encoding is the right thing to do.
  - To select a coding explicitly, pass a single character for this parameter. The parameter is case-insensitive. Possible values are:
  - n: This stands for “None.” Each byte in your string is treated as one character. Use this for ASCII text.
  - e: Enables the “EUC” encoding for Far East languages. 
  - s: Enables the Japanese “Shift-JIS” encoding.
  - u: Enables UTF-8, which uses one to four bytes per character and supports all languages in the Unicode standard (which includes all living languages of any significance).
  - When using a literal regular expression, you can set the encoding with the modifiers /n, /e, /s, and /u. Only one of these modifiers can be used for a single regular expression. They can be used in combination with any or all of the /x, /i, and /m modifiers.

### 3.5 Test If a Match Can Be Found Within a Subject String

- You want to check whether a match can be found for a particular regular expression in a particular string. A partial match is sufficient. For instance, the regex `‹regex●pattern›` partially matches `The regex pattern can be found`. You don’t care about any of the details of the match. You just want to know whether the regex matches the string.
- JavaScript
```js
if (/regex pattern/.test(subject)) {
    // Successful match
} else {
    // Match attempt failed
}
```
- Ruby
```rb
if subject =~ /regex pattern/
    # Successful match
else
    # Match attempt failed
end
```
or
```rb
if /regex pattern/ =~ subject
    # Successful match
else
    # Match attempt failed
end
```
- The most basic task for a regular expression is to check whether a string matches the regex. In most programming languages, a partial match is sufficient for the match function to return true
- The match function will scan through the entire subject string to see whether the regular expression matches any part of it. The function returns true as soon as a match is found. It returns false only when it reaches the end of the string without finding any matches.
- The code examples in this recipe are useful for checking whether a string contains certain data. If you want to check whether a string fits a certain pattern in its entirety (e.g., for input validation), use the next recipe instead.
- JavaScript
  - To test whether a regular expression can match part of a string, call the test() method on your regular expression. Pass the subject string as the only parameter.
  - regexp.test() returns true if the regular expression matches part or all of the subject string, and false if it does not.
- Ruby
  - The =~ operator is the pattern-matching operator. Place it between a regular expression and a string to find the first regular expression match. The operator returns an integer with the position at which the regex match begins in the string. It returns nil if no match can be found.
  - This operator is implemented in both the Regexp and String classes. In Ruby 1.8, it doesn’t matter which class you place to the left and which to the right of the operator. In Ruby 1.9, doing so has a special side effect involving named capturing groups. Recipe3.9 explains this.

### 3.6 Test Whether a Regex Matches the Subject String Entirely

- You want to check whether a string fits a certain pattern in its entirety. That is, you want to check that the regular expression holding the pattern can match the string from start to end. For instance, if your regex is ‹regex●pattern›, it will match input text consisting of regex pattern but not the longer string The regex pattern can be found.
- JavaScript
```js
if (/^regex pattern$/.test(subject)) {
    // Successful match
} else {
    // Match attempt failed
}
```
- Ruby
```rb
if subject =~ /\Aregex pattern\Z/
    # Successful match
else
    # Match attempt failed
end
```
- Normally, a successful regular expression match tells you that the pattern you want is somewhere within the subject text. In many situations you also want to make sure it completely matches, with nothing else in the subject text. Probably the most common situation calling for a complete match is validating input. If a user enters a phone number or IP address but includes extraneous characters, you want to reject the input.
- The solutions that use the anchors ‹$› and ‹\Z› also work when you’re processing a file line by line (Recipe3.21), and the mechanism you’re using to retrieve the lines leaves the line breaks at the end of the line. As Recipe2.5 explains, these anchors also match before a final line break, essentially allowing the final line break to be ignored.
- JavaScript
  - JavaScript does not have a function for testing whether a regex matches a string entirely. The solution is to add ‹^› to the start of your regular expression, and ‹$› to the end of your regular expression. Make sure that you do not set the /m flag for your regular expression. Only without /m do the caret and dollar match only at the start and end of the subject string. When you set /m, they also match at line breaks in the middle of the string.
- Ruby
  - Ruby’s Regexp class does not have a function for testing whether a regex matches a string entirely. The solution is to add the start-of-string anchor ‹\A› to the start of your regular expression, and the end-of-string anchor ‹\Z› to the end of your regular expression. This way, the regular expression can only match a string either in its entirety or not at all. If your regular expression uses alternation, as in ‹one|two|three›, make sure to group the alternation before adding the anchors: ‹\A(?:one|two|three)\Z›.

### 3.7 Retrieve the Matched Text

- You have a regular expression that matches a part of the subject text, and you want to extract the text that was matched. If the regular expression can match the string more than once, you want only the first match. For example, when applying the regex ‹\d+› to the string Do you like 13 or 42?, 13 should be returned.
- JavaScript
```js
var result = subject.match(/\d+/);
if (result) {
    result = result[0];
} else {
    result = '';
}
```
- Ruby
```rb
if subject =~ /regex pattern/
    result = $&
else
    result = ""
end
```
or
```rb
matchobj = /regex pattern/.match(subject)
if matchobj
    result = matchobj[0]
else
    result = ""
end
```
- Extracting the part of a longer string that fits the pattern is another prime job for regular expressions. 
- All programming languages discussed in this book provide an easy way to get the first regular expression match from a string. The function will attempt the regular expression at the start of the string and continue scanning through the string until the regular expression matches.
- JavaScript
  - The string.match() method takes a regular expression as its only parameter. You can pass the regular expression as a literal regex, a regular expression object, or as a string. If you pass a string, string.match() creates a temporary regexp object. 
  - When the match attempt fails, string.match() returns null. This allows you to differentiate between a regex that finds no matches, and a regex that finds a zero-length match. It does mean that you cannot directly display the result, as “null” or an error about a null object may appear.
  - When the match attempt succeeds, string.match() returns an array with the details of the match. Element zero in the array is a string that holds the text matched by the regular expression.
  - `Make sure that you do not add the /g flag to your regular expression`. If you do, string.match() behaves differently, as Recipe3.10 explains.
- Ruby
  - Recipe3.8 explains the `$~` variable and the MatchData object. In a string context, this object evaluates to the text matched by the regular expression. In an array context, this object evaluates to an array with element number zero holding the overall regular expression match.
  - `$&` is a special read-only variable. It is an alias for $~[0], which holds a string with the text matched by the regular expression.

### 3.8 Determine the Position and Length of the Match

- Instead of extracting the substring matched by the regular expression, as shown in the previous recipe, you want to determine the starting position and length of the match. With this information, you can extract the match in your own code or apply whatever processing you fancy on the part of the original string matched by the regex.
- JavaScript
```js
var matchstart = -1;
var matchlength = -1;
var match = /\d+/.exec(subject);
if (match) {
    matchstart = match.index;
    matchlength = match[0].length;
}
```
- Ruby
```rb
if subject =~ /regex pattern/
    matchstart = $~.begin()
    matchlength = $~.end() - matchstart
end
```
or
```rb
matchobj = /regex pattern/.match(subject)
if matchobj
    matchstart = matchobj.begin()
    matchlength = matchobj.end() - matchstart
end
```
- JavaScript
  - Call the exec() method on a regexp object to get an array with details about the match. This array has a few additional properties. 
  - The index property stores the position in the subject string at which the regex match begins. If the match begins at the start of the string, index will be zero. Element zero in the array holds a string with the overall regex match. Get the length property of that string to determine the length of the match.
  - Do not use the lastIndex property of the array returned by exec() to determine the ending position of the match. In a strict JavaScript implementation, the lastIndex does not exist in the returned array at all, but only in the regexp object itself. You shouldn’t use regexp.lastIndex either. It is unreliable, due to cross-browser differences (see Recipe3.11 for more details). Instead, simply add up match.index and match[0].length to determine where the regex match ended.
- Ruby
  - Recipe3.5 uses the =~ operator to find the first regex match in a string. A side effect of this operator is that it fills the special $~ variable with an instance of the MatchData class. This variable is thread-local and method-local. That means you can use the contents of this variable until your method exits or until the next time you use the =~ operator in your method, without worrying that another thread or another method in your thread will overwrite it.
  - If you want to keep the details of multiple regex matches, call the match() method on a Regexp object. This method takes a subject string as its only parameter. It returns a MatchData instance when a match can be found, or nil otherwise. It also sets the $~ variable to the same MatchObject instance, but does not overwrite other MatchObject instances stored in other variables.
  - The MatchData object stores all the details about a regular expression match. Recipes 3.7 and 3.9 explain how to get the text matched by the regular expression and by capturing groups.
  - he begin() method returns the position in the subject string at which the regex match begins. end() returns the position of the first character after the regex match. offset() returns an array with the beginning and ending positions. These three methods take one parameter. Pass 0 to get the positions of the overall regex match, or pass a positive number to get the positions of the specified capturing group. For example, begin(1) returns the start of the first capturing group.
  - Do not use length() or size() to get the length of the match. Both these methods return the number of elements in the array that MatchData evaluates to in array context, as explained in Recipe3.9.

### 3.9. Retrieve Part of the Matched Text

- As in Recipe3.7, you have a regular expression that matches a substring of the subject text, but this time you want to match just one part of that substring. To isolate the part you want, you added a capturing group to your regular expression, as described in Recipe2.9.
- For example, the regular expression `‹http://([a-z0-9.-]+)›` matches http://www.regexcookbook.com in the string Please visit http://www.regexcookbook.com for more information. The part of the regex inside the first capturing group matches www.regexcookbook.com, and you want to retrieve the domain name captured by the first capturing group into a string variable.
- JavaScript
```js
var result;
var match = /http:\/\/([a-z0-9.-]+)/.exec(subject);
if (match) {
    result = match[1];
} else {
    result = "";
}
```
- Ruby
```rb
if subject =~ %r!http://([a-z0-9.-]+)!
    result = $1
else
    result = ""
end
```
OR
```rb
matchobj = %r!http://([a-z0-9.-]+)!.match(subject)
if matchobj
    result = matchobj[1]
else
    result = ""
end
```
- Recipe2.10 and Recipe2.21 explain how you can use numbered backreferences in the regular expression and the replacement text to match the same text again, or to insert part of the regex match into the replacement text. You can use the same reference numbers to retrieve the text matched by one or more capturing groups in your code.
- In regular expressions, capturing groups are numbered starting at one. Programming languages typically start numbering arrays and lists at zero. All programming languages discussed in this book that store capturing groups in an array or list use the same numbering for capturing groups as the regular expression, starting at one. The zeroth element in the array or list is used to store the overall regular expression match. This means that if your regular expression has three capturing groups, the array storing their matches will have four elements. Element zero holds the overall match, and elements one, two, and three store the text matched by the three capturing groups.
- JavaScript
  - As explained in the previous recipe, the exec() method of a regular expression object returns an array with details about the match. 
  - Element zero in the array holds the overall regex match. Element one holds the text matched by the first capturing group, element two stores the second group’s match, etc. 
  - If the regular expression cannot match the string at all, regexp.exec() returns null.
- Ruby
  - Recipe3.8 explains the `$~` variable and the MatchData object. In an array context, this object evaluates to an array with the text matched by all the capturing groups in your regular expression. 
  - Capturing groups are numbered starting at 1, just like backreferences in the regular expression. Element 0 in the array holds the overall regular expression match.
  - $1, $2, and beyond are special read-only variables. $1 is a shortcut to $~[1], which holds the text matched by the first capturing group. $2 retrieves the second group, and so on.
  - Ruby 1.9 adds support for named capture to the regular expression syntax. It also extends the `$~` variable and the MatchData object explained in Recipe3.8 to support named capture. `$~["name"]` or `matchobj["name"]` returns the text matched by the named group “name.” Call matchobj.begin("name") and matchobj.end("name") to retrieve the beginning and ending positions of the match of a named group.

### 3.10. Retrieve a List of All Matches

- All the preceding recipes in this chapter deal only with the first match that a regular expression can find in the subject string. But in many cases, a regular expression that partially matches a string can find another match in the remainder of the string. And there may be a third match after the second, and so on. For example, the regex ‹`\d+›` can find six matches in the subject string `The lucky numbers are 7, 13, 16, 42, 65, and 99`: 7, 13, 16, 42, 65, and 99.
- You want to retrieve the list of all substrings that the regular expression finds when it is applied repeatedly to the remainder of the string, after each match.
- JavaScript
  - `var list = subject.match(/\d+/g);`
- Ruby
  - `result = subject.scan(/\d+/)`
- The Matches() method of the Regex class applies the regular expression repeatedly to the string, until all matches have been found. It returns a MatchCollection object that holds all the matches. The subject string is always the first parameter. This is the string in which the regular expression will try to find a match. The first parameter must not be null. Otherwise, Matches() will throw an ArgumentNullException.
- JavaScript
  - This code calls string.match(), just like the JavaScript solution to Recipe3.7. There is one small but very important difference: `the /g flag`. Regex flags are explained in Recipe3.4. \
  - The /g flag tells the match() function to iterate over all matches in the string and put them into an array. In the code sample, list[0] will hold the first regex match, list[1] the second, and so on. Check list.length to determine the number of matches. If no matches can be found at all, string.match returns null as usual.
  - The elements in the array are strings. `When you use a regex with the /g flag, string.match() does not provide any further details about the regular expression match`. If you want to get match details for all regex matches, iterate over the matches as explained in Recipe3.11.
- Ruby
  - The scan() method of the String class takes a regular expression as its only parameter. It iterates over all the regular expression matches in the string. When called without a block, scan() returns an array of all regex matches.
  - If your regular expression does not contain any capturing groups, scan() returns an array of strings. The array has one element for each regex match, holding the text that was matched.
  - When there are capturing groups, scan() returns an array of arrays. The array has one element for each regex match. Each element is an array with the text matched by each of the capturing groups. Subelement zero holds the text matched by the first capturing group, subelement one holds the second capturing group, etc. The overall regex match is not included in the array. If you want the overall match to be included, enclose your entire regular expression with an extra capturing group:
  - Ruby does not provide an option to make scan() return an array of strings when the regex has capturing groups. Your only solution is to replace all named and numbered capturing groups with noncapturing groups.

### 3.11. Iterate over All Matches

- The previous recipe shows how a regex could be applied repeatedly to a string to get a list of matches. Now you want to iterate over all the matches in your own code.
- JavaScript
  - If your regular expression may yield a zero-length match, or if you’re simply not sure about that, make sure to work around cross-browser issues dealing with zero-length matches and exec():
```js
var regex = /\d+/g;
var match = null;
while (match = regex.exec(subject)) {
  // Don't let browsers get stuck in an infinite loop
  if (match.index == regex.lastIndex) regex.lastIndex++;
  // Here you can process the match stored in the match variable
}
```
- If you know for sure your regex can never find a zero-length match, you can iterate over the regex directly:
```js
var regex = /\d+/g;
var match = null;
while (match = regex.exec(subject)) {
  // Here you can process the match stored in the match variable
}
```
- Ruby
```rb
subject.scan(/\d+/) {|match|
    # Here you can process the match stored in the match variable
}
```
- JavaScript
  - Before you begin, make sure to specify the /g flag if you want to use your regex in a loop. This flag is explained in Recipe3.4
  - Note that while (/\d+/g.exec()) (looping over a literal regex with /g) also will get stuck in the same infinite loop, at least with certain JavaScript implementations, because the regular expression is recompiled during each iteration of the while loop. When the regex is recompiled, the starting position for the match attempt is reset to the start of the string. Assign the regular expression to a variable outside the loop, to make sure it is compiled only once.
  - The workaround is to increment lastIndex in your own code if the exec() function hasn’t already done this. The first JavaScript solution to this recipe shows you how. If you’re unsure, simply paste in this one line of code and be done with it.
- Ruby
  - The scan() method of the String class takes a regular expression as its only parameter and iterates over all the regular expression matches in the string. When it is called with a block, you can process each match as it is found.
  - If your regular expression does not contain any capturing groups, specify one iterator variable in the block. This variable will receive a string with the text matched by the regular expression.
  - If your regex does contain one or more capturing groups, list one variable for each group. The first variable will receive a string with the text matched by the first capturing group, the second variable receives the second capturing group, and so on. No variable will be filled with the overall regex match. If you want the overall match to be included, enclose your entire regular expression with an extra capturing group.
```rb
subject.scan(/(a)(b)(c)/) {|a, b, c|
    # a, b, and c hold the text matched by the three capturing groups
}
```
- If you list fewer variables than there are capturing groups in your regex, you will be able to access only those capturing groups for which you provided variables. If you list more variables than there are capturing groups, the extra variables will be set to nil.
- If you list only one iterator variable and your regex has one or more capturing groups, the variable will be filled with an array of strings. The array will have one string for each capturing group. If there is only one capturing group, the array will have a single element:
```rb
subject.scan(/(a)(b)(c)/) {|abc|
    # abc[0], abc[1], and abc[2] hold the text
    # matched by the three capturing groups
}
```

### 3.12. Validate Matches in Procedural Code

- Recipe3.10 shows how you can retrieve a list of all matches a regular expression can find in a string when it is applied repeatedly to the remainder of the string after each match
- Now you want to get a list of matches that meet certain extra criteria that you cannot (easily) express in a regular expression. For example, when retrieving a list of lucky numbers, you only want to retain those that are an integer multiple of 13.
- JavaScript
```js
var list = [];
var regex = /\d+/g;
var match = null;
while (match = regex.exec(subject)) {
    // Don't let browsers get stuck in an infinite loop
    if (match.index == regex.lastIndex) regex.lastIndex++;
    // Here you can process the match stored in the match variable
    if (match[0] % 13 == 0) {
        list.push(match[0]);
    }
}
```
- Ruby
```rb
list = []
subject.scan(/\d+/) {|match|
    list << match if (Integer(match) % 13 == 0)
}
```
- Regular expressions deal with text. Though the regular expression ‹\d+› matches what we call a number, to the regular expression engine it’s just a string of one or more digits.
- If you want to find specific numbers, such as those divisible by 13, it is much easier to write a general regex that matches all numbers, and then use a bit of procedural code to skip the regex matches you’re not interested in.

### 3.13. Find a Match Within Another Match

- You want to find all the matches of a particular regular expression, but only within certain sections of the subject string. Another regular expression matches each of the sections in the string.
- Suppose you have an HTML file in which various passages are marked as bold with <b> tags. You want to find all numbers marked as bold. If some bold text contains multiple numbers, you want to match all of them separately. For example, when processing the string 1 <b>2</b> 3 4 <b>5 6 7</b>, you want to find four matches: 2, 5, 6, and 7.
- JavaScript
```js
var result = [];
var outerRegex = /<b>([\s\S]*?)<\/b>/g;
var innerRegex = /\d+/g;
var outerMatch;
var innerMatches;
while (outerMatch = outerRegex.exec(subject)) {
    if (outerMatch.index == outerRegex.lastIndex)
        outerRegex.lastIndex++;
    innerMatches = outerMatch[1].match(innerRegex);
    if (innerMatches) {
        result = result.concat(innerMatches);
    }
}
```
- Ruby
```rb
list = []
subject.scan(/<b>(.*?)<\/b>/m) {|outergroups|
    list += outergroups[1].scan(/\d+/)
}
```
- Regular expressions are well suited for tokenizing input, but they are not well suited for parsing input. Tokenizing means to identify different parts of a string, such as numbers, words, symbols, tags, comments, etc. It involves scanning the text from left to right, trying different alternatives and quantities of characters to be matched. Regular expressions handle this very well.
- Parsing means to process the relationship between those tokens. For example, in a programming language, combinations of such tokens form statements, functions, classes, namespaces, etc. Keeping track of the meaning of the tokens within the larger context of the input is best left to procedural code. In particular, regular expressions cannot keep track of nonlinear context, such as nested constructs.
- Trying to find one kind of token within another kind of token is a task that people commonly try to tackle with regular expressions. A pair of HTML bold tags is easily matched with the regular expression ‹<b>(.*?)</b>›.[7] A number is even more easily matched with the regex ‹\d+›. But if you try to combine these into a single regex, you’ll end up with something rather different: `\d+(?=(?:(?!<b>).)*</b>)`
- Though the regular expression just shown is a solution to the problem posed by this recipe, it is hardly intuitive. Even a regular expression expert will have to carefully scrutinize the regex to determine what it does, or perhaps resort to a tool to highlight the matches. And this is the combination of just two simple regexes.
- A better solution is to keep the two regular expressions as they are and use procedural code to combine them. The resulting code, while a bit longer, is much easier to understand and maintain, and creating simple code is the reason for using regular expressions in the first place.
- Either way, using two regular expressions together in a loop will be faster than using the one regular expression with its nested lookahead groups. The latter requires the regex engine to do a whole lot of backtracking

### 3.14. Replace All Matches

- 