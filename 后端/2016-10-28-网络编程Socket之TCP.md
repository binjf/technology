![img](http://images2015.cnblogs.com/blog/930246/201610/930246-20161028152253468-1007865536.png)       ![img](http://images2015.cnblogs.com/blog/930246/201610/930246-20161028152321921-1165080142.png)

服务端：

\1. 创建 ServerSocket 对象并监听一个端口

\2. 调用accept()方法等待客户端的连接(阻塞式)

\3. 输入流(记取客户端发送过来的数据)

\4. 输出流(响应客户端请求，即向客户端发送数据)

\5. 关闭资源

```
package cn.jmu.edu;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * 基于TCP 协议的Socket 通信，实现用户登陆
 * 服务器端
 *
 * @author Sky
 * @date 2016年10月27日
 */
public class Server {
    public static void main(String[] args) {
        try {    
            //1.创建一个服务端Socket，即ServerSocket，指定端口并监听此端口
            ServerSocket serverSocket = new ServerSocket(8888);
            System.out.println("***服务器即将启动，等待客户端的连接***");
            //2.调用accept方法开始监听，等待客户端的连接
            Socket socket = serverSocket.accept();
            
            //3.获取输入流，并读取用户信息
            InputStream is = socket.getInputStream(); //获取字节流入流
            InputStreamReader isr = new InputStreamReader(is); //将字节输入流转化为字符输入流
            BufferedReader br = new BufferedReader(isr); //为字符输入流添加缓冲
            String info = null;
            while((info=br.readLine()) != null){ //循环读取客户端发送的信息
                System.out.println("我是服务器，客户端说：" + info);
            }
            socket.shutdownInput(); //关闭输入流
            
            //4.获取输出流，响应客户端的请求
            OutputStream os = socket.getOutputStream();
            PrintWriter pw = new PrintWriter(os); //打包为打印流
            pw.write("欢迎您！");
            pw.flush(); //刷新缓冲区的数据，将数据输出
            
            //5.关闭资源
            pw.close();
            os.close();
            br.close();
            isr.close();
            is.close();
            socket.close();
            serverSocket.close();        
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

客户端：

\1. 创建 Socket 对象，传入服务器端的 IP 和监听的端口参数

\2. 输出流(给服务器端发送数据)

\3. 输入流(接收服务器端返回的信息)

\4. 关闭资源

```
package cn.jmu.edu;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.Socket;

/**
 * 客户端
 *
 * @author Sky
 * @date 2016年10月27日
 */
public class Client {
    public static void main(String[] args) {
        try {
            //1.创建客户端Socket，指定服务器地址和端口
            Socket socket = new Socket("localhost", 8888);
            //2.获取输出流，向服务器端发送信息
            OutputStream os = socket.getOutputStream(); //字节输出流
            PrintWriter pw = new PrintWriter(os); //将输出流打包为打印流
            pw.write("用户名：Sky;密码：winner");
            pw.flush(); //刷新数据，向服务器发送数据
            socket.shutdownOutput(); //关闭输出流
            
            //3.获取输入流，并读取服务器端的响应信息
            InputStream is = socket.getInputStream();
            BufferedReader br = new BufferedReader(new InputStreamReader(is));
            String info = null;
            while((info=br.readLine()) != null){
                System.out.println("我是客户端，服务端回答：" + info);
            }
            
            //4.关闭资源 
            br.close();
            is.close();
            pw.close();
            os.close();
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

 

上面是最基础的TCP通信编程，只能一个客户端连接一个服务器。下面改造成使用多线程实现多客户端的通信，一个服务器端有多个客户端连接。

服务端类 Server.java

```
package cn.jmu.edu;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * 基于TCP 协议的Socket 通信，实现用户登陆
 * 服务器端
 *
 * @author Sky
 * @date 2016年10月27日
 */
public class Server {
    public static void main(String[] args) {
        try {
            Socket socket = null;
            int count = 0; //记录客户端数量 
            //1.创建一个服务端Socket，即ServerSocket，指定端口并监听此端口
            ServerSocket serverSocket = new ServerSocket(8888);
            System.out.println("***服务器即将启动，等待客户端的连接***");
            //循环监听等待客户端的连接
            while(true){
                //调用accept方法开始监听，等待客户端的连接
                socket = serverSocket.accept();
                //创建一个新的线程
                ServerThread serverThread = new ServerThread(socket);
                //启动线程
                serverThread.start();
                
                count++; //统计客户端的数量
                System.out.println("客户端数量：" + count);
                //使用 InetAddress 对象获取客户端主机的IP地址
                InetAddress address = socket.getInetAddress();
                System.out.println("当前客户端的IP地址：" + address.getHostAddress());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

服务端多线程类 ServerThread.java

```
package cn.jmu.edu;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.Socket;

/**
 * 服务器端线程处理类
 *
 * @author Sky
 * @date 2016年10月28日
 */
public class ServerThread extends Thread {
    //和本线程相关的Socket
    Socket socket = null;
    //构造方法对socket进行初始化
    public ServerThread(Socket socket){
        this.socket = socket;
    }
    
    //执行线程的操作，响应客户端请求
    @Override
    public void run() {
        InputStream is = null;
        InputStreamReader isr = null;
        BufferedReader br = null;
        OutputStream os = null;
        PrintWriter pw = null;
        try {
            //3.获取输入流，并读取用户信息
            is = socket.getInputStream(); //获取字节流入流
            isr = new InputStreamReader(is); //将字节输入流转化为字符输入流
            br = new BufferedReader(isr); //为字符输入流添加缓冲
            
            String info = null;
            while((info=br.readLine()) != null){ //循环读取客户端发送的信息
                System.out.println("我是服务器，客户端说：" + info);
            }
            socket.shutdownInput(); //关闭输入流
            
            //4.获取输出流，响应客户端的请求
            os = socket.getOutputStream();
            pw = new PrintWriter(os); //打包为打印流
            pw.write("欢迎您！");
            pw.flush(); //刷新缓冲区的数据，将数据输出
            
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //5.关闭资源
            try {
                if (pw != null)
                    pw.close();
                if (os != null)
                    os.close();
                if (br != null)
                    br.close();
                if (isr != null)
                    isr.close();
                if (is != null)
                    is.close();
                if (socket != null)
                    socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        super.run();
    }
}
```

客户端类不变。

运行效果：

![img](http://images2015.cnblogs.com/blog/930246/201610/930246-20161028153203718-970414129.png)

 