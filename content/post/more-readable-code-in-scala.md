---
title: "More Readable Code in Scala"
date: 2018-05-23T16:33:30+04:30
draft: false
tags: [Scala,clean code]
---
As a Java developer, I 've always had argument with my colleagues to
use *Scala* instead *Java*. They complained that Scala is a complex
language and Java is a more readable and simple one so I never had the opportunity
to use Scala well. Fortunately, recently I 've just joined to the startup company called
[snapptrip.com](https://en.snapptrip.com/), and we use Scala for all the backend services.
A few days ago, I was working on a peace of code in Scala
to *register a new user* into the database. The code was fairly simple.
I had *UserRepository* object containing
*CRUD* operarions:
```
object UserRepository {

  def insert(user: User): Future[Int] = {
    db.run( users += user )
  }

  def getByUsername(username: String) : Future[Option[User]] {
   db.run(
     users.filter( users => users.username === username ).
           take(1).
           result.headOption
   )
  }
}
```
And I had a bunch of services to encapsulate business logic. For example, for registering a user
you probably check username uniqueness and password strengths. This is the code that I wrote first:
```
object UserService {
    def register(user: RegisterRequest) : Future[RegisterResponse] = {
        if(user.username.isEmpty){
          return Future.successful(RegisterResponse(errorMsg = "Please give a username"))
        }
        UserRepository.getByUsername(user.username).flatMap {
          case Some(someUser) => Future.successful(RegisterResponse(errorMsg = "Username already exists"))
          case None => if (user.password.length < 6) {
            Future.successful(RegisterResponse(errorMsg = "Password is too low"))
          } else {
            UserRepository.insert(User(None,user.username,user.password)).map { newId =>
              RegisterResponse(success = true,userID = newId)
            }.recover {
              case _ => RegisterResponse(errorMsg = "Unexpected error has occurred. Please try again later")
            }
          }
        }
    }
}
```
As a guy that always cares a lot to **clean and maintainable code**, this code absolutely is
not my cup of tea. The code was really complex for such a simple thing. Actully register method
should do two things:

1. a bunch of validation
2. save to database if validation succeeded

The picture below depicts the idea:
{{< fluid_imgs
        "center|/static/more-readable-code-scala/flow.png|algorithm"
>}}
So I decided to refactor this code, and the code transformed to this one:
```
 def register(newUser: RegisterRequest) : Future[Response] = {
     Validate(newUser) >
          usernameShouldNotBeEmpty >
          passwordShouldNotBeTooShort >
          usernameShouldNotBeTaken andThen
          saveToDatabase
 }
```
The above code is much more readable and more maintainable, doesn't it? Moreover, if
**functional programming** is all about using functions and composing them together,
I used this idea well. The above code works this way:

> *'newUser'* parameter passes to the *'usernameShouldNotBeEmpty'* function and if the
username is empty then *'register'* will be return early with some type of *Response* without
calling the other functions. On the other hand, if the username is not empty, *'newUser'* will
be passed to the second function. And this flow goes until all validation functions
have been executed. If the last function satisfy the condition, then the actual **action** will
be executed, in this case, saving to the database.

Also pay attention to the return type, that I changed from **Future[RegisterResponse]** to
**Future[Response]**. This way all of the functions can return any type as long as they are
subclasses of **Response** trait:
```
trait Response
case class ErrorResponse(message:String, success:Boolean=false) extends Response
case class SuccessResponse(message:String, success:Boolean=true) extends Response
case class RegisterUserResponse(userId: Long, success:Boolean=true) extends Response
```
And all of this workflow managed by **Validate** class. It defines like this:
```
class Validate[IN,OUT <: Response](input: IN,
                                   valFunctions: List[ValidateFunc[IN,OUT]]=List.empty) {

  def > (f: ValidateFunc[IN,OUT]) = new Validate(input,valFunctions :+ f)

  def andThen(action: IN => Future[OUT]): Future[OUT] = {
    val it = valFunctions.iterator
    while (it.hasNext){
      val nextFunc = it.next
      nextFunc(input) match {
        case Some(result) => return Future.successful(result)
        case _ => // Do nothing
      }
    }

    action(input)
  }

}
```
All validation functions are of type **ValidateFunc** which is basically a
function of type **IN => Option[OUT]**. If something goes wrong, validation functions should
return **Some(OUT)**, otherwise **None**. And OUT is restricted to all subtypes of **Response** by:
```
OUT <: Response
```
As you can see the *Validate* method **>** does nothing except appending validations function to a list.
All the logic of pipeline execution is in **andThen** method. It executes all
the validation functions sequentially and executes the main function, if and only
if all of them return **None**. You can find the full
source code [here](https://github.com/mostafa-asg/DemoAkkaHttp/blob/master/src/main/scala/com.github/services/validation/Validate.scala).

## Last world
I wanted to abstract validation in my code so I wrote Validate class for myself, and
of course, the next step for me, is to read [Cats Validated](https://typelevel.org/cats/datatypes/validated.html).