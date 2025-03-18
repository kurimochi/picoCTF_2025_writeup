## Tap into Hash

The `block_chain.py` contains the encryption program, and the `enc_flag` contains what appears to be the encryption key and the byte sequence to be decrypted.

Now, let's decipher `block_chain.py`. First, let's look at the `main` function... But most of the `main` function is code for attaching other data to the blockchain, and the actual encryption part is in the `encrypt` function, as you can see in the last sentence (`encrypted_blockchain = encrypt(blockchain_string, token, key)`). As you can see, the actual encryption part is in the `encrypt` function. Therefore, let's start with the `encrypt` function.

```python
def encrypt(plaintext, inner_txt, key):
    midpoint = len(plaintext) // 2

    first_part = plaintext[:midpoint]
    second_part = plaintext[midpoint:]
    modified_plaintext = first_part + inner_txt + second_part
    block_size = 16
    plaintext = pad(modified_plaintext, block_size)
    key_hash = hashlib.sha256(key).digest()

    ciphertext = b''

    for i in range(0, len(plaintext), block_size):
        block = plaintext[i:i + block_size]
        cipher_block = xor_bytes(block, key_hash)
        ciphertext += cipher_block

    return ciphertext
```

The points of interest here are as follows.

1. `key_hash = hashlib.sha256(key).digest()` -> *key is used through SHA256*
1. lines 13~16 of `encrypt` function -> *key is applied to xor with key_hash in block units*
1. `block_size = 16` -> *block size is 16 bytes*

The `xor_bytes` function used in 2 looks like it could be diverted as is, so we will write a solver here.

```python
import hashlib

def decrypt(key, encrypted_blockchain):
    # Initial value setting and calculation of key_hash
    block_size = 16
    key_hash = hashlib.sha256(key).digest()
    blockchain = b''

    # Apply to xor in blocks
    for i in range(0, len(encrypted_blockchain), block_size):
        encrypted_block = encrypted_blockchain[i:i + block_size]
        origin_block = xor_bytes(encrypted_block, key_hash)
        blockchain += origin_block

    return blockchain.decode()

# Adapted from block_chain.py
def xor_bytes(a, b):
    return bytes(x ^ y for x, y in zip(a, b))


key = b'\x8e\xdc\x08\xb8S\xee6\x0c\xf5\xfd\xceP\x15\xbf\xf6\xe2\x90\xf3\xd7F?,!\x1c\xb0D\x0cO\xcc\x04q\xb8'
encrypted_blockchain = b"\xb4\xc8\xbd\xec@A\xbd-\x1d\xfd\x16\xe1\xe3sW\x18\xb1\x99\xea\xb8\x15\x10\xe8{\x19\xacE\xb3\xb4w\nH\xe7\x9d\xea\xe9EC\xb9(N\xa8\x14\xe1\xb7t]\x1c\xb7\xc8\xb9\xbaAF\xea}L\xadF\xb4\xb1&\n\x19\xac\xcc\xbc\xedGI\xef~\x16\xab\x10\xb6\xb7'\x0cN\xe0\xca\xee\xba\x15D\xbcz\x17\xfa\x17\xe3\xe0sY\x14\xe5\xca\xb5\xecAB\xea{\x1e\xffD\xe4\xb0%\x0cO\xb0\xc9\xee\xefC\x12\xeb|N\xff\x16\xe8\xb4'[\x15\xe0\xd1\xbc\xec\x10F\xe8q\x17\xa8\x10\xe1\xedu\\N\xb6\xc5\xef\xba\x10@\xe8y\x1a\xadC\xe3\xb0p\nO\xb0\xce\xfc\xb5\x15\x1e\xcd\x1di\xb5B\xbc\xba \x05s\xb2\xaf\xde\xb4 \x18\xdc+{\xffQ\xb3\x8d\x1c6y\xeb\xb1\xbc\xaeBH\xed\x01p\xbfc\xaa\xb8\t4V\xc3\xb7\xd3\xe8G\x12\xbfy\x1c\xfd\x11\xad\xe4r\\\x1f\xb5\xc8\xbf\xeeCC\xefy\x18\xadC\xb1\xe3uXM\xe7\xc4\xe9\xeb\x17\x12\xeb{\x1b\xf7\x15\xb6\xf8s^\x1c\xe7\xca\xba\xe5\x10A\xb8-\x18\xffA\xe4\xe7u]J\xb1\xcf\xb9\xef\x12C\xbd+L\xfd\x19\xe6\xed'XN\xe0\xcf\xe9\xef\x17@\xbczI\xa8\x18\xb1\xe1vV\x1d\xe5\xcb\xbf\xec\x12\x15\xec|L\xabD\xb4\xb0n^\x1c\xb8\x98\xea\xea\x10A\xec/\x1c\xfa\x17\xb6\xecp\x08J\xb9\xc4\xed\xeb\x10\x17\xb6+\x1c\xf6\x11\xe5\xe0s\x0bI\xe3\xca\xb9\xbaGH\xb6}\x18\xf7A\xe0\xe5{X\x1f\xb4\x99\xe9\xbe@H\xbf*I\xfd\x14\xb6\xe7 l."

origin_blockchain = decrypt(key, encrypted_blockchain)
print(origin_blockchain)
```
(I put `solver.py` in the same directory, so please refer to it for actual operation.)
```
$ python solver.py
5410603d236160940efdcaf26beca4ddfaf5327aaf41b730645f77d4ccfdded5-00118a79e0fbdbba6bfc52384735078d69073d211d4efbc15b35ce5a168ad59a-00f7f88f01862b79cff1f05cc3e3dc12picoCTF{*************************************************}1123443252a07cca666af8e7ace2495f-000f669f06d71a4263f0353d23bc3968d6ba3e3a123ff8a4581d730ddb5cedde-009df6f0bf347f93ff88a7ff8b381550eeb65f198479a008635eeb691cf34f2c
```
We have not removed any other data, but we have obtained the flags.