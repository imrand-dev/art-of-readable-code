## Chapter 2: Packing Information into Names

Whether you’re naming a variable, a function, or a class, a lot of the same principles apply. We like to think of a name as a tiny comment. 

Even though there isn’t much room, you can convey a lot of information by choosing a good name.

### Choose Specific Words

For example, the word `get` is very unspecific, as in this example:

```python
def get_page(url):
  pass
```

The word `get` doesn’t really say much. Does this method get a page from a local cache, a database, or the Internet? If it’s from the Internet, a more specific name might be `fetch_page()` or `download_page()`.

Here’s an example of a BinaryTree class:

```cpp
class BinaryTree() {
  int size(){
    ...
  }
};
```

What would you expect the `size()` method to return? The height of the tree, the number of nodes, or the memory footprint of the tree?

The problem is that `size()` doesn’t convey much information. A more specific name would be `height()`, `numNodes()`, or `memoryBytes()`.

As another example, suppose you have some sort of Thread class:

```cpp
class Thread {
  void stop();
    ...
};
```

The name `stop()` is okay, but depending on what exactly it does, there might be a more specific name. For instance, you might call it `kill()` instead, if it’s a heavyweight operation that can’t be undone. Or you might call it `pause()`, if there is a way to `resume()` it.

### Finding More `Colorful` Words

Here are some examples of a word, as well as more `colorful` versions that might apply to your situation:

| Word      | Alternatives |
| ----------- | ----------- |
| send      | deliver, dispatch, announce, distribute, route       |
| find   | search, extract, locate, recover        |
| start      | launch, create, begin, open       |
| make   | create, set up, build, generate, compose, add, new        |

> It’s better to be clear and precise than to be cute.

### Avoid Generic Names Like tmp and retval

Names like `tmp`, `retval`, and `foo` are usually cop-outs that mean “I can’t think of a name.” Instead of using an empty name  like this, *pick a name that describes the entity’s value or purpose*.

For example, here’s a JavaScript function that uses retval:

```jsx
let euclideanNorm = function (v) {
  let retval = 0.0;
    
  for (let i = 0; i < v.length; i += 1)
    retval += (v[i] * v[i]);

  return Math.sqrt(retval);
}
```

It’s tempting to use `retval` when you can’t think of a better name for your return value. But `retval` doesn’t contain much information other than “I am a return value”.

A better name would describe the purpose of the variable or the value it contains. In this case, the variable is accumulating the sum of the squares of `v`. So a better name is `sumSquares`. This would announce the purpose of the variable upfront and might help catch a bug.

> The name `retval` doesn’t pack much information. Instead, use a name that describes the variable’s value.

There are, however, some cases where generic names do carry meaning. Let’s take a look at when it makes sense to use them.

Consider the classic case of swapping two variables:

```jsx
if (right < left) {
  let tmp = right;
  right = left;
  left = tmp;
}
```

In cases like these, the name `tmp` is perfectly fine. The variable’s sole purpose is temporary storage, with a lifetime of only a few lines. The name `tmp` conveys specific meaning to the reader—*that this variable has no other duties*. It’s not being passed around to other functions or being reset or reused multiple times.

But here’s a case where `tmp` is just used out of laziness:

```cpp
String tmp = user.name();
tmp += " " + user.phone_number();
tmp += " " + user.email();
...
template.set("user_info", tmp);
```

Even though this variable has a short lifespan, being temporary storage isn’t the most important thing about this variable. 

Instead, a name like `user_info` would be more descriptive.

> The name `tmp` should be used only in cases when being short-lived and temporary is the most important fact about that variable.

### Loop Iterators

Names like `i`, `j`, `iter`, and `it` are commonly used as indices and loop iterators. Even though these names are generic, they’re understood to mean “I am an iterator”.

But sometimes there are better iterator names than `i`, `j`, and `k`. For instance, the following loops find which users belong to which clubs:

```cpp
for (int i = 0; i < clubs.size(); i++)
  for (int j = 0; j < clubs[i].members.size(); j++)
    for (int k = 0; k < users.size(); k++)
      if (clubs[i].members[k] == users[j]) {
         cout << "user[" << j << "] is in club[" << i << "]" << endl;
      }
```

In the `if` statement, `members[]` and `users[]` are using the wrong index. Bugs like these are hard to spot.

In this case, using more precise names may have helped. Instead of naming the loop indexes `(i,j,k)`, another choice would be (`club_i`, `members_i`, `users_i`) or, more succinctly (`ci`, `mi`, `ui`). This approach would help the bug stand out more:

```cpp 
// Bug! First letters don't match up.
if (clubs[ci].members[ui] == users[mi])
```

When used correctly, the first letter of the index would match the first letter of the array:

```cpp 
// OK. First letters match.
if (clubs[ci].members[mi] == users[ui])
```

> If you’re going to use a generic name like `tmp`, `it`, or `retval`, have a good reason for doing so.

### Prefer Concrete Names over Abstract Names

When naming a `variable`, `function`, or other element, describe it concretely rather than abstractly.

For example, suppose you have an internal method named `serverCanStart()`, which tests whether the server can  listen on a given TCP/IP port. The name `serverCanStart()` is somewhat abstract, though. 
A more concrete name would be `canListenOnPort()`. This name directly describes what the method will do.

### Attaching Extra Information to a Name

Any extra information you squeeze into a name will be seen every time the variable is seen.

So if there’s something very important about a variable that the reader must know, it’s worth attaching an extra “word” to the name. For example,
suppose you had a variable that contained a hexadecimal string:

```cpp 
// Example: "af84ef845cd8"
string id;
```

You might want to name it `hex_id` instead.

### Values with Units

If your variable is a measurement (such as an amount of time or a number of bytes), it’s helpful to encode the units into the variable’s name.

For example, here is some JavaScript code that measures the load time of a web page:

```jsx
// top of the page
let start = new Date().getTime();
...
// bottom of the page
let elapsed = new Date().getTime() - start;
document.writeln("Load time was: " + elapsed + " seconds");
```

There is nothing obviously wrong with this code, but it doesn’t work, because `getTime()` returns milliseconds, not seconds.

By appending `_ms` to our variables, we can make everything more explicit:

```jsx 
// top of the page
let start_ms = new Date().getTime();
...
// bottom of the page
let elapsed_ms = new Date().getTime() - start_ms;
document.writeln("Load time was: " + elapsed_ms / 1000 + " seconds");
```

Besides time, there are plenty of other units that come up in programming. Here is a table of unitless function parameters, and better versions that include the units:

| Function parameter      | Renaming parameter to encode units |
| ----------- | ----------- |
| `start(int delay)`      | delay → delay_secs       |
| `createCache(int size)`   | size → size_mb        |
| `throttleDownload(float limit)`      | limit → max_kbps       |
| `rotate(float angle)`   | angle → degrees_cw        |

### Encoding Other Important Attributes

This technique of attaching extra information to a name isn’t limited to values with units. You should do it any time there’s something dangerous or surprising about the variable.

For example, many security exploits come from not realizing that some data your program receives is not yet in a safe state. 

For this, you might want to use variable names like `unTrustedUrl` or `unSafeMessageBody`. After calling functions that cleanse the unsafe input, the resulting variables might be `trustedUrl` or `safeMessageBody`.

### How Long Should a Name Be?

When picking a good name, there’s an implicit constraint that the name shouldn’t be too long. No one likes to work with identifiers like this:

```jsx
let newNavigationControllerWrappingViewControllerForDataSourceOfClass = 45;
```

The longer a name is, the harder it is to remember, and the more space it consumes on the screen, possibly causing extra lines to wrap.

On the other hand, programmers can take this advice too far, using only single-word names. So how should you manage this trade-off? How do you decide between naming a variable `d`, `days`, or `days_since_last_update`?

This decision is a judgment call whose best answer depends on exactly how that variable is being used. But here are some guidelines to help you decide.

### Shorter Names Are Okay for Shorter Scope

When you go on a short vacation, you typically pack less luggage than if you go on a long vacation. Similarly, identifiers that have a small “scope”  don’t need to carry as much information. That is, you can get away with shorter names because all that information is easy to see:

```cpp
if (debug) {
  map<string, int> m;
  LookUpNamesNumbers(&m);
  cout << m;
}
```

Even though `m` doesn’t pack any information, it’s not a problem, because the reader already has all the information she needs to understand this code.

If an identifier has a large scope, the name needs to carry enough information to make it clear.

### Throwing Out Unneeded Words

Sometimes words inside a name can be removed without losing any information at all. For instance, instead of `convertToString()`, the name `toString()` is smaller and doesn’t lose any real information. Similarly, instead of `doServeLoop()`, the name `serveLoop()` is just as clear.

### Use Name Formatting to Convey Meaning

```cpp
static const int kMaxOpenFiles = 100;

class LogReader {
  public:
    void openFile(string local_file);

  private:
    int offset_;
    DISALLOW_COPY_AND_ASSIGN(LogReader);
};
```

Most of the formatting in this example is pretty common—using `CamelCase` for class names, and using `lower_separated` for variable names. But some of the other conventions may have surprised you.

For instance, constant values are of the form `kConstantName` instead of `CONSTANT_NAME`. This style has the benefit of being easily distinguished from `#define` macros, which are `MACRO_NAME` by convention.

Class member variables are like normal variables, but must end with an underscore, like `offset_`. At first, this convention may seem strange, but being able to instantly distinguish members from other variables is very handy. For instance, if you’re glancing through the code of a large method, and see the line:

```
stats.clear();
```

you might wonder, *Does* `stats` *belong to this class? Is this code changing the internal state of the class?* If the `member_` convention is used, you can quickly conclude, *No,* `stats` must be a local variable. Otherwise it would be named `stats_`.

### Summary

The single theme for this chapter is: "pack information into your names". By this, we mean that the reader can extract a lot of information just from reading the name.

Here are some specific tips we covered:

- Use specific words—for example, instead of `get`, words like `fetch` or `download` might be better, depending on the context.
- Avoid generic names—like `tmp` and `retval`, unless there’s a specific reason to use them.
- Use concrete names—that describe things in more detail—the name `ServerCanStart()` is vague compared to `CanListenOnPort()`.
- Attach important details—to variable names for example, append `_ms` to a variable whose value is in milliseconds or prepend `raw_` to an unprocessed variable that needs escaping.
- Use longer names for larger scopes—don’t use cryptic one or two-letter names for variables that span multiple screens; shorter names are better for variables that span only a few lines.
- Use capitalization, underscores, and so on in a meaningful way—for example, you can append `_` to class members to distinguish them from local variables.
