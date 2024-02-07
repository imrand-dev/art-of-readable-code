## Chapter 10: Extracting Unrelated Subproblems

The advice for this chapter is to *aggressively identify and extract unrelated subproblems*. Here’s what we mean:

1. Look at a given function or block of code, and ask yourself, “What is the high-level goal of this code?”
2. For each line of code, ask, “Is it working *directly* to that goal? Or is it solving an *unrelated subproblem* needed to meet it?”
3. If enough lines are solving an unrelated subproblem, extract that code into a separate function.

### Introductory Example: `findClosestLocation()`

The high-level goal of the following JavaScript code is, to find the location that’s closest to a given point.

```jsx
// Return which element of 'array' is closest to the given latitude/longitude.
// Models the Earth as a perfect sphere.
var findClosestLocation = function (lat, lng, array) {
    var closest;
    var closest_dist = Number.MAX_VALUE;
    for (var i = 0; i < array.length; i += 1) {
        // Convert both points to radians.
        var lat_rad = radians(lat);
        var lng_rad = radians(lng);
        var lat2_rad = radians(array[i].latitude);
        var lng2_rad = radians(array[i].longitude);

		// Use the "Spherical Law of Cosines" formula.
        var dist = Math.acos(Math.sin(lat_rad) * Math.sin(lat2_rad) + 
                             Math.cos(lat_rad) * Math.cos(lat2_rad) *
                             Math.cos(lng2_rad - lng_rad));
        if (dist < closest_dist) {
            closest = array[i];
            closest_dist = dist;
        }
    }

    return closest;
};
```

Most of the code inside the loop is working on an unrelated subproblem: Compute the spherical distance between two lat/long points. Because there is so much of that code, it makes sense to extract it into a separate `spherical_distance()` function.

```jsx
var spherical_distance = function (lat1, lng1, lat2, lng2) {
    var lat1_rad = radians(lat1);
    var lng1_rad = radians(lng1);
    var lat2_rad = radians(lat2);
    var lng2_rad = radians(lng2);

    // Use the "Spherical Law of Cosines" formula.
    return Math.acos(Math.sin(lat1_rad) * Math.sin(lat2_rad) +
                     Math.cos(lat1_rad) * Math.cos(lat2_rad) *
                     Math.cos(lng2_rad - lng1_rad));
};
```

Now the remaining code becomes:

```jsx
var findClosestLocation = function (lat, lng, array) {
    var closest;
    var closest_dist = Number.MAX_VALUE;
    for (var i = 0; i < array.length; i += 1) {
		var dist = spherical_distance(lat, lng, array[i].latitude, array[i].longitude);
        if (dist < closest_dist) {
            closest = array[i];
            closest_dist = dist;
        }
    }

    return closest;
};
```

This code is far more readable because the reader can focus on the high-level goal without getting distracted by intense geometry equations.

### Other General-Purpose Code

For example, the following function call submits data to the server using Ajax and then displays the dictionary returned from the server.

```jsx
ajax_post({
    url: 'http://example.com/submit',
    data: data,
    on_success: function (response_data) {
        var str = "{\n";
        for (var key in response_data) {
            str += "  " + key + " = " + response_data[key] + "\n";
        }
        alert(str + "}");
              
        // Continue handling 'response_data' ...
    }
});
```

The high-level goal of this code is, Make an Ajax call to the server, and handle the response. But a lot of the code is solving the unrelated subproblem, Pretty-print a dictionary. It’s easy to extract that code into a function like `format_pretty(obj)`.

```jsx
var format_pretty = function (obj) {
    var str = "{\n";
    for (var key in obj) {
        str += "  " + key + " = " + obj[key] + "\n";
    }
    return str + "}";
};
```

There are a lot of reasons why extracting `format_pretty(obj)` is a good idea. It makes the calling code simpler, and `format_pretty(obj)` is a handy function to have around.

But there’s another great reason that’s not as obvious: it’s easier to improve `format_pretty(obj)` when the code is by itself.

Here are some cases `format_pretty(obj)` doesn’t handle:

- It expects `obj` to be an object. If instead, it’s a plain `string` or `undefined`, the current code will throw an exception.
- It expects each value of `obj` to be a simple type. If instead, it contains nested objects, the current code will display them as `[object Object]`, which isn’t very pretty.

But now, adding this functionality is easy. Here’s what the improved code looks like:

```jsx
var format_pretty = function (obj, indent) {
    // Handle null, undefined, strings, and non-objects.
    if (obj === null) return "null";
    if (obj === undefined) return "undefined";
    if (typeof obj === "string") return '"' + obj + '"';
    if (typeof obj !== "object") return String(obj);

    if (indent === undefined) indent = "";

    // Handle (non-null) objects.
    var str = "{\n";
    for (var key in obj) {
        str += indent + "  " + key + " = ";
        str += format_pretty(obj[key], indent + "  ") + "\n";
    }
    return str + indent + "}";
};
```

### Create a Lot of General-Purpose Code

* IS THIS TOP-DOWN OR BOTTOM-UP PROGRAMMING?

    * Top-down programming is a style where the highest-level modules and functions are designed first and the lower-level functions are implemented as needed to support them.
    * Bottom-up programming tries to anticipate and solve all the subproblems first and then build the higher-level components using these pieces.
    

### Project-Specific Functionality

This Python code creates a new Business object and sets its name, URL, and date_created.

```py
import re

business = Business()
business.name = request.POST["name"]

url_path_name = business.name.lower()
url_path_name = re.sub(r"['\.]", "", url_path_name)
url_path_name = re.sub(r"[^a-z0-9]+", "-", url_path_name)
url_path_name = url_path_name.strip("-")
business.url = "/biz/" + url_path_name

business.date_created = datetime.datetime.utcnow()
business.save_to_database()
```

The `url` is supposed to be a “clean” version of the `name`. For example, if the `name` is “A.C. Joe’s Tire & Smog, Inc.,” the `url` should be `"/biz/ac-joes-tire-smog-inc"`.

The unrelated subproblem in this code is: *Turn a name into a valid URL.* We can extract this code quite easily. While we’re at it, we can also precompile the regular expressions (and give them readable names):

```py
CHARS_TO_REMOVE = re.compile(r"['\.]+")
CHARS_TO_DASH = re.compile(r"[^a-z0-9]+")

def make_url_friendly(text):
    text = text.lower()
    text = CHARS_TO_REMOVE.sub('', text)
    text = CHARS_TO_DASH.sub('-', text)
    return text.strip("-")
```

Now the original code has a much more “regular” pattern:

```py
business = Business()
business.name = request.POST["name"]
business.url = "/biz/" + make_url_friendly(business.name)
business.date_created = datetime.datetime.utcnow()
business.save_to_database()
```

This code requires far less effort to read because you aren’t distracted by the regular expressions and deep string manipulation.

### Simplifying an Existing Interface

For example, let’s say you have a Python dictionary containing sensitive user information like `{ "username": "...", "password": "..." }` and you need to put all that information into a URL. Because it’s sensitive, you decide to encrypt the dictionary first, using a `Cipher` class.

But `Cipher` expects a string of bytes as input, not a dictionary. And `Cipher` returns a string of bytes, but we need something that’s URL-safe. `Cipher` also takes a number of extra parameters and is pretty cumbersome to use.

What started as a simple task turns into a lot of glue code:

```py
import json 

user_info = { "username": "...", "password": "..." }
user_str = json.dumps(user_info)

cipher = Cipher("aes_128_cbc", key=PRIVATE_KEY, init_vector=INIT_VECTOR, op=ENCODE)
encrypted_bytes = cipher.update(user_str)
encrypted_bytes += cipher.final()  # flush out the current 128 bit block

url = "http://example.com/?user_info=" + base64.urlsafe_b64encode(encrypted_bytes)
...
```

Even though the problem we’re tackling is Encrypt the user’s information into a URL, the majority of this code is just doing Encrypt this Python object into a URL-friendly string. It’s easy to extract that subproblem.

```py
import json 

def url_safe_encrypt(obj):
    obj_str = json.dumps(obj)
    cipher = Cipher("aes_128_cbc", key=PRIVATE_KEY, init_vector=INIT_VECTOR, op=ENCODE)
    encrypted_bytes = cipher.update(obj_str)
    encrypted_bytes += cipher.final()  # flush out the current 128 bit block
    
    return base64.urlsafe_b64encode(encrypted_bytes)
```

Then, the resulting code to execute the real logic of the program is simple:

```py
user_info = { "username": "...", "password": "..." }
url = "http://example.com/?user_info=" + url_safe_encrypt(user_info)
```

### Summary

A simple way to think about this chapter is to *separate the generic code from the project-specific code*. As it turns out, most code is generic. By building a large set of libraries and helper functions to solve the general problems, what’s left will be a small core of what makes your program unique.

The main reason this technique helps is that it lets the programmer focus on smaller, well-defined problems that are detached from the rest of your project. 

As a result, the solutions to those subproblems tend to be more thorough and correct. You might also be able to reuse them later.