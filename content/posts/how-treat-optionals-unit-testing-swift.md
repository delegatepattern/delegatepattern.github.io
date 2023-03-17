---
title: "How to unit-test optionals in Swift?"
date: 2023-01-15T14:08:49-05:00
draft: true
---
[XCTest](https://developer.apple.com/documentation/xctest) framework has a method that treats optionals. It’s named [XCTUnwrap](https://developer.apple.com/documentation/xctest/3380195-xctunwrap). 

This method takes the optional property as one of its parameters and dose of one the followings:

* Returns the unwrapped value (if there is a value)
* Throws an error if it finds nil (no value)

Because of the possibility of throwing error, you will need to use the **try** keyword before calling the method. In other words, you will also have to mark your unit testing method as throwing with the **throws** keyword.

Interestingly, XCTest framework treats any throwing error as a test failure. 

Let’s take a look at this example:

~~~
struct User {
    let name: String
    let username: String?
}

extension User {
    var profileURL: URL? {
        let baseURL = URL(string: "https://github.com/")!
        guard let id = username else {
            assertionFailure("The username is invalid")
            return nil

        }
        return baseURL.appendingPathComponent(id)
    }
}
~~~

Testing a valid profile URL

~~~
    func testValidProfileURL() throws {
        let user = User(name: "delegate", username:"delegatepattern")
        let profileString = "https://github.com/delegatepattern"
        let url = try XCTUnwrap(user.profileURL)

        XCTAssertEqual(url.absoluteString, profileString)
    }
~~~

Now it's time to test invalid profile URL:

