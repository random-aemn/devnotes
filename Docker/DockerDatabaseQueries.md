## How to Query Docker Database

In order to query the NI2 database while running the system in Docker, the following steps must be followed
1. Open Command Line terminal
2. Run the following command
   3. ```kubectl exec -it deploy/ni2-postgres -- bash```
3. Run 
   4. ```psql -U foci_user -d ni2_db```
4. Add the sql to perform the query you need, e.g., ```select * from users;```
<br>
<br>


`Kubectl` is the Kubernetes command line tool

`Exec` tells Kubernetes to execute a specific command _inside_ an existing and running container

`-it` means the terminal will be interactive _AND_ provide a terminal-like interface

`deploy/ni2-postgres` finds the pod/deployment that is running the PostgreSQL database.  ni2-postgres is the name of the deployment and Kubernetes will connect to one of the pods running under that deployment name

`--` is a separator that tells kubectl that the configuration options are finished and everything that follows should be passed as a command to be executed within the container

`bash` launches a Bash shell

At this point, we're now "inside" the database container and we can run commands to start querying the Postgres DB.

`psql` is the command line front end tool for PostgreSQL - allowing us to type and execute raw sql queries

`-U foci_user` tells the database that a User is trying to log-in and foci_user is the specific database username.  *The username _will_ change depending on the DB and schema

`-d ni2_db` says that I want to connect to a database and the name of the specific database I want to connect to.

Now we can directly query the database using the terminal
 

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
select sf.assigned_userid, u.cert_username, r.display_name as
from foci.sf328s sf
join core.users u on sf.assigned_userid = u.id
join core.users_roles ur on u.id = ur.user_id
join core.roles r on ur.role_id = r.id
where sf.id = 5001;
```

```
select id, display_name, description
from roles
order by id asc;```


