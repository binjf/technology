服务器端实现步骤：

\1. 创建 DatagramSocket，指定端口号

\2. 创建 DatagramPacket

\3. 接收客户端发送的数据信息

\4. 读取数据

```
package cn.jmu.edu;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

/**
 * 服务器端。实现基于UDP的用户登陆
 *
 * @author Sky
 * @date 2016年10月28日
 */
public class UDPServer {
    public static void main(String[] args) throws IOException {
        /*
         * 接收客户端发送的信息
         */
        //1.创建服务器端 DatagramSocket，并指定端口
        DatagramSocket socket = new DatagramSocket(8800);
        //2.创建数据报，用于接收客户发送的数据
        byte[] data = new byte[1024]; //创建字节数组，指定接收的数据包的大小
        DatagramPacket packet = new DatagramPacket(data, data.length);
        //3.接收客户端发送的数据(保存在packet中)
        System.out.println("***服务器端已经启动，等待客户端发送数据***");
        socket.receive(packet);
        //4.读取数据
        String info = new String(data, 0, packet.getLength());
        System.out.println("我是服务器，客户端说：" + info);
        
        /*
         * 向客户端响应数据
         */
        //1.定义客户端的地址、端口号、数据
        InetAddress address = packet.getAddress();
        int port = packet.getPort();
        byte[] data2 = "欢迎您！".getBytes();
        //2.创建数据报，包含响应的数据信息
        DatagramPacket packet2 = new DatagramPacket(data2, data2.length, address, port);
        //3.响应客户端
        socket.send(packet2);
        //4.关闭资源
        socket.close();
    }
}
```

 

客户端实现步骤：

\1. 定义发送的信息

\2. 创建 DatagramPacket，包含将要发送的信息

\3. 创建 DatagramSocket(用来发送数据)

\4. 发送信息

```
package cn.jmu.edu;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

/**
 * UDP客户端类
 *
 * @author Sky
 * @date 2016年10月28日
 */
public class UDPClient {
    public static void main(String[] args) throws IOException {
        /*
         * 向服务器端发送数据 
         */
        //1.定义服务器的地址、端口号、数据
        InetAddress address = InetAddress.getByName("localhost");
        int port = 8800;
        byte[] data = "用户名：Sky;密码：winner".getBytes();
        //2.创建数据报，包含发送的数据信息
        DatagramPacket packet = new DatagramPacket(data, data.length, address, port);
        //3.创建 DatagramSocket 对象(实现数据的发送)
        DatagramSocket socket = new DatagramSocket();
        //4.向服务器端发送数据报
        socket.send(packet);
        
        /*
         * 接收服务器端响应的数据
         */
        //1.创建数据报，用于接收 服务器端响应的数据
        byte[] data2 = new byte[1024];
        DatagramPacket packet2 = new DatagramPacket(data2, data2.length);
        //2.接收服务器响应的数据
        socket.receive(packet2);
        //3.读取数据
        String reply = new String(data2, 0, packet2.getLength());
        System.out.println("我是客户端，服务器端响应：" + reply);
        //4.关闭资源
        socket.close();
    }
}
```

 

运行结果：

![img](http://images2015.cnblogs.com/blog/930246/201610/930246-20161028164720625-576080230.png)

![img](http://images2015.cnblogs.com/blog/930246/201610/930246-20161028164731968-2107822012.png)

 