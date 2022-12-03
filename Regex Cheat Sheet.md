# Regex Cheat Sheet

## 0. Introduction

Regex (short for "regular expression") is a notation for describing text mode. It is used as a professional language to match some particular modes.

## 1. Syntax

```
/^hello/i --match--> "Hello regex"
```

## 2. Particles

These little parts will show how a regex composed of and their function.

### 2.1 Meta signs

| sign     | meaning                 |
| -------- | ----------------------- |
| .        | any single character    |
| \|       | "or", means select      |
| \d       | any number              |
| \D       | any non-number          |
| \s       | any space character     |
| \S       | any non-space character |
| \w       | any word                |
| \W       | any non-word            |
| \X       | unicode sequence        |
| \p{Prop} | unicode property        |
| \P{Prop} | unicode property        |

### 2.2 Special signs

| sign    | meaning                            |
| ------- | ---------------------------------- |
| \       | used to represent escape sequences |
| \f      | formfeed                           |
| \n      | newline                            |
| \r      | carriage return                    |
| \t      | horizontal tab                     |
| \v      | vertical tab                       |
| \0      | space                              |
| \num    | octal number                       |
| \xnum   | hexadecimal number                 |
| \x{num} | hexadecimal number                 |
| \unum   | unicode escape sequence            |
| \Unum   | unicode escape sequence            |
| \cchar  | control character                  |

### 2.3 Character sets

| character set | meaning                                                |
| ------------- | ------------------------------------------------------ |
| [...]         | single character of character set                      |
| [^...]        | character set except these characters                  |
| [a-z]         | characters between 'a' and 'z'                         |
| [^a-z]        | characters except 'a' to 'z'                           |
| [a-zA-Z]      | characters between 'a' and 'z' and between 'A' and 'Z' |

### 2.4 POSIX character sets

| character set | meaning                                   |
| ------------- | ----------------------------------------- |
| [[:alnum:]]   | any alphabet and number                   |
| [[:alpha:]]   | any alphabet                              |
| [[:ascii:]]   | ascii codes (0-127)                       |
| [[:blank:]]   | space or tab                              |
| [[:cntrl:]]   | control characters                        |
| [[:digit:]]   | digital numbers                           |
| [[:graph:]]   | visible characters except space           |
| [[:lower:]]   | lower characters                          |
| [[:print:]]   | printable characters                      |
| [[:punct:]]   | punctions                                 |
| [[:space:]]   | space characters                          |
| [[:upper:]]   | upper characters                          |
| [[:word:]]    | words, characters, numbers and underscore |
| [[:xdigit:]]  | hexadecimal numbers                       |
| [[:<:]]       | the beginning of word                     |
| [[:>:]]       | the end of word                           |

### 2.5 Anchors

| anchor | meaning                                        |
| ------ | ---------------------------------------------- |
| ^      | the beginning of strings/lines                 |
| \A     | the beginning of strings/lines, ignore m signs |
| $      | the end of strings/lines                       |
| \Z     | the end of strings/lines, ignore m signs       |
| \z     | match the end of strings only                  |
| \b     | match the boundary of words                    |
| \B     | match not the boundary of words                |
| \\<    | match the beginning of words                   |
| \\>    | match the end of words                         |
| \G     | match the begning location                     |

### 2.6 Quantitives

| quantitive | meaning                                                      |
| ---------- | ------------------------------------------------------------ |
| ?          | repeat 0 or 1 time                                           |
| ??         | repeat 0 or 1 time (not greedy match)                        |
| *          | repeat 0 or more times                                       |
| *?         | repeat 0 or more times (not greedy match)                    |
| +          | repeat 1 or more times                                       |
| +?         | repeat 1 or more times (not greedy match)                    |
| {n}        | repeat n times (n isn't negative integer)                    |
| {n,}       | repeat at least n times (n isn't negative integer)           |
| {n, m}     | repeat n times at least m times at most (n, m are both non-negative integers) |

### 2.7 Groups

| group           | meaning                    |
| --------------- | -------------------------- |
| (...)           | capturing group            |
| (?:...)         | non-capturing group        |
| (?P\<name\>...) | capturing group with name  |
| (?\<name\>...)  | capturing group with name  |
| (?'name'...)    | capturing group with name  |
| (?>...)         | non-capturing group        |
| (?\|...)        | reset child-mode number    |
| (?#...)         | comment the group          |
| (?=...)         | posible lookahead          |
| (?!...)         | negative lookahead         |
| (?<=...)        | positive reverse lookahead |
| (?<!...)        | negative reverse lookahead |

### 2.8 Replacements

| replacement | meaning                       |
| ----------- | ----------------------------- |
| $1          | capture contents in group 1   |
| $foo        | capture contents in group foo |

### 2.9 Mode prefix signs

| mode prefix sign | meaning                   |
| ---------------- | ------------------------- |
| g                | global mode               |
| i                | ignore upper and lower    |
| m                | multiple lines            |
| s                | single line               |
| x                | allow spaces and comments |
| u                | unicode characters        |
| U                | non-greedy match          |


### 2.10 Regex engines

* DFA
* POSIX DFA
* Traditional DFA
* DFA/NFA fusion

## 3. Websites

You can test your regex and understand it in visual way in these websites.

* regex101.com
* debuggex.com
* regexr.com
