# <center>**Small Subgroup Confinement Attack**</center>

### **Prerequisite**

- Know how [Diffie-Hellman key exchange](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange#Cryptographic_explanation) works

In this article, I hope to explain clearly and as simply as possible what a **small subgroup confinement attack** is and how this attack works.  
And finally, at the end of this article, how to protect your **Diffie-Hellman key exchange** from this attack.  

## **What’s a Small subgroup confinement attack ?**
***
The goal of a **small subgroup confinement attack** is to force the key to be confined to an unexpectedly small subgroup of the desire group and to recover a secret private integer (**a** or **b**).

## **Perform a Small Subgroup Confinement Attack on a Diffie Hellman Key Exchange**
***
### **Prior knowledge**

>To perform this attack and retrieve the secret integer chosen by our target, we must use a **man in the middle attack (MITM)** attack and give a tampered parameter to our target.
>
>_I’m not going to explain what’s a MITM attack and how to perform it in this article._

### **Situation**

We’ve intercepted Alice’s parameters, but we can only interact with Bob. How do we get the shared secret to decrypt Alice's encrypted message?

We will tamper Bob’s parameters in order to get his secret private integer **b**.  
For that, we’re going to use a small subgroup confinement attack.

### **Setup**

The _first step_ is to find a [smooth prime number](https://en.wikipedia.org/wiki/Smooth_number) **p** upper than the p given by Alice, with many small factors _(factors less than or equal to 2<sup>16</sup> are good because they don't require much computation time, which makes the attack faster)_.

You can find a very good function (get_smooth_prime) in this [CTF write up](https://github.com/KooroshRZ/CTF-Writeups/blob/faee0d5cc47e59f0dc2e23d9b9e368d038edd803/PicoCTF2022/Cryptography/VerySmooth/index.md), to make smooth prime where you can easily choose his length and smoothness.

With this smooth prime number **p**, we have a lot of small factors which divide **p-1**.

_The attack can start !_

### **Exploit the vulnerability of smooth prime number**

You will do these _4 steps_ on all factors of **p** to get the secret integer **b** of Bob.  
_We’re going to call each factor **f**._

> #### **Step 1**
>
>> To do that, we have to find a number A’ in order that:  
>> **<center>A’<sup>(rand_int(p-1) / p)</sup> (mod p) != 1.</center>**  
>> Example of a python function to find A’:  
>>
>> ```python
>> from random import randint
>> 
>> def get_tampered_public_key(f, p):
>>     tampered_A = 1
>>     while tampered_A == 1:
>>         tampered_A = pow(randint(1, p), (p-1)//f, p)
>>     return tampered_A
>> ```
>>
> #### **Step 2**
>
>> Send **A’** to Bob as _Alice public key_
>
> #### **Step 3**
>
>> Bob use **A’** to create his _shared secret_. [Bob shared secret = **A’<sup>b</sup> (mod p)**]
>
> #### **Step 4**
>
>> Intercept Bob’s _MAC(shared_secret, message)_  
>> With **A’** used in Bob shared secret, we have reduced the possible value of the _shared secret_, because Bob raised our A’ to a power b and the order of **A’** is **f**.  
>> We deliberately choose small factors **f** so we can brute force all possible value from **1** to **f** until we find an **x** such as  **A’<sup>x</sup> (mod p)** is equal to Bob _MAC(shared_secret, message)_.  
>> The result can be written like that: **x ≡ b<sub>1</sub> (mod f<sub>1</sub>)**

After repeating this fourth step, we have a lot of equations like this:

- x<sub>1</sub> ≡ b<sub>1</sub> (mod f<sub>1</sub>)
- x<sub>2</sub> ≡ b<sub>2</sub> (mod f<sub>2</sub>)
- …
- x<sub>n</sub> ≡ b<sub>n</sub> (mod f<sub>n</sub>)

### **Find the secret integer b of Bob**

With all these equations we can easily recover the secret integer of Bob using the **Chinese remainder theorem** (**CRT**).

Example to calculate the CRT of our equations in python:

```python
from sympy.ntheory.modular import crt

#Where f_n is the list of all the factors of our tampered p and x_n the list of all the x in our equations
b = crt(f_n, x_n)[0]
```

## **Countermeasure against the Small subgroup confinement attack**
***
One solution to protect your Diffie-Hellman key exchange from this attack is to use a **[safe prime p](https://en.wikipedia.org/wiki/Safe_and_Sophie_Germain_primes)** where ***p = 2q + 1*** and **q** is prime.

_Check that 2 <= y <= p-2 (otherwise there may be a 1 bit leak)_

With **p** a safe prime, **p-1** has a large prime factor that makes the small subgroup confinement attack impossible due to computational time.

Obviously, don’t use a prime number p with a small bit-length for a communication. According to the [Federal Office for Information Security](https://en.wikipedia.org/wiki/Federal_Office_for_Information_Security):
> “The length of p should be at least 2000 bits, and at least 3000 bits for a period of use beyond 2022” ([7.2.1 Diffie Hellman – Key length](https://www.bsi.bund.de/SharedDocs/Downloads/EN/BSI/Publications/TechGuidelines/TG02102/BSI-TR-02102-1.pdf?__blob=publicationFile&v=10))

## **References**
***
- [C.H. Lim and P.J. Lee. (1998). "A key recovery attack on discrete log-based schemes using a prime order subgroup"](https://link.springer.com/content/pdf/10.1007/BFb0052240.pdf)
- [Measuring small subgroup attacks against Diffie-Hellman](https://eprint.iacr.org/2016/995.pdf)
- [NDSS 2017: Measuring small subgroup attacks against Diffie-Hellman (video)](https://www.youtube.com/watch?v=noFbyPHXY0A)
