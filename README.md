# JSON library for Java

###### By RcCookie

This is a library to work with JSON files in Java.

## Overview

The parsing is handled by the `Json` class. It can handle json strings with common JSON formatting and additionally comments, trailing commas, and a JSON file containing only `null`;

JSON objects and arrays are represented by the classes `JsonObject` and `JsonArray`, respectively. Each of these classes has utilities to easily access elements from JSON files and read and write JSON from and to files.

Additionally, the class `JsonElement` functions as a wrapper class for values in JSON similar to `java.util.Optional` to simplify the process of performing many steps to check weather a value is present by checking each outer object / array for existence.

## How to use

Reading a JSON file into Java object is very simple - there are several ways to perform it:

````java
    // Option 1
    JsonObject o = new JsonObject(new File("xy.json"));
    JsonArray a = new JsonArray(new File("yz.json"));
    
    // Option 2
    o = Json.load(new File("xy.json")).asObject();
    a = Json.load(new File("yz.json")).asArray();
````

Option 1 will simply return `false` if an IOException occurres. The second option will throw an UncheckedIOException instead. Apart from that there is no difference from the two.

The second option can also come in handy when you don't know whether the JSON file describes an object or an array.

It is also possible to parse some JSON string that is already present as string, for example because you want to load the file yourself:

````java
    o = new JsonObject(jsonString);
    o = Json.parse(jsonString).asObject();
````

You can also create new JsonObjects and JsonArrays from scratch, using their default constructor.

Once the Java objects are created, they can be used like a map or list. In fact, JsonObject and JsonArray implement `Map` and `List`, respectively, so technically you could pass them into any method that expects that kind of data. But since you probably want to read specific fields, both classes contain methods that cast the result into the expected. Example:

````json
    {
        "key": "value",
        "array": [
            null,
            false,
            {
                "anotherKey": "anotherValue"
            },
            42
        ]
    }
````

To access the `anotherKey` field, we can simply write:

````java
    Stirng value = object.getArray("array").getObject(2).getString("anotherKey");

    // Instead of
    String value = (String)((JsonObject)((JsonArray)(object.get("array")).get(2)).get("anotherKey"));
````

---

You may need to check whether a JSON file actually defines a certain element. This is easy enough if we want to check in the above example, whether the `key`-mapping is defined. To do so, you could write:

````java
    String value;
    if(object.containsKey("key")) value = object.getString("key");
    else value = "defaultValue";
````

You could even inline this operation with the help of `?`, so this is a reasonable option. If however you want to check the presence of the `anotherKey`-mapping, you will first have to check the presence of the object it is contained in, before that whether the array has so many indices and at the very first weather there is actually a mapping "array" in the main object. This would look like this:

````java
    String value;
    if(object.containsKey("array") {
        JsonArray array = object.getArray("array");
        if(array.size() < 2) {
            JsonObject innerObject = array.getObject(2);
            if(innerObject.containsKey("anotherKey"))
                value = innerObject.getString("anotherKey");
            else value = "defaultValue";
        }
        else value = "defaultValue";
    }
    else value = "defaultValue";
````

This process can be simplified by the use of `JsonElement`. The following is equivalent to the above:

````java
    String value = object.getElement("array").get(2).get("anotherKey").or("defaultValue").asString();
````

If the default value should be `null`, `orNull()` can be used instead, and if the default value should only be generated on demand, `orGet()` supports the use of `Supplier`.

This can work because `JsonElement` can wrap any json value, both primitive and object-based. You can treat it as JsonObject, JsonArray, String and so on. If the value is not available at some point, it will ignore the following instructions and on demand return the specified default value instead.

---

Finally, storing a json structure into a string or file is just as simple as loading it:

````java
    String jsonString = object.toString();
    String unformattedJsonString = object.toString(false);

    // File can exist, but not necessary
    object.store(new File("xy.json"));
    Json.store(object, new File("xy.json"));
````

The same also applies for JsonArrays.
