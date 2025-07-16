## How to Query Docker Database

In order to query the NI2 database while running the system in Docker, the following steps must be followed
1. Open Command Line terminal
2. Run ```kubectl exec -it deploy/ni2-postgres --bash```
3. Commands must be prefixed with ```psql -U foci_user -d ni2_db -c```
   4. Add the sql to perform the query you need

An example query might look like ```psql -U foci_user -d ni2_db -c "select full_name from users where id = 99```

## Common Queries

### Find FOCI Users
*sf328id can be found by looking at the URL of the package in the SIT app*

- Get assigned userid for a specific sf328 
   - `select assigned_userid from foci.sf328s where id = sf328Id`

- Get assigned CI userid for a specific sf328
  - `select assigned_ci_userid from foci.sf328s where id = sf328Id`

- Get assigned userid, certificate username, and role title for a specific sf328
```
select sf.assigned_userid, u.cert_username, r.display_name as "Role Title"
from foci.sf328s sf
join core.users u on sf.assigned_userid = u.id
join core.roles r on u.id = r.id
where sf.id = 5001
```

- Get assigned CI userid, certificate username, and role title for a specific sf328
```
select sf.assigned_userid, u.cert_username, r.display_name as "Role Title"
from foci.sf328s sf
join core.users u on sf.assigned_userid = u.id
join core.roles r on u.id = r.id
where sf.id = 5001
```


