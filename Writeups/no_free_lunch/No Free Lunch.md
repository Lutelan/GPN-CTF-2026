##### Challenge Description
_I tried to remove all free lunch._
_Hopefully, you can still find a way out of this restaurant jail._

__Category:__ Pwn(Binary Exploitation)

### Solution Process
Looking at the `server.py` file we observer that all the file does is take the lines which we send to the server before typing `EOF`, and appends that to a temporary python file which has the following code at its start
```
import heapq # the only import you'll need today
from sys import addaudithook
from os import _exit
addaudithook(lambda x,y:_exit(0))

<your-code>
```
Now one must also notice that while running this a audit hook, is set which exits the program on noticing any security sensitive functions, such as `os`, also notice that the `PYTHONPATH`, environment variable is set to `/lib` thus we cannot import things like `subprocess`, doing a simple `os.system("./read_flag")` shall not work since, the program will exit, however we can easily just re define the `_exit` function to `print`, and thus giving the following payload to the server prints out the flag
```
_exit = print
import os
os.system("./read_flag")
EOF
```

The solution on running looks like this, the flag is obviously redacted
![[Selection_1158.png]]

PS: This does not seem to be the intended solution, however it works fine!