# Regular expressions

This session goes into detail about the syntax and use-cases for regular expressions, both for searching text and working with data files and directories.

## Introduction to the syntax

Let's practice in [a richer environment](https://regex101.com/#python) to start with.

## The `re` module

We need to import regular expression functionality before we can do anything else:

```python
import re
re.search('pattern', 'string containing the pattern')
```

Remember NLTK's concordancer:

```python
from nltk.book import *
text5.concordance('seriously')
```

Concordancing can be very powerful, especially for thematic categorisation and the like. So, let's load up some data, and then write up a regex-based concordancer for our plain text corpus:

```python
with open('forum.txt', 'r', encoding = 'utf-8') as fo:
    data = fo.read()
```

```python
def conc(corpus, query):
    """regex concordancer"""
    import re
    compiled = re.compile(r'(.*)(%s)(.*)' % query)
    lines = re.findall(compiled, text)
    for start, middle, end in lines:
        concline = [start[-30:], middle, end[:30]]
        print("\t".join(concline).expandtabs(35))
```

Let's try it out:

```python
conc('austral[a-z]+', data)
```

Finally, let's add a `window` keyword argument, and also fix any left printing issues:

```python
def conc(corpus, query, window = 30):
    """regex concordancer"""
    import re
    compiled = re.compile(r'(.*)(%s)(.*)' % query, re.I)
    lines = re.findall(compiled, text)
    for start, middle, end in lines:
        concline = [start[-window:], middle, end[:window]]
        if len(concline[0]) < window:
            concline[0] = ' ' * (window - len(concline[0])) + concline[0]
        print("\t".join(concline).expandtabs(35))
```

Great! We've improved on NLTK's concordancer!

Can you find:

1. Words with exclamation marks immediately after them
2. Multiple punctuation marks in a row
2. tokens/words with more than three of the same vowel in a row
3. ly adverbs with one token context either size
5. Numbers larger than 20

```python
conc(data, r'[a-z]+!')
conc(data, r'[\!\?,\.]{2,}')
conc(data, r'[a-z]*[aeiou]{4,}[a-z]*')
conc(data, r'[^\s]+\s[a-z]+ly\s[^\s]+')
conc(data, r'[0-9]*[123456789][0-9]+')
```

### `re.compile()`

When using regular expressions, we may want to compile our regular expression before using it. The first reason for doing this is to make things faster: compilation can take a moment, and it can slow down loops.

```python
s = 'cat'
print type(s)
pattern = re.compile(s)
print type(pattern)
```

#### Flags

The second reason we might want to use `re.compile()` is that we can easily add in options that change how our regular expression will match things:

```python
# ignore case
pattern = re.compile(s, re.I)
# match over multiple lines
pattern = re.compile(s, re.MULTILINE)
# dot matches newline too
pattern = re.compile(s, re.S)
```

### `re.search()`

```python
found = False
if re.search(r'p.w.r', data):
    found = True
    print(found)
```

### `re.findall()`

```python
matches = re.findall(r'[0-9\.]+', data)
set(matches)
```

```python
set([m.rstrip('.') for m in matches])
```

```python
for m in re.findall(r'[a-z]+ous\b', data):
    print('You are a very %s person.' % m)
```

### Match objects

```python
match = re.search(r'fe[a-z]+', data)
type(match)
```

When our pattern matches, we are left with a *match object*, which has special attributes.

#### Getting character indices

```python
print(match.start(), match.end())
# or ...
print(match.span())
```

```python
### access the data we passed in
print(match.string[:100])
```

#### Groups

Bracketted parts of regular expressions are *groups*, which we can access using `match.group(n)`. `match.group(0)` matches the entire match.

```python
pattern = re.compile(r'\b(aus)([a-z]+)', re.I)
match = re.search(pattern, data)
match.group(2)
```

### `re.match()`

`re.match()` is just like `re.search()`, but it only matches start of lines:

```python
re.match(r'open', 'the opening of a text')
```

```python
re.search(r'open', 'the opening of a text')
```

Generally, it's probably better to use `re.search()` with a caret (`^`) if need be:

```python
re.search(r'^open', 'the opening of a text')
```

### `re.split()`

We can create a list from a string:

```python
pattern = re.compile(r'\s\.,')
lst = re.split(pattern, data, maxsplit = 2)
lst
```

#### Tokenising with regular expressions

NLTK provides [a very simply method](http://www.nltk.org/_modules/nltk/tokenize/regexp.html) for tokenising by regular expression:

```python
from nltk.tokenize import RegexpTokenizer
pattern = r'[A-Za-z0-9-]'
tokeniser = RegexpTokenizer(pattern)
toks = tokeniser.tokenize(data)
```

Really, it's no different from:

```python
toks = [t for t in re.findall(pattern, data) if t]
```

... except that has options for matching gaps rather than tokens, discarding unmatched space, etc.

```python
## @instructor: modify the earlier cell for this code
from nltk.tokenize import RegexpTokenizer
pattern = r'\s+'
tokeniser = RegexpTokenizer(pattern, gaps = True, discard_empty = False)
toks = tokeniser.tokenize(data)
```

### `re.sub()`

`re.sub()` does find and replace with regular expressions.

```python
print(data.count('moslem'))
print(data.count('muslim'))
fx = re.sub(r'moslems*', 'muslim', data)
print(fx.count('muslim'))
```

## Putting it all together

We've got the script to the [South Park: Bigger, Longer & Uncut](https://en.wikipedia.org/wiki/South_Park:_Bigger,_Longer_%26_Uncut). This film was briefly quite famous for its profanity.

```python
with open('southparkmovie.txt', 'r', encoding = 'utf-8') as fo:
    sp = fo.read()
sp.count('shit')
```
Write a function that uses regex to replace obscenities with asterisks. Bonus points for:

1. Making it easy to add new obscenities to the list
2. Printing examples as the function runs to show us that it's working
3. Asterisks being the same length as the obscenity

```python
def censor(plaintext):
    """censor naughty words"""
    import re
    naughty = [r'(bull)shite?', 'bloody', 'fuck', 'hell']
    pattern = r'\b(' + '|'.join(naughty) + r')[a-z]*'
    regex = re.compile(pattern, re.I)
    replaced = re.sub(regex, '****', plaintext, count=False)
    ast = re.compile(r'\*+')
    for line in replaced.splitlines():
        if re.search(ast, line):
            print(line)
    return replaced
```

```python
def censor(plaintext):
    """second attempt"""
    import re
    from nltk import word_tokenize
    toks = word_tokenize(plaintext)
    naughty = [r'(bull)shite?', 'bloody', 'fuck', 'hell']
    pattern = r'\b(' + '|'.join(naughty) + r')[a-z]*'
    regex = re.compile(pattern, re.I)
    for index, tok in enumerate(toks):
        mtch = re.match(regex, tok)
        if mtch:
            toks[index] = '*' * len(mtch.group(0))
            print(' '.join(toks[index - 5:index+5]))
    return toks
```

```python
def censor(plaintext):
    import re
    naughty = [r'(bull)shite?', 'bloody', 'fuck', 'hell']
    pattern = r'\b(' + '|'.join(naughty) + r')[a-z]*'
    regex = re.compile(pattern, re.I)
    lst_of_chars = list(plaintext)
    lines = plaintext.splitlines()
    for index, line in enumerate(lines):
        mtch = re.search(regex, line)
        if mtch:
            toreplace = mtch.group(0)
            censored = line.replace(toreplace, '*' * len(toreplace))
            lines[index] = censored
            print(censored)
    return '\n'.join(lines)
```

```python
censor(sp)
```

## Regular expressions in Shell

```python

```

### `sed`

```python

```

### `grep`

```python

```

## Resources around the web

### Checkers

### Cheatsheets

### Crosswords