## PonyDB

# TL; DR
- The Goal:
  The Goal was to get "1337" as the favorite number

- The Problem:
  Only numbers between 0-100 were allowed so you had to inject it in some other way and simply using "number" and "1337" as favorite key/value wasn't an option, because the orginal number pair would always be at the end of the Json-string

- The Solution:
  You could overflow the favorite key or value which caused the stored Json-string to exceed the maximal lenght of 256 and thus get truncated. This meant the deserialization wouldn't error out because of trailing data.


# Structure
You can skip this part if you are already familiar with the challenge

The some important parts of the code

1.) The sql_mode

	'sql_mode': 'NO_BACKSLASH_ESCAPES'
 
 So that means Quotes can't be escaped by an backslash
 
 
 2.) The table creation
 
    cursor.execute('CREATE TABLE `ponies` (`name` varchar(64), `bio` varchar(256), `image` varchar(256), `favorites` varchar(256), `session` varchar(64))')
 
 3.) The insert query

    cur.execute(f"INSERT INTO `ponies` VALUES ('{name}', '{bio}', '{image}', " + \
		            f"'{{\"{favorite_key.lower()}\":\"{favorite_value}\"," + \
		            f"\"word\":\"{word.lower()}\",\"number\":{number}}}', " + \
		            f"'{session['id']}')")
 
4.) Input validation
  
    name = request.form['name']
	  if "'" in name: error = 'Name may not contain single quote'
	  if len(name) > 64: error = 'Name too long'
