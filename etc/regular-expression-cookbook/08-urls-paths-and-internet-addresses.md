# Chapter 8: URLs, Paths, and Internet Addresses

### 8.1. Validating URLs

- You want to check whether a given piece of text is a URL that is valid for your purposes.
- Allow almost any URL:
  - `^(https?|ftp|file)://.+$`
- Require a domain name, and don’t allow a username or password:
  - `^(https?|ftp)://[a-z0-9-]+(\.[a-z0-9-]+)+([/?].+)?$`
- Require a domain name, and don’t allow a username or password. Allow the scheme (http or ftp) to be omitted if it can be inferred from the subdomain (www or ftp):
  - `^((https?|ftp)://|(www|ftp)\.)[a-z0-9-]+(\.[a-z0-9-]+)+([/?].*)?$`
- Require a domain name and a path that points to an image file. Don’t allow a username, password, or parameters:
  - `^(https?|ftp)://[a-z0-9-]+(\.[a-z0-9-]+)+(/[\w-]+)*/[\w-]+\.(gif|png|jpg)$`
- You cannot create a regular expression that matches every valid URL without matching any invalid URLs. The reason is that pretty much anything could be a valid URL in some as of yet uninvented scheme.
- Validating URLs becomes useful only when we know the context in which those URLs have to be valid.
- ‹.+$› simply grabs everything until the end of the string, as long as the string doesn’t contain any line break characters.

### 8.2. Finding URLs Within Full Text

- You want to find URLs in a larger body of text. URLs may or may not be enclosed in punctuation, such as parentheses, that are not part of the URL.
- URL without spaces:
  - `\b(https?|ftp|file)://\S+`
- URL without spaces or final punctuation:
  - `\b(https?|ftp|file)://[-A-Z0-9+&@#/%?=~_|$!:,.;]*[A-Z0-9+&@#/%=~_|$]`
- URL without spaces or final punctuation. URLs that start with the www or ftp subdomain can omit the scheme:
  - `\b((https?|ftp|file)://|(www|ftp)\.)[-A-Z0-9+&@#/%?=~_|$!:,.;]*[A-Z0-9+&@#/%=~_|$]`

### 8.3.Finding Quoted URLs in Full Text

- You want to find URLs in a larger body of text. URLs may or may not be enclosed in punctuation that is part of the larger body of text rather than part of the URL. You want to give users the option to place URLs between quotation marks, so they can explicitly indicate whether punctuation, or even spaces, should be part of the URL.
```markdown
\b(?:(?:https?|ftp|file)://|(www|ftp)\.)[-A-Z0-9+&@#/%?=~_|$!:,.;]*
                                        [-A-Z0-9+&@#/%=~_|$]
|"(?:(?:https?|ftp|file)://|(www|ftp)\.)[^"\r\n]+"
|'(?:(?:https?|ftp|file)://|(www|ftp)\.)[^'\r\n]+'
```

### 8.4. Finding URLs with Parentheses in Full Text

- You want to find URLs in a larger body of text. URLs may or may not be enclosed in punctuation that is part of the larger body of text rather than part of the URL. You want to correctly match URLs that include pairs of parentheses as part of the URL, without matching parentheses placed around the entire URL.
- `\b(?:(?:https?|ftp|file)://|www\.|ftp\.)(?:\([-A-Z0-9+&@#/%=~_|$?!:,.]*\)|[-A-Z0-9+&@#/%=~_|$?!:,.])*(?:\([-A-Z0-9+&@#/%=~_|$?!:,.]*\)|[A-Z0-9+&@#/%=~_|$])`
- Pretty much any character is valid in URLs, including parentheses. Parentheses are very rare in URLs, however, and that’s why we don’t include them in any of the regular expressions in the previous recipes. But certain important websites have started using them:

### 8.5. Turn URLs into Links

- You have a body of text that may contain one or more URLs. You want to convert the URLs that it contains into links by placing HTML anchor tags around the URLs. The URL itself will be both the destination for the link and the text being linked.
- Replacement Text:
  - `<a●href="$&">$&</a>`

### 8.6. Validating URNs

- You want to check whether a string represents a valid Uniform Resource Name (URN), as specified in RFC 2141, or find URNs in a larger body of text.
```markdown
\Aurn:
# Namespace Identifier
[a-z0-9][a-z0-9-]{0,31}:
# Namespace Specific String
[a-z0-9()+,\-.:=@;$_!*'%/?#]+
\Z
```
- `^urn:[a-z0-9][a-z0-9-]{0,31}:[a-z0-9()+,\-.:=@;$_!*'%/?#]+$`
- A URN consists of three parts. The first part is the four characters urn:, which we can add literally to the regular expression.
- The second part is the Namespace Identifier (NID). It is between 1 and 32 characters long
- The third part of the URN is the Namespace Specific String (NSS). It can be of any length, and can include a bunch of punctuation characters in addition to letters and digits.

### 8.7. Validating Generic URLs

- You want to check whether a given piece of text is a valid URL according to RFC 3986.
```markdown
\A
(# Scheme
 [a-z][a-z0-9+\-.]*:
 (# Authority & path
  //
  ([a-z0-9\-._~%!$&'()*+,;=]+@)?              # User
  ([a-z0-9\-._~%]+                            # Named host
  |\[[a-f0-9:.]+\]                            # IPv6 host
  |\[v[a-f0-9][a-z0-9\-._~%!$&'()*+,;=:]+\])  # IPvFuture host
  (:[0-9]+)?                                  # Port
  (/[a-z0-9\-._~%!$&'()*+,;=:@]+)*/?          # Path
 |# Path without authority
  (/?[a-z0-9\-._~%!$&'()*+,;=:@]+(/[a-z0-9\-._~%!$&'()*+,;=:@]+)*/?)?
 )
|# Relative URL (no scheme or authority)
 (# Relative path
  [a-z0-9\-._~%!$&'()*+,;=@]+(/[a-z0-9\-._~%!$&'()*+,;=:@]+)*/?
 |# Absolute path
  (/[a-z0-9\-._~%!$&'()*+,;=:@]+)+/?
 )
)
# Query
(\?[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?
# Fragment
(\#[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?
\Z
```
- `^([a-z][a-z0-9+\-.]*:(\/\/([a-z0-9\-._~%!$&'()*+,;=]+@)?([a-z0-9\-._~%]+|↵
\[[a-f0-9:.]+\]|\[v[a-f0-9][a-z0-9\-._~%!$&'()*+,;=:]+\])(:[0-9]+)?↵
(\/[a-z0-9\-._~%!$&'()*+,;=:@]+)*\/?|(\/?[a-z0-9\-._~%!$&'()*+,;=:@]+↵
(\/[a-z0-9\-._~%!$&'()*+,;=:@]+)*\/?)?)|([a-z0-9\-._~%!$&'()*+,;=@]+↵
(\/[a-z0-9\-._~%!$&'()*+,;=:@]+)*\/?|(\/[a-z0-9\-._~%!$&'()*+,;=:@]+)↵
+\/?))
(\?[a-z0-9\-._~%!$&'()*+,;=:@\/?]*)?(#[a-z0-9\-._~%!$&'()*+,;=:@\/?]*)?$`
- The regular expressions in this recipe deal with generic URLs. They’re not intended for searching for URLs in larger text, but for validating strings that are supposed to hold URLs, and for splitting URLs into their various parts. They accomplish these tasks for any kind of URL, but in practice, you’ll likely want to make the regexes more specific. The recipes after this one show examples of more specific regexes.
- As a result, RFC 3986 is very broad, and a regular expression that implements it is quite long. The regular expressions in this recipe only implement the basics. They’re enough to reliably split the URL into its various parts, but not to validate each of those parts. Validating all the parts would require specific knowledge of each URL scheme anyway.
- RFC 3986 does not cover all URLs that you may encounter in the wild. For example, many browsers and web servers accept URLs with literal spaces in them, but RFC 3986 requires spaces to be escaped as %20.
- An absolute URL must begin with a scheme, such as http: or ftp:. The first character of the scheme must be a letter. The following characters may be letters, digits, and a few specific punctuation characters. We can easily match that with two character classes: ‹[a-z][a-z0-9+\-.]*›.
- Many URL schemes require what RFC 3986 calls an “authority.” The authority is the domain name or IP address of the server, optionally preceded by a username, and optionally followed by a port number.
- The username can consist of letters, digits, and a bunch of punctuation. It must be delimited from the domain name or IP address with an @ sign. ‹[a-z0-9\-._~%!$&'()*+,;=]+@› matches the username and delimiter.
- RFC 3986 is quite liberal in what it allows for the domain name. Recipe 8.15 explains what is commonly allowed for domains: letters, digits, hyphens, and dots. RFC 3986 also allows tildes, and any other character via the percentage notation. The domain name must be converted to UTF-8, and any byte that is not a letter, digit, hyphen, or tilde must be encoded as %FF, where FF is the hexadecimal representation of the byte.
- The port number, if specified, is simply a decimal number separated from the hostname with a colon. ‹:[0-9]+› is all we need.
- The query part of the URL is optional. If present, it must start with a question mark. The query runs until the first hash sign in the URL or until the end of the URL. Since the hash sign is not among valid punctuation characters for the query part of the URL, we can easily match this with ‹\?[a-z0-9\-._~%!$&'()*+,;=:@/?]*›. Both of the question marks in this regex are literal characters. The first one is outside a character class, and must be escaped. The second one is inside a character class, where it is always a literal character.
- The final part of a URL is the fragment, which is also optional. It begins with a hash sign and runs until the end of the URL. ‹\#[a-z0-9\-._~%!$&'()*+,;=:@/?]*› matches this.

### 8.8. Extracting the Scheme from a URL

- You want to extract the URL scheme from a string that holds a URL. For example, you want to extract http from http://www.regexcookbook.com.
- Extract the scheme from a URL known to be valid
  - `^([a-z][a-z0-9+\-.]*):`
- Extract the scheme while validating the URL
```markdown
\A
([a-z][a-z0-9+\-.]*):
(# Authority & path
 //
 ([a-z0-9\-._~%!$&'()*+,;=]+@)?              # User
 ([a-z0-9\-._~%]+                            # Named host
 |\[[a-f0-9:.]+\]                            # IPv6 host
 |\[v[a-f0-9][a-z0-9\-._~%!$&'()*+,;=:]+\])  # IPvFuture host
 (:[0-9]+)?                                  # Port
 (/[a-z0-9\-._~%!$&'()*+,;=:@]+)*/?          # Path
|# Path without authority
 (/?[a-z0-9\-._~%!$&'()*+,;=:@]+(/[a-z0-9\-._~%!$&'()*+,;=:@]+)*/?)?
)
# Query
(\?[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?
# Fragment
(\#[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?
\Z
```
- Extracting the scheme from a URL is easy if you already know that your subject text is a valid URL. A URL’s scheme always occurs at the very start of the URL. The caret (Recipe2.5) specifies that requirement in the regex. The scheme begins with a letter, which can be followed by additional letters, digits, plus signs, hyphens, and dots. We match this with the two character classes ‹[a-z][a-z0-9+\-.]*› (Recipe2.3).

### 8.9. Extracting the User from a URL

- You want to extract the user from a string that holds a URL. For example, you want to extract jan from ftp://jan@www.regexcookbook.com.
- Extract the user from a URL known to be valid
  - `^[a-z0-9+\-.]+://([a-z0-9\-._~%!$&'()*+,;=]+)@`
- Extract the user while validating the URL
```markdown
\A
[a-z][a-z0-9+\-.]*://                       # Scheme
([a-z0-9\-._~%!$&'()*+,;=]+)@               # User
([a-z0-9\-._~%]+                            # Named host
|\[[a-f0-9:.]+\]                            # IPv6 host
|\[v[a-f0-9][a-z0-9\-._~%!$&'()*+,;=:]+\])  # IPvFuture host
(:[0-9]+)?                                  # Port
(/[a-z0-9\-._~%!$&'()*+,;=:@]+)*/?          # Path
(\?[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?         # Query
(\#[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?         # Fragment
\Z
```
- The username, if present in the URL, occurs right after the scheme and the two forward slashes that begin the “authority” part of the URL. The username is separated from the hostname that follows it with an @ sign. Since @ signs are not valid in hostnames, we can be sure that we’re extracting the username portion of a URL if we find an @ sign after the two forward slashes and before the next forward slash in the URL. Forward slashes are not valid in usernames, so we don’t need to do any special checking for them.
- All these rules mean we can very easily extract the username if we know the URL to be valid. We just skip over the scheme with ‹[a-z0-9+\-.]+› and the ://. Then, we grab the username that follows. If we can match the @ sign, we know that the characters before it are the username. The character class ‹[a-z0-9\-._~%!$&'()*+,;=]› lists all the characters that are valid in usernames.
- If you don’t already know that your subject text is a valid URL, you can use a simplified version of the regex from Recipe 8.7. Since we want to extract the user, we can exclude URLs that don’t specify an authority. The regex in the solution actually matches only URLs that specify an authority that includes a username.

### 8.10. Extracting the Host from a URL

- You want to extract the host from a string that holds a URL. For example, you want to extract www.regexcookbook.com from http://www.regexcookbook.com/.
- Extract the host from a URL known to be valid
```markdown
\A
[a-z][a-z0-9+\-.]*://               # Scheme
([a-z0-9\-._~%!$&'()*+,;=]+@)?      # User
([a-z0-9\-._~%]+                    # Named or IPv4 host
|\[[a-z0-9\-._~%!$&'()*+,;=:]+\])   # IPv6+ host
```
- Extract the host while validating the URL
```markdown
\A
[a-z][a-z0-9+\-.]*://                       # Scheme
([a-z0-9\-._~%!$&'()*+,;=]+@)?              # User
([a-z0-9\-._~%]+                            # Named host
|\[[a-f0-9:.]+\]                            # IPv6 host
|\[v[a-f0-9][a-z0-9\-._~%!$&'()*+,;=:]+\])  # IPvFuture host
(:[0-9]+)?                                  # Port
(/[a-z0-9\-._~%!$&'()*+,;=:@]+)*/?          # Path
(\?[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?         # Query
(\#[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?         # Fragment
\Z
```
- Extracting the host from a URL is easy if you already know that your subject text is a valid URL. We use ‹\A› or ‹^› to anchor the match to the start of the string. ‹[a-z][a-z0-9+\-.]*://› skips over the scheme, and ‹([a-z0-9\-._~%!$&'()*+,;=]+@)?› skips over the optional user. The hostname follows right after that.
- RFC 3986 allows two different notations for the host. Domain names and IPv4 addresses are specified without square brackets, whereas IPv6 and future IP addresses are specified with square brackets. We need to handle those separately because the notation with square brackets allows more punctuation than the notation without. In particular, the colon is allowed between square brackets, but not in domain names or IPv4 addresses. The colon is also used to delimit the hostname (with or without square brackets) from the port number.
- If you don’t already know that your subject text is a valid URL, you can use a simplified version of the regex from Recipe 8.7. Since we want to extract the host, we can exclude URLs that don’t specify an authority. This makes the regular expression quite a bit simpler.

### 8.11. Extracting the Port from a URL

- You want to extract the port number from a string that holds a URL. For example, you want to extract 80 from http://www.regexcookbook.com:80/.
- Extract the port from a URL known to be valid
```markdown
\A
[a-z][a-z0-9+\-.]*://               # Scheme
([a-z0-9\-._~%!$&'()*+,;=]+@)?      # User
([a-z0-9\-._~%]+                    # Named or IPv4 host
|\[[a-z0-9\-._~%!$&'()*+,;=:]+\])   # IPv6+ host
:(?<port>[0-9]+)                    # Port number
```
- Extract the port while validating the URL
```markdown
\A
[a-z][a-z0-9+\-.]*://                       # Scheme
([a-z0-9\-._~%!$&'()*+,;=]+@)?              # User
([a-z0-9\-._~%]+                            # Named host
|\[[a-f0-9:.]+\]                            # IPv6 host
|\[v[a-f0-9][a-z0-9\-._~%!$&'()*+,;=:]+\])  # IPvFuture host
:([0-9]+)                                   # Port
(/[a-z0-9\-._~%!$&'()*+,;=:@]+)*/?          # Path
(\?[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?         # Query
(\#[a-z0-9\-._~%!$&'()*+,;=:@/?]*)?         # Fragment
\Z
```
- The port number is separated from the hostname with a colon, which we add as a literal character to the regular expression. The port number itself is simply a string of digits, easily matched with ‹[0-9]+›.

### 8.12. Extracting the Path from a URL

- You want to extract the path from a string that holds a URL. For example, you want to extract `/index.html` from http://www.regexcookbook.com/index.html or from /index.html#fragment.
- Extract the path from a string known to hold a valid URL. The following finds a match for all URLs, even for URLs that have no path:
  - `^([a-z][a-z0-9+\-.]*:(//[^/?#]+)?)?([a-z0-9\-._~%!$&'()*+,;=:@/]*)`
- Extract the path from a string known to hold a valid URL. Only match URLs that actually have a path:
  - `^([a-z][a-z0-9+\-.]*:(//[^/?#]+)?)?(/?[a-z0-9\-._~%!$&'()*+,;=@]+(/[a-z0-9\-._~%!$&'()*+,;=:@]+)*/?|/)([#?]|$)`
- Extract the path from a string known to hold a valid URL. Use atomic grouping to match only those URLs that actually have a path:
```markdown
\A
# Skip over scheme and authority, if any
(?>([a-z][a-z0-9+\-.]*:(//[^/?#]+)?)?)
# Path
([a-z0-9\-._~%!$&'()*+,;=:@/]+)
```
- You can use a much simpler regular expression to extract the path if you already know that your subject text is a valid URL. While the generic regex in Recipe 8.7 has three different ways to match the path, depending on whether the URL specifies a scheme and/or authority, the specific regex for extracting the path from a URL known to be valid needs to match the path only once.

### 8.13. Extracting the Query from a URL

- You want to extract the query from a string that holds a URL. For example, you want to extract param=value from http://www.regexcookbook.com?param=value or from /index.html?param=value.
- `^[^?#]+\?([^#]+)`
- Extracting the query from a URL is trivial if you know that your subject text is a valid URL. The query is delimited from the part of the URL before it with a question mark. That is the first question mark allowed anywhere in URLs. Thus, we can easily skip ahead to the first question mark with ‹^[^?#]+\?›. The question mark is a metacharacter only outside character classes, but not inside, so we escape the literal question mark outside the character class. The first ‹^› is an anchor (Recipe2.5), whereas the second ‹^› negates the character class (Recipe2.3).
- Question marks can appear in URLs as part of the (optional) fragment after the query. So we do need to use ‹^[^?#]+\?›, rather than just ‹\?›, to make sure we have the first question mark in the URL, and make sure that it isn’t part of the fragment in a URL without a query.
- The query runs until the start of the fragment, or the end of the URL if there is no fragment. The fragment is delimited from the rest of the URL with a hash sign. Since hash signs are not permitted anywhere except in the fragment, ‹[^#]+› is all we need to match the query. The negated character class matches everything up to the first hash sign, or everything until the end of the subject if it doesn’t contain any hash signs.

### 8.14. Extracting the Fragment from a URL

- You want to extract the fragment from a string that holds a URL. For example, you want to extract top from http://www.regexcookbook.com#top or from /index.html#top.
- `#(.+)`
- Extracting the fragment from a URL is trivial if you know that your subject text is a valid URL. The fragment is delimited from the part of the URL before it with a hash sign. The fragment is the only part of URLs in which hash signs are allowed, and the fragment is always the last part of the URL. Thus, we can easily extract the fragment by finding the first hash sign and grabbing everything until the end of the string. ‹#.+› does that nicely. Make sure to turn off free-spacing mode; otherwise, you need to escape the literal hash sign with a backslash.
- If you don’t already know that your subject text is a valid URL, you can use one of the regexes from Recipe8.7. The first regex in that recipe captures the fragment, if one is present in the URL, into capturing group number 13.

### 8.15. Validating Domain Names