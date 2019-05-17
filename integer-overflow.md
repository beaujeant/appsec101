# Integer overflow

#### 

DRAFT IDEAS

An overflow takes place whenever the result of an arithmetic operation with signed number doesn't fit the size of the register. For instance, `0x12345678` + `0xe0d1c2b3` = `0xf306192b`. In this example, the addition overflowed the most significant bit \(sign bit\), which incorrectly turns the sign integer as a _negative_ number. The OF can also be set with subtraction, e.g. `0x98765432` - `0x88776655` = `0x0ffeeddd`, which incorrectly turns the sign integer as a _positive_ number, so the **OF** would `true`.



In order to evaluate which value is greater/higher than another one, we can use once again the `sub` instruction and look if the **S**ign **F**lag \(**SF**\) has the same value as the **O**verflow **F**lag \(**OF**\). If it is the case, that means the _first_ operand is greater than the _second_ one and the jump is taken. If **SF** and **OF** are different, then, that means the _second_ operand is greater than the _first_ one and the jump is skipped. Remember, **SF** represent the result's most significant bit, and **OF** is set whenever an arithmetic overflow has occurred.

Here are four examples with all possible cases for the flag values. In this case, I will do subtraction on 8-bits value to make it easier and more readable:

```text
123 - 42: 01111011 - 00101010 = 01010001 (SF=0 | OF=0) -> 42 > 12
42 - 123: 00101010 - 01111011 = 10101111 (SF=1 | OF=0) -> 42 < 123
(-123) - 42: 10000101 - 00101010 = 01011011 (SF=0 | OF=1) -> -123 < 42
42 - (-123): 11010110 - 10000101 = 10100101 (SF=1 | OF=1) -> 42 > -123
```

The `jg` instruction also check a third flag, the **Z**ero **F**lag \(**ZF**\). Indeed, the condition verify if the first operand is greater than the second, so if the they are equals, this doesn't work.

```text
mov eax, 0x123
sub eax, 0x42
jg 0x08048400 ; jump is taken
```

```text
mov eax, 0x42
sub eax, 0x123
jg 0x08048400 ; jump is NOT taken
```

```text
mov eax, 0x42
sub eax, 0x42
jg 0x08048400 ; jump is NOT taken
```

#### jge - jump if greater or equal

