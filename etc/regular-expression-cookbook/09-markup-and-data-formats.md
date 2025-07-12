# Chapter 9: Markup and Data Formats

### 9.0. Processing Markup and Data Formats with Regular Expressions

- This final chapter focuses on common tasks that come up when working with an assortment of common markup languages and data formats: HTML, XHTML, XML, CSV, and INI
- This final chapter focuses on common tasks that come up when working with an assortment of common markup languages and data formats: HTML, XHTML, XML, CSV, and INI
- When programming, it’s usually best to use dedicated parsers and APIs instead of regular expressions when performing many of the tasks in this chapter, especially if accuracy is paramount (e.g., if your processing might have security implications)
- However, we don’t ascribe to a dogmatic view that XML-style markup should never be processed with regular expressions. There are cases when regular expressions are a great tool for the job, such as when making one-time edits in a text editor, scraping data from a limited set of HTML files, fixing broken XML files, or dealing with file formats that look like but aren’t quite XML. There are some issues to be aware of, but reading through this chapter will ensure that you don’t stumble into them blindly.

HTML
- Although processing HTML using regular expressions is a popular task, you should know up front that the language is poorly suited to the rigidity and precision of regular expressions. This is especially true of the bastardized HTML that is common on many web pages, thanks in part to the extreme tolerance for poorly constructed HTML that web browsers are known for.

XHTML
- Because the syntax of XHTML is a subset of HTML (as of HTML5) and is formed from XML, many regular expressions in this chapter are written to support all three of these markup languages. Recipes that refer to “(X)HTML” handle HTML and XHTML equally. You usually cannot depend on a document using only HTML or XHTML conventions, since mix-ups are common and web browsers generally don’t mind.

XML
- XML is a general-purpose language designed primarily for sharing structured data. It is used as the foundation to create a wide array of markup languages, including XHTML
- There are many other rules that must be adhered to when authoring well-formed XML documents, or if you want to write your own conforming XML parser. However, the rules we’ve just described (appended to the structure we’ve already outlined for HTML documents) are generally enough for simple regex searches.
- Because the syntax of XML is very similar to HTML and forms the basis of XHTML, many regular expressions in this chapter are written to support all three markup languages. Recipes that refer to “XML-style” markup handle XML, XHTML, and HTML equally.

CSV
- The CSV format is supported by most spreadsheets and database management systems, and is especially popular for exchanging data between applications.

### 9.1. Find XML-Style Tags

- You want to match any HTML, XHTML, or XML tags in a string, in order to remove, modify, count, or otherwise deal with them.
- The most appropriate solution depends on several factors, including the level of accuracy, efficiency, and tolerance for erroneous markup that is acceptable to you. Once you’ve determined the approach that works for your needs, there are any number of things you might want to do with the results. 
- But whether you want to remove the tags, search within them, add or remove attributes, or replace them with alternative markup, the first step is to find them.
- Be forewarned that this will be a long recipe, fraught with subtleties, exceptions, and variations. If you’re looking for a quick fix and are not willing to put in the effort to determine the best solution for your needs, you might want to jump to the section of this recipe, which offers a decent mix of tolerance versus precaution.

Quick and dirty
- This first solution is simple and more commonly used than you might expect, but it’s included here mostly for comparison and for an examination of its flaws. It may be good enough when you know exactly what type of content you’re dealing with and are not overly concerned about the consequences of incorrect handling. This regex matches a < symbol, then simply continues until the first > occurs:
- `<[^>]*>`

Allow > in attribute values
- This next regex is again rather simplistic and does not handle all cases correctly. However, it might work well for your needs if it will be used to process only snippets of valid (X)HTML. It’s advantage over the previous regex is that it correctly passes over > characters that appear within attribute values:
- `<(?:[^>"']|"[^"]*"|'[^']*')*>`
- Here is the same regex, with added whitespace and comments for readability:
```markdown
<
(?: [^>"']   # Non-quoted character
  | "[^"]*"  # Double-quoted attribute value
  | '[^']*'  # Single-quoted attribute value
)*
>
```

(X)HTML tags (loose)
- In addition to supporting > characters embedded in attribute values, this next regex emulates the lenient rules for (X)HTML tags that browsers actually implement. This both improves accuracy with poorly formed markup and lets the regex avoid content that does not look like a tag, including comments, DOCTYPEs, and unencoded < characters in text
- To get these improvements, two main changes are made. First, there is extra handling that helps determine where attribute values start and end in edge cases, such as when tags contain stray quote marks as part of an unquoted attribute value or separate from any legit attribute. Second, special handling is added for the tag name, including requiring the name to begin with a letter A–Z. The tag name is captured to backreference 1 in case you need to refer back to it:
- `</?([A-Za-z][^\s>/]*)(?:=\s*(?:"[^"]*"|'[^']*'|[^\s>]+)|[^>])*(?:>|$)`
```markdown
<
/?                  # Permit closing tags
([A-Za-z][^\s>/]*)  # Capture the tag name to backreference 1
(?:                 # Attribute value branch:
  = \s*             #   Signals the start of an attribute value
  (?: "[^"]*"       #   Double-quoted attribute value
    | '[^']*'       #   Single-quoted attribute value
    | [^\s>]+       #   Unquoted attribute value
  )
|                   # Non-attribute-value branch:
  [^>]              #   Character outside of an attribute value
)*
(?:>|$)             # End of the tag or string
```

(X)HTML tags (strict)
- This regex is more complicated than those we’ve already seen in this recipe, because it actually follows the rules for (X)HTML tags explained in the introductory section of this chapter. This is not always desirable, since browsers don’t strictly adhere to these rules. In other words, this regex will avoid matching content that does not look like a valid (X)HTML tag, at the cost of possibly not matching some content that browsers would in fact interpret as a tag (e.g., if your markup uses an attribute name that includes characters not accounted for here, or if attributes are included in a closing tag)
- `<(?:([A-Z][-:A-Z0-9]*)(?:\s+[A-Z][-:A-Z0-9]*(?:\s*=\s*(?:"[^"]*"|'[^']*'|[^"'`=<>\s]+))?)*\s*/?|/([A-Z][-:A-Z0-9]*)\s*)>`
```markdown
<
(?:                    # Branch for opening tags:
  ([A-Z][-:A-Z0-9]*)   #   Capture the opening tag name to backreference 1
  (?:                  #   This group permits zero or more attributes
    \s+                #   Whitespace to separate attributes
    [A-Z][-:A-Z0-9]*   #   Attribute name
    (?: \s*=\s*        #   Attribute name-value delimiter
      (?: "[^"]*"      #   Double-quoted attribute value
        | '[^']*'      #   Single-quoted attribute value
        | [^"'`=<>\s]+ #   Unquoted attribute value (HTML)
      )
    )?                 #   Permit attributes without a value (HTML)
  )*
  \s*                  #   Permit trailing whitespace
  /?                   #   Permit self-closed tags
|                      # Branch for closing tags:
  /
  ([A-Z][-:A-Z0-9]*)   #   Capture the closing tag name to backreference 2
  \s*                  #   Permit trailing whitespace
)
>
```

XML tags (strict)
- XML is a precisely specified language, and requires that user agents strictly adhere to and enforce its rules. This is a stark change from HTML and the longsuffering browsers that process it. We’ve therefore included only a “strict” version for XML:
- `<(?:([_:A-Z][-.:\w]*)(?:\s+[_:A-Z][-.:\w]*\s*=\s*(?:"[^"]*"|'[^']*'))*\s*/?|/([_:A-Z][-.:\w]*)\s*)>`
```markdown
<
(?:                  # Branch for opening tags:
  ([_:A-Z][-.:\w]*)  #   Capture the opening tag name to backreference 1
  (?:                #   This group permits zero or more attributes
    \s+              #   Whitespace to separate attributes
    [_:A-Z][-.:\w]*  #   Attribute name
    \s*=\s*          #   Attribute name-value delimiter
    (?: "[^"]*"      #   Double-quoted attribute value
      | '[^']*'      #   Single-quoted attribute value
    )
  )*
  \s*                #   Permit trailing whitespace
  /?                 #   Permit self-closed tags
|                    # Branch for closing tags:
  /
  ([_:A-Z][-.:\w]*)  #   Capture the closing tag name to backreference 2
  \s*                #   Permit trailing whitespace
)
>
```

Discussion
- Although it’s common to want to match XML-style tags using regular expressions, doing it safely requires balancing trade-offs and thinking carefully about the data you’re working with. Because of these difficulties, some people choose to forgo the use of regular expressions for any sort of XML or (X)HTML processing in favor of specialized parsers and APIs
- That’s an approach you should seriously consider, since such tools are sometimes easier to use and typically include robust detection or handling for incorrect markup. In browser-land, for example, it’s usually best to take advantage of the tree-based Document Object Model (DOM) for your HTML search and manipulation needs
- If you want to sterilize HTML from untrusted sources because you’re worried about specially-crafted malicious HTML and cross-site scripting (XSS) attacks, your safest bet is to first convert all <, >, and & characters to their corresponding named character references (&lt;, &gt;, and &amp;), then bring back tags that are known to be safe (as long as they contain no attributes or only use those within a select list of approved attributes). For example, to bring back <p>, <em>, and <strong> tags with no attributes after replacing <, >, and & with character references, search case-insensitively using the regex ‹&lt;(/?)(p|em|strong)&gt;› and replace matches with «<$1$2>» (or in Python and Ruby, «<\1\2>»). If necessary, you can then safely search your modified string for HTML tags using the regexes in this recipe.
- Quick and dirty
  - The advantage of this solution is its simplicity, which makes it easy to remember and type, and also fast to run. The trade-off is that it incorrectly handles certain valid and invalid XML and (X)HTML constructs. If you’re working with markup you wrote yourself and know that such cases will never appear in your subject text, or if you are not concerned about the consequences if they do, this trade-off might be OK. Another example of where this solution might be good enough is when you’re working with a text editor that lets you preview regex matches.
  - If you’re processing anything more than the most basic markup, especially if the subject text is coming from mixed or unknown sources, you will be better served by one of the more robust solutions further along in this recipe.
- Allow > in attribute values
  - Nevertheless, it covers the basics needed to match XML-style tags, and thus it might work well for your needs if it will be used to process snippets of valid markup that include only elements and text. The difference from the last regex is that it passes over > characters that appear within attribute values. For example, it will correctly match the entire <input> tags in the example subject strings we’ve previously shown: <input type="button" value=">>"> and <input type="button" onclick="alert(2>1)">.
  - Note that this solution has no special handling that allows it to exclude or properly match comments and other special nodes in your documents. Make sure you’re familiar with the kind of content you’re working with before putting this regex to use.
- (X)HTML tags (loose)
  - Via a couple main changes, this regex gets a lot closer to emulating the easygoing rules that web browsers use to identify (X)HTML tags in source code. That makes it a good solution in cases where you’re trying to copy browser behavior or the HTML5 parsing algorithm and don’t care whether the tags you match actually follow all the rules for valid markup
  - Keep in mind that it’s still possible to create horrifically invalid HTML that this regex will not handle in the same way as one or more browsers, since browsers parse some edge cases of erroneous markup in their own, unique ways.
  - This regex’s most significant difference from the previous solution is that it requires the character following the opening left angle bracket (<) to be a letter A–Z or a–z, optionally preceded by / (for closing tags). This constraint rules out matching stray, unencoded < characters in text, as well as comments, DOCTYPEs, XML declarations and processing instructions, CDATA sections, and so on. That doesn’t protect it from matching something that looks like a tag but is actually within a comment, scripting language code, the content of a <textarea> element, or other similar situation where text is treated literally. The upcoming section, Skip Tricky (X)HTML and XML Sections, shows a workaround for this issue. But first, let’s look at how this regex works.
  - Any other characters (even including quote marks and the equals sign) are treated as part of the tag’s name. That might seem a bit overly permissive, but it’s how most browsers operate. Bogus tags might not have any effect on the way a page is rendered, but they nevertheless become accessible via the DOM tree and are not rendered as text, although any content within them will show up.
  - After the tag name comes the attribute handling, which is significantly changed from the previous solution in order to more accurately emulate browser-style parsing of edge cases with poorly formed markup
- (X)HTML tags (strict)
  - By saying that this solution is strict, we mean that it attempts to follow the HTML and XHTML syntax rules explained in the introductory section of this chapter, rather than emulating the rules browsers actually use when parsing the source code of a document.
- XML tags (strict)
  - This regex is basically a simpler version of the “(X)HTML tags (strict)” regex, since we’re able to remove support for two HTML features that are not allowed in XML: unquoted attribute values and minimized attributes (attributes without an accompanying value).

Skip Tricky (X)HTML and XML Sections
- When trying to match XML-style tags within a source file or string, much of the battle is avoiding content that looks like a tag, even though its placement or other context precludes it from being interpreted as a tag
- a robust solution requires that we also avoid any content that appears within comments, scripting language code (which may use greater-than and less-than symbols for mathematical operations), XML CDATA sections, and various other constructs. We can solve this issue by first searching for these problematic sections, and then searching for tags only in the content outside of those matches.

### 9.2. Replace <b> Tags with <strong>

- You want to replace all opening and closing <b> tags in a string with corresponding <strong> tags, while preserving any existing attributes.
- This regex matches opening and closing <b> tags, with or without attributes:
  - <(/?)b\b((?:[^>"']|"[^"]*"|'[^']*')*)>
```markdown
<
(/?)             # Capture the optional leading slash to backreference 1
b \b             # Tag name, with word boundary
(                # Capture any attributes, etc. to backreference 2
    (?: [^>"']   # Any character except >, ", or '
      | "[^"]*"  # Double-quoted attribute value
      | '[^']*'  # Single-quoted attribute value
    )*
)
>
```
- To preserve all attributes while changing the tag name, use the following replacement text:
  - `<$1strong$2>`
- The word boundary (‹\b›) that follows the tag name is easy to forget, but it’s one of the most important pieces of this regex. The word boundary lets us match only <b> tags, and not <br>, <body>, <blockquote>, or any other tags that merely start with the letter “b.”
- When working with XML and XHTML, be aware that the colon used for namespaces, as well as hyphens and some other characters allowed as part of XML names, create a word boundary. For example, the regex could end up matching something like <b-sharp>. If you’re worried about this, you might want to use the lookahead ‹(?=[\s/>])› instead of a word boundary. It achieves the same result of ensuring that we do not match partial tag names, and does so more reliably.
- After the tag name, the pattern ‹((?:[^>"']|"[^"]*"|'[^']*')*)› is used to match anything remaining within the tag up until the closing right angle bracket. Wrapping this pattern in a capturing group as we’ve done here lets us easily bring back any attributes and other characters (such as the trailing slash for singleton tags) in our replacement string
- Replace a list of tags
  - Place all of the desired tag names within a group, and alternate between them.
  - `<(/?)([bi]|em|big)\b((?:[^>"']|"[^"]*"|'[^']*')*)>`

### 9.3. Remove All XML-Style Tags Except <em> and <strong>

- You want to remove all tags in a string except <em> and <strong>.
- In a separate case, you not only want to remove all tags other than <em> and <strong>, you also want to remove <em> and <strong> tags that contain attributes.
- This is a perfect setting to put negative lookahead (explained in Recipe 2.16) to use. Applied to this problem, negative lookahead lets you match what looks like a tag, except when certain words come immediately after the opening < or </. If you then replace all matches with an empty string (following the code in Recipe 3.14), only the approved tags are left behind.
- Solution 1: Match tags except <em> and <strong>
  - `</?(?!(?:em|strong)\b)[a-z](?:[^>"']|"[^"]*"|'[^']*')*>`
```markdown
< /?                   # Permit closing tags
(?!
    (?: em | strong )  # List of tags to avoid matching
    \b                 # Word boundary avoids partial word matches
)
[a-z]                  # Tag name initial character must be a-z
(?: [^>"']             # Any character except >, ", or '
  | "[^"]*"            # Double-quoted attribute value
  | '[^']*'            # Single-quoted attribute value
)*
>
```
- Solution 2: Match tags except <em> and <strong>, and any tags that contain attributes
  - `</?(?!(?:em|strong)\s*>)[a-z](?:[^>"']|"[^"]*"|'[^']*')*>`
```markdown
< /?                   # Permit closing tags
(?!
    (?: em | strong )  # List of tags to avoid matching
    \s* >              # Only avoid tags if they contain no attributes
)
[a-z]                  # Tag name initial character must be a-z
(?: [^>"']             # Any character except >, ", or '
  | "[^"]*"            # Double-quoted attribute value
  | '[^']*'            # Single-quoted attribute value
)*
>
```
- Apart from the negative lookahead added to prevent some tags from being matched, these regexes are nearly equivalent to the “(X)HTML tags (loose)” regex from Recipe 9.1. The other main difference here is that we’re not capturing the tag name to backreference 1.
- So let’s look more closely at what’s new in this recipe. Solution 1 never matches <em> or <strong> tags, regardless of whether they have any attributes, but matches all other tags. Solution 2 matches all the same tags as Solution 1, and additionally matches <em> and <strong> tags that contain one or more attributes.
- Since the point of these regexes is to replace matches with empty strings (in other words, remove the tags), Solution 2 is less prone to abuse of the allowed <em> and <strong> tags to provide unexpected formatting or other shenanigans.
- This recipe has (until now) intentionally avoided the word “whitelist” when describing how only a few tags are left in place, since that word has security connotations. There are a variety of ways to work around this pattern’s constraints using specially crafted, malicious HTML strings. If you’re worried about malicious HTML and cross-site scripting (XSS) attacks, your safest bet is to convert all <, >, and & characters to their corresponding named character references (&lt;, &gt;, and &amp;), then bring back tags that are known to be safe (as long as they contain no attributes or only use those within a select list of approved attributes).
- Whitelist specific attributes
  - Consider these new requirements: you need to match all tags except <a>, <em>, and <strong>, with two exceptions. Any <a> tags that have attributes other than href or title should be matched, and if <em> or <strong> tags have any attributes at all, match them too. All matched strings will be removed.
  - In other words, you want to remove all tags except those on your whitelist (<a>, <em>, and <strong>). The only whitelisted attributes are href and title, and they are allowed only within <a> tags. If a nonwhitelisted attribute appears in any tag, the entire tag should be removed.
  - `<(?!(?:em|strong|a(?:\s+(?:href|title)\s*=\s*(?:"[^"]*"|'[^']*'))*)\s*>)[a-z](?:[^>"']|"[^"]*"|'[^']*')*>`
```markdown
< /?          # Permit closing tags
(?!
  (?: em      # Dont match <em>
    | strong  #   or <strong>
    | a       #   or <a>
      (?:     # Only avoid matching <a> tags that use only
        \s+   #   href and/or title attributes
        (?:href|title)
        \s*=\s*
        (?:"[^"]*"|'[^']*')  # Quoted attribute value
      )*
  )
  \s* >       # Only avoid matching these tags when they're
)             #   limited to any attributes permitted above
[a-z]         # Tag name initial character must be a-z
(?: [^>"']    # Any character except >, ", or '
  | "[^"]*"   # Double-quoted attribute value
  | '[^']*'   # Single-quoted attribute value
)*
>
```
- This pushes the boundary of where it makes sense to use such a complicated regex. If your rules get any more complex than this, it would probably be better to write some code based on Recipe 3.11 or 3.16 that checks the value of each matched tag to determine how to process it (based on the tag name, included attributes, or whatever else is needed).

### 9.4. Match XML Names

- You want to check whether a string is a legitimate XML name (a common syntactic construct). XML provides precise rules for the characters that can occur in a name, and reuses those rules for element, attribute, and entity names, processing instruction targets, and more. Names must be composed of a letter, underscore, or colon as the first character, followed by any combination of letters, digits, underscores, colons, hyphens, and periods. That’s actually an approximate description, but it’s pretty close. The exact list of permitted characters depends on the version of XML in use.
- XML 1.0 names (approximate)
  - `^[:_\p{Ll}\p{Lu}\p{Lt}\p{Lo}\p{Nl}][:_\-.\p{L}\p{M}\p{Nd}\p{Nl}]*$`
- XML 1.1 names (exact)
  - `^[:_A-Za-z\u00C0-\u00D6\u00D8-\u00F6\u00F8-\u02FF\u0370-\u037D\u037F-\u1FFF\u200C\u200D\u2070-\u218F\u2C00-\u2FEF\u3001-\uD7FF\uF900-\uFDCF\uFDF0-\uFFFD][:_\-.A-Za-z0-9\u00B7\u00C0-\u00D6\u00D8-\u00F6\u00F8-\u036F\u0370-\u037D\u037F-\u1FFF\u200C\u200D\u203F\u2040\u2070-\u218F\u2C00-\u2FEF\u3001-\uD7FF\uF900-\uFDCF\uFDF0-\uFFFD]*$`

### 9.5. Convert Plain Text to HTML by Adding <p> and <br> Tags

- Given a plain text string, such as a multiline value submitted via a form, you want to convert it to an HTML fragment to display within a web page. Paragraphs, separated by two line breaks in a row, should be surrounded with <p>⋯</p>. Additional line breaks should be replaced with <br> tags.
- Step 1: Replace HTML special characters with named character references
  - As we’re converting plain text to HTML, the first step is to convert the three special HTML characters &, <, and > to named character references (see Table 9-3). Otherwise, the resulting markup could lead to unintended results when displayed in a web browser.
  - `&` => &amp;
  - `<` => &lt;
  - `>` => &gt;
- Step 2: Replace all line breaks with <br>
  - replace `\r\n?|\n` with `<br>`
- Step 3: Replace double <br> tags with </p><p>
  - replace `<br>\s*<br>` with `</p><p>`
- Step 4: Wrap the entire string with <p>⋯</p>
  - This step is a simple string concatenation, and doesn’t require regular expressions.
```js
function htmlFromPlainText(subject) {
    // Step 1 (plain text searches)
    subject = subject.replace(/&/g, "&amp;").
                      replace(/</g, "&lt;").
                      replace(/>/g, "&gt;");

    // Step 2
    subject = subject.replace(/\r\n?|\n/g, "<br>");

    // Step 3
    subject = subject.replace(/<br>\s*<br>/g, "</p><p>");

    // Step 4
    subject = "<p>" + subject + "</p>";

    return subject;
}

// Run some tests...
htmlFromPlainText("Test.");            // -> "<p>Test.</p>"
htmlFromPlainText("Test.\n");          // -> "<p>Test.<br></p>"
htmlFromPlainText("Test.\n\n");        // -> "<p>Test.</p><p></p>"
htmlFromPlainText("Test1.\nTest2.");   // -> "<p>Test1.<br>Test2.</p>"
htmlFromPlainText("Test1.\n\nTest2."); // -> "<p>Test1.</p><p>Test2.</p>"
htmlFromPlainText("< AT&T >");         // -> "<p>&lt; AT&amp;T &gt;</p>"
```

### 9.6. Decode XML Entities

- You want to convert all character entities defined by the XML standard to their corresponding literal characters. The conversion should handle named character references (such as &amp;, &lt;, and &quot;) as well as numeric character references (be they in decimal notation as &#0931; or &#931;, or in hexadecimal notation as &#x03A3;, &#x3A3;, or &#x3a3;).
- Regex
  - `&(?:#([0-9]+)|#x([0-9a-fA-F]+)|([0-9a-zA-Z]+));`
- Replace matches with their corresponding literal characters
```js
// Accepts the match ($0) and backreferences; returns replacement text
function callback($0, $1, $2, $3) {
    var charCode;

    // Name lookup object that maps to decimal character codes
    // Equivalent hexadecimal numbers are listed in comments
    var names = {
        quot: 34, // 0x22
        amp: 38, // 0x26
        apos: 39, // 0x27
        lt: 60, // 0x3C
        gt: 62 // 0x3E
    };

    // Decimal character reference
    if ($1) {
        charCode = parseInt($1, 10);
    // Hexadecimal character reference
    } else if ($2) {
        charCode = parseInt($2, 16);
    // Named entity with a lookup mapping
    } else if ($3 && ($3 in names)) {
        charCode = names[$3];
    // Invalid or unknown entity name
    } else {
        return $0; // Return the match unaltered
    }

    // Return a literal character
    return String.fromCharCode(charCode);
}

// Replace all entities with literal text
subject = subject.replace(
        /&(?:#([0-9]+)|#x([0-9a-fA-F]+)|([0-9a-zA-Z]+));/g,
        callback);
```

### 9.7. Find a Specific Attribute in XML-Style Tags

- Within an (X)HTML or XML file, you want to find tags that contain a specific attribute, such as id.
- Tags that contain an id attribute (quick and dirty)
  - `<[^>]+\sid\b[^>]*>`
```markdown
<         # Start of the tag
[^>]+     # Tag name, attributes, etc.
\s id \b  # The target attribute name, as a whole word
[^>]*     # The remainder of the tag, including the id attribute's value
>         # End of the tag
```
- Tags that contain an id attribute (more reliable)
  - `<(?:[^>"']|"[^"]*"|'[^']*')+?\sid\s*=\s*("[^"]*"|'[^']*')(?:[^>"']|"[^"]*"|'[^']*')*>`
```markdown
<
(?: [^>"']             # Tag and attribute names, etc.
  | "[^"]*"            #   and quoted attribute values
  | '[^']*'
)+?
\s id                  # The target attribute name, as a whole word
\s* = \s*              # Attribute name-value delimiter
( "[^"]*" | '[^']*' )  # Capture the attribute value to backreference 1
(?: [^>"']             # Any remaining characters
  | "[^"]*"            #   and quoted attribute values
  | '[^']*'
)*
>
```
- <div> tags that contain an id attribute
  
