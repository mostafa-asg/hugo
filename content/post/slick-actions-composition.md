---
title: "Slick Actions Composition"
date: 2018-06-04T11:28:54+04:30
draft: false
tags: [Scala,Slick]
---
## Problem
You want to insert a new row to a table which has auto increment primary key; Also
some additional information must be persist in other tables. At the end you also want
to return the inserted generated ID. Imaging you are developing
a new web application, and you are working on user registration code. Each time a
new user signs up, you want to store his credential in **users** table and all of his roles
in **userRoles** table. Here is the potential data model:
{{< fluid_imgs
	"center|/static/slick-actions-composition/data-model.png|data models"
>}}
How do you do this in [Slick](http://slick.lightbend.com/)? Sounds extremely simple!?
Believe me or not, many of my experienced colleagues do it the wrong way. Here is their's solution:

1. Insert new user into the **users** table and grab the auto generated primary key
2. Use the generated user id for inserting into other tables, for example **userRoles**
3. Return the generated user id to the caller

Although it works, it has some drawbacks. First of all, in this scenario, storing
user information in multiple tables should be done in an **atomic** way. All rows must
be inserted or none of them. However in this naive implementation, you may encounter
partial insert, because of failure in second or later inserts, leaving database in an inconsistent way.

Secondly, It has performance drawback because for inserting in each table,
new database roundtrip is involved.

## Solution
Slick uses **DBIOAction** for anything that can be executed on a database, whether it is
a getting the result of a query (myQuery.result), creating a table (myTable.schema.create),
inserting data (myTable += item) or something else. You can compose **DBIOAction**s using
provided methods like **map** and **flatMap** to construct new DBIOAction and run all of them
transactionaly using Slick.

## Source Code
To insert into users table and returning the inserted primary key:
```
def insert(user: User) = (users returning users.map(_.id)) += user
```
The return type of above method is **DBIOAction** parameterized by the result type
it will produce when you execute it which in this case is **Long**. Also note that
nothing has happend into database now. We 've just defined the action. Later
we will run these actions against database. For inserting row into userRoles:
```
def insert(rows: Seq[UserRole]) = userRoles ++= rows
```
Nothing special in the above code. Now the intersting part, composing actions togeher:
```
def register(user:User , roleIds: List[Long]) = {
 insert(user).flatMap { userId =>
    val insertUserRoleAction = insert( roleIds.map(roleId => UserRole(userId,roleId)) )
    insertUserRoleAction.map( _ => userId )
 }
}
```
We can rewrite above function even simpler using for-comprehension:
```
def register(user:User , roleIds: List[Long]) = {
  for {
       userId <- insert(user)
       _ <- insert( roleIds.map(roleId => UserRole(userId,roleId)) )
  } yield userId
}
```
And finally run *register* action **transactionaly**:
```
    val userId = Await.result(
        db.run(register(User("mostafa.asg","pass"),List(1,2)).transactionally),
        Duration.Inf
    )
```
You can find the full source code [here](https://gist.github.com/mostafa-asg/adbcf56f766273c84b6066d4b71f019e)