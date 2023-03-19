---
title: "How to Refactor Huge Codebase Incrementally"
date: 2023-03-18T08:12:35-06:00
draft: false
---

Today's topic about refactoring code incrementally without breaking the entire codebase.

Let's take a quick look at this minimal SwiftUI app we'll use for this lesson:


## User - Model Type

```
struct User {
    var followers: [Int] = []
}
```


## StateController - Global Shared Access

```
class StateController: ObservableObject {
    @Published var user = User(followers:[10,20])
}
```


## RefactorAPP - Environment Object 

```
struct RefactorApp: App {
    @StateObject var stateController = StateController()
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(stateController)
        }
    }
}
```

Finally, lets add some logic to the ContentView:


## ContentView - Add Logic

```
struct ContentView: View {
    @EnvironmentObject var stateController: StateController

    var followers: Int {
        var total = 0
        for follower in stateController.user.followers {
            total += follower
        }
        return total
    }

    var body: some View {
        VStack {
            Text("Total followers: \(followers)")
                .padding()
            Button("Add 10 more followers") {
                stateController.user.followers.append(10)
            }
        }
    }
}
```
</br>
This is how our our mini app looks

![Initial app look](/img/03/InitialAppLook.gif)

</br>
This is a simple straightforward example. I know and get it!  
However, imagine if this is would be a real app. You might have things like:

- **User** struct will have more properties
- **ContentView** with more complex layout
- **StateController** will save data either using file system or Core Data etc..
- Complex navigation stack that will deeply reference both the User and StageController instances.  



A couple of noticeable issues here:
- Putting domain business logic into the view layer while leaving the model layer super thin
- The current structure of this app is untestable.  

Why it’s not easy testable?  
- The **ContentView** struct reads its data directly from the **StateController** object. This object more likely will be connected to other objects like Network and Storage managers. This makes it super hard to setup unit tests. You will have to write Test Doubles to test code that likely to change and break you test cases. (Complex unit testing method to verify poorly written code). 
- **StateController** is an object and ContentView is a structure. It’s not a good practice to mix value types with reference types. This will make your structure behaves like an object. (This is an advanced topic that I cannot cover here. Feel free to reach out if you have any specific questions on this).
- Finaly, the code that adds new followers to the **User** instance is deeply nested in the view. Simply it’s not easily reachable so you can test it.

</br>

You might wondering how come this simple structure of an App comes with all these issues?  

I purposely showed you the most commonly used structure that you see other developers use so you can be familiar with it.   

The couple of noticeable issues HELPS us to identify the solution:
- Enhance the structure of our App
- Be able to write unit tests for it  

</br>

_________________

## Refactoring

We can go ahead and refactor the **User** struct (it’s nothing but a simple model). Model layer is the easiest to work with. So let’s improve it together:
</br>

```
struct Follower {
    let number: Int
}

struct User {
    private(set) var followers: [Follower] = []

    var totalFollowers: Int {
        var total = 0
        for follower in followers {
            total += follower.number
        }
        return total
    }

    mutating func addFollower(withNumber number: Int) {
        followers.append(Follower(number: number))
    }
}

```  

</br>

Our User struct is well refactored and ready for a straightforward tests. However, we can’t run the app anymore. Because of the changes we applied to the User type, the compiler is throwing multiple errors preventing us from compiling and building.

![Initial app look](/img/03/compiler-errors.png)  


That’s the main reason that developers are pushed back from refactoring. Small changes to an interface will highly leads to tens or even hundreds of compiler errors.  

While our app is simple and fixing these errors super easy, but with a real big codebase the task will be much more complex and time consuming.  

Imagine if your resources are limited and you short on time and need to release the app soon, what you would do?  

That’s where the ***Firebreak Types*** come handy.

# Introducing Firebreak Types...

Firebreak types allow you to fix all compiler errors and incrementally refactor bad code.

What is a Firebreak Type?  

It’s a temporary wrapper around our newly reworked type (*User* struct that we recently refactored).  

This wrapper mimics the **User** struct old interface.  

The purpose is to allow us to incrementally refactor bad code, write unit tests and fix all compiler errors with ease. All this is done in isolation of other parts of the codebase. This approach gives us the ability to ship and release new versions of the app while the refactoring is still in progress.  

Isn't that powerful?  

As I mentioned in this blog post, it’s highly recommended to break your project down into multiple modules using Swift Package Manager. Each module is a package and Xcode will create a separate target for each one.  

## Creating Model Package - Swift Package Manager


 You should already know how to create Swift Package. I named it *Model*, but you can name it anything you like.
 ![Swift Model Package](/img/03/swift-model-package.png) 

</br>

**NEXT:** Let's move the new refactored type (**User** and **Follower** structs) to the Swift Model package we just created.  

> **IMPORTANT NOTE:** We set the type and its properties and methods to *public* access, because this module is part of a different target and the default access control in Swift is *internal*.

</br>

This is how the our Model package and the refactored code looks like:

 ![Model Package With Refactored Code](/img/03/model-package-look.png)  
 
 So far we have done followings:
 - **Model** Module: Separate package for the model layer
 - **User** type is fully refactored and ready for unit tests ( can be tested in isolation of other parts). This type is part of the Model module now.

> **NOTE:** You can buid and run the app and everything should work. The reason is, the refctored types are part of a different target. We'd have to import the **Model** module in order to use it.

</br>

It's time to add some sample complmantry unit tests to our model types. Unit Testing files will be added to the SP **Tests** folder. Again, this is a great way to work on refactoring and adding unit tests in a complete isolation of other sections of the codebase.  

The reason I'm showing you this, is to encourge you to explore these powerful technuqies and use them wisely in your own projects.  


 ```  
import XCTest
@testable import Model

// MARK: - Follower Tests
final class FollowerTests: XCTestCase {
    func testZeroFollowers() {
        let follower = Follower(number: 0)
        XCTAssertEqual(follower.number, 0)
    }

    func testCountFollowers() {
        let followers = Follower(number: 20)
        XCTAssertEqual(followers.number, 20)
    }
}

// MARK: - User Tests
final class UserTests: XCTestCase {
    func testEmptyUser() {
        let user = User()
        XCTAssertEqual(user.followers.count, 0)
        XCTAssertEqual(user.totalFollowers, 0)
    }

    func testAppendFollowersToUser() {
        var user = User()
        user.addFollower(withNumber: 20)
        XCTAssertEqual(user.followers.count, 1)
        XCTAssertEqual(user.totalFollowers, 20)
    }

    func testAddMultipleFollowersToUser() {
        let user = User(followers: followers)
        XCTAssertEqual(user.followers.count, 3)
        XCTAssertEqual(user.totalFollowers, 60)
    }
}

// MARK: Private extenions.
private extension UserTests {
    var followers: [Follower] {
        return [Follower(number: 10),
                Follower(number: 20),
                Follower(number: 30),
        ]
    }
}

 ```  
</br>

You should see the test file added to the Model package inside Xcode Project Navigator:
 ![Add Unit Tests to ModelTests Folder](/img/03/Add-Unit-Tests.png) 

</br>


_________________

# Using Firebreak Type for Production

We need to apply the followings:
- Replace the old User type with the new refactored version.
- Update the UI to reflect the new changes.  

But we still can’t do this, the old interface is different and Xcode throwing compiler errors related to these lines in both **StateController** and **ContentView** files.

So it’s clear that we are ready to create our new Firebreak Type to wrap the Model.User type and mimics the old interface used in both **StateController** and **ContentView**.   

What I’m about to show you is that we are assuming our resources are limited and we are short on time and need to pause with refactoring now and continue ship and release new version of the app.  

 Again, we are strategically and incrementally refactoring bad code in isolations of other parts of the whole system.  


# When to Apply FireBreak Type?

1. You have finished partially or completely refactoring your old code and (maybe added required unit tests as well);
2. Your refactored code is isolated and independant before its first used. Using it directly will lead to so many compiler errors, and thats the purpose the Firebreak Type (wrapper) exist.
3. You are short on time and need to release and ship your app without having to wait for the complete refactoring process to be finished.

It's clear that it's the time to create this Wrapper around the model types we refactored and starting using them in the main target. Let's do this together :)  

## Identify the Areas that Generate Errors
- **StateController:** Old User init takes an array of integers. New type init takes an array of type Transaction. 
- **ContentView:** total followers computed property logic is based on calculating the total / sum of the integer values in the followers array. New type uses custom type for followers and is type of *[Follower]*. 
- **ContentView:** We can’t add new followers anymore. The property is private for setting new value.  

According to that, we have to accommodate these old interface requirements in our Firebreak Type.  

## Let’s implement the Firebreak Type now:
 I prefer to keep the old name as User. You don’t have to. This allows us to leave all other code untouched.  
 
  You might ask how about the **User** type that lives or part of the Model package? Well, since that in its own module, we will have to either import the module in the file where we want to use it or use namespaces to access it like this: **Model.User**

So, the Firebreak Type **User** (we will name our wrapper User) in the main target is NOT the same as the **Model.User** that is refactored and part of the package folder. Let’s move on…  

I want to to look at the 3 issues I listed above (Identify the Areas that Generate Errors) and write the solutions accordingly to satisfy the old interface.  

##### #1 Custom  Model.User init that takes an array of int:

 Let’s write an extension to the new type with a custom init: 
 
 > **NOTE:** Make sure to add the Model package to Xcode interface by going to the main target - > General tab -> Frameworks, Libraries and embedded content. Hit the + sign and choose the Model or whatever the name of the package you have chosen. This allows us to import this module into the main target.
 
 The Custom init in the extension that I'm about to show you, should satisfy the compiler error showing in the **StateController**  
 
 ##### #2 Firebreak Type that we are naming also *User* should provide a followers count computer property and a method to add followers.  
 
 Here is the final look of our User struct that acts as the Firebreak Type now:  
 
 
```
import Model

// MARK: - Firebreak Type - User - Mimicing the old api interface.
struct User {
    var underlying: Model.User

    var followers: [Int] {
        get {
            var num: [Int] = []
            for follower in underlying.followers {
                num.append(follower.number)
            }
            return num
        }
        set {
            underlying = Model.User(followers: newValue)
        }
    }

    init(followers: [Int]) {
        underlying = Model.User(followers: followers)
    }
}

// MARK: - Model.User extension.
extension Model.User {
    init(followers: [Int]) {
        self.init()
        for follower in followers {
            addFollower(withNumber: follower)
        }
    }
}

```
 
  
Build and Run. Our app should run without any compiler erros. Thanks to the Firebreak Type Wrapper. We did't have to touch any of these errors in order to fix them. That's the power of this method.

 ![Our app builds and runs with no compiler errors](/img/03/app-runs-no-compiler-errors.png) 
 
 
**NOW:** Our app builds and compiles with no errors. This technique gives us some golden benefits:
1. We can ship app updates while our code is in such intermediate state
2. We can take our time to slowly old wrong code with the new API.  

> **NOTE:** We left the compiler errors and both **StateController** and **ContentView** files untouched. Mimicing the old interface made this possible. 

_________________


## Replacing Old Code With The New API

By now, we assume that it's time to replace the old code and remove the Firebreak type (wrapper) and use our new refactored and tested code instead.  

It’s better to start with the view layer first. If you visualize the shape of the MVC pattern, you will see that the **StateController** is on the top as a global shared resource. Since views are usually down the MVC tree-shaped diagram, making the changes to the **StateController** will put us back into the the same cycle of compiler errors. **StateController** is referenced in these views. That’s why.  

</br>

Simple method to use when you ready to remove the Firebreak type:
- Start with the view layer. Cleaning up the views. You might still have to use the wrapper type temprorally until you finish cleaning the global shared controller. So keep this in mind
- Toggle the **StateController**: Other types that non-view based. Removing the Firebreak Type altogether.
- Go back to the *view layer* and update it according to any updates made to the controller layer.

## STEP #1
Let’s start replacing the old code from the view layer first:

 ![View layer first update](/img/03/ContentView-First-Update.png) 

Take a look at the blue lines showing in the image above. As I said, we are still partially using the wrapper in the view layer as we move up towrds the global shared layer. Once that's done, we'll be able to come back to **ContentView** and remove the Firebreak type altogether. 

## STEP #2
It’s time to update the **StateController**:

### StateController.swift:

```
import SwiftUI
import Model

// MARK: - StateController
class StateController: ObservableObject {
    @Published var User: Model.User = {
        var user = Model.User()
        user.addFollower(withNumber: 10)
        user.addFollower(withNumber: 20)
        user.addFollower(withNumber: 30)
        return user
    }()
}

```

## STEP #3

Finally, one last update to the view layer. Since **StateController** have direct access to the new User type. We should allow the View to access that info from the controller.  

Go back to the view layer and remove the wrapper type completely. This is the final step.

### ContentView.swift:


```
import SwiftUI
import Model

struct ContentView: View {
    @EnvironmentObject var stateController: StateController

    var followers: Int {
        stateController.user.totalFollowers
    }

    var body: some View {
        VStack {
            Text("Total followers: \(followers)")
                .padding()
            Button("Add 10 more followers") {
                stateController.user.addFollower(withNumber: 10)
            }
        }
    }
}

```

Now we have a fully functional, well refactored and unit tested app. Build and run and you should see yourself smiling :)  

 ![Final refactored app](/img/03/final-refactored-app.png) 
 
 </br>

## STEP #4

Removing the Firebreak Type (**User.swift**) file. *No longer needed*. I'm going to delete it from the project just to show you how we uitlized it for our advantage to make our refactoring easier and quicker.   

But once we are done. We should not let old and unused code creep into the codebase.


Our refactoring is complete. This process is incremental and considers making changes one item at a time without breaking the entire codebase and generating a long list of compiler errors.  

This should be enough to give you an overall idea of how to toggle refactoring in small pieces.  


## Final Thoughts
[You can download the complete Xcode project from my GitHub page here.](https://github.com/delegatepattern/Refactor.git)
 Each commit comes with a specific tag number that makes it easy for your to follow along this post step by step.  
 
Feel free to reach out for any questions or comments. Would love to hear from my fellow developers.

Extra Material:
- [Unit Testing Huge Codebase](/posts/unit-testing-huge-codebase/)
