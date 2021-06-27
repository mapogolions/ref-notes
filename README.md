#### Ref-Returns

```c#
public class Klass
{
    public int Id;
    public string Name;

    public override string ToString()
    {
        return $"Klass(Id: {Id}, Name: {Name})";
    }
}

public struct Struct
{
    public int Id;
    public string Name;

    public override string ToString()
    {
        return $"Struct(Id: {Id}, Name: {Name})";
    }
}
```

1)  Return ref to the field of the local class (Ok)
```c#
internal static ref string ReturnRefToFieldOfTheLocalClass()
{
    /*
                    Stack                                         Heap
end <~  |------------------------|           _________________________________________________
        |  hex (1 byte allocated)|    ~>    | 12 (4 bytes allocated) |  hex (1 byte allocated)|
call <~ |                        |           -------------------------------------------------
        |------------------------|
        |        .......         |
        |------------------------|
    */
    // Perfectly fine
    // return ref klass.Id
    // return ref klass.Name
    Klass klass = new() { Id = 12, Name = nameof(Klass) };
    return ref klass.Name;
}
```

2) Return ref to the field of the local struct variable (Invalid)
```c#
internal static ref string ReturnRefToFieldOfTheLocalStruct()
{
    /*
                  Stack                                    Heap
end <~  |------------------------|           ________________________________
        | hex (1 byte allocated) |    ~>    | char 1 | char 2 | ... | char N |
        |------------------------|           --------------------------------
        | 12 (4 bytes allocated) |
call <~ |                        |
        |------------------------|
        |        .......         |
        |------------------------|
    */
    Struct strct = new() { Id = 12, Name = nameof(Struct) }
    return ref strct.Name;
}
```

3) Return ref to the local class (Invalid)
```c#
internal static ref Klass ReturnRefToLocalClass()
{
    /*
                    Stack                                        Heap
end <~  |------------------------|           _________________________________________________
        | hex (1 byte allocated) |    ~>    | 12 (4 bytes allocated) |  hex (1 byte allocated)|
call <~ |                        |           --------------------------------------------------
        |------------------------|
        |        .......         |
        |------------------------|
    */
    Klass klass = new();
    return ref klass;
}
```

4) Return ref to the local string (Invalid)
```c#
internal static ref string ReturnRefToLocalString()
{
        /*
                    Stack                                 Heap
end <~  |------------------------|           _______________________________
        |  hex (1 byte allocated)|    ~>    |char 1 | char 2 | ... | char N |
call <~ |                        |           -------------------------------
        |------------------------|
        |        .......         |
        |------------------------|
    */
    string foo = "foo";
    return ref foo;
}
```

5) Return ref to the element of the local array (Ok)

5.1)

```c#
internal static ref string ReturnRefToStringElementOfLocalArray()
{
        /*
                    Stack                          Heap
end <~  |------------------------|           _____
        |  hex (1 byte allocated)|    ~>    | hex1| ~> [foo]
call <~ |                        |           -----
        |------------------------|          | ... | ~> [bar]
        |        .......         |           -----
        |------------------------|          | hexN| ~> [baz]
    */
    var arr = new[] { "foo", "bar", "baz" };
    return ref arr[0];
}
```

5.2)
```c#
internal static ref Struct ReturnRefToStructElementOfLocalArray()
{
        /*
                    Stack                                         Heap
end <~  |------------------------|           _______________________
        |  hex (1 byte allocated)|    ~>    | 13 (4 bytes allocated) |
call <~ |                        |           -----------------------       ____________________________
        |------------------------|          |  hex (1 byte allocated)| ~> | char1| char2 | ... | charN |
        |        .......         |           ------------------------      ----------------------------
        |------------------------|          | 14 (4 bytes allocated) |
                                            ------------------------      ----------------------------
                                            | hex (1 byte allocated) | ~> | char1| char2 | ... | charN |
                                            ------------------------      ----------------------------
    */
    var arr = new Struct[]
    {
        new() { Id = 13, Name = nameof(Struct) },
        new() { Id = 14, Name = string.Empty  }
    };
    return ref arr[0];
}
```
