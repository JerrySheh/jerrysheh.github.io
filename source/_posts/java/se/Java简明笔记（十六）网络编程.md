---
title: Java简明笔记（十六）网络编程
comments: true
categories:
- Java
- Java SE
tags: Java
abbrlink: bbbc0df0
date: 2019-01-10 18:01:15
---

![socket_title](../../../../images/Java/socket_title.jpg)

# 前言

网络编程，顾名思义就是编写通过网络通信的计算机程序。提到网络编程，一般指 socket 编程，之前我写过两篇相关的文章，分别是：[浅谈 socket 编程](../post/bfa70c14.html) 和 [Socket编程实践（Java & Python实现）](../post/78265215.html)，主要侧重于 socket 编程的理解，而这一篇侧重于使用 Java 进行 socket 编程的要点，作为简明笔记，以备后续用到时方便查阅。

<!-- more -->

# 使用Java进行 TCP socket 编程

## 服务器

socket编程涉及到多台计算机的连接，一般我们习惯分为服务器和客户端以区分服务提供者和使用者。服务器端首先要建立起一个socket连接服务，之后等待客户端来连接，当连接建立后，两方都用输入输出流来发送和接收数据，就像双向流水管道一样自然。

### 如何建立连接

对于服务器来说，要创建一个服务socket非常简单：

```java
int SERVER_PORT = 7706;
ServerSocket serverSocket = new ServerSocket(SERVER_PORT);
```

这样就创建了一个服务socket，之后用accept方法开始等待客户端来连接：

```java
Socket connectSocket = serverSocket.accept();
```

在没有客户端连接时，accept方法是一直阻塞的。直到有一个客户端进行连接，connectSocket对象才生成。<font color="red">注意，这里的connectSocket跟刚刚的serverSocket是两个概念，serverSocket用来欢迎并等待客户端的连接，connectSocket则是专门服务于某个连接的客户端的。</font>

连接建立后，我们得到了一个Socket对象（不是ServerSocket对象）。之后我们开一个线程，专门处理与这个客户端的数据传输。

```java
while (true){
    Socket connectSocket = serverSocket.accept();
    new sender(connectSocket).start();
}
```

而主线程的ServerSocket对象则在 while 循环内继续等待欢迎其他客户端来连接。

server.java 完整代码：

```java
public class server {

    // 端口号
    private static final int SERVER_PORT = 7767;

    public static void main(String[] args) throws IOException {

        boolean STOP = false;

        // 初始化ServerSocket服务
        ServerSocket serverSocket = new ServerSocket(SERVER_PORT);
        System.out.println("服务已启动，监听端口：" + SERVER_PORT);

        // 不断监听，欢迎客户端来连接
        while (!STOP){
            // accept方法是阻塞的，一旦有客户端连接，Socket对象才生成
            Socket connectSocket = serverSocket.accept();

            // 创建一个新的线程来服务特定的客户端
            // sender类是我们自定义的用于处理数据的类，继承于 Thread 类，下文介绍
            new sender(connectSocket).start();
        }
    }
}
```

### 如何发送和接收数据

接下来关心一下，sender类是如何处理数据的。首先，sender构造时传进了一个 connectSocket 对象，然后在线程的 run 方法里面编写如何跟客户端交互的代码：

```java
public class sender extends Thread {

    private Socket connectSocket;

    // 构造方法，从主线程传入一个 Socket 对象
    public sender(Socket connectSocket) {
        this.connectSocket = connectSocket;
    }

    // 在子线程进行处理
    @Override
    public void run(){
        // do something
    }
}
```

首先，用`connectSocket.getInputStream()`获取来自客户端的输入流，然后封装到 DataInputStream 对象中，最后用`readUTF()`方法来读取。输出流也是同理：

```java
@Override
public void run(){

    System.out.println("远程计算机 " + connectSocket.getRemoteSocketAddress() + "已连接");

    try {
        // 封装接收数据流
        InputStream in = connectSocket.getInputStream();
        DataInputStream dataIn = new DataInputStream(in);

        // 封装发送数据流
        OutputStream out = connectSocket.getOutputStream()
        DataOutputStream dataOut = new DataOutputStream(out);

        // while 循环用于不断接收和发送数据
        while (true){

            // 接收，该方法阻塞直到有数据接收
            String recv = dataIn.readUTF();
            System.out.println("成功接收来自 " + connectSocket.getRemoteSocketAddress() + "的数据：" + recv);

            // 数据处理
            String send = recv.toUpperCase();

            // 发送
            dataOut.writeUTF(send);
            System.out.println("向" + connectSocket.getRemoteSocketAddress() + "发送转换后的数据：" + send);

        }
    } catch (IOException e) {
        System.out.println("远程计算机 " + connectSocket.getRemoteSocketAddress() + "已断开");
    }

}
```

## 客户端

客户端相对来说就简单一些。先创建一个 Socket 对象，然后用同样的方法封装到输入流和输出流中进行处理即可。

创建socket对象

```java
// 创建Socket对象时需指定服务器的地址和端口
String serverNmae = "192.168.1.187";
Socket client = new Socket(serverName, 7767);
```

封装输入输出流

```java
// 封装输出流
DataOutputStream dataOut = new DataOutputStream(client.getOutputStream());
// 封装输入流
DataInputStream dataIn = new DataInputStream(client.getInputStream());
```

数据发送和接收

```java
// 发送数据
dataOut.writeUTF(send);
System.out.println("向服务器 " + client.getRemoteSocketAddress() + "发送了：" + send + "\n");

String recv = dataIn.readUTF();
System.out.println("接收来自服务器的数据：" + recv + '\n');
```

可以使用 Scanner 从键盘多次获取输入。

client.java 完整代码：

```java
public class client {

    public static void main(String[] args)  {

        // 从键盘获取服务器地址
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入服务器地址，如 192.168.1.1");
        String serverName = scanner.nextLine();

        try {
            // 创建 Socket 对象
            Socket client = new Socket(serverName, 7767);

            // 封装输出流
            DataOutputStream dataOut = new DataOutputStream(client.getOutputStream());
            // 封装输入流
            DataInputStream dataIn = new DataInputStream(client.getInputStream());

            // 循环发送和接收数据
            while (true){
                // 控制台循环获取输入
                System.out.println("\n请输入要发送的数据：(输入exit退出)");
                String send = scanner.next();

                if ("exit".equals(send)) break;

                // 发送数据
                dataOut.writeUTF(send);
                System.out.println("向服务器 " + client.getRemoteSocketAddress() + "发送了：" + send);

                // 接收数据
                String recv = dataIn.readUTF();
                System.out.println("接收来自服务器的数据：" + recv + '\n');
            }

        } catch (IOException e){
            System.out.println("服务器连接中断");
        }

    }
}
```


# 使用Java进行 UDP socket 编程

## 发送端

UDP socket 无需建立连接，但是在发送数据前需要先准备数据报包（packet）。类似于TCP里面我们把输入流封装到 DataInputStream 里，在 UDP 中我们把数据放在数据报包 DatagramPacket 当中，数据报包是载体。65508是每个数据报包可以容纳的最大数据量。

```java
// 数据
byte[] buffer = new byte[65508];

// 要发送的地址
InetAddress address = InetAddress.getByName("192.168.1.187");

// 数据报包里面存储包括数据内容、长度、要发送的地址和对方端口
DatagramPacket packet = new DatagramPacket(
    buffer, buffer.length, address, 9797);
```

有了数据报包之后，我们用 datagramSocket 对象来发送数据。<font color="red">可以这样理解，DatagramPacket 是包裹，用来装数据，而 datagramSocket 是车，用来把包裹运送出去。</font>

```java
// 准备车
DatagramSocket datagramSocket = new DatagramSocket();

// 将数据报包裹装车，并运送出去
datagramSocket.send(packet);
```

完整代码：

```java
/**
 * 客户端，发数据
 */
public class client {

    public static void main(String[] args) throws UnknownHostException, SocketException {

        // 要发送的数据
        byte[] buffer = "0123456789".getBytes();

        // 对方地址
        InetAddress address = InetAddress.getByName("192.168.1.187");

        // 准备包裹
        DatagramPacket packet = new DatagramPacket(buffer, buffer.length, address, 9797);

        // 准备车
        DatagramSocket datagramSocket = new DatagramSocket();

        try {
            // 运送出去
            datagramSocket.send(packet);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}
```

## 接收端

接收端一开始需要先创建一个 datagramSocket 对象，用来准备接收数据：

```java
// 准备卸货车，记下对方包裹的端口
DatagramSocket datagramSocket = new DatagramSocket(9797);
```

然后声明即将接收的数据报包格式：

```java
// 要接收的包裹长什么样
byte[] buffer = new byte[10];
DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
```

接收：

```java
// 接收包裹
datagramSocket.receive(packet);

// 把接收的内容显示出来
byte[] recv = packet.getData();
String s = new String(recv);
System.out.println("接收到：" + s);
```

完整代码：

```java
/**
 * 服务端，收数据
 */
public class server {

    public static void main(String[] args) throws SocketException {

        // 准备卸货车
        DatagramSocket datagramSocket = new DatagramSocket(9797);

        // 即将接收的包裹特征
        byte[] buffer = new byte[10];
        DatagramPacket packet = new DatagramPacket(buffer, buffer.length);

        try {
            // 接收包裹
            datagramSocket.receive(packet);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 显示包裹内容
        byte[] recv = packet.getData();
        String s = new String(recv);
        System.out.println("接收到：" + s);
    }

}
```
