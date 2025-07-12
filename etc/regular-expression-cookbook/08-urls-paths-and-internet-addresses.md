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


