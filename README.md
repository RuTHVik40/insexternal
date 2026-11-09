# insexternal

###pip install pycryptodome
###pip install cryptography
###sudo apt update
###sudo apt install openssl -y
###sudo apt update
###sudo apt install snort -y

###Caesar
def caesar(text, key, decrypt=False):
    if decrypt:
        key = -key
    result = ""
    for c in text:
        if c.isalpha():
            base = ord('A') if c.isupper() else ord('a')
            result += chr((ord(c) - base + key) % 26 + base)
        else:
            result += c
    return result
text = input("Enter plaintext: ")
key = int(input("Enter key: "))
cipher = caesar(text, key)
print("Encrypted:", cipher)
print("Decrypted:", caesar(cipher, key, True))

###Playfair
def matrix(key):
    s = ""
    for c in key.upper() + "ABCDEFGHIKLMNOPQRSTUVWXYZ":
        c = "I" if c == "J" else c
        if c not in s and c.isalpha():
            s += c
    return [s[i:i+5] for i in range(0, 25, 5)]
def prepare(text):
    text = "".join(
        c for c in text.upper() if c.isalpha()
    ).replace("J", "I")
    res = ""
    i = 0
    while i < len(text):
        a = text[i]
        b = text[i+1] if i+1 < len(text) else "X"
        res += a + ("X" if a == b else b)
        i += 1 if a == b else 2
    return res
def playfair(text, key, decrypt=False):
    m = matrix(key)
    pos = {
        m[r][c]: (r, c)
        for r in range(5)
        for c in range(5)
    }
    text = prepare(text)
    out = ""
    shift = -1 if decrypt else 1
    for i in range(0, len(text), 2):
        a, b = text[i], text[i+1]
        r1, c1 = pos[a]
        r2, c2 = pos[b]
        if r1 == r2:
            out += m[r1][(c1 + shift) % 5]
            out += m[r2][(c2 + shift) % 5]
        elif c1 == c2:
            out += m[(r1 + shift) % 5][c1]
            out += m[(r2 + shift) % 5][c2]
        else:
            out += m[r1][c2] + m[r2][c1]
    return out
key = input("Enter key: ")
text = input("Enter plaintext: ")
cipher = playfair(text, key)
print("Encrypted:", cipher)
print("Decrypted:", playfair(cipher, key, True))

###Hill Cipher
import numpy as np
def hill(text, key):
    text = "".join(
        c for c in text.upper() if c.isalpha()
    )
    while len(text) % 3:
        text += "X"
    nums = [ord(c) - 65 for c in text]
    result = ""
    for i in range(0, len(nums), 3):
        block = np.array(nums[i:i+3])
        result += "".join(
            chr(x % 26 + 65)
            for x in key.dot(block)
        )
    return result
key = np.array([
    [6, 24, 1],
    [13, 16, 10],
    [20, 17, 15]
])
text = input("Enter plaintext: ")
cipher = hill(text, key)
print("Encrypted:", cipher)

###Rail fence 
def encrypt(text, key):
    rail = [['\n' for _ in range(len(text))] for _ in range(key)]
    row, col = 0, 0
    direction_down = False
    for ch in text:
        if row == 0 or row == key - 1:
            direction_down = not direction_down
        rail[row][col] = ch
        col += 1
        row += 1 if direction_down else -1
    cipher = ""
    for i in range(key):
        for j in range(len(text)):
            if rail[i][j] != '\n':
                cipher += rail[i][j]
    return cipher
def decrypt(cipher, key):
    rail = [['\n' for _ in range(len(cipher))] for _ in range(key)]
    row, col = 0, 0
    direction_down = False
    for _ in range(len(cipher)):
        if row == 0 or row == key - 1:
            direction_down = not direction_down
        rail[row][col] = '*'
        col += 1
        row += 1 if direction_down else -1
    index = 0
    for i in range(key):
        for j in range(len(cipher)):
            if rail[i][j] == '*' and index < len(cipher):
                rail[i][j] = cipher[index]
                index += 1
    result = ""
    row, col = 0, 0
    direction_down = False
    for _ in range(len(cipher)):
        if row == 0 or row == key - 1:
            direction_down = not direction_down
        result += rail[row][col]
        col += 1
        row += 1 if direction_down else -1
    return result
text = input("Enter plaintext: ")
key = int(input("Enter number of rails: "))
cipher = encrypt(text, key)
print("Encrypted Text:", cipher)
plain = decrypt(cipher, key)
print("Decrypted Text:", plain)

###Row Transpostion 
import math
def get_order(key):
    return sorted(range(len(key)), key=lambda k: key[k])
def encrypt(text, key):
    cols = len(key)
    rows = math.ceil(len(text) / cols)
    text += 'X' * (rows * cols - len(text))
    matrix = [list(text[i:i+cols]) for i in range(0, len(text), cols)]
    order = get_order(key)
    cipher = ""
    for c in order:
        for r in range(rows):
            cipher += matrix[r][c]
    return cipher
def decrypt(cipher, key):
    cols = len(key)
    rows = math.ceil(len(cipher) / cols)
    order = get_order(key)
    matrix = [['' for _ in range(cols)] for _ in range(rows)]
    index = 0
    for c in order:
        for r in range(rows):
            matrix[r][c] = cipher[index]
            index += 1
    plain = ""
    for r in matrix:
        plain += "".join(r)
    return plain.rstrip('X')
text = input("Enter plaintext: ")
key = input("Enter key: ")
cipher = encrypt(text, key)
print("Encrypted:", cipher)
plain = decrypt(cipher, key)
print("Decrypted:", plain)

###Columnar transposition
import math
def get_key_order(key):
    return sorted(range(len(key)), key=lambda k: key[k])
def encrypt(plaintext, key):
    plaintext = plaintext.replace(" ", "").upper()
    cols = len(key)
    rows = math.ceil(len(plaintext) / cols)
    while len(plaintext) < rows * cols:
        plaintext += "X"
    matrix = []
    index = 0
    for r in range(rows):
        row = []
        for c in range(cols):
            row.append(plaintext[index])
            index += 1
        matrix.append(row)
    order = get_key_order(key)
    ciphertext = ""
    for col in order:
        for row in matrix:
            ciphertext += row[col]
    return ciphertext
def decrypt(ciphertext, key):
    cols = len(key)
    rows = math.ceil(len(ciphertext) / cols)
    order = get_key_order(key)
    matrix = [['' for _ in range(cols)] for _ in range(rows)]
    index = 0
    for col in order:
        for row in range(rows):
            matrix[row][col] = ciphertext[index]
            index += 1
    plaintext = ""
    for row in matrix:
        plaintext += "".join(row)
    return plaintext.rstrip("X")
text = "HELLO WORLD"
key = "HACK"
cipher = encrypt(text, key)
plain = decrypt(cipher, key)
print("Plaintext :", text)
print("Key       :", key)
print("Ciphertext:", cipher)
print("Decrypted :", plain)

###DES
from Crypto.Cipher import DES
from Crypto.Util.Padding import pad, unpad
key = b"12345678"
cipher = DES.new(key, DES.MODE_ECB)
plaintext = "Hello DES"
encrypted = cipher.encrypt(pad(plaintext.encode(), DES.block_size))
decipher = DES.new(key, DES.MODE_ECB)
decrypted = unpad(decipher.decrypt(encrypted), DES.block_size)
print("Encrypted:", encrypted.hex())
print("Decrypted:", decrypted.decode())

###AES
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os
key = AESGCM.generate_key(bit_length=256)
aes = AESGCM(key)
nonce = os.urandom(12)
data = b"Hello, AES!"
ciphertext = aes.encrypt(nonce, data, None)
plaintext = aes.decrypt(nonce, ciphertext, None)
print("Cipher Text:", ciphertext)
print("Decrypted Text:", plaintext)

###rsa
from math import gcd
p, q = 3, 11
n = p * q
phi = (p - 1) * (q - 1)
e = 3
while gcd(e, phi) != 1:
    e += 2
d = pow(e, -1, phi)
m = int(input("Enter message (number): "))
c = pow(m, e, n)
print("Encrypted:", c)
decrypted = pow(c, d, n)
print("Decrypted:", decrypted)

###diffie helman ket exchange 
p = 23
g = 5
a = int(input("Enter Alice's private key: "))
b = int(input("Enter Bob's private key: "))
A = pow(g, a, p)
B = pow(g, b, p)
key_alice = pow(B, a, p)
key_bob = pow(A, b, p)
print("Alice's public key:", A)
print("Bob's public key:", B)
print("Shared secret:", key_alice)

###sha
import hashlib
def sha256_hash(text):
    h = hashlib.sha256()
    h.update(text.encode('utf-8'))
    return h.hexdigest()
text = input("Enter message: ")
hash_value = sha256_hash(text)
print("Original Message:", text)
print("SHA-256 Hash:", hash_value)
print("Hash Length:", len(hash_value))
print("Hash in Bytes:", hashlib.sha256(text.encode()).digest())
print("Verification:", sha256_hash(text) == hash_value)

###dsa
from cryptography.hazmat.primitives.asymmetric import dsa
from cryptography.hazmat.primitives import hashes
from cryptography.exceptions import InvalidSignature
private_key = dsa.generate_private_key(key_size=2048)
public_key = private_key.public_key()
message = input("Enter message: ").encode()
signature = private_key.sign(message, hashes.SHA256())
print("Digital Signature:", signature.hex())
try:
    public_key.verify(signature, message, hashes.SHA256())
    print("Signature Verified: Valid")
except InvalidSignature:
    print("Signature Verified: Invalid")

###ssl/tls
####Certificate Generation Command
openssl req -x509 -newkey rsa:2048 -keyout server.key -out server.crt -days 365 -nodes
####server
import socket
import ssl
context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain("server.crt", "server.key")
server = socket.socket()
server.bind(("localhost", 12345))
server.listen(1)
with context.wrap_socket(server, server_side=True) as ssock:
    conn, addr = ssock.accept()
    print("Secure connection established")
    data = conn.recv(1024).decode()
    print("Client:", data)
    conn.send(b"Hello from secure server")
    conn.close()
####client
import socket
import ssl
context = ssl.create_default_context()
context.check_hostname = False
context.verify_mode = ssl.CERT_NONE
with socket.create_connection(("localhost", 12345)) as sock:
    with context.wrap_socket(sock, server_hostname="localhost") as ssock:
        ssock.send(b"Hello from secure client")
        print("Server:", ssock.recv(1024).decode())

###snort
####verify snort
snort -V
####nano /etc/snort/rules/local.rules
alert icmp any any -> any any (msg:"ICMP Packet Detected"; sid:1000001; rev:1;)
####generate traffic
ping 127.0.0.1
####snort running
sudo snort -A console -q -c /etc/snort/snort.conf -i lo
