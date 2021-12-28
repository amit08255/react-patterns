# State Reducer Pattern

The benefit of the state reducer pattern is in the fact that it allows "inversion of control" which is basically a mechanism for the author of the API to allow the user of the API to control how things work internally.

> It gives an advanced way for the user to change how your component operates internally. 

The code is similar to Custom Hook Pattern, but in addition the user defines a reducer which is passed to the hook. This reducer will overload any internal action of your component.
