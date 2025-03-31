## SongCipher

### Description
"Sombody once told me the cipher was gonna roll me. You are the sharpest tool in the shed  After this challenge, you might want to add that song to your playlist :)"

Attached is a file with hex data ``aa, d7, ce, d9, 82, d0, d6, de, 40, e8, dd, d8, 85, 84, e3, d8, da, cb, 40, d6, d3, 40, e1, e1, 85, 93, ee, d0, df, dc, a3, 40, bc, ea, 81, d4, df, 8f, 8e, b4, 97, d3, dc, dc, 8d, 40, c0, dc, 81, b6, 90, 82, 89, bd, 8f, a0, 40, d8, cd, c6, 92, 94, 88, b8, da, df, c6, 94, 94, 61, e0, db, 8f, de, 89, d0, d6, 94, a0, 88, cc, 85, e7, 88, d4, d9, 94, 73, d7, cb, 40, df, c6, e5, 85, 9a, 8f, b0, d7, d5, 8e, d6, 86, 8b, e2, dd, d9, 4c, 8f, d3, 8f, da, da, 8d, cb, 94, 98, 89, b7, d7, 8d, cd, 85, e1, 8e, 87, 89, ba, cc, d9, 99, 93, 81, d5, d3, 41, 88, b9, da, 85, 94, ce, e1, ce, c9, 40, b2, e1, 40, e7, df, c6, 8d, e3, ab, b5, b6, e0, 73, a0, d3, 90, cd, a1, 7f, 75, 7c, 90, 87, ce, 9e, 74, b8, c4, b5, 51, d6, d7, a5, d7, e5`` 

### Initial analysis
The file is relatively short at around 180 bytes, so statistical analysis isn't likely to get us far. The clue strongly suggests the lyrics to Smash Mouth's All Star will be relevant in breaking this cipher but that still leaves a lot of variance in exactly how it's applied, and which parts of the lyrics were used.

The first thing I did was convert the hex to its decimal and ascii representation. As ascii it was mostly unprintable. As decimal we see this.

``[170, 215, 206, 217, 130, 208, 214, 222, 64, 232, 221, 216, 133, 132, 227, 216, 218, 203, 64, 214, 211, 64, 225, 225, 133, 147, 238, 208, 223, 220, 163, 64, 188, 234, 129, 212, 223, 143, 142, 180, 151, 211, 220, 220, 141, 64, 192, 220, 129, 182, 144, 130, 137, 189, 143, 160, 64, 216, 205, 198, 146, 148, 136, 184, 218, 223, 198, 148, 148, 97, 224, 219, 143, 222, 137, 208, 214, 148, 160, 136, 204, 133, 231, 136, 212, 217, 148, 115, 215, 203, 64, 223, 198, 229, 133, 154, 143, 176, 215, 213, 142, 214, 134, 139, 226, 221, 217, 76, 143, 211, 143, 218, 218, 141, 203, 148, 152, 137, 183, 215, 141, 205, 133, 225, 142, 135, 137, 186, 204, 217, 153, 147, 129, 213, 211, 65, 136, 185, 218, 133, 148, 206, 225, 206, 201, 64, 178, 225, 64, 231, 223, 198, 141, 227, 171, 181, 182, 224, 115, 160, 211, 144, 205, 161, 127, 117, 124, 144, 135, 206, 158, 116, 184, 196, 181, 81, 214, 215, 165, 215, 229]``

The first thing that stood out to me is that these numbers do not look very random. We have a lot of numbers around 210, and almost none below 100.

If you look at an ascii table like https://www.rapidtables.com/code/text/ascii-table.html . You'll notice the lowercase alphabet characters all sit at around 110. So the decimal numbers we're seeing would be coherent with two lowercase alphabet characters added together, with a modulo 256 to make sure we stay in range for hexadecimal representation. If the process was an XOR instead, then the decimal representation would look essentially random. 

There's another clue in the decimal data, at the 9th position we have a very low value, "64", this would match up with 32\*2, and 32 is the spacebar character. So it looks like both the original plaintext, and the key, have a space in position 9. That would make sense if the first word is "Somebody ". Giving us a strong hint.
### Approach
We can assume our cipher text is the result of the original text and some All Star lyrics having their decimal values added together directly. To reverse it we just need to know the exact lyrics that were used, then subtract their decimal values, from our ciphertext decimal values.

I'll use my own program to achieve this but the function names should be descriptive enough.
### Solution
```
def main():
    #The lyrics we're testing
    KEY = """Somebody once told me the world is gonna roll me I ain't the sharpest tool in the shed She was looking kind of dumb with her finger and her thumb In the shape of an "L" on her forehead"""
    #Load the hex data and clean it
    songcipher = load_data(OLD_DATA, "songdata.txt")
    songcipher = songcipher.split(",")
    songcipher = "".join(songcipher)
    #Convert ascii lyrics, and hex ciphertext to decimal
    songcipher = conv.hex_to_dec(songcipher)
    key_dec = conv.char_to_dec(KEY)
    decrypted = []

    #Setting range to the smaller list to avoid out of index errors
    for i in range(min(len(songcipher), len(key_dec))):
        output = songcipher[i] - key_dec[i]
        #Handles any negative numbers and keeps things in range
        output = output + 256 % 256
        decrypted.append(output)
    decrypted = conv.dec_to_char(decrypted)
    print(decrypted)
```
Output > ``What are you doing in my swamp? Swamp! Swamp! Swamp! Oh, dear! Whoa! All right, get out of here. All of you, move it! Come on! Let's go! The flag is swampCTF{S1mpl3_S0ng_0TP_C1ph3r}``