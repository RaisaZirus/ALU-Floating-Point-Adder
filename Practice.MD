I’ll build these like an evaluation sheet: decimal operands, the operation to perform, the FP16 inputs you should enter in Logisim, and a checked 16-bit answer key covering ordinary results, signs, cancellation, rounding, overflow, and underflow.

## Below are exam-style decimal questions covering normal addition/subtraction, negative results, cancellation, rounding, overflow, and underflow.

> For subtraction `A − B`, enter `A` normally and flip the sign bit of `B`, because the circuit performs addition using signed operands:
>
> ```text
> A − B = A + (−B)
> ```

## Quick solving procedure

For each decimal operand:

1. Determine the sign bit.
2. Convert the magnitude to binary.
3. Normalize it as `1.fraction × 2ᵉ`.
4. Stored exponent = `e + 15`.
5. Keep 10 fraction bits.
6. Enter the two 16-bit values.
7. Read and interpret `RESULT`, `OVF`, and `UNF`.

---

# Practice questions

Try solving these before checking the answers.

1. `5.5 + 2.25`
2. `12.0 + 0.5`
3. `−6.5 + 2.25`
4. `−3.5 + (−2.25)`
5. `8.0 − 3.5`
6. `3.5 − 8.0`
7. `1.0 + (−1.0)`
8. `65504 + 65504`
9. `−65504 + (−65504)`
10. `0.000091552734375 + (−0.0000762939453125)`
11. `0.00006109476089477539 − 0.00006103515625`
12. `1.0 + 0.00048828125`
13. `1.0009765625 + 0.00048828125`
14. `65504 + 16`

---

# Answer key

## 1. Normal addition

```text
5.5 + 2.25 = 7.75
```

```text
XA     = 0 | 10001 | 0110000000
XB     = 0 | 10000 | 0010000000
RESULT = 0 | 10001 | 1111000000
```

```text
OVF=0, UNF=0
```

---

## 2. Different exponents

```text
12.0 + 0.5 = 12.5
```

```text
XA     = 0 | 10010 | 1000000000
XB     = 0 | 01110 | 0000000000
RESULT = 0 | 10010 | 1001000000
```

```text
OVF=0, UNF=0
```

---

## 3. Opposite signs, negative result

```text
−6.5 + 2.25 = −4.25
```

```text
XA     = 1 | 10001 | 1010000000
XB     = 0 | 10000 | 0010000000
RESULT = 1 | 10001 | 0001000000
```

```text
OVF=0, UNF=0
```

---

## 4. Both numbers negative

```text
−3.5 + (−2.25) = −5.75
```

```text
XA     = 1 | 10000 | 1100000000
XB     = 1 | 10000 | 0010000000
RESULT = 1 | 10001 | 0111000000
```

```text
OVF=0, UNF=0
```

---

## 5. Positive subtraction result

Question:

```text
8.0 − 3.5
```

For the circuit:

```text
8.0 − 3.5 = 8.0 + (−3.5)
```

```text
XA         = 0 | 10010 | 0000000000
B normally = 0 | 10000 | 1100000000
XB entered = 1 | 10000 | 1100000000
RESULT     = 0 | 10001 | 0010000000
```

```text
RESULT = 4.5
OVF=0, UNF=0
```

---

## 6. Negative subtraction result

```text
3.5 − 8.0 = 3.5 + (−8.0) = −4.5
```

```text
XA         = 0 | 10000 | 1100000000
B normally = 0 | 10010 | 0000000000
XB entered = 1 | 10010 | 0000000000
RESULT     = 1 | 10001 | 0010000000
```

```text
OVF=0, UNF=0
```

---

## 7. Exact cancellation

```text
1.0 + (−1.0) = 0
```

```text
XA     = 0 | 01111 | 0000000000
XB     = 1 | 01111 | 0000000000
RESULT = 0 | 00000 | 0000000000
```

```text
OVF=0, UNF=0
```

Important internal event:

```text
SUM = 000000000000000
IS_ZERO = 1
```

The final zero-selection MUX forces `RESULT=0x0000`.

---

## 8. Positive overflow

```text
65504 + 65504 = +Infinity
```

```text
XA     = 0 | 11110 | 1111111111
XB     = 0 | 11110 | 1111111111
RESULT = 0 | 11111 | 0000000000
```

```text
OVF=1, UNF=0
```

The calculated exponent reaches `31`, so `FLAGS_PACK` forces exponent `11111` and clears the fraction.

---

## 9. Negative overflow

```text
−65504 + (−65504) = −Infinity
```

```text
XA     = 1 | 11110 | 1111111111
XB     = 1 | 11110 | 1111111111
RESULT = 1 | 11111 | 0000000000
```

```text
OVF=1, UNF=0
```

The overflow handling is identical, but the sign is `1`.

---

## 10. Subnormal output

```text
0.000091552734375
+ (−0.0000762939453125)
= 0.0000152587890625
```

Equivalent values:

```text
1.5×2⁻¹⁴ − 1.25×2⁻¹⁴ = 0.25×2⁻¹⁴
```

```text
XA     = 0 | 00001 | 1000000000
XB     = 1 | 00001 | 0100000000
RESULT = 0 | 00000 | 0100000000
```

```text
OVF=0, UNF=1
```

This activates `DENORM`.

---

## 11. Smallest positive subnormal

```text
0.00006109476089477539
− 0.00006103515625
= 0.000000059604644775390625
```

For subtraction, negate the second operand:

```text
XA         = 0 | 00001 | 0000000001
B normally = 0 | 00001 | 0000000000
XB entered = 1 | 00001 | 0000000000
RESULT     = 0 | 00000 | 0000000001
```

```text
OVF=0, UNF=1
```

The result is the smallest positive FP16 subnormal:

```text
2⁻²⁴
```

---

## 12. Halfway rounding: round downward

```text
1.0 + 0.00048828125
= 1.00048828125 exactly
```

Inputs:

```text
XA = 0 | 01111 | 0000000000
XB = 0 | 00100 | 0000000000
```

The exact answer is halfway between:

```text
1.0000000000₂
1.0000000001₂
```

Round-to-nearest-even selects the first value because its retained LSB is `0`.

```text
RESULT = 0 | 01111 | 0000000000
       = 1.0
```

```text
OVF=0, UNF=0
```

---

## 13. Halfway rounding: round upward

```text
1.0009765625 + 0.00048828125
= 1.00146484375 exactly
```

```text
XA = 0 | 01111 | 0000000001
XB = 0 | 00100 | 0000000000
```

The answer lies halfway between fraction values:

```text
0000000001    retained LSB is odd
0000000010    retained LSB is even
```

Nearest-even therefore rounds upward:

```text
RESULT = 0 | 01111 | 0000000010
       = 1.001953125
```

```text
OVF=0, UNF=0
```

---

## 14. Overflow caused by rounding

```text
65504 + 16 = 65520
```

```text
XA = 0 | 11110 | 1111111111
XB = 0 | 10011 | 0000000000
```

`65520` is the overflow rounding boundary. Rounding attempts to increase the maximum significand, producing an exponent carry.

```text
RESULT = 0 | 11111 | 0000000000
       = +Infinity
```

```text
OVF=1, UNF=0
```

## Most important cases to practise first

If evaluation time is limited, prioritize these:

1. `12.0 + 0.5` — alignment  
2. `−6.5 + 2.25` — opposite signs  
3. `3.5 − 8.0` — negative subtraction result  
4. `1.0 + (−1.0)` — exact cancellation  
5. `65504 + 65504` — overflow  
6. Smallest-normal subtraction — subnormal output  
7. `1.0 + 2⁻¹¹` — round-to-nearest-even