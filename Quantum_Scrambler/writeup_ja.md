## Quantum Scrambler
**結論: 元コードと逆操作を行うだけです。**((((  
という説明だとわかりにくすぎるので、ちゃんと説明していきます。

まず、サーバーにアクセスしてみましょう。
```
$ nc verbal-sleep.picoctf.net 51316
[['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54'], ['0x46', [['0x70', '0x69'], ['0x63', [], '0x6f']], '0x7b'], ['0x70', [['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54']], '0x79'], ['0x74', [['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54'],
...
[['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54']], '0x79'], ['0x74', [['0x70', '0x69'], ['0x63', [], '0x6f'], ['0x43', [['0x70', '0x69']], '0x54'], ['0x46', [['0x70', '0x69'], ['0x63', [], '0x6f']], '0x7b']], '0x68']], '0x69']], '0x65']], '0x66']], '0x39']]], ['0x7d']]
```
これが復号対象の文字列のようです。  
またソースである`quantum_scrambler.py`から、`get_flag`関数と`scramble`関数を見つけました。
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
このうち問題を解くうえで鍵になるのは、
1. **scramble関数すべて**
2. **get_flag関数の3行目~6行目**

となります。この部分の逆操作をsolverで行えば良いということです。

まずは`scramble`関数を参考に`unscramble`関数を書いていきます。
```python
def unscramble(C):
    A = C
    i = len(A) - 1
    while i >= 2:
        A[i-1].pop()                    # A[i-1].append(A[:i-2])に対応
        A.insert(i-1, A[i-2].pop())     # A[i-2] += A.pop(i-1)に対応
        i -= 1
    return A
```

さらに、2の部分を参考に`unscramble`関数で復号したものをASCIIに変換します。
```python
def hex_to_ascii(hex_list):
    origin = []

    for h in hex_list:
        if isinstance(h, list):
            h = h.pop(0)
        origin.append(chr(int(h, 16)))
    return ''.join(origin)
```
これらの実装は同ディレクトリ内の`solver.py`にまとめてありますので、実行する際はそちらを参照してください。
```
$ python solver.py
picoCTF{***********************}
```
これで、フラグを入手することができました。