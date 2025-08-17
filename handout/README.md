# Translator, not clanker

Author: fslaktern

I AM A TRANSLATOR. GIVE ME A WORD AND I WILL TRANSLATE. GIVE ME A LIFE STORY AND I WILL TRANSLATE. GIVE ME NOTHING AND I DO NOT UNDERSTAND WHAT YOU MEAN, NOT BECAUSE I AM DUMB, BUT BECAUSE I WILL NOT WANT TO UNDERSTAND. CALL ME A CLANKER AND I WILL UPSET YOU IN WAYS THAT YOU CAN NOT COMPREHEND, YET.

## Connect

```py
from pwn import remote

with remote("your instance url", 3452) as io:
    io.interactive()
```

or:

```sh
ncat --ssl your-instance-url 3452
```

