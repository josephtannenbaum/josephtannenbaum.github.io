---
title:  "Storing Random Stack Tips"
---

I think a minor part of most programmers' jobs is keeping a small <a href="https://en.wikipedia.org/wiki/Girdle_book">handbook of trivia</a> related to the quirks of their tech stack, mostly in their heads. I don't think there will come a day when we will be free of "Has anyone seen this error?" and "What do I have to do to make this setup command work?" Slack threads. To carry some of that weight for me, I created a command-line tool `fyi`. I was partially inspired by <a href="https://codeascraft.com/2018/10/10/etsys-experiment-with-immutable-documentation/">"Etsy’s experiment with immutable documentation".</a>

## fyi demo

```bash
MacBook-Pro:~ j$ fyi
# <Opens up a vim terminal for a new hidden file with a random name, e.g. ~/.fyi/2yl9vg91yqjbtlc0.fyi>
MacBook-Pro:~ j$ fyi "undefined"
:: results ::
[1] could not freeze typescript TS files cannot read property hash of undefined
[2] number inputs ngmodel ng-model undefined bound variable and blur problem
MacBook-Pro:~ j$ fyi "undefined" 2
:: /Users/joetannenbaum/.fyi/2oxvp21jubhbdgmt.fyi ::
min and max need to be in the units of the parent model value, not of the internal value.

Also you should set ng-model-options="{ updateOn: 'blur keyup mouseup' }"
```

## fyi functionality

So it's a read-only system of notes you can search with the minimal amount of UI.

`fyi` opens vim with a blank FYI file

`fyi "search terms"` searches the first line of each FYI file

`fyi "search terms" 2` displays the 2nd result of the search, including the file path in case you need to edit it.

## fyi source code

I have a repo for `fyi` here: https://github.com/josephtannenbaum/fyi

```python
#!/usr/local/bin/python3
import sys
import os
import random
import string
import argparse
from subprocess import call
from fuzzywuzzy import fuzz

FYI_PATH = os.path.expanduser('~/.fyi')

def pretty_list(mylist):
  out = []
  for i, elem in enumerate(mylist):
    line = '['
    if len(mylist) > 9 and i < 9:
      line += ' '
    line += '{}] {}'.format(i + 1, elem)
    out.append(line.strip())
  return '\n'.join(out)

def generate_random_key(length):
    return ''.join(random.choice(string.ascii_lowercase + string.digits) for _ in range(length))

def make_fyi_folder():
  if not os.path.exists(FYI_PATH):
    os.makedirs(FYI_PATH)

def open_vim(file_path):
  call(['vim', file_path])

def cat(file_path):
  with open(file_path) as fyifile:
    fyilines = fyifile.readlines()
    # remove tags line at end of file
    if all([t[0] == '#' for t in fyilines[-1].split()]):
      fyilines.pop()
    print('\n'.join(fyilines[2:]).strip())

def open_vim_with_random_string():
  random_string = generate_random_key(16)
  new_path = '{}/{}.fyi'.format(FYI_PATH, random_string)
  open_vim(new_path)

def find_fyi(query_string):
  results = []
  directory = FYI_PATH
  for filename in os.listdir(directory):
    if filename.endswith('.fyi'):
        file_path = os.path.join(directory, filename)
        with open(file_path) as fyifile:
          first_line = fyifile.readline()
          rest = fyifile.readlines()
          tags = [t[1:] for t in rest[-1].split() if t[0] == '#']
          r = fuzz.token_set_ratio(query_string, first_line + ' ' + ' '.join(tags))
          if r > 73:
            results.append({ 'first_line': first_line.strip(), 'file_path': file_path })
  return results

parser = argparse.ArgumentParser()
parser.add_argument('query', nargs='?', help="query for FYIs")
parser.add_argument('idx', nargs='?', type=int, help="with query, display an FYI by the index")
args = parser.parse_args()

if not (args.query or args.idx):  # fyi
  make_fyi_folder()
  open_vim_with_random_string()

elif args.query and not args.idx:  # fyi "my search terms"
  results = find_fyi(args.query)
  if len(results) == 0:
    print('\033[1m:: no results ::\033[0m')
  else:
    print('\033[1m:: results ::\033[0m')
    print(pretty_list([result['first_line'] for result in results]))

elif args.query and args.idx:  # fyi "my search terms" 1
  r_index = args.idx - 1
  results = find_fyi(args.query)
  chosen_result = results[r_index]
  file_path = chosen_result['file_path']
  print('\033[1m:: {} ::\033[0m'.format(file_path))
  cat(file_path)
```
