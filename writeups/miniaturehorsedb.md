# PonyDB

## TL; DR
- The Goal:
  The Goal was to get "1337" as the favorite number

- The Problem:
  Only numbers between 0-100 were allowed so you had to inject it in some other way and simply using "number" and "1337" as favorite key/value wasn't an option, because the orginal number pair would always be at the end of the Json-string

- The Solution:
  You could overflow the favorite key, value and word with the char "İ" which doubles in size as beeing formated in a string (in certain circumstances) which caused the stored Json-string to exceed the maximal lenght of 256 and thus get truncated. This meant the deserialization wouldn't error out because of trailing data.


## Structure
You can skip this part if you are already familiar with the challenge

The some important parts of the code

### 1.) The sql_mode

	'sql_mode': 'NO_BACKSLASH_ESCAPES'
 
 So that means Quotes can't be escaped by an backslash
 
 
 ### 2.) The table creation
 
    cursor.execute('CREATE TABLE `ponies` (`name` varchar(64), `bio` varchar(256), `image` varchar(256), `favorites` varchar(256), `session` varchar(64))')
 
 ### 3.) The insert query

    cur.execute(f"INSERT INTO `ponies` VALUES ('{name}', '{bio}', '{image}', " + \
		            f"'{{\"{favorite_key.lower()}\":\"{favorite_value}\"," + \
		            f"\"word\":\"{word.lower()}\",\"number\":{number}}}', " + \
		            f"'{session['id']}')")
 
### 4.) Input validation

(If error has a value not null the insert query doesn't get executed)

Same for all other fields...

    name = request.form['name']
	  if "'" in name: error = 'Name may not contain single quote'
	  if len(name) > 64: error = 'Name too long'

Except for the favorite number:

	number = int(request.form['number'])
	if number >= 100: error = "Ponies can't count that high"
	if number < 0: error = "Ponies can't count that low"

(this validates that the number is between 0 and 100)


### 5.) Flag reveal

	{% if favorite == 'number' and pony['favorites'][favorite] == 1337 %}
	Favorite flag: {{ flag }}
	{% endif %} {% endfor %}

The Flag will be shown if the favorite number is 1337


## Exploitation

Let's recap:
- the favorites Column (JSON Data) is maxed at 256 chars
- the original number/value pair gets checked and will always be appended to the end of the Json array
- from ponydb we know that we have to overflow somehow the JSON data

After playing around I noticed that the chr(304) (Python syntax) sometimes gets converted in an invisible + "i" character

So the sub-string expands if beeing inserted into the main-string

## The Payload

this was pretty straight forward
important: this time three variables were needed to insert the exploit

	word='aaaİİİİİİİİİİİİİİİİİİİİİİİ"}'
	favorite_key='İİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİ'
	favorite_value='","number":1337, "İİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİİ": "'


## Thank you for reading :)
