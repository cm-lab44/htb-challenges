28<sup>th</sup> 05 2023

3ntity1325
genghi

### Description:

A fun easy challange for beginners to become comfortable in reverse engineering encryption algorithms. The algorithm is kept simple and unobfuscated to allow for easy understanding of the algorithm which keeps the focus on reverse engineering not figuring out the encryption.

### Objective

Reverse engineer the encryption algorithm 

### Difficulty:

`EASY`

### Flag:

`HTB{TheFriendsWeMade}`

# Challenge

After downloading we find two files:

- Flag.txt
- Encrypt.py

Flag.txt:
`
The key is: InsideYouAlAlong
The flag is: HTB{0011101 0000110 0010110 0101111 0010110 0001100 0111100 0000001 0010001 0110010 0111011 0100100 0100001 0001110 0001010 0000010}
`

Encrypt.py:
```python
def ToBin(string):
    binary_string = ""
    for char in string:
        ascii_value = ord(char)
        binary_value = bin(ascii_value)[2:]
        binary_string += binary_value + " "
    return binary_string.strip()

def encode(st, key):
    st = ToBin(st)
    key = ToBin(key)
    out = []
    for i, j in zip(st, key):
        if i == ' ':
            out.append(' ')
        elif i != j:
            out.append('1')
        else:
            out.append('0')
    return ''.join(out)
```

The algorithm itself is quite simple: 
- Convert the string and key to binary
- Compare each bit of the string and key
- If the string and key bit match return 0 if they don't return 1 (XOR)
- Take all the comparison bits and return them as a string that looks like binary

To reverse engineer we write a function using the following algorithm:
- Convert the key to binary
- Compare each bit of the encrypted flag and the key
- If the flag bit is 0 return the key bit if the flag bit is 1 return the inverse of the key bit

A function like this will do the trick:

```python
def ToBin(string):
    binary_string = ""
    for char in string:
        ascii_value = ord(char)
        binary_value = bin(ascii_value)[2:]
        binary_string += binary_value + " "
    return binary_string.strip()

def decode(st, key):
    key = ToBin(key)
    out = []
    for i, j in zip(st, key):
        if i == ' ':
            out.append(' ')
        elif i == '1':
            if j == '0':
                out.append('1')
            else:
                out.append('0')
        else:
            out.append(j)
    return ''.join(out)
```

Running 0011101 0000110 0010110 0101111 0010110 0001100 0111100 0000001 0010001 0110010 0111011 0100100 0100001 0001110 0001010 0000010 through it with the key InsideYouAlAlong should yield the flag: TheFriendsWeMade

# Solver

Solver.py:
```python
def ToBin(string):
    binary_string = ""
    for char in string:
        ascii_value = ord(char)
        binary_value = bin(ascii_value)[2:]
        binary_string += binary_value + " "
    return binary_string.strip()

def ToText(binary_string):
    binary_list = binary_string.split()

    text = ""
    for binary in binary_list:
        decimal = int(binary, 2)
        character = chr(decimal)
        text += character

    return text

def GetFlagContent(FlagPath):
    #Read the file containing the flag
    f = open(FlagPath, "r", encoding='utf-8')
    content = f.read()
    #Convert it to a list of characters
    content = list(content)
    #Declare a variable to track which characters should be kept and which removed
    #to leave only the characters in the HTB{}
    cptre = False
    #Loop through the characters
    for i in range(len(content)):
        #When the { character is reached start keeping the characters
        if content[i] == '{':
            cptre = True
            content[i] = ''
        #When the } character is reached stop keeping the characters
        if content[i] == '}':
            cptre = False
        if cptre == False:
            #Remove the character by replacing it with an empty string
            content[i] = ''
    #Return the list of characters that now only feature the characters from the HTB{} as a string
    return(''.join(content))

def decode(st, key):
    #Convert the key to binary
    key = ToBin(key)
    out = []
    #Go through the flag and key bit by bit.
    for i, j in zip(st, key):
        if i == ' ':
            out.append(' ')
        #If the flag bit is 1 return the opposite of the key bit
        elif i == '1':
            if j == '0':
                out.append('1')
            else:
                out.append('0')
        #Otherwise return the key bit
        else:
            out.append(j)
    #Return the entire flag binary as a string
    return ''.join(out)

print("HTB{" + ToText(decode(GetFlagContent("Flag.txt"), "InsideYouAlAlong")) + "}")
```
