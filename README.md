# 95-702 Lab 5 Cryptographic Protocols          

## Diffie-Hellman-Merkle Key Exchange

In lecture, we discussed the famous key exchange problem. How do two parties,
operating at a distance, agree on a symmetric key? The approach we study in this
lab (a classic cryptographic protocol) was invented prior to the invention of RSA.
It was one of the very first public key approaches ever discovered.

In this lab, you will modify existing UDP-based client and server programs to
implement the Diffie-Hellman-Merkle key exchange.

1. Copy UDPClient.java and UDPServer.java. Create an IntelliJ project with a
package named "cmu.edu.ds". Paste the two classes into this package.

UDPClient.java

```
package cmu.edu.ds;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;
import java.util.Scanner;

/*
Based on Coulouris UDP socket code
 */
public class UDPClient {
    private DatagramSocket socket = null;
    private InetAddress host = null;
    private int port;

    public static void main(String[] args) {
        Scanner keyboard = new Scanner(System.in);
        System.out.print("Enter an integer: ");
        int value = Integer.parseInt(keyboard.nextLine());
        System.out.println("value = " + value);
        UDPClient udpClient = new UDPClient();
        udpClient.init("localhost", 7272);
        udpClient.send(Integer.toString(value));
        value = Integer.parseInt(udpClient.receive());
        System.out.println("Answer: " + value);
        udpClient.close();
    }

    private void init(String hostname, int portNumber) {
        try {
            host = InetAddress.getByName(hostname);
            port = portNumber;
            socket = new DatagramSocket();
        } catch (SocketException e) {
            System.out.println("Socket error " + e.getMessage());
        } catch (IOException e) {
            System.out.println("IO error " + e.getMessage());
        }
    }

    private void send(String message) {
        byte[] m = message.getBytes();
        DatagramPacket packet = new DatagramPacket(m, m.length, host, port);
        try {
            socket.send(packet);
        } catch (IOException e) {
            System.out.println("IO error " + e.getMessage());
        }
    }

    private String receive() {
        byte[] answer = new byte[256];
        DatagramPacket reply = new DatagramPacket(answer, answer.length);
        try {
            socket.receive(reply);
        } catch (IOException e) {
            System.out.println("IO error " + e.getMessage());
        }
        return(new String(reply.getData(), 0, reply.getLength()));

    }

    private void close() {
        if (socket != null) socket.close();
    }
}

```

UDPServer.java

```
package cmu.edu.ds;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.net.SocketException;

/*
Based on Coulouris UDP socket code
 */
public class UDPServer {
    private DatagramSocket socket = null;
    private InetAddress inetAddress = null;
    private int port;

    public static void main(String[] args) {
        UDPServer udpServer = new UDPServer();
        udpServer.init(7272);
        int value = Integer.parseInt(udpServer.receive());
        System.out.println("Server received: " + value);
        value++;
        String message = Integer.toString(value);
        udpServer.send(message);
        udpServer.close();
    }

    private void init(int portnumber) {
        try {
            socket = new DatagramSocket(portnumber);
            System.out.println("Server socket created");
        } catch (SocketException e) {
            System.out.println("Socket error " + e.getMessage());
        } catch (IOException e) {
            System.out.println("IO error " + e.getMessage());
        }
    }

    private void send(String message) {
        byte[] buffer = new byte[256];
        buffer = message.getBytes();
        DatagramPacket reply = new DatagramPacket(buffer, buffer.length, inetAddress, port);
        try {
            socket.send(reply);
        } catch (SocketException e) {
            System.out.println("Socket error " + e.getMessage());
        } catch (IOException e) {
            System.out.println("IO error " + e.getMessage());
        }

    }

    private String receive() {
        byte[] buffer = new byte[256];
        DatagramPacket request = new DatagramPacket(buffer, buffer.length);

        try {
            socket.receive(request);
            inetAddress = request.getAddress();
            port = request.getPort();
        } catch (SocketException e) {
            System.out.println("Socket error " + e.getMessage());
        } catch (IOException e) {
            System.out.println("IO error " + e.getMessage());
        }
        return new String(request.getData(), 0, request.getLength());
    }

    private void close() {
        if (socket != null) socket.close();
    }
}

```

Compile and run the server code first. If an exception is thrown, change the
port number and try again (and use that port number in the client). Then compile
and run the client code. Enter an integer into the running client and then verify that the client
receives back the integer that you entered + 1.

:checkered_flag:**This is the checkpoint for this lab. Show your TA that you have the client and server
running.**

2. Read the following article about the Diffie-Hellman Key Exchange algorithm:
https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange

3. Implement the algorithm by changing the client and server code. The client
will be Alice and the server will be Bob. As described in the article on the Diffie-
Hellman Key exchange, Alice and Bob will begin with a modulus p = 23 and a base g = 5.
Have Alice choose a secret random integer a between 2 and 22. Have Bob choose a
secret random integer b between 2 and 22.

Have Alice transmit g^a mod p to Bob. As a response to the transmission from Alice, Bob
replies with g^b mod p.

Both Alice and Bob compute s (the shared secret.)

On Alice's console, display the secret integer that she now shares with Bob (s). On Bob's
console, display the secret integer that he shares with Alice (s).

4. The modulus p above is too small. Change your modulus p to the following integer.
This integer is very probably prime. It is about 2,048 bits in size. Use Java's
BigInteger class.

```
294558318881405180764747479252007358319960875235150893513057100495960335262381639732
393624382991877148611640594583065379669231891214833093801938123911763243718214043283
060093720669049649181956712189051916260382176617240174711734510352477962712574583690
779486253846522009126482319144984230256476305809392243435136726060071627481596350642
241513558954925792693196456498326057846493955255568347280893811272095586783577349445
131066561096635908313303089526419052508796347391313473326110069433039169945763380273
958809155750154147725521635748917952339066093424140296680685333565455781078703656353
98276428848740477292742280559
```
The values for a and b are also too small. Use the following Java method to create
random values for a and b that are 2,046 bits in length.

public BigInteger(int numBits, Random rnd)

We can leave the value of g at 5.

:checkered_flag:**Show your TA that Alice and Bob are able to arrive at a shared secret using these very
large BigIntegers.**
