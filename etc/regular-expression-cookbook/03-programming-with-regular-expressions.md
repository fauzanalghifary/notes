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
- 