## Resources

<details> <summary>Challenge Source Code</summary>
#!/usr/bin/env python3

import json
from Crypto.Hash import CMAC
from Crypto.Cipher import AES
from Crypto.PublicKey import RSA
from Crypto.Signature import pkcs1_15
from secret import FLAG, CMAC_KEY, PUBKEY_TAG, try_read_cmac_key


def mac(msg):
    cmac = CMAC.new(CMAC_KEY, ciphermod=AES)
    cmac.oid = '2.16.840.1.101.3.4.2.42'
    cmac.update(msg)
    return cmac


def do_update():
    update = json.loads(input('update package: '))
    n_bytes = bytes.fromhex(update['pubkey'])
    signature_bytes = bytes.fromhex(update['signature'])
    payload_bytes = bytes.fromhex(update['payload'])

    key_tag = mac(n_bytes).digest()
    if key_tag != PUBKEY_TAG:
        print('verification failed')
        return

    n = int.from_bytes(n_bytes, 'big')
    e = 0x10001
    pubkey = RSA.construct((n, e))
    verifier = pkcs1_15.new(pubkey)

    h = mac(payload_bytes)
    signature = signature_bytes

    try:
        verifier.verify(h, signature)
    except:
        print('verification failed')
        return
    print('signature correct')

    if payload_bytes == b'Gimmie a flag, pretty please.':
        print(FLAG)

    print('update succesfull')


def main():
    while True:
        print('1. try to read cmac key')
        print('2. do an update')
        choice = int(input('your choice: '))
        if choice == 1:
            print(try_read_cmac_key())
        elif choice == 2:
            do_update()
        else: break

main()
</details>

<details> <summary>Known Valid JSON</summary>
{"payload": "02a9bae06574e74b0c8afa7839c215a39621428b10d3bff2736493ffb7eeca12b7bb36aee3085e040475faa0d3b3cfac85ed6bedc1caee35ad0ce42c3fa06d42f3224a267d190aada84a3b76807d266ead2b1f5e2eddc3c7772227a4dfbe894bb1d3f47cf4e5aacd653a08c36d57c07c778ab55d9aaba0c99d874eec7ddb8f28", "pubkey": "babca97e73b969bd26a13a489f942c508d06774c3f246f7c2e64139932d3ad3276ece5126c4e9e1c5cc541adccf789d47d843d2ea83138810d7ac58da422f23a7e8c5d4a10cf62510eed6c060f9f01087c7ed4ba82ae95a40260b9848a7630f2d2ada3075f656f5e4b56cd94cd3bcd7ee1d8d3427336916b8f4e06207c4577d8f9cfa8bea054c54db5b39fe932d60826fc240235761be17982bb7af46d8a133db836a5b74466b81f13c4be42b76b57c36b8292e509ed35325bceba79591804e084c5caf79c661f22c09a3e5d3da5cb2b64f3e8d451becbe4ff317cf479d475e3bed24c022b3b865d13e8381579ec97ec2e0fe7474cc9db7c8c85036958b77e91", "signature": "048a5520c6d1e3859d52056bc6a75fb0d9130cb29abf6bb6d186a6e89d90a2bad67969b5e54a62d80c537bd96accd038e00acd5321be551925972ac82c743a399ef442beca5232bba8dd913c53ba31ffec32b35cc5ec4d9078cae7abfe03081e57db519d9fe7199e6801dfa291fdc32f7fe2c5f79445750bc8c6cd1d0cd4469428d40ba98303005eb058baabd0f25d85a07fb0c5e196e1a5b982cf11f64409d2ab6bf66df4b1e709eb6572149d3316018af73e2cabeed7639e80f6ddc0ed18199e56873fb0cc03dc4e37b02ef8a8557969a8b05f6366c99f36ea05f384a25c81043302c7d296f12b64a43bfc55774bd095dbc529e1a29dcac4fe59b7d51ef83c"}
</details>

## Challenge Overview

Our goal is to send the payload "b'Gimmie a flag, pretty please.'" and receive a flag in return. However, changing our payload invalidates our signature, so our main challenge is signing our new payload to pass the verification checks.

The challenge is based on a real-life exploit called EntrySign ([Original blog here](https://bughunters.google.com/blog/5424842357473280/zen-and-the-art-of-microcode-hacking)). The vulnerability stems from using a CMAC digest instead of a proper hash, combined with our ability to read the CMAC key using our handy dandy electron microscope. This means we can craft collisions (different inputs that produce the same hashed output) with the original CMAC digests that will pass verification.

We can't simply cause a collision with our payload since it has strict requirements on its contents. We also can't use our known valid signature as it won't work on our new payload. Outright forging a new signature would be akin to breaking RSA and not computationally feasible. This leaves the pubkey field as our attack vector.

During the challenge, I initially tried making the shortest possible pubkey that would collide with the old digest. This seemed promising since the source code uses the pubkey as the modulus N in the standard RSA operation (m^e)^d ≡ m (mod N). I thought reducing the modulus might make brute force feasible, but this misunderstood what actually makes RSA secure.

The correct approach is to craft a matching pubkey with small factors. Once we can factor our pubkey, we calculate phi(N) and from there derive d = e^(-1) mod phi(N) where d is our private exponent. Important note: we're not recovering the original private exponent, but rather creating one that works with our public exponent and satisfies (m^e)^d ≡ m (mod N).

With this valid private exponent matching our known public exponent "e" and our self-defined modulus N, we can sign any message we want.

The main challenge that I haven't explored here would be how to craft an optimal pubkey with easily factored N while maintaining the CMAC collision. One thought is to automate incrementing the pubkey and adjusting the compensating bytes to maintain validity, then brute force while checking for small factors. There may be a more intelligent deterministic approach instead.