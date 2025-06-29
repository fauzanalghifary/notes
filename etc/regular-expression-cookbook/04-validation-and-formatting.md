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

### 4.5. Validate Traditional Date Formats, Excluding Invalid Dates

- You want to validate dates in the traditional formats mm/dd/yy, mm/dd/yyyy, dd/mm/yy, and dd/mm/yyyy, as shown in Recipe4.4. But this time, you also want to weed out invalid dates, such as February 31st.
- There are essentially two ways to accurately validate dates with a regular expression. One method is to use a simple regex that merely captures groups of numbers that look like a month/day/year combination, and then use procedural code to check whether the date is correct.
- The other method is to do everything with a regular expression.
- The pure regex solution is interesting only in situations where one regex is all you can use, such as when you’re using an application that offers one box to type in a regex. When programming, make things easier with a bit of extra code. This will be particularly helpful if you want to add extra checks on the date later.

### 4.6. Validate Traditional Time Formats

- You want to validate times in various traditional time formats, such as hh:mm and hh:mm:ss in both 12-hour and 24-hour formats.
- Hours and minutes, 12-hour clock:
  - `^(1[0-2]|0?[1-9]):([0-5]?[0-9])(●?[AP]M)?$`
- Hours and minutes, 24-hour clock:
  - `^(2[0-3]|[01]?[0-9]):([0-5]?[0-9])$`
- Hours, minutes, and seconds, 12-hour clock:
  - `^(1[0-2]|0?[1-9]):([0-5]?[0-9]):([0-5]?[0-9])(●?[AP]M)?$`
- Hours, minutes, and seconds, 24-hour clock:
  - `^(2[0-3]|[01]?[0-9]):([0-5]?[0-9]):([0-5]?[0-9])$`
- The question marks in all of the preceding regular expressions make leading zeros optional. Remove the question marks to make leading zeros mandatory.

Discussion

- Validating times is considerably easier than validating dates. Every hour has 60 minutes, and every minute has 60 seconds. This means we don’t need any complicated alternations in the regex
- We put parentheses around the parts of the regex that match the hours, minutes, and seconds. That makes it easy to retrieve the digits for the hours, minutes, and seconds, without the colons
- The parentheses around the hour part keeps two alternatives for the hour together. If you remove those parentheses, the regex won’t work correctly. Removing the parentheses around the minutes and seconds has no effect, other than making it impossible to retrieve their digits separately.
- In large text:
  - `\b(2[0-3]|[01]?[0-9]):([0-5]?[0-9])\b`

### 4.7. Validate ISO 8601 Dates and Times

- You want to match dates and/or times in the official ISO 8601 format, which is the basis for many standardized date and time formats. For example, in XML Schema, the built-in date, time, and dateTime types are all based on ISO 8601.

Dates

- `^([0-9]{4})-(1[0-2]|0[1-9])$`
- `^(?<year>[0-9]{4})-(?<month>1[0-2]|0[1-9])$`
- ISO 8601 allows hyphens to be omitted from calendar dates, making both 2010-08-20 and 20100820 valid representations of the same date. The following regex accounts for this, but also allows for invalid formats like YYYY-MMDD and YYYYMM-DD.
  - `^([0-9]{4})-?(1[0-2]|0[1-9])-?(3[01]|0[1-9]|[12][0-9])$`
  - `^(?<year>[0-9]{4})-?(?<month>1[0-2]|0[1-9])-?↵
(?<day>3[01]|0[1-9]|[12][0-9])$`
- Calendar date, such as 2008-08-30 or 20080830. The hyphens are optional. This regex uses a capturing group and a backreference to match YYYY-MM-DD or YYYYMMDD, but not YYYY-MMDD or YYYYMM-DD.
  - `^([0-9]{4})(-?)(1[0-2]|0[1-9])\2(3[01]|0[1-9]|[12][0-9])$`
  - `^(?<year>[0-9]{4})(?<hyphen>-?)(?<month>1[0-2]|0[1-9])\k<hyphen>(?<day>3[01]|0[1-9]|[12][0-9])$`
- Ordinal date (e.g., 2008-243). The hyphen is optional:
  - `^([0-9]{4})-?(36[0-6]|3[0-5][0-9]|[12][0-9]{2}|0[1-9][0-9]|00[1-9])$`

Weeks

- Week of the year (e.g., 2008-W35). The hyphen is optional:
  - `^([0-9]{4})-?W(5[0-3]|[1-4][0-9]|0[1-9])$`
  - `^(?<year>[0-9]{4})-?W(?<week>5[0-3]|[1-4][0-9]|0[1-9])$`
- Week date (e.g., 2008-W35-6). The hyphens are optional.
  - `^([0-9]{4})-?W(5[0-3]|[1-4][0-9]|0[1-9])-?([1-7])$`
  - `^(?<year>[0-9]{4})-?W(?<week>5[0-3]|[1-4][0-9]|0[1-9])-?(?<day>[1-7])$`

Times

- Hours and minutes (e.g., 17:21). The colon is optional:
  - `^(2[0-3]|[01][0-9]):?([0-5][0-9])$`
  - `^(?<hour>2[0-3]|[01][0-9]):?(?<minute>[0-5][0-9])$`
- Hours, minutes, and seconds (e.g., 17:21:59). The colons are optional:
  - `^(2[0-3]|[01][0-9]):?([0-5][0-9]):?([0-5][0-9])$`
  - `^(?<hour>2[0-3]|[01][0-9]):?(?<minute>[0-5][0-9]):?↵(?<second>[0-5][0-9])$`
- Time zone designator (e.g., Z, +07 or +07:00). The colons and the minutes are optional:
  - `^(Z|[+-](?:2[0-3]|[01][0-9])(?::?(?:[0-5][0-9]))?)$`
- Hours, minutes, and seconds with time zone designator (e.g., 17:21:59+07:00). All the colons are optional. The minutes in the time zone designator are also optional:
  - `^(2[0-3]|[01][0-9]):?([0-5][0-9]):?([0-5][0-9])(Z|[+-](?:2[0-3]|[01][0-9])(?::?(?:[0-5][0-9]))?)$`
  - `^(?<hour>2[0-3]|[01][0-9]):?(?<minute>[0-5][0-9]):?(?<second>[0-5][0-9])(?<timezone>Z|[+-](?:2[0-3]|[01][0-9])(?::?(?:[0-5][0-9]))?)$`

Date and time

- Calendar date with hours, minutes, and seconds (e.g., 2008-08-30 17:21:59 or 20080830 172159). A space is required between the date and the time. The hyphens and colons are optional. This regex matches dates and times that specify some hyphens or colons but omit others. This does not follow ISO 8601.
  - `^([0-9]{4})-?(1[0-2]|0[1-9])-?(3[01]|0[1-9]|[12][0-9])●(2[0-3]|[01][0-9]):?([0-5][0-9]):?([0-5][0-9])$`
  - `^(?<year>[0-9]{4})-?(?<month>1[0-2]|0[1-9])-?(?<day>3[01]|0[1-9]|[12][0-9])●(?<hour>2[0-3]|[01][0-9]):?(?<minute>[0-5][0-9]):?(?<second>[0-5][0-9])$`
- A more complicated solution is needed if we want to match date and time values that specify either all of the hyphens and colons, or none of them. The cleanest solution is to use conditionals. But only some flavors support conditionals.
  - `^([0-9]{4})(-)?(1[0-2]|0[1-9])(?(2)-)(3[01]|0[1-9]|[12][0-9])●(2[0-3]|[01][0-9])(?(2):)([0-5][0-9])(?(2):)([0-5][0-9])$`
- If conditionals are not available, then we have to use alternation to spell out the alternatives with and without delimiters.
  - `^(?:([0-9]{4})-?(1[0-2]|0[1-9])-?(3[01]|0[1-9]|[12][0-9])●(2[0-3]|[01][0-9]):?([0-5][0-9]):?([0-5][0-9])|([0-9]{4})(1[0-2]|0[1-9])(3[01]|0[1-9]|[12][0-9])●(2[0-3]|[01][0-9])([0-5][0-9])([0-5][0-9]))$`

XML Schema dates and times

- The date and time types defined in the XML Schema standard are based on the ISO 8601 standard. The date types allow negative years for years before the start of the calendar (B.C. years). It also allows for years with more than four digits, but not for years with fewer than four digits. Years with more than four digits must not have leading zeros.
  - `^(-?(?:[1-9][0-9]*)?[0-9]{4})-(1[0-2]|0[1-9])-(3[01]|0[1-9]|[12][0-9])(Z|[+-](?:2[0-3]|[01][0-9]):[0-5][0-9])?$`
  - `^(?<year>-?(?:[1-9][0-9]*)?[0-9]{4})-(?<month>1[0-2]|0[1-9])-(?<day>3[01]|0[1-9]|[12][0-9])(?<timezone>Z|[+-](?:2[0-3]|[01][0-9]):[0-5][0-9])?$`
- Time, with optional fractional seconds and time zone (e.g., 01:45:36 or 01:45:36.123+07:00). There is no limit on the number of digits for the fractional seconds. This is the XML Schema time type:
  - `^(2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9])(\.[0-9]+)?(Z|[+-](?:2[0-3]|[01][0-9]):[0-5][0-9])?$`
  - `^(?<hour>2[0-3]|[01][0-9]):(?<minute>[0-5][0-9]):(?<second>[0-5][0-9])(?<frac>\.[0-9]+)?(?<timezone>Z|[+-](?:2[0-3]|[01][0-9]):[0-5][0-9])?$`
- Date and time, with optional fractional seconds and time zone (e.g., 2008-08-30T01:45:36 or 2008-08-30T01:45:36.123Z). This is the XML Schema dateTime type:
  - `^(-?(?:[1-9][0-9]*)?[0-9]{4})-(1[0-2]|0[1-9])-(3[01]|0[1-9]|[12][0-9])T(2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9])(\.[0-9]+)?(Z|[+-](?:2[0-3]|[01][0-9]):[0-5][0-9])?$`
  - `^(?<year>-?(?:[1-9][0-9]*)?[0-9]{4})-(?<month>1[0-2]|0[1-9])-(?<day>3[01]|0[1-9]|[12][0-9])T(?<hour>2[0-3]|[01][0-9]):(?<minute>[0-5][0-9]):(?<second>[0-5][0-9])(?<ms>\.[0-9]+)?(?<timezone>Z|[+-](?:2[0-3]|[01][0-9]):[0-5][0-9])?$`

Discussion

- ISO 8601 defines a wide range of date and time formats. The regular expressions presented here cover the most common formats, but most systems that use ISO 8601 only use a subset
- For most regexes, we also show an alternative using named capture. Some of these date and time formats may be unfamiliar to you or your fellow developers. Named capture makes the regex easier to understand. .NET, Java 7, XRegExp, PCRE 7, Perl 5.10, and Ruby 1.9 support the ‹(?<name>⋯)› syntax used in the solutions in this recipe.
- If the delimiters are all the same, we can do this quite easily using a capturing group for the first delimiter and backreferences for the remaining delimiters. The “dates” subsection of the “Solution” section shows an example. For the first hyphen, we use ‹(-?)›, ‹(?<hyphen>-?)› or ‹(?P<hyphen>-?)› to match an optional hyphen and capture it into a named or numbered group
- When using numbered capture, make sure to use the correct number for the backreference.
- Only some flavors support conditionals. If conditionals are not available, the only solution is to use alternation to spell out the two alternatives with and without delimiters. The disadvantage of this solution is that it results in two capturing groups for each part of the date and time. Only one of the two sets of capturing groups will participate in the match. Code that uses this regex will have to check both groups.


#### 4.8. Limit Input to Alphanumeric Characters

- Your application requires that users limit their responses to one or more alphanumeric English characters (letters A–Z and a–z, and digits 0–9).
  - `^[A-Z0-9]+$`

Limit input to ASCII characters

- The following regular expression limits input to the 128 characters in the seven-bit ASCII character table.
  - `^[\x00-\x7F]+$`

Limit input to ASCII noncontrol characters and line breaks

- Use the following regular expression to limit input to visible characters and whitespace in the ASCII character table, excluding control characters. The line feed and carriage return characters (at positions 0x0A and 0x0D, respectively) are the most commonly used control characters, so they’re explicitly included using ‹\n› (line feed) and ‹\r› (carriage return):
  - `^[\n\r\x20-\x7E]+$`

Limit input to shared ISO-8859-1 and Windows-1252 characters

- ISO-8859-1 and Windows-1252 (often called ANSI) are two commonly used eight-bit character encodings that are both based on the Latin-1 standard (or more formally, ISO/IEC 8859-1). 
  - `^[\x00-\x7F\xA0-\xFF]+$`

Limit input to alphanumeric characters in any language

- `^[\p{L}\p{M}\p{Nd}]+$`
- This uses a character class that includes shorthands for all code points in the Unicode Letter, Mark, and Decimal Number categories, which follows the official Unicode definition of an alphanumeric character. The Mark category is included since marks are required for words of many languages. Marks are code points that are intended to be combined with other characters (for example, to form an accented version of a base letter).
- Unfortunately, Unicode categories are not supported by all of the regular expression flavors covered by this book.

### 4.9. Limit the Length of Text

- You want to test whether a string is composed of between 1 and 10 letters from A to Z.
- All the programming languages covered by this book provide a simple, efficient way to check the length of text
- However, using regular expressions to check text length can be useful in some situations, particularly when length is only one of multiple rules that determine whether the subject text fits the desired pattern
- RegEx: `^[A-Z]{1,10}$`

Discussion
```markdown
^         # Assert position at the beginning of the string.
[A-Z]     # Match one letter from A to Z
  {1,10}  #   between 1 and 10 times.
$         # Assert position at the end of the string.
```
- By combining the interval quantifier with the surrounding start- and end-of-string anchors, the regex will fail to match if the subject text’s length falls outside the desired range.

Limit the length of an arbitrary pattern

- As explained in Recipe2.16, lookaheads (and their counterpart, lookbehinds) are a special kind of assertion that, like ‹^› and ‹$›, match a position within the subject string and do not consume any characters.
- Lookaheads can be either positive or negative, which means they can check if a pattern follows or does not follow the current position in the match.
- A positive lookahead, written as ‹(?=⋯)›, can be used at the beginning of the pattern to ensure that the string is within the target length range. The remainder of the regex can then validate the desired pattern without worrying about text length
- `^(?=.{1,10}$).*` (not for JS)
- `^(?=[\S\s]{1,10}$)[\S\s]*`
- It is important that the ‹$› anchor appears inside the lookahead because the maximum length test works only if we ensure that there are no more characters after we’ve reached the limit. Because the lookahead at the beginning of the regex enforces the length range, the following pattern can then apply any additional validation rules. In this case, the pattern `‹.*›` (or `‹[\S\s]*›` in the version that adds native JavaScript support) is used to simply match the entire subject text with no added constraints.

Limit the number of nonwhitespace characters

- `^\s*(?:\S\s*){10,100}$`

Limit the number of words

- The following regex is very similar to the previous example of limiting the number of nonwhitespace characters, except that each repetition matches an entire word rather than a single nonwhitespace character. It matches between 10 and 100 words, skipping past any nonword characters, including punctuation and whitespace:
  - `^\W*(?:\w+\b\W*){10,100}$`
- In Java 4 to 6, JavaScript, PCRE, Python 2.x, and Ruby, the word character token ‹\w› in this regex will match only the ASCII characters A–Z, a–z, 0–9, and _, and therefore this cannot correctly count words that contain non-ASCII letters and numbers.
- A possible workaround is to reframe the regex to count whitespace rather than word character sequences, as shown here:
  - `^\s*(?:\S+(?:\s+|$)){10,100}$`
- In many cases, this will work the same as the previous solutions, although it’s not exactly equivalent. For example, one difference is that compounds joined by a hyphen, such as “far-reaching,” will now be counted as one word instead of two. The same applies to words with apostrophes, such as “don’t.”

### 4.10. Limit the Number of Lines in Text

- You need to check whether a string is composed of five or fewer lines, without regard for how many total characters appear in the string.
- The exact characters or character sequences used as line separators can vary depending on your operating system’s convention, application or user preferences, and so on. Crafting an ideal solution therefore raises questions about what conventions for indicating the start of a new line should be supported. The following solutions support the standard MS-DOS/Windows (‹\r\n›), legacy Mac OS (‹\r›), and Unix/Linux/BSD/OS X (‹\n›) line break conventions.
- `\A(?>[^\r\n]*(?>\r\n?|\n)){0,4}[^\r\n]*\z`
- JS: `^(?:[^\r\n]*(?:\r\n?|\n)){0,4}[^\r\n]*$`

Discussion
```markdown
\A          # Assert position at the beginning of the string.
(?>         # Group but don't capture or keep backtracking positions:
  [^\r\n]*  #   Match zero or more characters except CR and LF.
  (?>       #   Group but don't capture or keep backtracking positions:
    \r\n?   #     Match a CR, with an optional following LF (CRLF).
   |        #    Or:
    \n      #     Match a standalone LF character.
  )         #   End the noncapturing, atomic group.
){0,4}      # End group; repeat between zero and four times.
[^\r\n]*    # Match zero or more characters except CR and LF.
\z          # Assert position at the end of the string.
```
- The leading ‹\A› matches the position at the beginning of the string, and ‹\z› matches at the end. This helps to ensure that the entire string contains no more than five lines, because unless the regex is anchored to the start and end of the text, it can match any five lines within a longer string.

### 4.11. Validate Affirmative Responses

- You need to check a configuration option or command-line response for a positive value. You want to provide some flexibility in the accepted responses, so that true, t, yes, y, okay, ok, and 1 are all accepted in any combination of uppercase and lowercase.
- `^(?:1|t(?:rue)?|y(?:es)?|ok(?:ay)?)$`

Discussion
```markdown
^            # Assert position at the beginning of the string.
(?:          # Group but don't capture:
  1          #   Match "1".
 |           #  Or:
  t(?:rue)?  #   Match "t", optionally followed by "rue".
 |           #  Or:
  y(?:es)?   #   Match "y", optionally followed by "es".
 |           #  Or:
  ok(?:ay)?  #   Match "ok", optionally followed by "ay".
)            # End the noncapturing group.
$            # Assert position at the end of the string.
```
- It could be written in a number of ways. For example, `‹^(?:[1ty]|true|yes|ok(?:ay)?)$›` is an equally good approach
- Simply alternating between all seven values as `‹^(?:1|t|true|y|yes|ok|okay)$›` would also work fine, although for performance reasons it’s generally better to reduce the amount of alternation via the pipe ‹|› operator in favor of character classes and optional suffixes (using the ‹?› quantifier). 
- In this case, the performance difference is probably no more than a few microseconds, but it’s a good idea to keep regex performance issues in the back of your mind. Sometimes the difference between these approaches can surprise you.
- All of these examples surround the potential match values with a noncapturing group to limit the reach of the alternation operators. If we omit the grouping and instead use something like ‹^true|yes$›, the regex engine will search for “the start of the string followed by ‘true’” or “‘yes’ followed by the end of the string.” ‹^(?:true|yes)$› tells the regex engine to find the start of the string, then either “true” or “yes,” and then the end of the string.

### 4.12. Validate Social Security Numbers

- You need to check whether a user has entered a valid Social Security number in your application or website form.
- If you simply need to ensure that a string follows the basic Social Security number format and that obvious, invalid numbers are eliminated, the following regex provides an easy solution. If you need a more rigorous solution that checks with the Social Security Administration to determine whether the number belongs to a living person, refer to the section of this recipe.
- `^(?!000|666)[0-8][0-9]{2}-(?!00)[0-9]{2}-(?!0000)[0-9]{4}$`

Discussion

- United States Social Security numbers are nine-digit numbers in the format AAA-GG-SSSS:
- The first three digits were historically (prior to mid-2011) assigned by geographical region, and are thus called the area number. The area number cannot be 000, 666, or between 900 and 999. Digits four and five are called the group number and range from 01 to 99. The last four digits are serial numbers from 0001 to 9999.
```markdown
^            # Assert position at the beginning of the string.
(?!000|666)  # Assert that neither "000" nor "666" can be matched here.
[0-8]        # Match a digit between 0 and 8.
[0-9]{2}     # Match a digit, exactly two times.
-            # Match a literal "-".
(?!00)       # Assert that "00" cannot be matched here.
[0-9]{2}     # Match a digit, exactly two times.
-            # Match a literal "-".
(?!0000)     # Assert that "0000" cannot be matched here.
[0-9]{4}     # Match a digit, exactly four times.
$            # Assert position at the end of the string.
```
- The first set allows any number from 000 to 899, but uses the preceding negative lookahead ‹(?!000|666)› to rule out the specific values 000 and 666. This kind of restriction can be pulled off without lookahead, but having this tool in our arsenal dramatically simplifies the regex.
- If you wanted to remove 000 and 666 from the range of valid area numbers without using any sort of lookaround, you’d need to restructure `‹(?!000|666)[0-8][0-9]{2}›` as `‹(?:00[1-9]|0[1-9][0-9]|[1-578][0-9]{2}|6[0-57-9][0-9]|66[0-57-9])›`. This far less readable approach uses a series of numeric ranges, which you can read all about in Recipe6.7.

### 4.13. Validate ISBNs

- 