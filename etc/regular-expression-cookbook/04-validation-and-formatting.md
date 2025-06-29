# Chapter 4: Validation and Formatting

### 4.0. Intro

- This chapter contains recipes for validating and formatting common types of user input. 

### 4.1. Validate Email Address

- You have a form on your website or a dialog box in your application that asks the user for an email address. You want to use a regular expression to validate this email address before trying to send email to it. This reduces the number of emails returned to you as undeliverable.

Simple

- This first solution does a very simple check. It only validates that the string contains an at sign (@) that is preceded and followed by one or more nonwhitespace characters.
- `^\S+@\S+$`

Simple, with restrictions on characters

- The domain name, the part after the @ sign, is restricted to characters allowed in domain names. Internationalized domain names are not allowed. The local part, the part before the @ sign, is restricted to characters commonly used in email local parts, which is more restrictive than what most email clients and servers will accept:
- `^[A-Z0-9+_.-]+@[A-Z0-9.-]+$`

Simple, with all valid local part characters

- This regular expression expands the previous one by allowing a larger set of rarely used characters in the local part. Not all email software can handle all these characters, but we’ve included all the characters permitted by RFC 5322, which governs the email message format. Among the permitted characters are some that present a security risk if passed directly from user input to an SQL statement, such as the single quote (') and the pipe character (|). Be sure to escape sensitive characters when inserting the email address into a string passed to another program, in order to prevent security holes such as SQL injection attacks
- `^[A-Z0-9_!#$%&'*+/=?`{|}~^.-]+@[A-Z0-9.-]+$`

No leading, trailing, or consecutive dots

- Both the local part and the domain name can contain one or more dots, but no two dots can appear right next to each other. Furthermore, the first and last characters in the local part and in the domain name must not be dots
- `^[A-Z0-9_!#$%&'*+/=?`{|}~^-]+(?:\.[A-Z0-9_!#$%&'*+/=?`{|}~^-]+↵)*@[A-Z0-9-]+(?:\.[A-Z0-9-]+)*$`

Top-level domain has two to six letters

- This regular expression adds to the previous versions by specifying that the domain name must include at least one dot, and that the part of the domain name after the last dot can only consist of letters. That is, the domain must contain at least two levels, such as secondlevel.com or thirdlevel.secondlevel.com. The top-level domain (.com in these examples) must consist of two to six letters.
- `^[\w!#$%&'*+/=?`{|}~^-]+(?:\.[\w!#$%&'*+/=?`{|}~^-]+)*@↵
(?:[A-Z0-9-]+\.)+[A-Z]{2,6}$`

Discussion

- If you thought something as conceptually simple as validating an email address would have a simple one-size-fits-all regex solution, you’re quite wrong. This recipe is a prime example that before you can start writing a regular expression, you have to decide exactly what you want to match. There is no universally agreed-upon rule as to which email addresses are valid and which not. It depends on your definition of valid.
- Because you ultimately have to check whether the address exists by actually sending email to it, you can decide to use a simpler or more relaxed regular expression. Allowing invalid addresses to slip through may be preferable to annoying people by blocking valid addresses. For this reason, you may want to select the “simple” regular expression. Though it obviously allows many things that aren’t email addresses, such as #$%@.-, the regex is quick and simple, and will never block a valid email address.
- If you want to avoid sending too many undeliverable emails, while still not blocking any real email addresses, the regex in Top-level domain has two to six letters is a good choice.
- You have to consider how complex you want your regular expression to be. If you’re validating user input, you’ll likely want a more complex regex, because the user could type in anything. But if you’re scanning database files that you know contain only valid email addresses, you can use a very simple regex that merely separates the email addresses from the other data. Even the solution in the earlier subsection may be enough in this case.
- Finally, you have to consider how future-proof you want your regular expression to be
- This recipe illustrates how you can build a regular expression step-by-step

### 4.2. Validate and Format North American Phone Numbers

- You want to determine whether a user entered a North American phone number, including the local area code, in a common format. These formats include 1234567890, 123-456-7890, 123.456.7890, 123 456 7890, (123) 456 7890, and all related combinations. If the phone number is valid, you want to convert it to your standard format, (123) 456-7890, so that your phone number records are consistent.
- Regex:
  - `^\(?([0-9]{3})\)?[-.●]?([0-9]{3})[-.●]?([0-9]{4})$`
- Replacement:
  - `($1)●$2-$3`
- JS Example
```js
var phoneRegex = /^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$/;

if (phoneRegex.test(subjectString)) {
    var formattedPhoneNumber =
        subjectString.replace(phoneRegex, "($1) $2-$3");
} else {
    // Invalid phone number
}
```
- Discussion
```markdown
^        # Assert position at the beginning of the string.
\(       # Match a literal "("
  ?      #   between zero and one time.
(        # Capture the enclosed match to backreference 1:
  [0-9]  #   Match a digit
    {3}  #     exactly three times.
)        # End capturing group 1.
\)       # Match a literal ")"
  ?      #   between zero and one time.
[-. ]    # Match one hyphen, dot, or space
  ?      #   between zero and one time.
⋯        # [Match the remaining digits and separator.]
$        # Assert position at the end of the string.
```
- As we’ve repeatedly seen, parentheses are special characters in regular expressions, but in this case we want to allow a user to enter parentheses and have our regex recognize them. This is a textbook example of where we need a backslash to escape a special character so the regular expression treats it as literal input
- The parentheses that appear without backslashes are capturing groups and are used to remember the values matched within them so that the matched text can be recalled later
- Character classes allow you to match any one out of a set of characters. ‹[0-9]› is a character class that matches any digit.
- ‹[-.●]› is another character class, one that allows any one of three separators. It’s important that the hyphen appears first or last in this character class, because if it appeared between other characters, it would create a range, as with ‹[0-9]›. Another way to ensure that a hyphen inside a character class matches a literal version of itself is to escape it with a backslash.
- Finally, quantifiers allow you to repeatedly match a token or group. ‹{3}› is a quantifier that causes its preceding element to be matched exactly three times. The regular expression ‹[0-9]{3}› is therefore equivalent to ‹[0-9][0-9][0-9]›, but is shorter and hopefully easier to read

Find phone numbers in documents

- Two simple changes allow the previous regular expressions to match phone numbers within longer text:
  - `\(?\b([0-9]{3})\)?[-.●]?([0-9]{3})[-.●]?([0-9]{4})\b`
- Here, the ‹^› and ‹$› assertions that bound the regular expression to the beginning and end of the text have been removed. In their place, word boundary tokens (‹\b›) have been added to ensure that the matched text stands on its own and is not part of a longer number or word.
- Similar to ‹^› and ‹$›, ‹\b› is an assertion that matches a position rather than any actual text. Specifically, ‹\b› matches the position between a word character and either a nonword character or the beginning or end of the text. Letters, numbers, and underscore are all considered word characters
- Note that the first word boundary token appears after the optional, opening parenthesis. This is important because there is no word boundary to be matched between two nonword characters, such as the opening parenthesis and a preceding space character. The first word boundary is relevant only when matching a number without parentheses, since the word boundary always matches between the opening parenthesis and the first digit of a phone number.

Allow a leading “1”

- You can allow an optional, leading “1” for the country code (which covers the North American Numbering Plan region) via the addition shown in the following regex:
  - `^(?:\+?1[-.●]?)?\(?([0-9]{3})\)?[-.●]?([0-9]{3})[-.●]?([0-9]{4})$`
- It uses a noncapturing group, written as ‹(?:⋯)›. When a question mark follows an unescaped left parenthesis like this, it’s not a quantifier, but instead helps to identify the type of grouping.
- Standard capturing groups require the regular expression engine to keep track of backreferences, so it’s more efficient to use noncapturing groups whenever the text matched by a group does not need to be referenced later
- Another reason to use a noncapturing group here is to allow you to keep using the same replacement string as in the previous examples. If we added a capturing group, we’d have to change $1 to $2 (and so on) in the replacement text shown earlier in this recipe.

Allow seven-digit phone numbers

- To allow matching phone numbers that omit the local area code, enclose the first group of digits together with its surrounding parentheses and following separator in an optional, noncapturing group:
  - `^(?:\(?([0-9]{3})\)?[-.●]?)?([0-9]{3})[-.●]?([0-9]{4})$`
- Since the area code is no longer required as part of the match, simply replacing any match with «($1)●$2-$3» might now result in something like () 123-4567, with an empty set of parentheses. To work around this, add code outside the regex that checks whether group 1 matched any text, and adjust the replacement text accordingly.

### 4.3. Validate International Phone Numbers

- You want to validate international phone numbers. The numbers should start with a plus sign, followed by the country code and national number.
- Regex:
  - `^\+(?:[0-9]●?){6,14}[0-9]$`
- JS Example:
```js
function validate(phone) {
    var regex = /^\+(?:[0-9] ?){6,14}[0-9]$/;

    if (regex.test(phone)) {
        // Valid international phone number
    } else {
        // Invalid international phone number
    }
}
```

Discussion

- Fortunately, there is a simple, industry-standard notation specified by ITU-T E.123. This notation requires that international phone numbers include a leading plus sign (known as the international prefix symbol), and allows only spaces to separate groups of digits.
```md
^         # Assert position at the beginning of the string.
\+        # Match a literal "+" character.
(?:       # Group but don't capture:
  [0-9]   #   Match a digit.
  \x20    #   Match a space character
    ?     #     between zero and one time.
)         # End the noncapturing group.
  {6,14}  #   Repeat the group between 6 and 14 times.
[0-9]     # Match a digit.
$         # Assert position at the end of the string.
```

Validate international phone numbers in EPP format

- `^\+[0-9]{1,3}\.[0-9]{4,14}(?:x.+)?$`
- This regular expression follows the international phone number notation specified by the Extensible Provisioning Protocol (EPP). EPP is a relatively recent protocol (finalized in 2004), designed for communication between domain name registries and registrars. It is used by a growing number of domain name registries, including .com, .info, .net, .org, and .us. The significance of this is that EPP-style international phone numbers are increasingly used and recognized, and therefore provide a good alternative format for storing (and validating) international phone numbers.
- EPP-style phone numbers use the format +CCC.NNNNNNNNNNxEEEE, where C is the 1–3 digit country code, N is up to 14 digits, and E is the (optional) extension. The leading plus sign and the dot following the country code are required. The literal “x” character is required only if an extension is provided.

### 4.4. Validate Traditional Date Formats

- You want to validate dates in the traditional formats mm/dd/yy, mm/dd/yyyy, dd/mm/yy, and dd/mm/yyyy. You want to use a simple regex that simply checks whether the input looks like a date, without trying to weed out things such as February 31st.
- Match any of these date formats, allowing leading zeros to be omitted:
  - `^[0-3]?[0-9]/[0-3]?[0-9]/(?:[0-9]{2})?[0-9]{2}$`
- Match any of these date formats, requiring leading zeros:
  - `[0-3][0-9]/[0-3][0-9]/(?:[0-9][0-9])?[0-9][0-9]$`
- Match m/d/yy and mm/dd/yyyy, allowing any combination of one or two digits for the day and month, and two or four digits for the year:
  - `^(1[0-2]|0?[1-9])/(3[01]|[12][0-9]|0?[1-9])/(?:[0-9]{2})?[0-9]{2}$`
- Match mm/dd/yyyy, requiring leading zeros:
  - `^(1[0-2]|0[1-9])/(3[01]|[12][0-9]|0[1-9])/[0-9]{4}$`
- Match d/m/yy and dd/mm/yyyy, allowing any combination of one or two digits for the day and month, and two or four digits for the year:
  - `^(3[01]|[12][0-9]|0?[1-9])/(1[0-2]|0?[1-9])/(?:[0-9]{2})?[0-9]{2}$`
- Match dd/mm/yyyy, requiring leading zeros:
  - `^(3[01]|[12][0-9]|0[1-9])/(1[0-2]|0[1-9])/[0-9]{4}$`
- Match any of these date formats with greater accuracy, allowing leading zeros to be omitted:
  - `^(?:(1[0-2]|0?[1-9])/(3[01]|[12][0-9]|0?[1-9])|↵
(3[01]|[12][0-9]|0?[1-9])/(1[0-2]|0?[1-9]))/(?:[0-9]{2})?[0-9]{2}$`
  - We can use the free-spacing option to make this regular expression easier to read:
```markdown
^(?:
  # m/d or mm/dd
  (1[0-2]|0?[1-9])/(3[01]|[12][0-9]|0?[1-9])
|
  # d/m or dd/mm
  (3[01]|[12][0-9]|0?[1-9])/(1[0-2]|0?[1-9])
)
# /yy or /yyyy
/(?:[0-9]{2})?[0-9]{2}$
```
- `^(?:(1[0-2]|0[1-9])/(3[01]|[12][0-9]|0[1-9])|↵
(3[01]|[12][0-9]|0[1-9])/(1[0-2]|0[1-9]))/[0-9]{4}$`
- The same solution using the free-spacing option to make it easier to read:
```markdown
^(?:
  # mm/dd
  (1[0-2]|0[1-9])/(3[01]|[12][0-9]|0[1-9])
|
  # dd/mm
  (3[01]|[12][0-9]|0[1-9])/(1[0-2]|0[1-9])
)
# /yyyy
/[0-9]{4}$
```

Discussion

- The other issue is that regular expressions don’t deal directly with numbers. You can’t tell a regular expression to “match a number between 1 and 31”, for instance. Regular expressions work character by character. We use ‹3[01]|[12][0-9]|0?[1-9]› to match 3 followed by 0 or 1, or to match 1 or 2 followed by any digit, or to match an optional 0 followed by 1 to 9
- Because of this, you have to choose how simple or how accurate you want your regular expression to be
- The first two solutions for this recipe are quick and simple, too, and they also match invalid dates, such as 0/0/00 and 31/31/2008
- (?:[0-9]{2})?[0-9]{2}› allows the year to consist of two or four digits.
- ‹[0-9]{2}› matches exactly two digits
- ‹(?:[0-9]{2})?› matches zero or two digits. The noncapturing group (see Recipe2.9) is required, because the question mark needs to apply to the character class and the quantifier ‹{2}› combined
- We use alternation (see Recipe 2.8) inside a group to match various pairs of digits to form a range of two-digit numbers. We use capturing groups here because you’ll probably want to capture the day and month numbers anyway.
- If you want to search for dates in larger bodies of text instead of checking whether the input as a whole is a date, you cannot use the anchors ‹^› and ‹$›. Merely removing the anchors from the regular expression is not the right solution. That would allow any of these regexes to match 12/12/2001 within 9912/12/200199, for example. Instead of anchoring the regex match to the start and end of the subject, you have to specify that the date cannot be part of longer sequences of digits.
  - `\b(1[0-2]|0[1-9])/(3[01]|[12][0-9]|0[1-9])/[0-9]{4}\b`