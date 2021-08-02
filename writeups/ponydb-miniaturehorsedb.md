## PonyDB

# TL; DR
- The Goal:
  The Goal was to get "1337" as the favorite number

- The Problem:
  Only numbers between 0-100 were allowed so you had to inject it in some other way and simply using "number" and "1337" as favorite key/value wasn't an option, because the orginal number value would always be at the end of the Json-string

- The Solution:
  You could overflow the favorite key or value which caused the stored Json-string to exceed the max lenght of 256 and thus get truncated. This meant the deserialization wouldn't error
