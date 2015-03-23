# Grep

In the shell world one of the most used commands is grep. Grep and it's faster cousin ack give you the ability to search for occurence of a string of text or a pattern inside files or streams.

## Looking for a string in a stream

One of the most common use of grep is to check if a process is running. To do so we usually forward the exit of the *ps* command to *grep* :

```
> ps aux | grep unicorn
```

If any line of the *ps aux* output contains unicorn we will see those lines in the terminal as any other will have been filtered out by grep.

If there is no output it means no lines contain unicorn. In the end, since ps lists running processes it means no unicorn process is running (or no commands containing the unicorn string).

## Looking for a string inside files

You can also use a similar approach to figure out where a specific string is contained.

If you do the following :

```
grep git somefile.txt
```

Grep will output the lines that do include the string "git" in the file "somefile.txt".

If you have multiple files to test you can list more than one :

```
grep git somefile1.txt somefile2.txt
```

In that case grep will add the file name as prefix in front of each line that match the searched string.

And if you have many files you can just pass a directoy or a glob of course :

```
grep git *.txt
```

## More context

By default grep will only show you the line matching the search, but you might want to see more than that line to remember or figure out the context.

You can use a context param for that with the number of lines you want to see before and after the matched line :

```
grep -C2 git somefile1.txt
```

## Just a count

Maybe you just want to know how many lines match the search. To do so you can use the count parameter :

```
grep -c git somefile1.txt
```

## Reversing the search

Sometimes you want to know what lines don't match a specific string, you can use the invert parameter to do so :

```
grep -v git somefile1.txt
```

That is mostly helpful when you chain several grep together :

```
grep git *txt | grep -v hg
```

That for example will show the lines that include git but not hg.

## Using regex

Regex are another very powerful shell thing, with grep you can use them to specify the pattern you are looking for :

```
grep -e "g.t" somefile.txt
```

That one will list you all the lines that match the regex "g.t" (g followed by 1 character, followed by a t).

## Recursively

You can then use the classic "-R" for recusrivity inside directories. It's helpful when looking inside a whole project of course.

## Epilogue

Grep are handy when you are on a Unix based system for many things. From process searches to configuration check etc ... It's one of the top swiss army knives of the unix toolbox. A definite must know.

Ack is very similar but with added sugar such as colours and better performances.

