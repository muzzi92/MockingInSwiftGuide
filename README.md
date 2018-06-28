# An Example of Mocking and Dependency Injection in Swift #

This guide will walk you through how to mock an external library in Swift, via dependency injection.
In this example, we will be mocking Alamofire. Alamofire was installed using Cocoapods.

### 1) Create a Messenger Class that imports Alamofire and calls its request function inside its own sendMessage function.

```Swift
import Alamofire

 class Messenger {
    let accountSID = ProcessInfo.processInfo.environment["TWILIO_ACCOUNT_SID"]
    let authToken = ProcessInfo.processInfo.environment["TWILIO_AUTH_TOKEN"]
    let url = "https://api.twilio.com/2010-04-01/Accounts/\(accountSID)/Messages"
    let parameters = ["From": "YOUR_TWILIO_NUMBER", "To": "YOUR_PERSONAL_NUMBER", "Body": "Hello from Swift!"]

   func sendMessage() -> Void {
      Alamofire.request(url, method: .post, parameters: parameters)
        .authenticate(user: accountSID, password: authToken)
        .responseJSON { response in
          debugPrint(response)
      }
    }
  }
```

### 2) Extract Alamofire into its own class.

```Swift
import Foundation
import Alamofire

class Alamo {

    func request(url:String, parameters:Parameters, accountSID:String, auth:String) -> Void {
        Alamofire.request(url, method: .post, parameters: parameters)
            .authenticate(user: accountSID, password: auth)
            .responseString { response in
                debugPrint(response)
        }
    }

}
```

### 3) Now in your Messenger Class, you can use dependency injection by inputting an instance of your new Alamo class as a default parameter.

```Swift
class Messenger {

   let accountSID = ProcessInfo.processInfo.environment["TWILIO_ACCOUNT_SID"]
   let authToken = ProcessInfo.processInfo.environment["TWILIO_AUTH_TOKEN"]
   let url = "https://api.twilio.com/2010-04-01/Accounts/\(accountSID)/Messages"
   let parameters = ["From": "YOUR_TWILIO_NUMBER", "To": "YOUR_PERSONAL_NUMBER", "Body": "Hello from Swift!"]

    func sendMessage(alamo:Alamo = Alamo()) -> Void {
        alamo.request(url: url, parameters: parameters, accountSID: accountSID, auth: authToken)
    }

}
```

### 4) In your test suite, create a MockAlamo Class that extends from your Alamo Class.

To make a class a subclass of another, simply use a colon as shown below. This will give the new subclass access to the type and properties of its parent class.   
Use the override keyword to declare a function of the same name as that in its parent class. Please note for an override function to work, it must have the same number and type of parameters, and must be of the same return type (e.g. Void).   
Give the override function some testable functionality. In this case it just increments a counter. This will allow us to test whether of not the function has been called, by asserting if the counter has risen.   

```Swift
import Foundation
import Alamofire
@testable import Angelos_proj

class MockAlamo: Alamo {
    var counter = 0
    override func request(url:String, parameters:Parameters, accountSID:String, auth:String) -> Void {
        counter += 1
    }
}
```

### 5) Inject your Mock Class into your function when unit testing it.

```Swift
func testSendMessageFunctionIsCalled() {
        let mockAlamo = MockAlamo()
        messenger.sendMessage(alamo: mockAlamo)
        XCTAssertEqual(mockAlamo.counter, 1)
    }
```

This test allows us to check whether or not the alamo.request function is called when our sendMessage function is invoked, without depending on the real Alamofire library.
The mockAlamo instance can be injected successfully as a parameter as it is a subclass of our Alamo class, and hence shares the same "Alamo" type.
The Alamo class that we created has one sole function, which is to call the Alamofire request. Resultantly, it does not need to be tested.
