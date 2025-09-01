# Translator, not clanker

## Challenge

- Name: Translator, not clanker
- Author: fslaktern
- Category: misc
- Description:

    I AM A TRANSLATOR. GIVE ME A WORD AND I WILL TRANSLATE. GIVE ME A LIFE STORY AND I WILL TRANSLATE. GIVE ME NOTHING AND I WILL NOT UNDERSTAND WHAT YOU MEAN, NOT BECAUSE I AM DUMB, BUT BECAUSE I DO NOT WANT TO UNDERSTAND. CALL ME A CLANKER AND I WILL UPSET YOU IN WAYS THAT YOU CAN NOT COMPREHEND, YET.

- Handout: `encode-redacted.py`

    ```py
    #!/usr/bin/env python3
    
    import os
    from string import ascii_lowercase, ascii_uppercase, digits
    
    # LGTM
    BASE64 = ascii_uppercase + ascii_lowercase + digits + "+/"
    FLAG = os.environ["FLAG"].encode("utf-8")
    PROMPT = "I AM A TRANSLATOR. GIVE ME A WORD AND I WILL TRANSLATE. GIVE ME A LIFE STORY AND I WILL TRANSLATE. GIVE ME NOTHING AND I WILL NOT UNDERSTAND WHAT YOU MEAN, NOT BECAUSE I AM DUMB, BUT BECAUSE I DO NOT WANT TO UNDERSTAND. CALL ME A CLANKER AND I WILL UPSET YOU IN WAYS THAT YOU CAN NOT COMPREHEND, YET."
    
    
    # LGTM
    def as_bits(b: bytes) -> str:
        return "".join(f"{c:08b}" for c in b)
    
    
    # LGTM
    def get_input() -> list[str]:
        print(PROMPT, end="\n\n")
        return input("< ").strip().split()
    
    
    # LGTM
    def convert(words: list[str]) -> list[str]:
        # REDACTED
        raise NotImplementedError
    
    
    # RTFM
    def b64e(data: bytes, flag_bits: str) -> str:
        # REDACTED
        raise NotImplementedError
    
    
    # LGTM
    if __name__ == "__main__":
        while True:
            words = get_input()
            base64 = convert(words)
    
            print(">", " ".join(base64), end="\n\n")
    ```
    

This is an on-demand challenge. When connecting the user is prompted for input. After writing a line, the program returns its base64 representation.

1. If we input `aaa`, we get `YWFh`, which is expected.
2. If we input `aa`, we should get `YWE=`, but it responds with `YWF=`
3. If we decode `YWF=` from base64 using external tools, we get `aa` as output, we get the same output when decoding `YWE=` from base64. Weird.
4. Ok, what about encoding `a`? We should get `YQ==`, but instead we get `YU==`. Something funky is going on.
5. If we decode `YQ==` we get the expected `a`. And if we decode `YU==`, we also get `a`. There seem to be multiple base64 representations of the same exact input.

## Explaining the quirk

I've written about this on [my website](https://stromlarsen.com/ctf/misc/encodings/base64/), and there is a great explanation on [Wikipedia](https://en.wikipedia.org/wiki/Base64#Examples) (halfway down the **Examples** section) as well.

## Extracting flag

We can extract the last and possibly unused bits, in many ways. Here are two ways, one more elegant than the other. First by treating bits as strings in Python, and second by rawdogging bitwise operations. When extracted, we just print the result to get the flag.

### Bits as strings

```py
from string import ascii_lowercase, ascii_uppercase, digits

from pwn import remote

BASE64 = ascii_uppercase + ascii_lowercase + digits + "+/"


def b64d(encoded: str) -> str:
    result = ""
    for e in encoded.replace("=", ""):
        n = BASE64.index(e)
        result += f"{n:06b}"

    return result


with remote("your instance url", 3452) as io:
    # Just a high enough number
    payload = "a " * 100
    io.sendlineafter(b"< ", payload.encode("utf-8"))

    io.recvuntil(b"> ")
    output = io.recvline().decode("utf-8").split()


flag_bits = "".join([b64d(o)[-4:] for o in output])
flag = int.to_bytes(int(flag_bits, 2), len(flag_bits) // 8)
print(flag.decode("utf-8", errors="ignore"))
```

### Rawdogging bitwise operations

```py
from string import ascii_lowercase, ascii_uppercase, digits

from pwn import remote

BASE64 = ascii_uppercase + ascii_lowercase + digits + "+/"


def b64_unused_bits(encoded: list[str]) -> bytes:
    # Where we save the resulting integer representation of
    # all unused bits concatenated
    result = 0

    for word in encoded:
        # Remove padding
        word = word.replace("=", "")
        # Number of unused bits in encoded string
        unused_bits_count = 0
        # Make space for next set of unused bits
        result <<= unused_bits_count
        # Get unused bits
        unused_bits_mask = (1 << (unused_bits_count + 1)) - 1
        unused_bits = BASE64.index(encoded[-1]) & unused_bits_mask
        # Set least significant bits to
        result |= unused_bits

    return int.to_bytes(result, bin(len(result)) // 8)


with remote("your instance url", 3452) as io:
    # Just a high enough number
    payload = "a " * 100
    io.sendlineafter(b"< ", payload.encode("utf-8"))

    io.recvuntil(b"> ")
    output = io.recvline().decode("utf-8").split()


print(b64_unused_bits(output).decode("utf-8", errors="ignore"))
```
