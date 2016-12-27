---
layout: post
title: 基于RSA非对称加密算法的UDP加密传输
date: 2014-07-24 12:00:00.000000000 +08:00
tags: Java
---

作者：杨振林

UDP 是User Datagram Protocol的简称， 中文名是用户数据报协议，是OSI（Open System Interconnection，开放式系统互联） 参考模型中一种无连接的传输层协议，提供面向事务的简单不可靠信息传送服务。

不加密情况下使用UDP传输的过程如下。

接收方先运行等待发送方的数据。

```
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetSocketAddress;
import java.net.SocketAddress;

public class DatagramReciver {
	
	private static String ip="127.0.0.1";
	private static int reciveport=15000;
	
	public static void main(String args[]) throws Exception {
		// 1.创建要用来接收的本地地址对象
		SocketAddress localAddr = new InetSocketAddress(ip, reciveport);
		// 2.接收的服务器UDP端口
		DatagramSocket recvSocket = new DatagramSocket(localAddr);
		while (true) {
			// 3.指定接收缓冲区大小
			byte[] buffer = new byte[20];
			// 4.创建接收数据包对象,指定接收大小
			DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
			// 5.阻塞等待数据到来,如果收到数据,存入packet中的缓冲区中
			System.out.println("UDP服务器等待接收数据:"
					+ recvSocket.getLocalSocketAddress());
			recvSocket.receive(packet);
			// 得到发送方的IP和端口
			SocketAddress address = packet.getSocketAddress();
			// 转换接收到的数据为字符串
			String msg = new String(packet.getData()).trim();
			// 接收到后,打印出收到的数据长度
			System.out.println("recv is:" + msg + " from:" + address);
		}
	}
}
```

发送方发送数据

```
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetSocketAddress;
import java.net.SocketAddress;

public class DatagramSender {
	
	private static String ip="127.0.0.1";
	private static int sendport =13000;
	private static int reciveport=15000;
	public static void main(String args[]) throws Exception {
		// 1.创建要用来发送的本地地址对象
		SocketAddress localAddr = new InetSocketAddress(ip, sendport);
		// 2.创建发送的Socket对象
		DatagramSocket dSender = new DatagramSocket(localAddr);
		int count = 0;
		while (true) {
			// 创建要发送的数据,字节数组
			count++;
			// 3.要发送的数据
			byte buffer[] = (count + "-hello").getBytes();
			// 4.发送数据的目标地址和端口
			SocketAddress destAdd = new InetSocketAddress(ip,
					reciveport);
			// 5.创建要发送的数据包,指定内容,指定目标地址
			DatagramPacket dp = new DatagramPacket(buffer, buffer.length,
					destAdd);
			dSender.send(dp);// 6.发送
			System.out.println("数据已发送: " + count);
			Thread.sleep(1000);
		}
	}
}
```

不加密情况下，我们传输的数据是暴露给外界的，使用Wireshark等抓包工具就可以轻松的看到发送的具体内容。

下面分析使用RSA非对称加密算法对UDP通信进行加密。

非对称加密算法需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）。公开密钥与私有密钥是一对，如果用公开密钥对数据进行加密，只有用对应的私有密钥才能解密；如果用私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。因为加密和解密使用的是两个不同的密钥，所以这种算法叫作非对称加密算法。

RSA是目前最有影响力的公钥加密算法，它能够抵抗到目前为止已知的绝大多数密码攻击，已被ISO推荐为公钥数据加密标准。RSA算法基于一个十分简单的数论事实：将两个大素数相乘十分容易，但是想要对其乘积进行因式分解却极其困难，因此可以将乘积公开作为加密密钥。

通过上述简介可以知道，使用RSA加密的步骤是：接收方创建一对公钥+私钥。然后将公钥传给发送方。发送方使用公钥对数据进行加密，然后将加密后的数据发送给接收方。接收方收到加密数据之后用私钥进行解密。这样就可以保证了通信过程中的安全，因为无论是公钥还是加密数据泄露，只要没有私钥，就无法解密。私钥只保存在接收方本地，不使用网络传输。

创建公钥+私钥

```
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

public class RSA {
	public static void main(String args[]) throws Exception {
		/**
		 * RSA加密
		 */
		// 1.取得RSA算法的密钥生成器对象

		KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");

		// 2.设定密钥长度为1024位

		keyPairGen.initialize(1024);

		// 3.生成 "密钥对 "对象

		KeyPair keyPair = keyPairGen.generateKeyPair();

		// 4.分别取得私钥privateKey和公钥publicKey对象

		RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
		RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();

		/**
		 * 公钥写入
		 */
		FileOutputStream oospublic = new FileOutputStream("public-yzl.key");
		ObjectOutputStream ooppublic = new ObjectOutputStream(oospublic);
		ooppublic.writeObject(publicKey);
		ooppublic.flush();
		oospublic.close();
		System.out.println("公钥写入文件成功！");

		/**
		 * 私钥写入
		 */
		FileOutputStream oosprivate = new FileOutputStream("private-yzl.key");
		ObjectOutputStream oopprivate = new ObjectOutputStream(oosprivate);
		oopprivate.writeObject(privateKey);
		oopprivate.flush();
		oopprivate.close();
		System.out.println("私钥写入文件成功！");
	}
}
```

这样，在接收方本地，就创建了公钥（public-yzl.key）和私钥（private-yzl.key）两个文件。下面对公钥文件（public-yzl.key）进行传输，私钥保存在本地。这里使用UDP进行传输。传输公钥的过程不需要加密。

信息发送方此时是公钥文件的接收方，先运行等待接收公钥。

```
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetSocketAddress;


public class PublicKeyRecive {
	private String ip;
	private int serverPort;
	private int clientPort;
	private String path;


	public PublicKeyRecive(String ip, int serverPort, int clientPort,
			String path) {
		this.ip = ip;
		this.serverPort = serverPort;
		this.clientPort = clientPort;
		this.path = path;
	}

	public void receive() {
		try {
			// 接收文件监听端口
			DatagramSocket receive = new DatagramSocket(serverPort);
			// 写文件路径
			BufferedOutputStream bos = new BufferedOutputStream(
					new FileOutputStream(new File(path)));

			// 读取文件包
			byte[] buf = new byte[200];
			DatagramPacket pkg = new DatagramPacket(buf, buf.length);
			// 发送收到文件后 确认信息包
			byte[] messagebuf = new byte[2];
			messagebuf = "ok".getBytes();
			DatagramPacket messagepkg = new DatagramPacket(messagebuf, messagebuf.length,
					new InetSocketAddress(ip, clientPort));
			while (true) {
				receive.receive(pkg);
				if (new String(pkg.getData(), 0, pkg.getLength()).equals("end")) {
					System.out.println("文件接收完毕");
					bos.close();
					receive.close();
					break;
				}
				receive.send(messagepkg);
				System.out.println(new String(messagepkg.getData()));
				bos.write(pkg.getData(), 0, pkg.getLength());
				bos.flush();
			}
			bos.close();
			receive.close();
		} catch (Exception e) {
			e.printStackTrace();
		} 
	}
	
	public static void main(String[] args) {
		String ip="127.0.0.1";
		int serverPort=8085;
		int clientPort=8086;
		String path="new-public-yzl.key";
		new PublicKeyRecive(ip, serverPort, clientPort,path).receive();
	}

}
```

信息的接收方此时是公钥的发送方，向对方发送公钥。

```
import java.io.BufferedInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetSocketAddress;

public class PublicKeySend {
	private int serverPort;
	private int clientPort;
	private String path;
	private String ip;

	public PublicKeySend(int serverPort, int clientPort, String path, String ip) {
		this.serverPort = serverPort;
		this.clientPort = clientPort;
		this.path = path;
		this.ip = ip;
	}

	public void send() {
		try {
			// 文件发送者设置监听端口
			DatagramSocket send = new DatagramSocket(clientPort);
			BufferedInputStream bis = new BufferedInputStream(
					new FileInputStream(new File(path)));
			// 确认信息包
			byte[] messagebuf = new byte[2];
			DatagramPacket messagepkg = new DatagramPacket(messagebuf,
					messagebuf.length);
			// 文件包
			byte[] buf = new byte[200];
			int len;
			while ((len = bis.read(buf)) != -1) {

				DatagramPacket pkg = new DatagramPacket(buf, len,
						new InetSocketAddress(ip, serverPort));
				// 设置确认信息接收时间，3秒后未收到对方确认信息，则重新发送一次
				send.setSoTimeout(3000);
				while (true) {
					send.send(pkg);
					send.receive(messagepkg);
					System.out.println(new String(messagepkg.getData()));
					break;
				}
			}
			// 文件传完后，发送一个结束包
			buf = "end".getBytes();
			DatagramPacket endpkg = new DatagramPacket(buf, buf.length,
					new InetSocketAddress(ip, serverPort));
			System.out.println("文件发送完毕");
			send.send(endpkg);
			bis.close();
			send.close();

		} catch (Exception e) {
			e.printStackTrace();
		}

	}

	public static void main(String[] args) {
		int serverPort = 8085;
		int clientPort = 8086;
		String path = "public-yzl.key";
		String ip = "127.0.0.1";

		new PublicKeySend(serverPort, clientPort, path, ip).send();
	}
}
```

在公钥发送成功之后，开始进行数据加密传输。

发送方使用公钥加密数据的过程如下。输入一个字符串，输出一个加密后的数组。使用GBK进行编码。

```
import java.io.FileInputStream;
import java.io.ObjectInputStream;
import java.security.interfaces.RSAPublicKey;

import javax.crypto.Cipher;

public class SenderSecret {
	String str;

	public SenderSecret(String str) {
		this.str = str;
	}

	public byte[] getsendmsg() throws Exception {
		FileInputStream fins = new FileInputStream("new-public-yzl.key");
		ObjectInputStream ois = new ObjectInputStream(fins);
		RSAPublicKey publicKey = (RSAPublicKey) ois.readObject();
		fins.close();

		byte[] srcData = str.getBytes("GBK");

		Cipher cipher = Cipher.getInstance("RSA");
		cipher.init(Cipher.ENCRYPT_MODE, publicKey);

		// 返回加密后的内容

		return cipher.doFinal(srcData);
	}
}
```

接收方接收到加密数据后，用私钥进行解密的过程如下。输入一个加密后的数组，输出一个解密后的字符数组。

```
import java.io.FileInputStream;
import java.io.ObjectInputStream;
import java.security.interfaces.RSAPrivateKey;
import javax.crypto.Cipher;

public class ReciverSecret {
	private byte[] recivebyte=null;

	public ReciverSecret(byte[] recivebyte,int len) {
		this.recivebyte=new byte[len];
		for(int i=0;i&lt;recivebyte.length;i++){
			this.recivebyte[i] = recivebyte[i];
		}
	}

	public byte[] getrecivemsg() throws Exception {
		FileInputStream fins = new FileInputStream("private-yzl.key");
		ObjectInputStream ois = new ObjectInputStream(fins);
		RSAPrivateKey privateKey = (RSAPrivateKey) ois.readObject();
		fins.close();

		Cipher cipher = Cipher.getInstance("RSA");

		cipher.init(Cipher.DECRYPT_MODE, privateKey);

		// 返回解密后的内容

		return cipher.doFinal(recivebyte);

	}

}
```

有了加密和解密的功能之后，下面开始进行先加密后发送和先接收后解密。

发送方先加密后发送

```
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetSocketAddress;
import java.net.SocketAddress;

public class DatagramSender {
	private static int ports = 13000;
	private static int portr = 15005;

	public static void main(String args[]) throws Exception {
		// 1.创建要用来发送的本地地址对象
		SocketAddress localAddr = new InetSocketAddress("127.0.0.1", ports);
		// 2.创建发送的Socket对象
		DatagramSocket dSender = new DatagramSocket(localAddr);
		int count = 0;
		while (true) {
			// 创建要发送的数据,字节数组
			count++;
			// 3.要发送的数据

			String str = count + "-hello";
			// byte buffer[] = (count + "-hello").getBytes();

			/**
			 * 加密
			 */
			SenderSecret ss = new SenderSecret(str);
			byte[] bytestr = ss.getsendmsg();

			// 4.发送数据的目标地址和端口
			SocketAddress destAdd = new InetSocketAddress("127.0.0.1", portr);
			// 5.创建要发送的数据包,指定内容,指定目标地址
			DatagramPacket dp = new DatagramPacket(bytestr, bytestr.length,
					destAdd);

			dSender.send(dp);// 6.发送
			System.out.println("数据已发送: " + count);
			Thread.sleep(1000);
		}
	}
}
```

接收方先接收后解密

```
import java.io.FileOutputStream;
import java.io.ObjectOutputStream;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetSocketAddress;
import java.net.SocketAddress;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;

public class DatagramReciver {
	private static int port = 15005;

	public static void main(String args[]) throws Exception {	

		// 1.创建要用来接收的本地地址对象
		SocketAddress localAddr = new InetSocketAddress("127.0.0.1", port);
		// 2.接收的服务器UDP端口
		DatagramSocket recvSocket = new DatagramSocket(localAddr);
		while (true) {
			// 3.指定接收缓冲区大小
			byte[] buffer = new byte[128];
			// 4.创建接收数据包对象,指定接收大小
			DatagramPacket packet = new DatagramPacket(buffer, buffer.length);

			recvSocket.receive(packet);

			// 转换接收到的数据为字符串

			byte[] recivebyte = packet.getData();

			ReciverSecret rs = new ReciverSecret(recivebyte, recivebyte.length);
			byte[] bytestr = rs.getrecivemsg();
			
			String finalmsg = new String(bytestr, "GBK");			
			System.out.println(finalmsg);

		}
	}
}
```

通过RSA加密之后，使得私钥不进行网络传输，只保存在本地。解密过程只在本地进行。从而保证了数据的安全性。