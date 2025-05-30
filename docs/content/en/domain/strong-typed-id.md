# Strongly Typed Entity ID

## Introduction

In domain-driven design modeling, the entity ID is very important. It is the unique identifier of an entity, the primary key of the entity, and an important attribute of the entity. Usually, we use types such as `int`, `long`, `Guid`, and `string` to define entity IDs. However, using these types in the system can lead to the following problems:

1. When defining an ID value, it is impossible to determine from the type whether it represents an entity ID or another type, which reduces code readability.
2. When assigning a value to an ID field referenced by an entity, it is easy to mistakenly assign a non-expected value of another type to the ID field, leading to errors.

To solve the above problems, we recommend using `strongly typed entity IDs`. Strongly typed entity IDs are encapsulations based on basic types. The currently supported basic types are:

- Int32
- Int64
- Guid
- String

## How to Use

Strongly typed IDs need to be modified with the keywords `public`, `partial`, and `record`, and need to implement one of the interfaces `IInt32StronglyTypedId`, `IInt64StronglyTypedId`, `IGuidStronglyTypedId`, or `IStringStronglyTypedId`.

1. Add a reference to the `NetCorePal.Extensions.Domain.Abstractions` package.

    ```bash
    dotnet add package NetCorePal.Extensions.Domain.Abstractions
    ```

2. You can write code similar to the following to implement a strongly typed entity ID:

    ```csharp
    using NetCorePal.Extensions.Domain;
    namespace YourNamespace;

    public partial record Int64OrderId : IInt64StronglyTypedId;
    ```

    The following code is automatically generated by `SourceGenerator`:

    ```csharp
    using NetCorePal.Extensions.Domain;
    using System;
    using System.ComponentModel;
    namespace YourNamespace;

    [TypeConverter(typeof(EntityIdTypeConverter<Int64OrderId, Int64>))]
    public partial record Int64OrderId(Int64 Id) : IInt64StronglyTypedId
    {
        public static implicit operator Int64(Int64OrderId id) => id.Id;
        public static implicit operator Int64OrderId(Int64 id) => new Int64OrderId(id);
        public override string ToString()
        {
            return Id.ToString();
        }
    }
    ```

    More examples:

    ```csharp
    using NetCorePal.Extensions.Domain;
    namespace YourNamespace;

    // Int32 strongly typed entity ID
    public partial record Int32OrderId : IInt32StronglyTypedId;

    // Guid strongly typed entity ID
    public partial record GuidOrderId : IGuidStronglyTypedId;

    // String strongly typed entity ID
    public partial record StringOrderId : IStringStronglyTypedId;
    ```

## Json Serialization Support

In scenarios such as inter-service calls and WebAPI, it usually involves serialization and deserialization between entity types and JSON strings. To ensure that strongly typed entity IDs work properly in these scenarios, we provide serialization support based on `System.Text.Json` and `Newtonsoft.Json`.

1. For `System.Text.Json`, you can use the following code to implement:

    ```csharp
    using NetCorePal.Extensions.Domain.Json;
    var builder = WebApplication.CreateBuilder(args);
    builder.Services.AddMvc().AddControllersAsServices().AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.Converters.Add(new EntityIdJsonConverterFactory());
    });
    ```

2. For `Newtonsoft.Json`, you can use the following code to implement:

    Add the `NetCorePal.Extensions.AspNetCore` package

    ```bash
    dotnet add package NetCorePal.Extensions.AspNetCore
    ```

    Add the following code to the `Startup.cs` file:

    ```csharp
    builder.Services.AddControllers().AddNewtonsoftJson(options =>
    {
        options.SerializerSettings.Converters.Add(new NewtonsoftEntityIdJsonConverter());
    });
    ```

    The following example shows the effect of serialization and deserialization:

    ```csharp
    JsonSerializerOptions options = new();
    options.Converters.Add(new EntityIdJsonConverterFactory());

    var id = JsonSerializer.Deserialize<OrderId1>("\"12\"", options);
    Assert.NotNull(id);
    Assert.True(id.Id == 12);
    var id2 = new OrderId2(2);
    var json = JsonSerializer.Serialize(id2, options);
    Assert.Equal("\"2\"", json);
    ```

Note: Strongly typed entity IDs will be serialized as strings.

## ID Generation

For ID generation, see the [ID Generation](../data/id-generator.md) document.
