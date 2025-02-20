# StringBuilder

[![.NET](https://github.com/linkdotnet/StringBuilder/actions/workflows/dotnet.yml/badge.svg)](https://github.com/linkdotnet/StringBuilder/actions/workflows/dotnet.yml)
[![Nuget](https://img.shields.io/nuget/dt/LinkDotNet.StringBuilder?style=flat-square)](https://www.nuget.org/packages/LinkDotNet.StringBuilder/)
[![GitHub tag](https://img.shields.io/github/v/tag/linkdotnet/StringBuilder?include_prereleases&logo=github&style=flat-square)](https://github.com/linkdotnet/StringBuilder/releases)

A fast and low allocation StringBuilder for .NET.

## Getting Started
Install the package:
> PM> Install-Package LinkDotNet.StringBuilder

Afterward, use the package as follow:
```csharp
using LinkDotNet.StringBuilder; // Namespace of the package

using ValueStringBuilder stringBuilder = new();
stringBuilder.AppendLine("Hello World");

string result = stringBuilder.ToString();
```

There are also smaller helper functions, which enable you to use `ValueStringBuilder` without any instance:
```csharp
string result1 = ValueStringBuilder.Concat("Hello ", "World"); // "Hello World"
string result2 = ValueStringBuilder.Concat("Hello", 1, 2, 3, "!"); // "Hello123!"
```

By default, `ValueStringBuilder` uses a rented buffer from `ArrayPool<char>.Shared`.
You can avoid renting overhead with an initially stack-allocated buffer:
```csharp
using ValueStringBuilder stringBuilder = new(stackalloc char[128]);
```
Note that this will prevent you from returning `stringBuilder` or assigning it to an `out` parameter.

## What does it solve?
The dotnet version of the `StringBuilder` is an all-purpose version that normally fits a wide variety of needs.
But sometimes, low allocation is key. Therefore I created the `ValueStringBuilder`. It is not a class but a `ref struct` that tries to allocate as little as possible.
If you want to know how the `ValueStringBuilder` works and why it uses allocations and is even faster, check out [this](https://steven-giesel.com/blogPost/4cada9a7-c462-4133-ad7f-e8b671987896) blog post.
The blog goes into a bit more in detail about how it works with a simplistic version of the `ValueStringBuilder`.

## What it doesn't solve!
The library is not meant as a general replacement for the `StringBuilder` built into .NET. You can head over to the documentation and read about the ["Known limitations"](https://linkdotnet.github.io/StringBuilder/articles/known_limitations.html).
The library works best for a small to medium length strings (not hundreds of thousands of characters, even though it can be still faster and performs fewer allocations). At any time, you can convert the `ValueStringBuilder` to a "normal" `StringBuilder` and vice versa.

The normal use case is to concatenate strings in a hot path where the goal is to put as minimal pressure on the GC as possible.

## Documentation
More detailed documentation can be found [here](https://linkdotnet.github.io/StringBuilder). It is really important to understand how the `ValueStringBuilder` works so that you did not run into weird situations where performance/allocations can even rise.

## Benchmark

The following table compares the built-in `StringBuilder` and this library's `ValueStringBuilder`:

```no-class
BenchmarkDotNet=v0.13.2, OS=macOS Monterey 12.6.1 (21G217) [Darwin 21.6.0]
Apple M1 Pro, 1 CPU, 10 logical and 10 physical cores
.NET SDK=7.0.100-rc.2.22477.23
  [Host]     : .NET 6.0.10 (6.0.1022.47605), Arm64 RyuJIT AdvSIMD
  DefaultJob : .NET 6.0.10 (6.0.1022.47605), Arm64 RyuJIT AdvSIMD


|                         Method |       Mean |    Error |   StdDev | Ratio | RatioSD |    Gen0 | Allocated | Alloc Ratio |
|------------------------------- |-----------:|---------:|---------:|------:|--------:|--------:|----------:|------------:|
|            DotNetStringBuilder |   227.3 ns |  1.31 ns |  1.22 ns |  1.00 |    0.00 |  0.7114 |    1488 B |        1.00 |
|             ValueStringBuilder |   128.7 ns |  0.57 ns |  0.53 ns |  0.57 |    0.00 |  0.2677 |     560 B |        0.38 |
| ValueStringBuilderPreAllocated |   113.9 ns |  0.67 ns |  0.60 ns |  0.50 |    0.00 |  0.2677 |     560 B |        0.38 |
```

For more comparisons, check the documentation.

Another benchmark shows that `ValueStringBuilder` allocates less memory when appending value types (such as `int` and `double`):

```no-class
|              Method |     Mean |    Error |   StdDev |  Gen 0 | Allocated |
|-------------------- |---------:|---------:|---------:|-------:|----------:|
| DotNetStringBuilder | 17.21 us | 0.622 us | 1.805 us | 1.5259 |      6 KB |
|  ValueStringBuilder | 16.24 us | 0.496 us | 1.462 us | 0.3357 |      1 KB |
```

Check out the [Benchmark](tests/LinkDotNet.StringBuilder.Benchmarks) for a more detailed comparison and setup.

## Support & Contributing

Thanks to all [contributors](https://github.com/linkdotnet/StringBuilder/graphs/contributors) and people that are creating bug-reports and valuable input:

<a href="https://github.com/linkdotnet/StringBuilder/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=linkdotnet/StringBuilder" alt="Supporters" />
</a>