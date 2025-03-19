## Quantum Scrambler
First, access the server.
```
$ nc verbal-sleep.picoctf.net 51316
[['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54'], ['0x46', [['0x70', '0x69'], ['0x63', [], '0x6f']], '0x7b'], ['0x70', [['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54']], '0x79'], ['0x74', [['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54'],
...
[['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54']], '0x79'], ['0x74', [['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54'], ['0x46', [['0x70', '0x69'], ['0x63', [], '0x6f']], '0x7b']], '0x68']], '0x69']], '0x65']], '0x66']], '0x39']]], ['0x7d']]
```
This appears to be the string to be decrypted.  
I also found the `get_flag` and `scramble` functions in the source, `quantum_scrambler.py`.
```python
def scramble(L):
  A = L
  i = 2
  while (i < len(A)):
    A[i-2] += A.pop(i-1)
    A[i-1].append(A[:i-2])
    i += 1
    
  return L

def get_flag():
  flag = open('flag.txt', 'r').read()
  flag = flag.strip()
  hex_flag = []
  for c in flag:
    hex_flag.append([str(hex(ord(c)))])

  return hex_flag
```
Of these, the key to solving the problem is as follows
1. all of the scramble functions
2. lines 3 to 6 of the get_flag function

The key to solving this problem is to use solver to perform the reverse operation of this part.

First, we will write an `unscramble` function with reference to the `scramble` function.
```python
def unscramble(C):
    A = C
    i = len(A) - 1
    while i >= 2:
        A[i-1].pop()                    # Corresponds to A[i-1].append(A[:i-2])
        A.insert(i-1, A[i-2].pop())     # Corresponds to A[i-2] += A.pop(i-1)
        i -= 1
    return A
```
In addition, convert decrypted with the `unscramble` function to ascii, referring to part 2.
```python
def hex_to_ascii(hex_list):
    return ''.join([chr(int(h.pop(0), 16)) if isinstance(h, list) else chr(int(h, 16))
                    for h in hex_list])
```
These implementations are summarized in `solver.py` in the same directory, so please refer to it when executing.
```
$ python solver.py
picoCTF{***********************}
```
Now you have obtained the flag.