##### Challenge Description
_This week's special: A chance to compete against our esteemed guest, the rock-paper-scissors grand master. If you manage to beat them 100 out of 100 times we will reward you with a flag specially made for you._

__Category__: Cryptography

### Solution Process
The source code of the program is available to us this is the main code which determines if we win against the machine
```
    print("I want to play a game...")
    already_seen = {}
    for _ in range(NUM_ROUNDS):
        com = bytes.fromhex(input("Commitment (hex): "))
        my_choice = choice(["rock", "paper", "scissors"])
        print(f"I choose {my_choice}.")
        your_choice = input("What did you choose? ")
        if your_choice not in {"rock", "paper", "scissors"}:
            print("*Your opponent just staress at you, seeming very confused*")
            return
        unveil_info = tuple(bytes.fromhex(x) for x in input("Proof (hex): ").split())
        if not verify(com, your_choice.encode("ascii"), unveil_info):
            print("Hey, no cheating! Do that again and I will eat all your flags")
            return
        elif my_choice == your_choice:
            print("Sorry, that was a draw. No flag for you")
            return
        elif (my_choice, your_choice) in {
            ("rock", "scissors"),
            ("scissors", "paper"),
            ("paper", "rock"),
        }:
            print("Sorry, you lose. No flag for you")
            return
        elif com in already_seen and already_seen[com] != your_choice:
            print("Something fishy is going on here. What are you doing?")
            return
        already_seen[com] = your_choice
    print(f"How can that be? Well, a deal is a deal. Here is your flag: {environ['FLAG']}")
```
Now one can see that obviously to beat the machine we must actually beat it in rock, paper, scissors which is not a hard task as it tells us its choice before it asks us for our's however to keep the loop going and not exit out one must also satisfy the following condition
```
if not verify(com, your_choice.encode("ascii"), unveil_info):
            print("Hey, no cheating! Do that again and I will eat all your flags")
            return
```
Now let us look at the `verify` function and see what is does
```
def verify(commitment: bytes, message: bytes, unveil_info: tuple[bytes, bytes]) -> bool:
    """
    Check if a commitment was correctly unveiled.

    :param commitment: The commitment that was unveiled
    :param message: The message the commitment was unveiled to
    :param unveil_info: The unveil information used to unveil the commitment

    :return: Whether the commitment was correctly unveiled
    """

    r1, r2 = unveil_info  # two is better than one, right?

    return commitment == sha256(r1 + message + r2).digest()
```
Looking at this we can see that what the function seems to do is take the two hex inputs we give it from the Proof prompt and then adds them before and after our choice in the game and checks if the commitment hex we gave it earlier matches the hash of this. Immediately one might think to solve this one needs to somehow brute-force a pre-image for the `sha256` algorithm to reverse what input one might need, but not only is that impossible, it is also not a practical!

But what we can do is somehow make sure that the `r1 + message + r2` thing is just right so that it always hashes to whatever `commitement` is, their might be other ways to do this, but i chose the following method.

Firstly, since the program also makes sure we do no repeat the `commitment` string we will use python's random library to generate two hex codes which we can add to our commitment string in any manner to make sure it doesn't repeat. 

The main trick is that we will always give the program `commitement`(its hash i.e.) in the following format `hash(random_hex1 + rockpaperscissors(in hex) + random_hex2)`, what this enables us to do is, regardless of the choice the computer takes we can give it the two hex strings such that it gives us `random_hex1 + rockpaperscissors(in hex) + random_hex2`. For example if we have to make the choice `rock` then we can just remove rock from the string and give the program `hex(random_hex1), hex(paperscissors + random_hex2`. Then the program will add everything together and the verify function will give us a value of true, looping this 100 times gives us the flag easily. Following is the code for this solution(I known it is pretty jank!)
```
from hashlib import sha256
from pwn import *
import random

# p = process(["python3", "main.py"])
p = remote("fermented-meatball-over-whipped-tahini-hoxe.gpn24.ctf.kitctf.de", 443, ssl=True)

for i in range(1,101):
    rand_num_1 = f"{random.randint(0,255):02x}"
    rand_num_2 = f"{random.randint(0,255):02x}"

    hash_bytes = bytes.fromhex(rand_num_1) + b"rockpaperscissors" + bytes.fromhex(rand_num_2)
    com = sha256(hash_bytes).hexdigest()

    p.recvuntil(b"Commitment (hex): ")
    p.sendline(com)

    bot_choice = p.recvline().split()[-1].rstrip(b'.')
    p.recvuntil(b"What did you choose? ")

    if bot_choice == b"rock":
        p.sendline(b"paper")
        r1 = rand_num_1 + "rock".encode().hex()
        r2 = "scissors".encode().hex() + rand_num_2
        p.recvuntil(b"Proof (hex): ")
        p.sendline(f"{r1} {r2}")
    if bot_choice == b"paper":
        p.sendline(b"scissors")
        r1 = rand_num_1 + "rockpaper".encode().hex()
        r2 = rand_num_2
        p.recvuntil(b"Proof (hex): ")
        p.sendline(f"{r1} {r2}")
    if bot_choice == b"scissors":
        p.sendline(b"rock")
        r1 = rand_num_1
        r2 = "paperscissors".encode().hex() + rand_num_2
        p.recvuntil(b"Proof (hex): ")
        p.sendline(f"{r1} {r2}")

    print(f"Won {i} times")

print(p.recvall().decode())
```
Running this program gives us the flag, (output is below, flags is redacted)
![[Selection_1161.png]]

