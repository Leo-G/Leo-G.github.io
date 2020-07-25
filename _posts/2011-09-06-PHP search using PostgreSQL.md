A simple search form to search  the db and display the contents via PHP and PostgreSQL

Name the below as index.html

 

<h2>Search</h2>
<form name="search" method="post" action="search.php">
Seach for: <input type="text" name="find" /> in
<Select NAME="field">
<Option VALUE="emailaddress">emailaddress</option>
<Option VALUE="domainname">domainname</option>
</Select>
<input type="hidden" name="searching" value="yes" />
<input type="submit" name="search" value="Search" />
</form>


Name the below as search.php

 

 

<?
$searching = $_POST['searching'];
$find = $_POST['find'];
$field = $_POST['field'];
//This is only displayed if they have submitted the form
if ($searching =="yes")
{
echo "<h2>Results</h2><p>";
//If they did not enter a search term we give them an error
if ($find == "")
{
echo "<p>You forgot to enter a search term";
exit;
}
// We preform a bit of filtering
//$find = strtoupper($find);
$find = strip_tags($find);
$find = trim ($find);
// attempt a connection
// $dbh = pg_connect("host=hostname port=5432 dbname=dbname  user= username password=password");
//if (!$dbh) {
//  die("Error in connection: " . pg_last_error());
// }
include_once("connection/connection.php");
// execute query
$sql = "SELECT * from table where $field = '$find' limit 30 ";
$result = pg_query($dbh, $sql);
if (!$result) {
die("Error in SQL query: " . pg_last_error());
}
echo "<table border='1'>
<tr>
<th>Firstname</th>
<th>Lastname</th>
<th>Email address</th>
</tr>";
while ($row = pg_fetch_array($result)) {
echo "<tr>";
echo "<td> " . $row[0] . "</td>";
echo " <td>" . $row[1] . "</td>";
echo "<td> " . $row[2] . "</td>";
echo "</tr>";
}
echo "</table>";
$rows = pg_num_rows($result);
if ($rows == 0)
{
echo "Sorry, but we can not find an entry to match your query<br><br>";
}
//And we remind them what they searched for
echo "<b>Searched For:</b> " .$find;
echo "<b>Number of rows :</b> " .$rows;
// free memory
pg_free_result($result);
pg_close($dbh);
}
?>
 
