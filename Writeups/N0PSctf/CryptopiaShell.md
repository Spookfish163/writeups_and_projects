
## Source code
```
import os
from base64 import b64decode

class CrypTopiaShell():

    # THE VALUES OF P AND K ARE NOT THE CORRECT ONES AND ARE ONLY PLACEHOLDERS
    P = 0xdf5321e0a509b27419d9680b0a20698c841b6420906047d58b15ae331df19f0ac38703bd109e64098e77567ffb62fe2814be54e0e1a3aef9a5e58f5bf7a8437d41a6402aad078ae4d118274337bb0b1e2c943ae7c3f9f12c3602560434e5fc1dc373a272259b6d803731e696e4c9f9ef0420ff95225f321d81c3650a469240c523e81a26134dcbdf0b12ba941c09b0aae856fc4fdd6b8f1cf7a7e61796d042dc3921d7d0231338008ee1fe8f2f9d33ea0d669d9c25af51df10ab3ef612e3071088abef9572aa82228791a7bf218771de5db5ebec68a405f9646e05d44cae5932c5e0a1c95b672fd3d1a2f120b918391391c7cd569e59656904ac7f14cb33e4bb
    K = 0x9f9798d08dc88586a04a234525f591413e8d45b13d1ffe2c071e281d28bd8381

    G = 0x8b6eec60fae5681c
    MAGIC = b"\x01\x02CrypTopiaSig\x03\x04"

    def __sign(self, gen, key, mod):
        bl = gen.bit_length()
        for i in range(len(self.data)):
            gen = (gen ^ (self.data[i] << (i % bl))) & 2**bl-1
        s = 1
        while key:
            if key & 1:
                s = s * gen % mod
            key = key >> 1
            gen = gen * gen % mod
        return s

    def create(self, data):
        self.data = data
        self.signature = self.__sign(self.G, self.K, self.P).to_bytes(self.P.bit_length()//8, 'big')
        self.header = self.MAGIC + len(self.data).to_bytes(6, 'big') + len(self.signature).to_bytes(6, 'big')

    def parse(self, data):
        if data[:len(self.MAGIC)]!= self.MAGIC:
            print("Missing magic bytes")
            return False
        length = int.from_bytes(data[len(self.MAGIC):len(self.MAGIC)+6], 'big')
        signature_length = int.from_bytes(data[len(self.MAGIC)+6:len(self.MAGIC)+12], 'big')
        if len(data) > len(self.MAGIC)+12+length+signature_length:
            print("Invalid data size")
            return False
        self.data = data[len(self.MAGIC)+12:len(self.MAGIC)+12+length]
        self.signature = data[len(self.MAGIC)+12+length:len(self.MAGIC)+12+length+signature_length]
        if self.__sign(self.G, self.K, self.P).to_bytes(self.P.bit_length()//8, 'big') != self.signature:
            print("Invalid signature")
            return False
        return True

    def run(self):
        try:
            os.system(self.data)
        except Exception as e:
            print(f"Woops! Something went wrong")

    def dump(self):
        return self.header + self.data + self.signature

ctc = CrypTopiaShell()

print("Welcome to CrypTopiaShell!\nProvide base64 encoded shell commands in the CrypTopiaSig format in order to get them executed.")

while True:
    try:
        data = input("$ ")
        try:
            data = b64decode(data)
        except:
            print("Invalid base64 data")
            continue
        try:
            if not ctc.parse(data):
                continue
            ctc.run()
        except:
            print(f"Invalid CrypTopiaSig file")
    except:
        break
```

## Attack Method

The key thing to realise is that the message data only effects the $gen variable, and the signing algorithm then runs on $gen. Clashing the signature would be essentially impossible, but clashing $gen is quite easy since it's a completely known system.

There are actually two approaches here. I opted to clash the original gen and thus be able to reuse our known valid signature. You could however also ensure your gen is 0 and so the value of the a valid signature would also be 0.

In order to generate the exact same $gen while sending any message we want, I opted to take my input data, pad it to 64 bytes, and then calculate its gen using the exact same code from the source. Then knowing my current gen and the target gen, I used this function to flip the necessary bits until they matched.

```
def bit_correcter(starting_bits, target_bits):
    correctors = b""
    for i in range(64):
        if starting_bits[63-i] == target_bits[63-i]:
            correctors += b"\x00"
        else:
            correctors += b"\x01"
    return correctors
```
It compares our current and target bits, if the bit at current index needs to be flipped, we pass a \x01 byte to our message, if it's already correct we pass a \x00 byte.
## Attack code
```
from base64 import b64encode

G = 0x8b6eec60fae5681c

MAGIC = b"\x01\x02CrypTopiaSig\x03\x04"

signature = bytes.fromhex("8e4d1b7ba81471161e66432aadeb28e4115704877ef6dc9caf26cea2773c570369190d7a02b7adcad6bd470e61c58b3ec339810a02c7dfce2c6ae1fca13f66c7eefd3944659099f488b3b8a6ff212d9aa26b82ef8e6704eaae81cf8f7263c969f188747168e01eac3bfd0d5459f70ea4213f98ec4232206ed4abbc20d93572ba628f1b88e2e8a34210bec63097cbb3365284a307a749ba2309b2fcfa57b302b6604f65a7537f5b57eb8decca0357cb07ef32241a02c977152b35b2cc9fc20ea134fccfa891f4b92b2fd01b9160f48a6696b7aaf3a7c8e23bafd2c07297a5a32fd8ca246d2ee84e7afdcd66f7f15df06823f6b651d2d71fd0d135b20fe8625457")

first_message = b'echo "The date is $(date)\x0aYou are $(whoami)\x0aCurrent directory is $(pwd)"'

def main():
    x = gen_sim(first_message)
    target_bits = format(x, '064b')

    data = input("What would you like to send?: ").encode()
    padded = pad_to_64(data)

    starting_gen = gen_sim(padded)

    starting_bits = format(starting_gen, '064b')
    correcting_bytes = bit_correcter(starting_bits, target_bits)
    new_message = padded + correcting_bytes

    test_gen = gen_sim(new_message)
    new_bits = format(test_gen, '064b')

    if new_bits == target_bits:
        print("match")
    else:
        print("no match")

    header = MAGIC + len(new_message).to_bytes(6, 'big') + len(signature).to_bytes(6, 'big')
    payload = header + new_message + signature
    payload = b64encode(payload)
    print(payload.decode('ascii'))  # Print the result

def pad_to_64(message):
   return message + b' ' * (64 - len(message))

def gen_sim(subject):
    data = subject
    gen = G
    for i in range(len(data)):
        gen = (gen ^ (data[i] << (i % 64))) & 2**64-1
    return gen

def bit_correcter(starting_bits, target_bits):
    correctors = b""
    for i in range(64):
        if starting_bits[63-i] == target_bits[63-i]:  # Fixed indexing
            correctors += b"\x00"
        else:
            correctors += b"\x01"
    return correctors

main()
```


## The Problem

In the end my implementation still failed. I passed all authentication checks but the os.system() call reported an error and I could not execute commands. If I had time I could have tweaked the source code, tested locally, and gotten more useful error reporting and learned that the issue was the null bytes I was sending.

Even though I was sending valid Bash and passing all checks, os.system() does not allow any null bytes to be included. This would have been fairly easy to work around, I would just need to use a different byte when doing the correcting step. In principle there are endless ways to achieve that, they're just slightly more complex than my original implementation and I was unfortunately out of time.