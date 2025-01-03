# IS-Lab02-02FIE-22110065
# Task 1: Public-key based authentication 
**Question 1**: 
Two Docker containers were established: `userver` and `uclient`, creating a controlled network environment for the authentication experiment.

![alt text](image-1.png)

**Answer 1**:

## Step 1: Preparing the environment
 began by setting up two Docker containers - `userver` and `uclient`.

![image](/img/2.0.png)

The following packages were systematically installed on both machines to support the authentication process:

Using this command: `apt update && apt install openssl netcat-traditional net-tools xxd`


1. `openssl`: To use openssl
2. `netcat` (traditional): To send and receive files, specifically the public key.
3. `net-tools`: To use the command `ifconfig` to see the IP address of the machine.
4. `xxd`: To inspect hex values from binary files

*Note: While 512-bit encryption was selected for educational purposes, it is acknowledged that this key length is insufficient for production security environments.*


## Step 2: Create client's private and public keys
### 2.1. Create private key

A 512-bit RSA private key was generated using the following command:

Command: `openssl genrsa -out client_private.pem 512`

Explain the command:

- `genrsa`: Generate RSA private key.
- `client_private.pem`: The file name to save the private key in.
- `512`: The length of the private key.

The private key is saved in file named `client_private.pem`

![image](/img/2.1.png)

### 2.2. Create public key

The public key was extracted from the private key using:

```
openssl rsa -in client_private.pem -pubout -out client_public.pem
```

The above command uses the `client_private.pem` as input, and extract a public key from the private key (`-pubout`) and save the public key to `client_public.pem` (`-out client_public.pem`)

![image](/img/2.2.png)

### 2.3. Send public key to server machine

In order to send the public key `client_public.pem` to the server machine, we use `netcat`.

First, on the server, we have to know the IP address of the machine:

![image](/img/2.3.png)

First, the server have to be listening on a port (I used port 4444) using the command 

```
nc -lv -p 4444 > client_public.pem
```

![image](/img/2.4.png)

The server machine will be listening (`-l`) on port 4444 (`-p 4444`), and will print out the log (`-v`). The receive bits will be saved into `client_public.pem` on the server machine.

Then, the client can send the file using
```
nc [ip address] 4444 < client_public.pem
```
The client machine will send the file `client_public.pem` to the server's IP address, port 4444

![image](/img/2.5.png)

Now, the client_public is now on the server machine

![image](/img/2.6.png)

Result: The public key is sent successfully.

## Step 3: Encrypt the message

First, I create a common `challenge.txt` file
```
echo "Hello from the server" > challenge.txt
```

And encrypt the message using the following command
```
openssl pkeyutl -encrypt -inkey client_public.pem -pubin -in challenge.txt -out challenge.enc
```

The command will use RSA to encrypt the file `challenge.txt` using the public key from `client_public.pem`. The result will be saved in `challenge.enc`.

![image](img/2.7.png)

## Step 4: Send the encrypted message to the client.

The public key was securely transmitted between machines using `netcat`, demonstrating a fundamental principle of public-key infrastructure (PKI).

First, the client must be listening on port 4445, then the server can send the file.

![image](/img/2.8.png)

The client now have the encrypted challenge file:

![image](/img/2.9.png)


## Step 5: Decrypt the challenge

The encrypted challenge was decrypted using the client's private key:
```
openssl pkeyutl -decrypt -inkey client_private.pem -in challenge.enc -out decrypted_challenge.txt
```

The command is used to decrypt the encrypted `challenge.enc` using the client's private key (`-inkey client_private.pem`), and save the decrypted to `decrypted_challenge.txt`.
 
![image](/img/3.0.png)

## Step 6: Sign the challenge using client's private key

A digital signature was generated to authenticate the decrypted challenge:
```
openssl dgst -sha256 -sign client_private.pem -out signed_challenge.bin decrypted_challenge.txt
```

## Step 7: Send the signature back to the server

![image](/img/3.1.png)


## Step 7: Verify the signature
The signature was verified using the public key:

```
openssl dgst -sha256 -verify client_public.pem -signature signed_challenge.bin challenge.txt
```

Explain the command:

- `-sha256`: Use SHA-256 for hashing.
- `-verify client_public.pem`: Verify the signature using the public key in client_public.pem.
- `-signature signed_challenge.bin`: Specifies the received `signed_challenge.bin` as the signature file to check.
- `challenge.txt`: The file to verify.

![image](/img/3.2.png)

The file is now verifed.

**Conclusion:** The laboratory exercise successfully demonstrated the technical implementation of public-key based authentication, providing practical insights into cryptographic communication principles.





