##### Challenge Description
_I always feel like cooking is such a chore... You have to chop up all your ingredients, cook them for hours and then make the plating look half-decent.
But not with this new machine I got! You just have to put in your recipe (weirdly, the interface calls it a flag...) and it will get cooked for you. It's so easy, even someone with no experience in ~~cooking~~ reverse engineering can do it._

Category: Reverse Engineering

### Solution Process
I personally use a combination of Ghidra and GDB for reverse engineering binaries, at least ELF ones. If one gives the `autocooker` binary to Ghidra, and then looks at the `main` functions decompilation, one notices the following code
```
undefined8 main(int param_1,long param_2)

{
   int iVar1;
   undefined4 local_c;
   
   local_c = 0;
   if (1 < param_1) {
      iVar1 = strcmp(*(char **)(param_2 + 8),"cooking_class");
      if (iVar1 == 0) {
          local_c = 1;
      }
   }
------------------------------------------------------------------------(reduced)
```
Looking at the code, one realises all it does is check the the argument supplied to the binary is equal to the string `cooking_class`, so we shall run the program as follows `./autocooker cooking_class`. Now the next check is the function `check_recipe_length`, looking at its disassembly one can see that all it is doing is checking whether the length of the input is equal to a certain `TARGET_LENGTH`, which turns out to be `60` characters, (actually 61 but that is the null byte, thus we only give the program sixty characters).

Thus if we run the program as `./autocooker cooking_class`, and then give it a simple input of 60 characters, say 60 `A`. We get the following output
```
     _   _   _ _____ ___     ____ ___   ___  _  _______ ____
    / \ | | | |_   _/ _ \   / ___/ _ \ / _ \| |/ / ____|  _ \
   / _ \| | | | | || | | | | |  | | | | | | | ' /|  _| | |_) |
  / ___ \ |_| | | || |_| | | |__| |_| | |_| | . \| |___|  _ <
 /_/   \_\___/  |_| \___/   \____\___/ \___/|_|\_\_____|_| \_\

Welcome to the auto cooker. We'll cook any recipe for you under one condition: It must actually taste good.

Enter your recipe (flag) you want to cook and confirm with [ENTER]:
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
We now have the following state of our kitchen:
41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 0A 00 00 00
Salting...
We now have the following state of our kitchen:
EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB EB A0 AA AA AA
Frying...
We now have the following state of our kitchen:
BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE 0A AA AA AA
Oops, it burned :(
Cutting off the burnt bits...
We now have the following state of our kitchen:
BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE 0A 0A 0A 0A
Mixing...
We now have the following state of our kitchen:
0A 0A 0A 0A BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE BE
Taste testing...
YUCK!
```
The step that seems to fail is the testing step, which we can see in the disassembly as the test function, analysing it, one observers all it does it checks whether our input string after manipulation by salting, frying, cutting and mixing is equal to a certain hex string which is hard coded into the binary, one can look at this using Ghidra, or objdump, this hex string is as follows
```
0a 0a 0a 0a 7d dd 5c 4e 5f 9f fc 2c f9 3c ee 5f ef f9 fc ec 8d e9 2e 5f 8f ff a9 5f 8f a9 cc 5f 3f ec be fe 8d 5f fe 8d e9 5f fd a9 3d 5f 99 1e 3e 6e 5f 6c 99 99 cc 5f 3c 1d ce ef 9e 4e af de
```
So all we must do, is reverse the steps which result in this output and that should be our target input or in this case the flag, analysing the various functions `mix, trim, fry, salt`. We see that mix just reverses the bytes, trim removes the last four which don't even matter so we can ignore that function, `fry` perform's a nibble swap on the hex, and `salt` xor's each byte with `0xAA`. One can reverse these steps using a python program or something else, gives us the flag.
```
DELICIOUS = bytes.fromhex(
    "0a 0a 0a 0a 7d dd 5c 4e 5f 9f fc 2c f9 3c ee 5f "
    "ef f9 fc ec 8d e9 2e 5f 8f ff a9 5f 8f a9 cc 5f "
    "3f ec be fe 8d 5f fe 8d e9 5f fd a9 3d 5f 99 1e "
    "3e 6e 5f 6c 99 99 cc 5f 3c 1d ce ef 9e 4e af de"
)

data = DELICIOUS[::-1]

data = bytes(((b & 0x0f) << 4) | ((b & 0xf0) >> 4) for b in data)

data = bytes(b ^ 0xaa for b in data)

print(data.decode())
```
Running this code gives us the bytes, which when converted to ASCII give the required flag.