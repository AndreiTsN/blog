When working with higher-level languages like Python, you rarely think about how data structures are stored in memory. However, in Rust, memory placement plays a crucial role, directly impacting performance and code behavior. Let's explore how arrays (array) and vectors (Vec) work, along with their key similarities and differences.

1. Key Similarities and Differences and Memory Management.

Array (Array)

An array is a fixed-size container allocated on the stack. Its size is known at compile time, making it highly efficient to work with.
Example of an array in Rust:

let a: [i32; 5] = [10, 20, 30, 40, 50];

Similarities with Vec:

✅ Both store elements of the same type.

✅ Both support iteration using for.

Differences:

❌ Fixed size: An array cannot be resized after creation.

⚡ Stack allocation: Access to elements is extremely fast.

In Rust, arrays are placed on the stack by default since their size is known at compile time. However, if an array is too large, it can be allocated on the heap using wrappers like Box<[T; N]>. Without such wrappers, arrays remain on the stack.
Example of heap allocation for a large array:

fn large_array() -> Box<[i32; 1_000_000]>
{
    Box::new([0; 1_000_000]) 
    // This array will be allocated on the heap
}


Vector (Vec)

A vector (Vec) is a dynamic data structure stored in the heap, allowing it to grow and shrink at runtime.
Example of a vector in Rust:

let vec1: Vec<i32> = vec![1, 2, 3, 4, 5];

Similarities with Array:

✅ Supports indexed access (vec[0]).

✅ Can be passed to functions that work with sequences.

Differences:

✅ Dynamic size: Elements can be added or removed.

🔄 Heap allocation: Requires memory management.


By understanding these differences, we can optimize Rust programs for better performance and memory efficiency!


2.1. Iterating over Arrays

Next, let's consider an interesting aspect of iterating over these structures. Let's start with an array.
In Rust, there are several ways to iterate over arrays ([T; N]), each with its own characteristics.

2.1.1 Iterating by Value (for i in arr)

fn main() {
    let arr = [1, 2, 3, 4, 5];
    
    for i in arr {
        println!("{}", i);
    }
    // The array remains available after iteration.
    println!("{:?}", arr);
}

🔹 How it works:

    The type T must implement Copy, otherwise the elements will be moved.
    Values are copied during iteration if T: Copy.
    This method does not modify the original array, and the array remains available after iteration.

❌ What if T does not implement Copy?

#[derive(Debug)]
struct MyStruct {
    value: i32,
}

fn main() {
    let arr = [MyStruct { value: 1 }, MyStruct { value: 2 }];
    
    for i in arr {
        println!("{:?}", i);
    }
    // arr is no longer available
}

🔹 How it works:

    MyStruct does not implement Copy, so the elements are moved during iteration.
    After the loop, arr is no longer available.
    To avoid this, you can use for i in &arr or .iter().

2.1.2 Iterating by Reference (for i in &arr)

fn main() {
    let arr = [1, 2, 3, 4, 5];
    
    for i in &arr {
        println!("{}", i);
    }
    // The array remains available after iteration.
    println!("{:?}", arr);
}

🔹 How it works:

    &arr transforms into a slice &[i32], which is a reference to the entire array.
    This happens thanks to automatic dereferencing (dereference coercion) in Rust.
    Rust converts &[T; N] into &[T], making the code more universal for working with sequences.
    The iteration happens by reference (&T).
    The original array remains available after the loop.

2.1.3. Iterating using .iter() (for i in arr.iter())

fn main() {
    let arr = [1, 2, 3, 4, 5];
    
    for i in arr.iter() {
        println!("{}", i);
    }
    // The array remains available after iteration.
    println!("{:?}", arr);
}

🔹 How it works:

    .iter() creates an iterator over references (&T), but not a slice.
    .iter() is useful for functional programming, such as when using sum():

fn main() {
    let arr = [1, 2, 3, 4, 5];
    let sum: i32 = arr.iter().sum();
    println!("Sum of elements: {}", sum);
}

2.1.4. Iterating with .into_iter() (why it's better to use .iter())

fn main() {
    let arr = [1, 2, 3, 4, 5];
    
    for i in arr.into_iter() {
        println!("{}", i);
    }
    // The array remains available after iteration.
    println!("{:?}", arr);
}

🔹 How it works:

    .into_iter() moves the elements of the array during iteration if the type does not implement Copy, or copies them if it does.
    In Rust versions 1.53 and higher, the array remains available after iteration if the elements implement Copy.

Why is it better to use .iter()?

    Performance: .iter() does not require copying or moving elements, which can be more efficient for larger data types.
    Explicit behavior: .iter() guarantees that the array will remain available after iteration.

Use .into_iter() if you need to move the elements out of the array and no longer need the array after the iteration.

2.1.5. Iterating with .iter_mut() (modifying elements)

fn main() {
    let mut arr = [1, 2, 3, 4, 5];
    
    for i in arr.iter_mut() {
        *i *= 2;
    }
    // The array remains available after iteration.
    println!("{:?}", arr);
}

2.1.6. Performance Considerations
When choosing an iteration method, you should consider performance, especially when working with large arrays:

    Using references (&arr, .iter(), .iter_mut()):
        Iterating by reference does not require copying elements, making it more efficient for arrays where elements consume significant memory.
        It also prevents data from being moved, which is useful if you want to keep the array available after iteration.

    Using for i in arr:
        If the type T implements Copy (e.g., primitive types like i32), this method can be efficient, as the data is just copied and the array remains available.
        However, for larger or more complex data types (e.g., structs), copying might be less efficient. In such cases, iterating by reference is preferable.

Example with large arrays: If an array contains a large number of elements, especially if each element is a complex structure with a large amount of data, using .iter() or &arr can avoid unnecessary copying and improve performance.

Summary


| Method                    | Ownership                     | Modification | Is the original array available? |
| ------------------------- | ----------------------------- | ----------- | ------------------------------- |
| `for i in arr`            | **Copies** (if `Copy`)        | ❌           | ✅                               |
| `for i in arr` (`!Copy`)  | **Moves** elements            | ❌           | ❌                               |
| `for i in &arr`           | **Reference (`&T`)**          | ❌           | ✅                               |
| `for i in arr.iter()`     | **Reference (`&T`)**          | ❌           | ✅                               |
| `for i in arr.into_iter()`| **Copies** (if `Copy`)        | ❌           | ✅                               |


💡 What to choose?

    for i in arr — if T: Copy, as the array remains available.
    for i in &arr or .iter() — if T does not implement Copy, to avoid moving elements.
    .iter_mut() — if you need to modify elements.
    
