# **Public-Key vs Symmetric-Key Performance Study (OpenSSL Lab)**

## **Objective**

The goal of this lab is to compare the performance of public-key encryption (RSA) and symmetric-key encryption (AES) using OpenSSL.
We generate a small message, encrypt and decrypt it using RSA and AES, measure the time taken, and compare the results with OpenSSL’s built-in benchmarking tool.

---

## **1. Preparing the 16-byte Message**

We create a file containing exactly 16 bytes.

```bash
echo -n "ThisIs16ByteMsg!" > message.txt
wc -c message.txt
```

Output should be:

```
16 message.txt
```

---

## **2. Generate RSA-1024 Key Pair**

Generate the private key:

```bash
openssl genrsa -out private.pem 1024
```

Extract the public key:

```bash
openssl rsa -in private.pem -pubout -out public.pem
```

---

## **3. RSA Encryption and Decryption**

### Encrypt using public key

```bash
time openssl rsautl -encrypt -inkey public.pem -pubin -in message.txt -out message_enc.txt
```

### Decrypt using private key

```bash
time openssl rsautl -decrypt -inkey private.pem -in message_enc.txt -out message_dec.txt
```

Verify:

```bash
cat message_dec.txt
```

---

## **4. AES-128 Encryption**

Generate a random 128-bit key:

```bash
openssl rand -hex 16 > aes.key
```

Encrypt the message:

```bash
time openssl enc -aes-128-cbc -pbkdf2 -salt -in message.txt -out message_aes.enc -pass file:aes.key
```

Decrypt:

```bash
openssl enc -aes-128-cbc -pbkdf2 -d -in message_aes.enc -out message_aes.dec -pass file:aes.key
cat message_aes.dec
```

---

## **5. Performance Measurement**

To get meaningful timing results, each operation is repeated many times.

### RSA Encryption (1000 times)

```bash
time for i in {1..1000}; do
  openssl rsautl -encrypt -inkey public.pem -pubin -in message.txt -out /dev/null
done
```

### RSA Decryption (1000 times)

```bash
time for i in {1..1000}; do
  openssl rsautl -decrypt -inkey private.pem -in message_enc.txt -out /dev/null
done
```

### AES Encryption (1000 times)

```bash
time for i in {1..1000}; do
  openssl enc -aes-128-cbc -pbkdf2 -salt -in message.txt -out /dev/null -pass file:aes.key
done
```

Average time is calculated as:

```
Average = Total Time / Number of Runs
```

---

## **6. OpenSSL Speed Benchmark**

RSA benchmark:

```bash
openssl speed rsa
```

AES benchmark:

```bash
openssl speed aes
```

---

## **7. Compare Multiple Operations**

```bash
time for i in {1..1000}; do openssl rsautl -decrypt -inkey private.pem -in message_enc.txt -out /dev/null; done
time for i in {1..1000}; do  openssl enc -aes-128-cbc -pbkdf2 -salt -in message.txt -out /dev/null -pass file:aes.key; done
```

## **8. Observations**

* RSA encryption is much slower than AES.
* RSA decryption is even slower than RSA encryption.
* AES-128 encryption is extremely fast, even when repeated many thousands of times.
* OpenSSL’s `speed` command confirms that AES processes data at hundreds of MB/s, while RSA can only perform a few thousand operations per second.

---

## **9. Conclusion**

This experiment demonstrates why hybrid encryption is used in practice:

* **RSA** is used to securely exchange keys.
* **AES** is used to encrypt actual data efficiently.

Symmetric encryption (AES) is several orders of magnitude faster than public-key encryption (RSA), which makes it ideal for encrypting large amounts of data.

---
