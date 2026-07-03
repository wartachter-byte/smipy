!!! WIP !!!
## Scopes and bridges
A scope is a standalone container with no initial elements other than read-only globals.
+ A local is a variable in the current scope. 
+ A global is a variable available from all scopes, standard read-only, but can be defined as writable from a scope.
+ A bridge is a variable retrieved from 1 scope up or down. Bridges can be stacked, meaning that scope A has bridge B and scope C can then make bridge D bridging to bridge B bridging to some variable.

## Variable storage
A variable is defined as an object with a [numerical ID](#numerical-id) and an [item on the stack](#stack-item) that may point to a [item on the heap](#heap-item) or be a stack type.
The numerical ID is a 16-bit value that describes locals, globals and bridged variables.
### Numerical ID
The way that a numerical ID is ordered is following this method:
```
0 (local / global) 0 (if global, bridged / global) 00000000000000 (ID)
```
### Stack item
A stack item is a 64-bit float. Thanks to [NaN-tagging](#nan-tagging) it is possible to put more [types](#stack-types) in it.
#### NaN-tagging
NaN-tagging is possible thanks to the definition of a NaN in IEEE754:
```
"NaN: exponent = e-max + 1 and trailing significand ≠ 0"
This means that the exponent is all 1's and the fraction part is not 0.
```
A sNaN is a signaling NaN.
It signals an exception and is made by having the significant bit in the fraction set to 0 and at least one other bit being 1.
To not accidentaly signal a sNaN you must set the most significant bit to 1, which will make it a qNaN which is a quiet NaN that doesn't cause any exceptions.
This means that we have 51 bits of data left over to use as we permanentaly set 1 from the 52 original bits.
#### Stack types
There are 8 types. Each one of them has a binary tag, except for the float, a function and a way that it is ordered:
+ Pointer (Tag: 000)
  ```
  [000000000000000000000000000000000000000000000000] (pointer data, 48 bits) [000] (Tag)
  ```
+ Int (Tag: 001)
  ```
  [0000000000000000] (future use) [00000000000000000000000000000000] (32-bit data) [001] (Tag)
  ```
+ Boolean (Tag: 010, combined with None)
+ None (Tag: 010, combined with Boolean)
  ```
  [0000000000000000000000000000000000000000000000] (Empty) [0] (is_None flag) [0] (Boolean data) [010] (Tag)
  ```
+ Short String (Tag: 011)
  ```
  The short string type operates on 2 modes:
    ascii (max. 6 char.)
    UTF-8 (max. 5 bytes)

  ascii mode:
  [0] (mode flag) [00000] (future use) [[0000000] (one char) [0000000] [0000000] [0000000] [0000000] [0000000]] (string) [011] (Tag)

  UTF-8 mode:
  [1] (mode flag) [00000 00] (future use) [[00000000] (one byte) [00000000] [00000000] [00000000] [00000000]] (string) [011] (Tag)
  ```
+ Exception (Tag: 100)
  ```
  [0000000000000000] (Exception type, 16-bit) [00000000000000000000000000000000] (additional info, 32-bit) [100] (Tag)
  ```
+ Future use (Tag: 101 and 110)
+ Float (No Tag)
    + Whenever it isn't NaN for putting data types in.
+ NaN (Tag: 111)
  ```
  [000000000000000000000000000000000000000000000000] (empty) [111] (Tag)
  The NaN type is neccesary for normal float arithmetic
  ```

### Heap Item
An heap item is an 32-byte block on the heap.
## Mathematical functions and order of operations

| Item 1 token | Item 1 function | Item 2 token | Item 2 function | Item 3 token | Item 3 function |
| ---: | :--- | ---: | :--- | ---: | :--- |
| ( \<value\> ) | Parentheses | - | - | - | - |
| \<a\> ^ \<b\> | exponentiation | - | - | - | - |
| \<a\> * \<b\> | multiplication | \<a\> / \<b\> | division | - | - |
| \<a\> + \<b\> | addition | \<a\> - \<b\> | subtraction | - | - |
| \<a\> ble \<b\> | bit shift left | \<a\> bri \<b\> | bit shift right | - | - |
| \<a\> band \<b\> | bitwise and | \<a\> bor \<b\> | bitwise or | \<a\> xbo \<b\> | exclusive bitwise or |
| not \<a\> | inversion | - | - | - | - |
| \<a\> smt \<b\> | smaller than | \<a\> lrt \<b\> | larger than | - | - |
| \<a\> eqs \<b\> | equals | \<a\> neqs \<b\> | not equals | - | - |
| \<a\> and \<b\> | boolean and | \<a\> or \<b\> | boolean or | - | - |
