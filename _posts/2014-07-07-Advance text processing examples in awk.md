Last week I was reading up on awk and was fascinated by its text processing capabilities despite been written more than 3 decades ago.

Let’s take a simple example of awk on the following “/etc/passwd” file.

leog:x:502:502::/home/leog:/bin/bash
leo:x:506:506::/home/leo:/bin/bash
awk:x:911:911::/home/awk:/sbin/nologin

This file has fields separated by colons.

If I wanted to print only the GID and UID columns using awk then I would do

[leo@awk-examples]$ awk -F: ‘{print $3,$4}' /etc/passwd
502 502
506 506
911 911

The “-F” indicates the field separator by default it is a space.
“$1, $2, $3” are awk built-in variables which indicate the column number. (Just remember to separate them with a comma else they will print output without any space)

This is all easy stuff, let’s take this further and use awk to print the GID and UID of only a user leo.

[leo@awk-examples]$ awk -F: '$1 ~ /^leo$/ {print $3,$4}' /etc/passwd
506 506

“$1 ~ /^leo$/” This expression compares only fields in the first column to leo since “^” indicates start’s with and “$” indicates ends with, this expression will only match leo and not leog.
“~” is used to match a regular expression match.

Things to note
* Awk will only execute the code in the bracket if the preceding expression evaluates to true.
* If you use the expression “/^leo$/ {print $3,$4}” without “$1”this will do a line match and try and match a line which starts with and ends with leo.
* The field separator can also takes an expression, for example to match a file with multiple tabs

 awk -F"\t+"

or a file separated by word and two or may three digits

 awk -F  “word[0-9][0-9][0-9]”

Let’s take another awk example.
Finding the value for a specific row and column
The below file contains the list of products with their prices for different brands.

ProductName:Brand1:Brand2:Brand3
monitor:100:200:300
keyboard:50:100:150
mouse:10:20:30

Now if you followed my earlier example then finding the prices for a particular product would be as follows

awk -F:  ‘$1 ~/^monitor$/ {print}’ filename.

But what if I want to find the price for a particular brand as well? No problem awk can do that as well

[leo@awk-examples]$ awk -F: 'NR==1  {for(i=2; i < NF+1; i++){ if ($i~/Brand2/) { cn = i; break; }}} $1 ~/mouse/  { print $cn; exit; }' prices.txt
20

“NR “ is an awk built-in variable which stores the number of records. The above code will only execute on the first line.
“NF” is another awk built-in variable which stores the number of fields.

The above code will evaluate the first record for “Brand2” and store its column value in “cn”, it will then check the first column for mouse and print the price of the column stored in “$cn”.

Variables in AWK
You can even have predefined variables and access awk code from a file to keep it simple.

[leo@awk-examples]$ vim prices.awk
NR==1  {
    for(i=2; i < NF; i++)
      { if ($i == c) { cn = i; break; }}} $1 == p  { print $cn; exit; }
[leo@dns awk-examples]$ awk -F: -f prices.awk -v p="monitor" c="Brand2" prices.txt


To awk, everything is a string although we perform numeric comparisons, all awk variables are stored internally as strings, awk automatically takes care of the string-to-number conversion steps.

[leo@awk-examples]$ vim prices.awk
NR==1  {
    for(i=2; i < NF; i++)
      { if ($i == c) { cn = i; break; }}} $1 == p  { print $cn; exit; }
END {
total = $cn*2
 print total
}
[leo@awk-examples]$ awk -F: -f prices.awk -v p="monitor" c="Brand2" prices.txt
200
400
Much more convienent than bash where you would need to enclose numeric calculations in $(()).

END is a special block, Awk executes this block after all lines in the input file have been processed.

You may also like to read about BASH if statement examples

Ref
http://www.ibm.com/developerworks/library/l-awk2/
http://en.wikibooks.org/wiki/An_Awk_Primer/Variables
