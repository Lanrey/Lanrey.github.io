# Using Router Design Pattern in Node.js

## Introduction
 How do you tame a bunch of if-else statements or switch statement that grows with every feature request? How do you handle conditions that require multiple evaluations?
 
 Typically, in Node.js, asides using if-else statements and switch statements, ternary operators can also be used in evaluating multiple conditions, but this approach becomes harder to read when the conditions become large, and even harder to read when the ternary operators become nested.
 
 The router pattern approaches this problem by using objects for lookups, think *hashmaps* or *dictionaries* and replaces control flow with datastructures. This approach also enables separation of concerns, wrapping business logic (conditions) as routes, then creating a dispatcher function (router). Using this approach makes your codebase more functional, improving code readability, and makes your codebase easier to test.
 
 In this article, we will refactor an existing sample codebase, that is riddled with numerous if-else statements. The article would also provide a step-by-step refactor process, and eliminate all if-else statements with object lookups **(router pattern)**.
 
 ## Example
 We will be refactoring some code for an order service that changes the state of an order based on different order status events.

 ```
    /// List of possible events order service can process
    let orderStatusEvents = {
        orderCreated: 'order-made',
        orderCheck: 'order-check-in-progress'
        orderAccepted: 'order-accepted',
        orderConfirmed: 'order-confirmed',
        orderDelivered: 'order-delivered',
    }
    
    let orderDispatcher = (orderStatus) => {
        if (Object.is(orderStatus, orderStatusEvents.orderCreated)) {
            //hypothetical function for processing order created
            orderCreated(orderStatus);
        } else if (Object.is(orderStatus, orderStatusEvents.orderCheck) {
            orderCheck(orderStatus);
        } else if (Object.is(orderStatus, orderStatusEvents.orderAccepted) {
            orderAccepted(orderStatus);
        } else if (Object.is(orderStatus, orderStatusEvents.orderConfirmed) {
            orderConfirmed(orderStatus);
        } else if (Object.is(orderStatus, orderStatusEvents.orderDelivered) {
            orderDelivered(orderStatus);
        }
        return "There has been an error processing this error!!";
    };
 ```
 
 So far, this order service can respond to a set of events. If you ask the order service to process an event it doesn't understand, it responds with a fallback message.
 
 At the moment, all of the logic lives in the `orderDispatcher` function. The function currently has 6 behaviours, so there are 5 if-else statements and one fallback return statement. The code is still relatively short right now, due to the number of events the order dispatcher currently supports, for each new event added, a new if-else statement would be added to the `orderDispatcher` function .
 
 Ballooning if-else or switch statements are a code smell that suggests the `orderDispatcher` function might have too many responsibilities.
 
 So let's refactor this codebase with the Router design pattern.
 
 ## Step 1 - Extract each case into a separate function, list them in a data structure
 
 Let's move the code for the order status received into an object called `orderStatusResponses` using the events as the key.
 ```
 let orderStatusResponses = {
     'order-made': (orderStatus) => orderCreated(orderStatus),
     'order-check-in-progress': (orderStatus) => ordercheck(orderStatus),
     'order-accepted': (orderStatus) => orderAccepted(orderStatus),
     'order-confirmed': (orderStatus) => orderConfirmed(orderStatus),
     'order-delivered': (orderStatus) => orderDelivered(orderStatus),
 }
 ```
 
 Now that we've moved each event into the `orderStatusResponses` object, we can replace the cases by looking up the appropriate `orderstatusResponses` function and invoking it. At this point, our code should still work exactly as before.
 
 ```
 
     let orderDispatcher = (orderStatus) => {
        if (Object.is(orderStatus, orderStatusEvents.orderCreated)) {
            return orderStatusResponses['order-made'](orderStatus);
        } else if (Object.is(orderStatus, orderStatusEvents.orderCheck) {
            return orderStatusResponses['order-check-in-progress'](orderStatus);
        } else if (Object.is(orderStatus, orderStatusEvents.orderAccepted) {
            return orderStatusResponses['order-accepted'](orderStatus);
        } else if (Object.is(orderStatus, orderStatusEvents.orderConfirmed) {
            return orderStatusResponses['order-confirmed'](orderStatus);
        } else if (Object.is(orderStatus, orderStatusEvents.orderDelivered) {
            return orderStatusResponses[order-delivered'](orderStatus);
        }
        return "There has been an error processing this error!!";
    };
 ```
 
 ## Step 2 - Replace the body of each case with an invocation of that function
 
 Let's collapse the cascading if-else statements. Since each `orderStatusEvents` is listed with the corresponding event  in the `orderStatusResponses` object, we can use the orderstatus to directly look up the appropriate response function.
 
 If a matching response function is found we'll invoke it.
 
 ```
 
 let orderDispatcher = (orderStatus) => {
     let orderStatusResponse = orderStatusResponses[orderStatus];
     if (orderStatusResponse) {
         return orderStatusResponse(orderStatus);
     }
     return "There has been an error processing this error!!";
 }
 ```
 
 So we have collapsed a 5 case if-else statement into one! In the next step we'll eliminate the if-else statement altogether by extracting the fallback behaviour into a function of its own. if no response is matched, we'll use the double pipe operator to insert the fallback response.
 
 ## Step 3 - Replace the `if...else` statement with one function lookup and invocation.
 
 ```
 let fallback = () => "There has been an error processing this error!!";
 
  let orderDispatcher = (orderStatus) => {
     let orderStatusResponse = orderStatusResponses[orderStatus] || fallback;
    return orderStatusResponse(orderStatus);
 }
 ```

Another refactor that can happen will be to put a representation of a fallback e.g, an empty string " ", when the required event isn't defined, in `orderStatusResponses` object, this would eliminate the need for the pipe operator in the `orderDispatcher` function.

## Conclusion

In refactoring the sample codebase, we've learnt how to use the router pattern to prevent `if...else` ballooning, and have shown how to replace control flow with datastructures. To extend the codebase i.e, add and process a new type of event, we just put the new event and it's corresponding response in the `orderStatusEvents` object, there won't be any changes to the `orderDispatcher` function.

The router pattern helps us discover common needs across if-else cases and provide a flexible interface to DRY them up. 

 
 
 
 





  
