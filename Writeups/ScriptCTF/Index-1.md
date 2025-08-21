We're presented with a menu like 
```
1. Read data
2. Write data
3. Print flag
4. Exit
```
When selecting an option you are then prompted to give an index, and you can then read or write 7 bytes of data to/from that index. Each index being 8 bytes apart. The "print flag" option at this point just being a joke.

I first tried passing a format string but found no unusual behaviour. Also no obvious overflow, my input was getting properly cut off at 7 bytes + a likely null byte.

I then tried an integer overflow when selecting an index. There wasn't actually an integer overflow but it was possible to read/write items outside of the expected range. Attempting a write like this would trigger segfault, but we can essentially read anywhere in the program, being allowed to pass negative numbers also.

I pulled the binary up in Cutter and can see that the menu has a secret option. if ```atoi(&str) == 0x539``` where &str is our input, we pass the check. atoi is ascii to int, 0x539 is 1337 in hex, so passing 1337 as our option passes this check. This doesn't directly print the flag but it does load it in memory to a variable called flag.

By poking around a bit we can find the flag loaded at index 8, 64 bytes from the index initialization. We're able to read the full at once rather than just 7 bytes at a time as the flag gets loaded in without the forced null byte terminator at the 8th byte.

scriptCTF{4rr4y_00B_unl0ck3d_0bc37a3928c5}
