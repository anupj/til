`impl <trait>` makes the compiler do the hard work of figuring out the exact type:

Oftentimes, you have a situation where you don't care exactly what type something has,
you simply care that it implements particular traits. For example, you might
have a function that accepts any kind of iterator as long as it produces integers.
In this case, you could write the following:

```Rust
fn sum_integers(integers: impl Iterator<Item = u32>) -> u32 {
    //                    ^^^^^^^^^^^^^
    let mut sum = 0;
    for integer in integers {
        sum += x;
    }
    sum
}
```

Here, the impl Iterator type is a special sort of type. It doesn't
actually name any particular type. Instead, it is a shorthand for a
generic type. In other words, the function above is roughly equivalent
to the following desugared version:

```Rust
fn sum_integers<I>(integers: I) -> u32
where
    I: Iterator<Item = u32>
{
    let mut sum = 0;
    for integer in integers {
        sum += x;
    }
    sum
}
```

Intuitively, a function that has an argument of type impl Iterator is saying
"you can give me any sort of iterator that you like".

## Resources

- [Impl trait Initiative](https://rust-lang.github.io/impl-trait-initiative/explainer.html)
