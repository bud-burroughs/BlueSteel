![HBC Digital logo](https://raw.githubusercontent.com/gilt/Cleanroom/master/Assets/hbc-digital-logo.png)     
![Gilt Tech logo](https://raw.githubusercontent.com/gilt/Cleanroom/master/Assets/gilt-tech-logo.png)

# BlueSteel

An Avro encoding/decoding library for Swift.

BlueSteel is part of [the Cleanroom Project](https://github.com/gilt/Cleanroom) from [Gilt Tech](http://tech.gilt.com).


### Swift compatibility

This is the `master` branch. It uses **Swift 4.1** and **requires Xcode 9.3** to compile.


#### Current status

Branch|Build status
--------|------------------------
[`master`](https://github.com/gilt/BlueSteel)|[![Build status: master branch](https://travis-ci.org/gilt/BlueSteel.svg?branch=master)](https://travis-ci.org/gilt/BlueSteel)


### Never heard of Avro?

Take a gander at the [official documentation for Avro](http://avro.apache.org/docs/current/) before reading further.


### License

BlueSteel is distributed under [the MIT license](https://github.com/gilt/BlueSteel/blob/master/LICENSE).

BlueSteel is provided for your use—free-of-charge—on an as-is basis. We make no guarantees, promises or apologies. *Caveat developer.*


### Adding BlueSteel to your project

#### Carthage

[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)

The simplest way to integrate BlueSteel is with the [Carthage](https://github.com/Carthage/Carthage) dependency manager.

First, add this line to your [`Cartfile`](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md#cartfile):

```
github "gilt/BlueSteel" ~> 2.0.0
```

Then, use the `carthage` command to [update your dependencies](https://github.com/Carthage/Carthage#upgrading-frameworks).

Finally, you’ll need to [integrate BlueSteel into your project](https://github.com/gilt/BlueSteel/blob/master/INTEGRATION.md) in order to use [the API](https://rawgit.com/gilt/BlueSteel/master/Documentation/API/index.html) it provides.

#### Swift Package Manager

[![Swift Package Manager](https://rawgit.com/jlyonsmith/artwork/master/SwiftPackageManager/swiftpackagemanager-compatible.svg)](https://swift.org/package-manager/)

Simply add BlueSteel to your `Package.swift`:

```
dependencies: [
    .package(url: "https://github.com/Myotest/BlueSteel.git", from: "3.1.0")
]
```

Version 3.1.0 is the first version supported in Swift Package Manager.

#### Use

Once successfully integrated, just add the following statement to any Swift file where you want to use BlueSteel:

```swift
import BlueSteel
```

See [the Integration document](https://github.com/gilt/BlueSteel/blob/master/INTEGRATION.md) for additional details on integrating BlueSteel into your project.

### API documentation

For detailed information on using BlueSteel, [API documentation](https://rawgit.com/gilt/BlueSteel/master/Documentation/API/index.html) is available.


## Usage

Since Avro data is not self describing, we're going to need to supply an Avro Schema before we can (de)serialize any data. Schema enums are constructed from a JSON schema description, in either String or NSData form.

```swift
import BlueSteel

let jsonSchema = "{ \"type\" : \"string\" }"
let schema = Schema(jsonSchema)
```

### Deserializing Avro data

Using the Schema above, we can now decode some Avro binary data.

```swift
let rawBytes: [Byte] = [0x6, 0x66, 0x6f, 0x6f]
let avro = AvroValue(schema: schema, withBytes: rawBytes)
```

We can now get the Swift String from the Avro value above using an optional getter.
```swift
if let avroString = avro.string {
    print(avroString) // Prints "foo"
}
```

### Serializing Swift data

We can use the same Schema above to serialize an AvroValue to binary.

```swift
if let serialized = avro.encode(schema) {
    print(serialized) // Prints [6, 102, 111, 111]
}
```

#### But how do we convert our own Swift types to AvroValue?

By conforming to the AvroValueConvertible protocol! You just need to extend your types with one function:
```swift
func toAvro() -> AvroValue
```

Suppose we wanted to serialize a NSUUID with the following schema:

```JSON
{
    "type" : "fixed",
    "name" : "UUID",
    "size" : 16
}
```

We could extend NSUUID as follows:

```swift
extension NSUUID : AvroValueConvertible {
    public func toAvro() -> AvroValue {
        var uuidBytes: [Byte] = [Byte](count: 16, repeatedValue: 0)
        self.getUUIDBytes(&uuidBytes)
        return AvroValue.AvroFixedValue(uuidBytes)
    }
}
```

To generate and serialize a NSUUID, we could now do:

```swift
let serialized: [Byte]? = NSUUID().toAvro().encode(uuidSchema)
```
Hey presto! We now have a byte array representing an NSUUID serialized to Avro according to the fixed schema provided.
Okay, so the example above is maybe a little bit too simple. Let's take a look at a more complex example. Suppose we have a record schema as follows:

```JSON
{
    "type": "record", 
        "name": "test",
        "fields" : [
        {"name": "a", "type": "long"},
        {"name": "b", "type": "string"}
    ]
}
```

We could create a corresponding type Swift that might look something like this:
```swift
struct testStruct {
    var a: Int64 = 0
    var b: String = ""
}
```

To convert testStruct to AvroValue, we could extend it like this:

```swift
extension testStruct : AvroValueConvertible {
    func toAvro() -> AvroValue {
        return AvroValue.AvroRecordValue([
                "a" : self.a.toAvro(),
                "b" : self.b.toAvro()])
    }
}
```

You might've noticed above that we called .toAvro() on Int64 and String values. We didn't have to define these ourselves because BlueSteel provides AvroValueConvertible extensions for Swift primitives.

So that just about covers a very quick introduction to BlueSteel. Please note that BlueSteel is still very early in development and may change significantly.


## About

The Cleanroom Project began as an experiment to re-imagine Gilt’s iOS codebase in a legacy-free, Swift-based incarnation.

Since then, we’ve expanded the Cleanroom Project to include multi-platform support. Much of our codebase now supports tvOS in addition to iOS, and our lower-level code is usable on macOS and watchOS as well.

Cleanroom Project code serves as the foundation of Gilt on TV, our tvOS app [featured by Apple during the launch of the new Apple TV](http://www.apple.com/apple-events/september-2015/). And as time goes on, we'll be replacing more and more of our existing Objective-C codebase with Cleanroom implementations.

In the meantime, we’ll be tracking the latest releases of Swift & Xcode, and [open-sourcing major portions of our codebase](https://github.com/gilt/Cleanroom#open-source-by-default) along the way.


### Contributing

BlueSteel is in active development, and we welcome your contributions.

If you’d like to contribute to this or any other Cleanroom Project repo, please read [the contribution guidelines](https://github.com/gilt/Cleanroom#contributing-to-the-cleanroom-project). If you have any questions, please reach out to project owner [Paul Lee](http://github.com/pauljlee).


### Acknowledgements

API documentation is generated using [Realm](http://realm.io)’s [jazzy](https://github.com/realm/jazzy/) project, maintained by [JP Simard](https://github.com/jpsim) and [Samuel E. Giddins](https://github.com/segiddins).
