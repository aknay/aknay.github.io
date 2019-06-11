---
layout: post
title:  "Basic Reactive Programming with Python"
date:   2019-06-11 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will learn how to use basic reactive programming with Python language.  
<!-- excerpt -->

#### **Prerequisites**
1. Python Programming

#### **Overview**
In this tutorial, we will discuss basic Reactive Programming Concept and why we need to code in a reactive way. We will then compare the imperative way and reactive way of coding with code snippets.

#### **Dependencies**
We need to install `RxPy`. You can use `pip install rx` to install. We will be using Python 3. You can look at the official website from [here](https://github.com/ReactiveX/RxPY).


#### **Why Reactive Programming**
Before we go into reactive programming, let's look at the imperative way of coding first. 

{% highlight java linenos %}

def printResponse(response: str):
    print(f'The response: {response}')

class ImperativePerson:
    def __init__(self, name: str):
        self._name: str = name

    def sayHiWithName(self, otherPersonName: str) -> str:
        return f'Hi {otherPersonName}, nice to meet you. My name is {self._name}.'


imperativePerson = ImperativePerson('John')

imperativeResponse1 = imperativePerson.sayHiWithName('David')
printResponse(imperativeResponse1)

imperativeResponse2 = imperativePerson.sayHiWithName('Jane')
printResponse(imperativeResponse2)
{% endhighlight %}

The Result:
```
The response: Hi David, nice to meet you. My name is John.
The response: Hi Jane, nice to meet you. My name is John.
```

The above code is not doing anything much. Just printing out the response from the `ImperativePerson` class when we say 'Hi' to that person. As you can see, we need to call and get the response back every time we say 'Hi'. Imagine we are using it in UI or WebApp. It can be very painful and confusing as we need to remember to get back the data after we call it. 

We don't want to call the function every time. We just want to wait for it to call it back. It will be then responsive, isn't? Next, we will change the above code using `RxPy` to make it reactive.

{% highlight java linenos %}
from typing import Callable

from rx.subjects import Subject


def printResponse(response: str):
    print(f'The response: {response}')


class ReactivePerson:
    def __init__(self, name: str):
        self._name: str = name
        self._responseSubject = Subject()

    def sayHiWithName(self, otherPersonName: str):
        self._responseSubject.on_next(f'Hi {otherPersonName}, nice to meet you. My name is {self._name}.')

    def subscribe(self, func: Callable):
        self._responseSubject.subscribe(func)

    def __del__(self):
        self._responseSubject.dispose()


reactivePerson = ReactivePerson('Clark')

reactivePerson.subscribe(printResponse)

reactivePerson.sayHiWithName('Emma')

reactivePerson.sayHiWithName('James')
{% endhighlight %}

The Result:
```
The response: Hi Emma, nice to meet you. My name is Clark.
The response: Hi James, nice to meet you. My name is Clark.
```

Can you see the difference? In the reactive programming code above, we introduce `Subject` type by importing from `rx.subjects`. This Subject type can broadcast data to all subscribers. That is the reason we need to pass `printResponse` function to `subscribe`. By doing that, the function will be called whenever we `sayHiWithMyName`. Inside that `sayHiWithMyName` function, we need to add an element into `on_next` function. Then the added element will be notified to all subscribers.

Now our little `ReactivePerson` class becomes responsive. We can observe any response as soon as we say Hi. No more getting the data back and print it out. We can even subscribe to multiple functions. How awesome. 
