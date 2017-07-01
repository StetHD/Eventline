# Threads

As previously mentioned the bot (name TBC) hasn't been given any routes to
follow.

We do this using the concept of threads. Threads are simply a pattern to match
on the event payload followed by a series of actions to perform.

Here is an example of this.

```
import Bot from 'microbot'

var bot = new Bot()
```

The next step is to tell the bot (name TBC) to start listening to events. Right now
this won't do anything as we haven't declared any routing logic.

```
bot.on({
    type: 'message'
})
.then(DoSomething)

bot.listen()
```

When we get an event with a key of `type` set to the value of `message` then this
thread will trigger the `DoSomething` action.

We will cover actions in the next tutorial.

When declaring threads these should always occur before the `listen` method as
this is what builds them ready to be executed at run time.

The priority of each thread is determined in the order in which you declare them, ones
defined earlier in your code will be checked first to see if the event matches the
specified pattern before moving onto the next one.

## Patterns

The pattern for a thread determines what should match in the payload for an event before
that thread can be ran. Below we outline the supported forms and techniques for creating
patterns for your threads.

There are two kinds of matchers for patterns:

- `compositional` who filter events based on the structure of the event payload.
- `value` who filter based on the value of a keypath of the event payload.

### Object

You will use this the most often, this will only allow a thread to be
triggered if each key path specifies matches the value of the same key
path in the payload of the event.

```
{
    type: 'message'
}
```

For example the above matches events with a payload that have a key called `type`
with the value `message`.

Microbot (name TBC), also supports deep-level keys when it comes to objects:

```
{
    sender: {
        id: 1
    }
}
```

The above will match events with a payload that have a key called `sender`
which in turn has a key called `id` with the value `1`.

Matching deep-level keys usually requires alot of typing and brackets,
so we support dot-notation key paths.

The pattern above for example could be re-phrased as:

```
{
    "sender.id": 1
}
```

### Array

When using array in your pattern, a thread will only be triggered if all
conditions within that array match.

```
[
    {
        type: 'message'
    },
    {
        "sender.id": 1
    }
]
```

This pattern above only matches events with a payload that have a key called `type`
with the value `message` and that have a key called `sender` which in turn has a key called `id` with the value `1`.

We could have esily done this purley with an object based pattern but using arrays
allows us to easily compose patterns for a thread from multiple indvidual ones.

i.e "When we recieve a message containing a location from sender with an id of 1"

### Function

Sometimes matching raw patterns isn't enougth and we need more complecated expressions.
Functions allow us to express more complicated rules for an event payload which must match.

```
function TextIsTooShort(event) {
    return event.text && event.text.length < 5
}

bot.on(TextIsTooShort)
```

When TextIsTooShort returns true the thread is triggered. In this example the function
is passed the event to match, it checks if the event has a key called `text` which has
a string shorter than 5 characters.

Functions can also be passed into object patterns to validate the value of a key.

```
function TextIsTooShort(text) {
    return text && text.length < 5
}

bot.on({
    "text": TextIsTooShort
})
```

Now this function is passed in as a `value` based matcher, it will be given the
value to match and if it returns true then all of the keys for the object 
will match and the thread will be triggered.

Functions can also be passed into arrays and as mentioned before as long as it
and the other patterns in that array match then the thread will be triggered.

### Value

And by now you can see we support matching by value, as long as the value
of a key is equal then it will match. We support or javascript types
that can be compared using the equality operator.

```
bot.on({
    "text": "I should be equal"
})
```

If the key `text` above matches the value passed in, then it should match :)

### Regular Expressions

For more complex value based matching we support regular expressions in addition
to functions.

```
bot.on({
    'text': /\d+/
})
```

In the pattern above we match an event with a payload with the key `text` with a value
which represents a digit of 0 or more characters.

The built in regular expressions can be very limited and their syntax lags behind, so
we have built in support for the `Xregexr` library which allows you to provide named
capture groups in patterns to reference later.

```
bot.on({
    'text': /(?<age>\d+)/
})
```

This pattern works same as before but the value will be captured into a capture group
named `age`. Of course you could do this with standard regular expressions but you
could only reference them by position within the expression which can be fragile.

To extract the value of these capture groups we provide a utility function named
` getCaptureGroups`.

Inside of your action (convered in the next tutorial) you will pass in the event and the keypath for the regular
expression and it will return the cpature groups.

```
import getCaptureGroups from 'microbot'

function MyAction(event) {

    var textCaptureGroups = getCaptureGroups(event, 'text')

    // Regexr
    console.log("The user said they were " + textCaptureGroups[0] + " old!")

    // Xregexr
    console.log("The user said they were " + textCaptureGroups.age + " old!")

    return event
}
```
