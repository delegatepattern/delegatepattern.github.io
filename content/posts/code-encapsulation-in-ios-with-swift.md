---
title: "Code Encapsulation in iOS With Swift"
date: 2016-01-04T07:17:21-05:00
draft: true
highlightjs: true
highlightjslanguages: ["swift"]
---

One of the places where most iOS developers make object oriented design mistakes is putting all UI code inside view controllers.

This will lead to an unmanageable and massive view controller. Thus, this clearly violates the MVC pattern guideline.

Let’s be real. You found your way here because you have an interest to know how to make your view controller cleaner and separate concerns and responsibilities.

**But why would developers use this method?**

*   Quick and easy way – which often leads to more problems later on
*   Bad practices – Following some online tutorials

A couple of years ago, I had an interview with a big company for an open iOS developer position. The technical interview went really well, however I failed the practical coding test. I were shocked to know this!

I reached out to the hiring manager to know why they turned me down, he literally said:

> “lack of most important object oriented programming design principles”

I was like, WTF?

I had 3+ years of experience in iOS at that time and were following the best OOP concepts and MVC design pattern guidelines.

I decided to reach out to my mentor and share with him my Xcode project for review. His response was:

*   You are creating massive view controllers by putting all UI code inside.
*   You are NOT using encapsulation for your code ( this shows if you have a good OOP software design mindset).

Dose any of the above sound familiar? It does to me and I bit you used to do the same thing.

Why not just write UI View code inside View Controllers?
========================================================

Well, that’s fine for a simple design that consist of a few UI objects, however when you are designing a more complex screen with many views and other UI objects, this approach makes it extremely hard to manage your view controller and keep it smaller.

At the end, it’s not the concern of view controllers to have the UI code and its actions inside.

By following the quick and easy way, you will end up with the following problems:

*   Massive view controller
*   Hard to manage your code and objects
*   Your code is not reusable on other parts of the app
*   Your code is dependent and if you make a change here, something breaks there

I’m going to create a simple Xcode project to demonstrate this. Go ahead and quickly prototype a simple login screen that consist of the following:

*   UILabel for the login screen name
*   UITextField for username and one for password
*   UIButton for login button

Your final screen show look similar to this:

<p><center><br />
<a href="/img/simple-ios-login-screen.png" target="_blank"><img src="/img/simple-ios-login-screen.png" alt="This is my greatest accomplishment of 2016" width="250" style="border: solid 4px #ffffff; box-shadow: 0px 0px 3px 0px rgba(0,0,0,.15);" /></a><em><small>Simple iOS login screen design.</small></em><br /><br />
</center></p>
  

Make appropriate connections for your UI objects inside your view controller:

    
    class ViewController: UIViewController {
        
        // MARK: Properties:
        @IBOutlet weak var loginScreenLabel: UILabel!
        @IBOutlet weak var userTextField: UITextField!
        @IBOutlet weak var passwordTextField: UITextField!
        
        override func viewDidLoad() {
            super.viewDidLoad()
           
        }
        
        // MARK: Actions
        @IBAction func loginButton(_ sender: Any) {
        }
        
    }
    
    

Now, I want you to imagine that you are working on a complex screen with so many objects, how many lines your UI code will take of your view controller space?

All the actions and properties settings will live inside YOUR massive view controller!

How to Encapsulate UI Code into Custom View?

*   Drag a new UIView from the object library and embed it under the main’s view for your view controller.
*   Embed your UI objects inside a stack view and make the appropriate constraints.
*   Create a new Swift subclass of UIView.
*   Assign the new class to the new added custom view on Storyboard.

Now, you need to write all your outlets and actions inside this custom class for the new custom view. Look at the code below:

    
    class CustomView: UIView {
        
        // MARK: Properties
        @IBOutlet fileprivate weak var loginScreenLabel: UILabel!
        @IBOutlet fileprivate weak var userTextField: UITextField!
        @IBOutlet fileprivate weak var passwordTextField: UITextField!
        
        // MARK: Actions
        @IBAction func loginButton(_ sender: UIButton) {
        }
        
    }
    
    

Do you notice how I controlled the access level to these properties?

Why Code Encapsulation Is So Important?
=======================================

*   Encapsulation promotes maintenance;
*   Increase reusability of your code;
*   Code changes can be made independently
*   Increase security and restricting access to objects

When you apply this to your UI code in any iOS project, you will end up with a much cleaner view controller. Not only this, you’ll be able to reuse the whole login screen code in other projects.

Do you want to see how clean is your old view controller after using code encapsulation?

    
    class ViewController: UIViewController {
        
        // MARK: Properties:
        var newCustomView: CustomView {
            return view as! CustomView
        }
        
        override func viewDidLoad() {
            super.viewDidLoad()
        
        }
        
    }
    
    

We only have one variable instantiated inside the view controller!

**Pay attention, I’m about to mess up with the truth :**

*   The very FIRST thing Employers and professional developers look at when reviewing your project is your view controller class to see if you understand the basics of good software design principles.
*   No good encapsulation means you are NOT good at OOP design principles.

Summary
=======

UI code encapsulation avoids massive view controller, promotes maintainability, testability and restrict objects access to increase security.