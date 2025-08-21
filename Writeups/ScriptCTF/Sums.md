<details>
<summary>server.py</summary>

```python
import random
import subprocess
import sys
import time

start = time.time()

n = 123456

nums = [str(random.randint(0, 696969)) for _ in range(n)]

print(' '.join(nums), flush=True)

ranges = []
for _ in range(n):
    l = random.randint(0, n - 2)
    r = random.randint(l, n - 1)
    ranges.append(f"{l} {r}") #inclusive on [l, r] 0 indexed
    print(l, r)

big_input = ' '.join(nums) + "\n" + "\n".join(ranges) + "\n"

proc = subprocess.Popen(
    ['./solve'],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True
)

stdout, stderr = proc.communicate(input=big_input)

out_lines = stdout.splitlines()
ans = [int(x) for x in out_lines[:n]]

urnums = []
for _ in range(n):
    urnums.append(int(input()))

if ans != urnums:
    print("wawawawawawawawawa")
    sys.exit(1)

if time.time() - start > 10:
    print("tletletletletletle")
    sys.exit(1)

print(open('flag.txt', 'r').readline())
```
</details>

The challenge - "Find the sum of a[i] for i in [l, r]"

My first step is to rework the server code and ensure I can solve the sums locally in a timely manner, then I'll worry about server I/O. We only have a 10 second window to return a correct answer and a naive solve will be much too slow, doing 10000+ calculations for each range.

Instead we will do one full set of prefix calculations first, where we calculate the sum of all items in a up to range x, and save that number as prefix[x], then when it's time to do the range calculations we only have to do one calculation each time, which is prefix[r] - prefix[l], with some tweaks to make sure we're getting the inclusive range properly.

```python
import random

n = 123456

nums = [str(random.randint(0, 696969)) for _ in range(n)]

ranges = []
for _ in range(n):
    l = random.randint(0, n - 2)
    r = random.randint(l, n - 1)
    ranges.append(f"{l} {r}")

prefix = [0]
for num_str in nums:
    prefix.append(prefix[-1] + int(num_str))

for item in ranges:
    left_str, right_str = item.split(" ")
    left_int = int(left_str)
    right_int = int(right_str)
    result = prefix[right_int+1] - prefix[left_int]
    print(result)
```

Then to actually grab live input from the server and return it correctly.

```python
import random
import io
import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(("play.scriptsorcerers.xyz", 10480))
sock.settimeout(2)

n = 123456

file_in = sock.makefile('r', buffering=8192)
file_out = sock.makefile('w', buffering=8192)

nums_str = file_in.readline()

ranges = []

for i in range(n):
    ranges.append(file_in.readline())

nums = nums_str.split(" ")

prefix = [0]
for num_str in nums:
    prefix.append(prefix[-1] + int(num_str))

for item in ranges:
    left_str, right_str = item.split(" ")
    left_int = int(left_str)
    right_int = int(right_str)
    result = prefix[right_int+1] - prefix[left_int]
    file_out.write(f"{result}\n")

file_out.flush()

flag = file_in.readline()
print(flag)
```

scriptCTF{1_w4n7_m0r3_5um5_75b43c900d21}
