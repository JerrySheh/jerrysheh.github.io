---
title: Socket编程实践（Java & Python实现）
categories: 计算机网络
tags:
  - 计算机网络
  - Python
abbrlink: '78265215'
date: 2017-11-29 00:01:36
---

之前在[浅谈 Socket 编程](../post/bfa70c14.html)一篇中，初步使用 Socket 套接字，在 linux 下用 C 实现了一个客户端/服务器 通信的例子。但是由于C语言比较隐晦难懂，很多自定义的数据结构也偏向于底层，导致我对于 TCP/UDP Socket 的过程理解不深。这一篇，主要站在应用的角度，用实际的例子，补充实践 Socket 通信的过程。一开始为了更好地理解，使用了 Python 语言。**2019年1月9日闲来无事，用java重写了一遍，感受对比一下 java 的繁琐和 python 的简洁**（真是无趣的家伙）。

- 参考书籍：《计算机网络：自顶向下方法》（第2章）。

---

<!-- more -->

# TCP Socket的过程

## 1. TCP连接

TCP是面向连接的协议。客户端和服务器在发送数据之前，必须先握手和创建一个TCP连接。

一个TCP连接模型如下：

客户端应用程序 <---> `客户端Socket` <---> TCP连接 <--->  `服务器Socket` <--->服务器应用程序


## 2. 欢迎之门和连接之门

TCP连接中，客户端需要首先向服务器发起接触。也就是说，服务器必须提前准备好（即服务器应用必须先运行起来），而且，服务器必须有一扇“特殊的门”，我们可以称之为“欢迎之门”（`欢迎Socket`，ServerSocket），欢迎来自任意主机上的客户端进程来敲门。

客户端要向服务器发起连接的时候，首先创建一个TCP Socket，这个 Socket 指定了服务器中`欢迎Socket`的地址（即服务器IP和端口号）。创建完毕后，客户端即可向服务器发起三次握手并建立与服务器的TCP连接了。

在三次握手期间，客户端敲的是服务器的“欢迎之门”。当服务器听到敲门后，将生成一个新的门，这个新门就是`连接Socket`（connection Socket），专门用于特定的客户。

对于应用程序来说，`客户端Socket`和`服务器连接Socket`（注意不是欢迎Socket）直接通过一根管道连接。服务器和客户端可以互相发送或接收字节。

![Socket](../../../../images/TCPconnect.gif)

---

# Python 实现 TCP Socket 的例子

## TCPClient.py 客户端

```Python
from socket import *

serverName = "servername"
serverPort = 12000

# 初始化客户端socket
clientSocket = socket(AF_INET,SOCK_STREAM)

# 客户端socket向服务器发起连接
clientSocket.connect((serverName,serverPort))

sentence = input("Input lowercase sentence:")

# 客户端socket向服务器发送内容
clientSocket.send(sentence)

# 客户端socket接收来自服务器的内容
modifiedSentence = clientSocket.recv(1024)
print("From Server:", modifiedSentence)

# 关闭客户端socket
clientSocket.close()

```


逐行解释:

### 1. clientSocket = socket(AF_INET,SOCK_STREAM)

使用`socket()`初始化函数，创建了一个`客户端Socket`，第一个参数`AF_INET`指明底层网络使用的是IPv4，第二个参数`SOCK_STREAM`指明该Socket是SOCK_STREAM类型，也就是TCP。 `clientSocket`就是一个 Socket对象，它具有connect、send、recv等方法。

### 2. clientSocket.connect((serverName,serverPort))

前面提到，当客户端创建完一个TCP Socket之后，就可以向服务器发起三次握手并建立与服务器的TCP连接了，这一句就是连接。第一个参数`serverName`指明服务器的名字（即ip地址），第二个参数`serverPort`指明了服务器进程的端口。

### 3. sentence = input("Input lowercase sentence:")

用户输入一个句子，并存储在 `sentence` 变量中。

### 4. clientSocket.send(sentence)

clientSocket对象的`send`方法，将用户输入的句子放到TCP连接中去，交给TCP去发送。

### 5. modifiedSentence = clientSocket.recv(1024)

当字符到达服务器时，就会被放在modifiedSentence这个字符串变量中，字符持续积累，直到遇到结束符。clientSocket对象的`recv`方法，把服务器发回来的字符串放入modifiedSentence中。

### 6. clientSocket.close()

关闭Socket，关闭了客户端和服务器之间的TCP连接。


## TCPServer.py 服务器

```Python
from socket import *
serverPort = 12000

# 初始化服务器Socket
serverSocket = socket(AF_INET,SOCK_STREAM)

# 绑定端口号
serverSocket.bind(("",serverPort))

# 服务器开始监听
serverSocket.listen(1)
print("The server is ready to receive")

# 一旦收到客户端的connect，立即接受（accept）并建立连接，成立特定服务于该客户端的 connectionSocket
while True:
  connectionSocket, addr = serverSocket.accept()
  sentence = connectionSocket.recv(1024)
  capitalizedSentence = sentence.upper()

  # 服务器连接socket向客户端发送数据
  connectionSocket.send(capitalizedSentence)

  # 关闭连接socket
  connectionSocket.close()

```
逐行解释:

### 1. serverSocket = socket(AF_INET,SOCK_STREAM)

使用`socket()`初始化函数，创建了一个服务器Socket。也就是`serverSocket`，这是上文提到的欢迎Socket。

### 2. serverSocket.bind(("",serverPort))

`bind`方法绑定一个端口号。

### 3. serverSocket.listen(1)

一切准备就绪，开始聆听某个客户端来敲门。参数`1`表示最大连接客户数量为1

### 4. connectionSocket, addr = serverSocket.accept()

当有客户敲门时，服务器的欢迎Socket通过`accept()`函数创建了一个新的Socket（连接Socket），为这个特定的客户专用。客户端和服务器完成了握手，这时候，在客户端的`clientSocket`和服务器的`serverSocket`之间创建了一个TCP连接，这个TCP连接让客户端的`clientSocket`服务器的`connectionSocket`之间互传数据。

### 5. connectionSocket.close()

传输完数据后，我们关闭的是`connectionSocket`，但`serverSocket`保持打开。所以另一个客户敲门时，服务器仍继续响应。

---

# Python 实现 UDP Socket 的例子

UDP是无连接的，不可靠的数据传送服务。当使用UDP时，必须先将`目的地址`和`源地址`附在分组上面。目的地址和源地址，都包括其`IP地址`和Socket应用程序的`端口号`。

需要注意的是，将源地址附在分组上这个动作是由底层操作系统来完成的，不用我们关心。

## UDPClient.py 客户端

```Python
from socket import *
serverName = 'hostname'
serverPort = 12000

# 初始化一个客户端Socket
clientSocket = socket(AF_INET, SOCK_DGRAM)

message = input('Input lowercase sentence:')

# 客户端socket向服务器发送数据
clientSocket.sendto(message,(serverName,serverPort))

# 客户端socket接收来自服务器的数据
modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print(modifiedMessage)

# 关闭客户端socket
clientSocket.close()

```

逐行解释：

### 1. clientSocket = socket(AF_INET, SOCK_DGRAM)

使用`socket()`初始化函数，创建了一个`客户端Socket`，第一个参数`AF_INET`指明底层网络使用的是IPv4，第二个参数`SOCK_DGRAM`指明该Socket是SOCK_DGRAM类型，也就是UDP。 `clientSocket`就是一个 Socket对象，它具有connect、send、recv等方法。

<font color="red">注意，创建客户端Socket时，并没有指定`客户端的端口号`，这件事由操作系统来做。</font>

### 2. message = input('Input lowercase sentence:')

用户输入一个句子，并存储在 `sentence` 变量中。

### 3. clientSocket.sendto(message,(serverName,serverPort))

clientSocket对象的`sendto`方法，将用户输入的句子放到UDP连接中去，交给UDP去发送。第一个参数是刚刚用户输入的内容，第二个参数指定了服务器的地址和端口号。

### 4. `modifiedMessage, serverAddress = clientSocket.recvfrom(2048)`

当一个来自服务器的分组到达这个客户端Socket的时候，该分组的数据就会被放到`modifiedMessage`这个变量中，对方的源地址（包含IP和端口号）被放置到变量`serverAddress`中。事实上，在这个UDP的例子中，UDPClient并不需要服务器的地址信息，因为它一开始就已经知道了。但这行代码仍然提供了服务器的地址。

### 5. clientSocket.close()

关闭Socket，关闭了客户端和服务器之间的UDP连接。

## UDPServer.py 服务器

```Python
from socket import *
serverPort = 12000

# 初始化服务器socket
serverSocket = socket(AF_INET, SOCK_DGRAM)

# 绑定服务器端口
serverSocket.bind(('', serverPort))
print("The server is ready to receive")

# 接收来自客户端的消息，处理并发送
while True:
  message, clientAddress = serverSocket.recvfrom(2048)
  modifiedMessage = message.upper()
  serverSocket.sendto(modifiedMessage, clientAddress)

```

逐行解释：

### 1. serverSocket = socket(AF_INET, SOCK_DGRAM)

使用`socket()`初始化函数，创建了一个服务器Socket。

### 2. serverSocket.bind(('', serverPort))

`bind`方法绑定一个端口号。

### 3. message, clientAddress = serverSocket.recvfrom(2048)

当一个来自客户端的分组到达这个服务器Socket的时候，该分组的数据就会被放到`message`这个变量中，对方的源地址（包含IP和端口号）被放置到变量`clientAddress`中。使用该源地址信息，服务器就可知道接下来的应答要发往何处。

### 4. modifiedMessage = message.upper()

把接收到的数据`message`，转化成大写，并存在`modifiedMessage`这个变量中。

### 5. serverSocket.sendto(modifiedMessage, clientAddress)

erverSocket对象的`sendto`方法，将转换成大写的数据，放到UDP连接中去，交给UDP去发送。第一个参数是刚刚转换过的内容，第二个参数指定了客户端的地址和端口号。（客户端的地址和端口号在第3步就接收到了）

---

# Java 实现 TCP socket

## 服务器

```java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class server {

    // 服务器端口
    private static final int SERVER_PORT = 7767;

    public static void main(String[] args) throws IOException {

        boolean STOP = false;

        ServerSocket serverSocket = new ServerSocket(SERVER_PORT);
        System.out.println("服务已启动，监听端口：" + SERVER_PORT);
        while (!STOP){
            // accept方法是阻塞的
            Socket connectSocket = serverSocket.accept();

                try {
                    System.out.println("远程计算机 " + connectSocket.getRemoteSocketAddress() + "已连接");

                    // 从连接socket获取来自客户端的输入流
                    InputStream in = connectSocket.getInputStream();

                    // 将输入流封装到数据流中（方便后续操作字符串）
                    DataInputStream dataIn = new DataInputStream(in);

                    // 从数据流中读UTF
                    String recv = dataIn.readUTF();
                    System.out.println("成功接收来自 " + connectSocket.getRemoteSocketAddress() + "的数据：" + recv);

                    String send = recv.toUpperCase();

                    // 封装输出流
                    OutputStream out = connectSocket.getOutputStream();
                    DataOutputStream dataOut = new DataOutputStream(out);

                    // 发送
                    dataOut.writeUTF(send);
                    System.out.println("向" + connectSocket.getRemoteSocketAddress() + "发送转换后的数据：" + send);

                    // 关闭资源
                    dataIn.close();
                    dataOut.close();
                    connectSocket.close();

                } catch (IOException e) {
                    e.printStackTrace();
                }
        }
    }
}
```

## 客户端

```java
import java.io.*;
import java.net.Socket;
import java.util.Scanner;

public class client {

    public static void main(String[] args) throws IOException {

        // 设定服务器地址
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入服务器地址，如 192.168.1.1");
        String serverName = scanner.nextLine();

        Socket client = new Socket(serverName, 7767);

        // 封装输出流
        OutputStream out = client.getOutputStream();
        DataOutputStream dataOut = new DataOutputStream(out);

        // 控制台获取输入
        System.out.println("\n请输入要发送的数据：");
        String send = scanner.next();

        // 发送数据
        dataOut.writeUTF(send);
        System.out.println("向服务器 " + client.getRemoteSocketAddress() + "发送了：" + send);

        // 接收数据
        InputStream in = client.getInputStream();
        DataInputStream dataIn = new DataInputStream(in);
        String recv = dataIn.readUTF();
        System.out.println("接收来自服务器的数据：" + recv + '\n');

        // 关闭资源
        dataIn.close();
        dataOut.close();
        client.close();
        scanner.close();
    }

}
```

Java 的 UDP socket ？ 下次再说吧。
