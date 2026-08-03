description: create a single dummy user in database
allowed tools : Read, Bash(python3:*)

Read database.py file to understand the users table scheme and get_db() helper.

Then wrine and run a python script using bash 
1.that generates a realisting Indian users own knowledge of common Indian names accross regions:
Name : A realisting Indian first + last name
email: derived from the name with a randon 2-3 digit number suffix (example :rahul.sharma91@gmail.com)
password:"password123" hasshed with the werkzeug's generate_password_hash
created at current datetime
check if the generated email is already exist in user table.  if it does regenerate until unique
Inserts the user into the database using the same 
get_db() pattern found in db.py
prints confirmation
id
name
email




