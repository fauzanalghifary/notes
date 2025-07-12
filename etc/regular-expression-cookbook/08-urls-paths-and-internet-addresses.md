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

- You want to check whether a string looks like it may be a valid, fully qualified domain name, or find such domain names in longer text.
- Check whether a string looks like a valid domain name:
  - `^([a-z0-9]+(-[a-z0-9]+)*\.)+[a-z]{2,}$`
- Find valid domain names in longer text:
  - `\b([a-z0-9]+(-[a-z0-9]+)*\.)+[a-z]{2,}\b`
- Check whether each part of the domain is not longer than 63 characters:
  - `\b((?=[a-z0-9-]{1,63}\.)[a-z0-9]+(-[a-z0-9]+)*\.)+[a-z]{2,63}\b`
- Allow internationalized domain names using the punycode notation:
  - `\b((xn--)?[a-z0-9]+(-[a-z0-9]+)*\.)+[a-z]{2,}\b`
- Check whether each part of the domain is not longer than 63 characters, and allow internationalized domain names using the punycode notation:
  - `\b((?=[a-z0-9-]{1,63}\.)(xn--)?[a-z0-9]+(-[a-z0-9]+)*\.)+[a-z]{2,63}\b`
- A domain name has the form of domain.tld, or subdomain.domain.tld, or any number of additional subdomains. The top-level domain (tld) consists of two or more letters. That’s the easiest part of the regex: ‹[a-z]{2,}›.
- The domain, and any subdomains, consist of letters, digits, and hyphens. Hyphens cannot appear in pairs, and cannot appear as the first or last character in the domain. We handle this with the regular expression ‹[a-z0-9]+(-[a-z0-9]+)*›. This regex allows any number of letters and digits, optionally followed by any number of groups that consist of a hyphen followed by another sequence of letters and digits. Remember that the hyphen is a metacharacter inside character classes (Recipe 2.3) but an ordinary character outside of character classes, so we don’t need to escape any hyphens in this regex.
- The domain and the subdomains are delimited with a literal dot, which we match with ‹\.› in a regular expression. Since we can have any number of subdomains in addition to the domain, we place the domain name part of the regex and the literal dot in a group that we repeat: ‹([a-z0-9]+(-[a-z0-9]+)*\.)+›. Since the subdomains follow the same syntax as the domain, this one group handles both.
- Our first set of regular expressions doesn’t check whether each part of the domain is no longer than 63 characters.
- What we can do is to use lookahead to match the same text twice. Review Recipe 2.16 first if you’re not familiar with lookahead. We use the same regex ‹[a-z0-9]+(-[a-z0-9]+)*\.› to match a domain name with valid hyphens, and add ‹[-a-z0-9]{1,63}\.› inside a lookahead to check that its length is also 63 characters or less. The result is ‹(?=[-a-z0-9]{1,63}\.)[a-z0-9]+(-[a-z0-9]+)*\.›.
- The lookahead ‹(?=[-a-z0-9]{1,63}\.)› first checks that there are 1 to 63 letters, digits, and hyphens until the next dot. It’s important to include the dot in the lookahead. Without it, domains longer than 63 characters would still satisfy the lookahead’s requirement for 63 characters. Only by putting the literal dot inside the lookahead do we enforce the requirement that we want at most 63 characters.

### 8.16. Matching IPv4 Addresses

- You want to check whether a certain string represents a valid IPv4 address in 255.255.255.255 notation. Optionally, you want to convert this address into a 32-bit integer.
- Simple regex to check for an IP address:
  - `^(?:[0-9]{1,3}\.){3}[0-9]{1,3}$`
- Accurate regex to check for an IP address, allowing leading zeros:
  - `^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$`
- Accurate regex to check for an IP address, disallowing leading zeros:
  - `^(?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.){3}(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])$`
- Simple regex to extract IP addresses from longer text:
  - `\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b`
- Accurate regex to extract IP addresses from longer text, allowing leading zeros:
  - `\b(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\b`
- Accurate regex to extract IP addresses from longer text, dis25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9]allowing leading zeros:
  - `\b(?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.){3}(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\b`
- Simple regex that captures the four parts of the IP address:
  - `^([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})$`
- Accurate regex that captures the four parts of the IP address, allowing leading zeros:
  - `^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.↵
(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.↵
(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.↵
(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$`
- Accurate regex that captures the four parts of the IP address, disallowing leading zeros:
  - `^(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.↵
(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.↵
(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.↵
(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])$`
- A version 4 IP address is usually written in the form 255.255.255.255, where each of the four numbers must be between 0 and 255. Matching such IP addresses with a regular expression is very straightforward.
- In the solution, we present four regular expressions. Two of them are billed as “simple,” while the other two are marked “accurate.”

### 8.17. Matching IPv6 Addresses

- You want to check whether a string represents a valid IPv6 address using the standard, compact, and/or mixed notations.

Standard notation
- Match an IPv6 address in standard notation, which consists of eight 16-bit words using hexadecimal notation, delimited by colons (e.g.: 1762:0:0:0:0:B03:1:AF18). Leading zeros are optional.
- Check whether the whole subject text is an IPv6 address using standard notation:
  - `^(?:[A-F0-9]{1,4}:){7}[A-F0-9]{1,4}$`
- Find an IPv6 address using standard notation within a larger collection of text:
  - `(?<![:.\w])(?:[A-F0-9]{1,4}:){7}[A-F0-9]{1,4}(?![:.\w])`

Mixed notation
- Match an IPv6 address in mixed notation, which consists of six 16-bit words using hexadecimal notation, followed by four bytes using decimal notation. The words are delimited with colons, and the bytes with dots. A colon separates the words from the bytes. Leading zeros are optional for both the hexadecimal words and the decimal bytes. This notation is used in situations where IPv4 and IPv6 are mixed, and the IPv6 addresses are extensions of the IPv4 addresses. 1762:0:0:0:0:B03:127.32.67.15 is an example of an IPv6 address in mixed notation.
- Check whether the whole subject text is an IPv6 address using mixed notation:
  - `^(?:[A-F0-9]{1,4}:){6}(?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.){3}(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])$`
- Find IPv6 address using mixed notation within a larger collection of text:
  - `(?<![:.\w])(?:[A-F0-9]{1,4}:){6}↵
(?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.){3}↵
(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])(?![:.\w])`

Standard or mixed notation
- Check whether the whole subject text is an IPv6 address using standard or mixed notation:
  - `^(?:[A-F0-9]{1,4}:){6}(?:[A-F0-9]{1,4}:[A-F0-9]{1,4}|(?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.){3}(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9]))$`
- Find IPv6 address using standard or mixed notation within a larger collection of text:
```markdown
(?<![:.\w])                                              # Anchor address
(?:[A-F0-9]{1,4}:){6}                                        # 6 words
(?:[A-F0-9]{1,4}:[A-F0-9]{1,4}                               # 2 words
|  (?:(?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])\.){3}  # or 4 bytes
   (?:25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9]?[0-9])
)(?![:.\w])    
```

Compressed notation
Compressed mixed notation
Standard, mixed, or compressed notation

### 8.18. Validate Windows Paths

- You want to check whether a string looks like a valid path to a folder or file on the Microsoft Windows operating system.
- Drive letter paths
```markdown
\A
[a-z]:\\                    # Drive
(?:[^\\/:*?"<>|\r\n]+\\)*   # Folder
[^\\/:*?"<>|\r\n]*          # File
\Z
```
- Drive letter and UNC paths
```markdown
\A
(?:[a-z]:|\\\\[a-z0-9_.$\●-]+\\[a-z0-9_.$\●-]+)\\  # Drive
(?:[^\\/:*?"<>|\r\n]+\\)*                          # Folder
[^\\/:*?"<>|\r\n]*                                 # File
\Z
```
- Drive letter, UNC, and relative paths
```markdown
\A
(?:(?:[a-z]:|\\\\[a-z0-9_.$\●-]+\\[a-z0-9_.$\●-]+)\\|  # Drive
   \\?[^\\/:*?"<>|\r\n]+\\?)                           # Relative path
(?:[^\\/:*?"<>|\r\n]+\\)*                              # Folder
[^\\/:*?"<>|\r\n]*                                     # File
\Z
```
- Matching a full path to a file or folder on a drive that has a drive letter is very straightforward. The drive is indicated with a single letter, followed by a colon and a backslash. We easily match this with ‹[a-z]:\\›. The backslash is a metacharacter in regular expressions, and so we need to escape it with another backslash to match it literally.
- Folder and filenames on Windows can contain all characters, except these: \/:*?"<>|. Line breaks aren’t allowed either. We can easily match a sequence of all characters except these with the negated character class ‹[^\\/:*?"<>|\r\n]+›. The backslash is a metacharacter in character classes too, so we escape it. ‹\r› and ‹\n› are the two line break characters
- Folders are delimited with backslashes. We can match a sequence of zero or more folders with ‹(?:[^\\/:*?"<>|\r\n]+\\)*›, which puts the regex for the folder name and a literal backslash inside a noncapturing group (Recipe2.9) that is repeated zero or more times with the asterisk (Recipe2.12).
- To match the filename, we use ‹[^\\/:*?"<>|\r\n]*›. The asterisk makes the filename optional, to allow paths that end with a backslash. If you don’t want to allow paths that end with a backslash, change the last ‹*› in the regex into a ‹+›.

### 8.19.Split Windows Paths into Their Parts

- You want to check whether a string looks like a valid path to a folder or file on the Microsoft Windows operating system. If the string turns out to hold a valid Windows path, then you also want to extract the drive, folder, and filename parts of the path separately.
- Drive letter paths
```markdown
\A
(?<drive>[a-z]:)\\
(?<folder>(?:[^\\/:*?"<>|\r\n]+\\)*)
(?<file>[^\\/:*?"<>|\r\n]*)
\Z
```
- Drive letter and UNC paths
```markdown
\A
(?<drive>[a-z]:|\\\\[a-z0-9_.$●-]+\\[a-z0-9_.$●-]+)\\
(?<folder>(?:[^\\/:*?"<>|\r\n]+\\)*)
(?<file>[^\\/:*?"<>|\r\n]*)
\Z
```
- Drive letter, UNC, and relative paths
```markdown
\A
(?<drive>[a-z]:\\|\\\\[a-z0-9_.$●-]+\\[a-z0-9_.$●-]+\\|\\?)
(?<folder>(?:[^\\/:*?"<>|\r\n]+\\)*)
(?<file>[^\\/:*?"<>|\r\n]*)
\Z
```
- The regular expressions in this recipe are very similar to the ones in the previous recipe. This discussion assumes you’ve already read and understood the discussion of the previous recipe.
- We’ve made only one change to the regular expressions for drive letter paths, compared to the ones in the previous recipe. We’ve added three capturing groups that you can use to retrieve the various parts of the path: ‹drive›, ‹folder›, and ‹file›. You can use these names if your regex flavor supports named capture (Recipe2.11). If not, you’ll have to reference the capturing groups by their numbers: 1, 2, and 3.

### 8.20. Extract the Drive Letter from a Windows Path

- You have a string that holds a (syntactically) valid path to a file or folder on a Windows PC or network. You want to extract the drive letter, if any, from the path. For example, you want to extract c from c:\folder\file.ext.
- `^([a-z]):`
- Extracting the drive letter from a string known to hold a valid path is trivial, even if you don’t know whether the path actually starts with a drive letter. The path could be a relative path or a UNC path.
- Colons are invalid characters in Windows paths, except to delimit the drive letter. Thus, if we have a letter followed by a colon at the start of the string, we know the letter is the drive letter.

### 8.21. Extract the Server and Share from a UNC Path

- You have a string that holds a (syntactically) valid path to a file or folder on a Windows PC or network. If the path is a UNC path, then you want to extract the name of the network server and the share on the server that the path points to. For example, you want to extract server and share from \\server\share\folder\file.ext.
- `^\\\\([a-z0-9_.$●-]+)\\([a-z0-9_.$●-]+)`
- UNC paths begin with two backslashes. Two consecutive backslashes are not allowed in Windows paths, except to begin a UNC path. Thus, if a known valid path begins with two backslashes, we know that the server and share name must follow.

### 8.22. Extract the Folder from a Windows Path

- You have a string that holds a (syntactically) valid path to a file or folder on a Windows PC or network, and you want to extract the folder from the path. For example, you want to extract \folder\subfolder\ from c:\folder\subfolder\file.ext or \\server\share\folder\subfolder\file.ext.
- `^([a-z]:|\\\\[a-z0-9_.$●-]+\\[a-z0-9_.$●-]+)?((?:\\|^)(?:[^\\/:*?"<>|\r\n]+\\)+)`
- Extracting the folder from a Windows path is a bit tricky if we want to support UNC paths, because we can’t just grab the part of the path between backslashes. If we did, we’d be grabbing the server and share from UNC paths too.
- The first part of the regex, ‹^([a-z]:|\\\\[a-z0-9_.$●-]+\\[a-z0-9_.$●-]+)?›, skips over the drive letter or the network server and network share names at the start of the path.
- The question mark after the group makes it optional. This allows us to support relative paths, which don’t have a drive letter or network share.
- The folders are easily matched with ‹(?:[^\\/:*?"<>|\r\n]+\\)+›. The character class matches a folder name. The noncapturing group matches a folder name followed by a literal backslash that delimits the folders from each other and from the filename. We repeat this group one or more times. This means our regular expression will match only those paths that actually specify a folder. Paths that specify only a filename, drive, or network share won’t be matched.

### 8.23. Extract the Filename from a Windows Path

- You have a string that holds a (syntactically) valid path to a file or folder on a Windows PC or network, and you want to extract the filename, if any, from the path. For example, you want to extract file.ext from c:\folder\file.ext.
- `[^\\/:*?"<>|\r\n]+$`
- The filename always occurs at the end of the string. It can’t contain any colons or backslashes, so it cannot be confused with folders, drive letters, or network shares, which all use backslashes and/or colons.
- The negated character class ‹[^\\/:*?"<>|\r\n]+› (Recipe 2.3) matches the characters that can occur in filenames. Though the regex engine scans the string from left to right, the anchor at the end of the regex makes sure that only the last run of filename characters in the string will be matched, giving us our filename.

### 8.24. Extract the File Extension from a Windows Path

- You have a string that holds a (syntactically) valid path to a file or folder on a Windows PC or network, and you want to extract the file extension, if any, from the path. For example, you want to extract .ext from c:\folder\file.ext.
- `\.[^.\\/:*?"<>|\r\n]+$`
- We can use the same technique for extracting the file extension as we used for extracting the whole filename in Recipe 8.23. 
- The only difference is in how we handle dots. The regex in Recipe 8.23 does not include any dots. The negated character class in that regex will simply match any dots that happen to be in the filename.
- A file extension must begin with a dot. Thus, we add ‹\.› to match a literal dot at the start of the regex.

### 8.25. Strip Invalid Characters from Filenames

- You want to strip a string of characters that aren’t valid in Windows filenames. For example, you have a string with the title of a document that you want to use as the default filename when the user clicks the Save button the first time.
- Regex: `[\\/:"*?<>|]+`
- Replacement: ``
- The characters \/:"*?<>| are not valid in Windows filenames. These characters are used to delimit drives and folders, to quote paths, or to specify wildcards and redirection on the command line.
- We can easily match those characters with the character class ‹[\\/:"*?<>|]›. The backslash is a metacharacter inside character classes, so we need to escape it with another backslash. All the other characters are always literal characters inside character classes.
- We repeat the character class with a ‹+› for efficiency. This way, if the string contains a sequence of invalid characters, the whole sequence will be deleted at once, rather than character by character. You won’t notice the performance difference when dealing with very short strings, such as filenames, but it is a good technique to keep in mind when you’re dealing with larger sets of data that are more likely to have longer runs of characters that you want to delete.

