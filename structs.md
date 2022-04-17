# OpenStruct, Introduced
## A quick recap on a Ruby Class and an object

What is a Ruby class?

Ruby allows us to define classes. These Classes then provides a blueprint for the construction of similar objects. Each Class defines methods which are definitions of behaviour, and attributes which are definitions of variables. The methods will get invoked in response to a message.

A class can be used to repeatedly instantiate, or create, new instances of an object.

All this talk about objects, but what is a Ruby object?

An object is a data specific instance of a class.
Every newly instantiated Class implements the same methods and uses the same attribute names, but each contains its own personal data.

They share the same methods, so they all behave like the Class. However they contain different data, hence they represent different ones.

Every class object like String, Integer is a data-specific instance of the Class class.




## An Struct


Struct = structure???

Sometimes you may not want to create an entire class, but instaed you just want to use a "template Class" that will allow you to create an object. This is where OpenStructs are useful.

An OpenStruct can be defined as a built in Ruby class that allows you define attributes with accompanying values, thus producing objects which store these related attributes together.

An Openstruct employs a Hash internallu to store the attributes and values.


 A hash can be used to initialize the Open struct


Whne would you use  ahash instaed



### How to craete a Struct
The most common method is to create a structure that is assigned to a constant.

```ruby
Dog = Struct.new(:name, :age, :gender, :weight)
```


Once you've created the struct, you can then instantiate the object:

```ruby
ty = Dog.new("Ty", 2, "male", 10 )
```

The above example is used with positional parameters, and usually works great with a limited number of arguments. However as the number of arguments incraese, it becoems more readable to use keyword arguments. In addition, If you do not provide the ciorrrect number of arguments fo rthe s contructore, it will simply set the parameter to nil.

In ruby 2.5 , keywork arguments were added 🎉.
When we want to indicate to a STruct that we will be passing keyword_arguments, we simply add `keyword_init` as an identifier.

```ruby
Dog = Struct.new(:name, :age, :gender, :weight, keyword_init: true)
```


Now, you will instantiate your object with keywork arguments:

```ruby
ty = Dog.new(name: "Ty", age: 2, gender: "male", weight: 10 )
```

Id you do not set a value for a keyword argument, it will simply set it to `nil`.



Some other advantages include:

1. The Struct basically wires up the accessor methods for reading and writing internally so that you don't need to take care of it.

2. Since the Struct producedc value objects, you are able to compare structs directly based on their attributes. This is unlike the produce of identity objects for a normal class.

    An example is:

    ```
    ty1 = Dog.new("Ty", 2, "male", 10 )
    ty2 = Dog.new("Ty", 2, "male", 10 )
    ty3 = Dog.new("Ty", 2, "male", 12 )
    ```

    which yields the following results when compared:
    ```
    ty1 == ty2
    => true

    ty2 == ty3
    => false
    ```

3. You can access Struct members using methods. YOu can think of a Struct member as an attribute.

    An example:

    ```
    ty = Dog.new("Ty", 2, "male", 10 )
    ```

    ```
    ty.gender
    => "male"
    ```

    This becomes more useful when you want to act upon the values of your members.

    Example:
    ```
      ty1 = Dog.new("Ty", 2, "male", 10 )
      ty2 = Dog.new("Ty", 2, "male", 20 )
    ```

    ```
    [ty1, ty2].min_by(&:weight)
    => #<struct Dog name="Ty", age=2, gender="male", weight=10>
    ```

    You can use methods like, max, min, sum, select etc.





Refernce my PR.

Look into the source code and walk through it.


## What si an OpenStruct

If you need just a once off object then you should create an OpenStruct. ]]

An Openstruct can be much slower than a Hash , as there is more overhead in the setting of these properties when it craetes objects. YOu can look at this study here that measures the speed. [Insert link here]

It employs a Hash internally to store the attributes and values and it can even be initialized with a Hash

