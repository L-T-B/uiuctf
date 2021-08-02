# PonyDB

## TL; DR
- The Goal:
  The Goal was to get "1337" as the favorite number

- The Problem:
  Only numbers between 0-100 were allowed so you had to inject it in some other way and simply using "number" and "1337" as favorite key/value wasn't an option, because the orginal number pair would always be at the end of the Json-string

- The Solution:
  You could overflow the favorite key or value which caused the stored Json-string to exceed the maximal lenght of 256 and thus get truncated. This meant the deserialization wouldn't error out because of trailing data.


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


And except for **favortite key/value pair**:

	favorite_key = request.form['favorite_key']
	if "'" in favorite_key: error = 'Custom favorite name may not contain single quote'
	if len(favorite_key) > 64: 'Custom favorite name too long'

	favorite_value = request.form['favorite_value']
	if "'" in favorite_value: error = 'Custom favorite may not contain single quote'
	if len(favorite_value) > 64: 'Custom favorite too long'

They have a small diffrenze they don't define the error variable if the lenght isn't right

A SQL injection isn't possible but the injection of arbitrary JSON data is...

### 5.) Flag reveal

	{% if favorite == 'number' and pony['favorites'][favorite] == 1337 %}
	Favorite flag: {{ flag }}
	{% endif %} {% endfor %}

The Flag will be shown if the favorite number is 1337


## Exploitation

Let's recap:
- the favorites Column (JSON Data) is maxed at 256 chars
- we are able to insert an arbitrary amount of characters into a string which will get stored in this column
- the original number/value pair gets checked and will always be appended to the end of the Json array

Simply closing the Json Array with an Exploit like

	favorite_key = 'somevalue'
	favorite_value = 'a", "number": 1337}'

would result in a JSON data like
	
	{"somevalue":"a", "number": 1337}","word":"someword,"number":10}

This isn't a valid JSON Array for the python JSON parser and it will thow an error ("json trailing data") and stop the application

So we had to trim the Json Data in some way...

So I asked myself what would happen if the string overflows the maximal characters...

and BINGO!

If the string is longer than the maximal amount, Mysql will just show a waring and trim the string to the maximal size

## The Payload

