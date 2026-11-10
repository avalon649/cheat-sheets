# git bisect — Binary Search for a Bad Commit

```bash
git bisect start
git bisect bad                    # current commit is bad
git bisect good <commit|tag>      # last known good
# git checks out a midpoint — test it, then:
git bisect good                   # or
git bisect bad                    # or
git bisect skip                   # can't test this one
# repeat until git prints "<hash> is the first bad commit"
git bisect reset                  # return to original HEAD
```

## Shortcuts
```bash
git bisect start <bad> <good>
git bisect start HEAD v1.2.0 -- src/     # limit to path
git bisect log                    # show steps so far
git bisect log > bisect.log
git bisect replay bisect.log
git bisect visualize              # (alias: view)
git bisect terms --term-old=fast --term-new=slow   # custom terms
```

## Automated bisect
```bash
git bisect start HEAD v1.2.0
git bisect run ./test.sh          # exit 0=good, 1-124/126-127=bad, 125=skip
git bisect run npm test
git bisect reset
```
