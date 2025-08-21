The server would randomly generate a number, accept a number from user input then run ```decimal = Decimal(user_input)``` using the Python [Decimal](https://docs.python.org/3/library/decimal.html) library. The server would then check ```if random_number / decimal == 0:``` and pass the check would print a flag.
This is mathematically impossible so we go take a look at anything unusual the Decimal function can do. We see "Decimals also include special values such as `Infinity`, `-Infinity`, and `NaN`."
So if we provide the string "Infinity" instead of a number, we pass the check and get the flag.

scriptCTF{70_1nf1n17y_4nd_b3y0nd_072bbcc9b2ba}
