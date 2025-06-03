Upon connecting to the server via netcat we're presented with:

```
Welcome to our first version of CrypTopia Stream Cipher!
You can here encrypt any message you want.
Oh, one last thing: 3a72d30da1d40c9e5f991f6b7a0fc9f6a83284d5fff2fc
Enter your message:
```

The program will then return the encrypted version of whatever string we pass is. This gives us the opportunity to do a known plain text attack. Whatever we pass the program, we can simply xor it with the given cipher text and return a valid key to decrypt the flag.

```encr_flag = "1181e7bc65573fdeed20a91c527540b73b06463f48d09b"
    known_plain = "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
    known_encr = "3ed0d68e7f436bcab80d9730026775e231547470079f8719ed1ca0731330fd2d07617365302d5d0d311ef7d4d07ea657a637542b929350408b0edb420e8d67eae0370c4240d6e932c1d7c51a957293675d4d4d883dffa2aad7e61e371483d3"
    known_encr_text = conv.hex_to_text(known_encr)
    key = conv.xor_text(known_encr_text, known_plain)
    encr_flag_text = conv.hex_to_text(encr_flag)
    plaintext_flag = conv.xor_text(encr_flag_text, key)
    print(plaintext_flag)
```

Outputs:

```
N0PS{u5u4L_M1sT4k3S...}
```

#known_plaintext