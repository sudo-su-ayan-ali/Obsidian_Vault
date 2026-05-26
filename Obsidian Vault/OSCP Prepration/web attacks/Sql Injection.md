# solving labs of web security acedamy
## lab number 1:-/
- topic: **# SQL injection vulnerability in WHERE clause allowing retrieval of hidden data.**
	- step 1:
		- select any one catagory then go to url and add single qoute at the end of url and see for error like internal server error.
	- step2:
		- and now add (--) this to comment out rest of the remeaning sql query in the backend of server.
	- step3:
		- now adding a filter to get query true so we use OR operator `OR 1=1`.
	- step4:
		- combinne all the queries and the final query be like .
```sql
' OR 1=1--
```
now you solved the lab .
## lab number 2:-/
- topic: **# SQL injection vulnerability allowing login bypass.**
	- step1:
		- go to login and we try to bypass and get login as administrator trying to get error internal server via old school techniqes.
	- step2:
		- commenting out rest of all query after administrator and the single quote by add ( -- ) this.
	- step3:
		- the final query is be like.
```sql
administrator'--
```

- now you solved the lab.
