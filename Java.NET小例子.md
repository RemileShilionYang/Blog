---
title: Java.NET小例子
tags: Java,NET,Socket,ServerSocket
---

```javascript?linenums
--Client.java
public class Client {

	private static String ip = null;
	private static String websiteName = null;
	private static int port = 10000;
	private static int time = 50000;
	
	public static void main(String[] args) throws Exception{
		
		/**
		 * 解析XML配置文件
		 */
		
		DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
		DocumentBuilder builder = factory.newDocumentBuilder();
		Document document = builder.parse(new File("/Work Files/Java/SocketNetworkWithXML/src/socketNetworkWithXML/Configure.xml"));
		XML xml = new XML(document);
		Element root = xml.getRoot();
		NodeList nodeList = xml.getChild(root);
		xml.getItem(nodeList);
		ip = XML.ip;
		websiteName = XML.websiteName;
		port = XML.port;
		time = XML.time;
		
		/**
		 * 判断远程地址提供类型(ip/site name)
		 */
		
		if(ip == null && websiteName == null){
			System.err.println("No Server Completed!");
			System.exit(0);
		}
		else if(ip == null && websiteName != null){
			try{
				InetAddress aInetAddress = InetAddress.getByName(websiteName);
				InetSocketAddress address = new InetSocketAddress(aInetAddress, port);
				Socket socket = new Socket();
				socket.connect(address, time);
				ClientHandler clientHandler = new ClientHandler();
				
				OutputStream outputStream = socket.getOutputStream();
				PrintWriter printWriter = new PrintWriter(outputStream,true);
				Scanner scanner = new Scanner(System.in);
				
				InputStream inputStream = socket.getInputStream();
				Scanner scanner2 = new Scanner(inputStream);
				
				String string = null;
				while(true){
					clientHandler.send(scanner,printWriter);
					string = clientHandler.receive(scanner2);
					if(string.equals("Bye!")) break;
				}
				socket.close();
			}
			catch(Exception exception){
				System.err.println("Server Not Found!");
			}
		}
		else{
			try{
				InetSocketAddress address = new InetSocketAddress(ip, port);
				Socket socket = new Socket();
				socket.connect(address, time);
				ClientHandler clientHandler = new ClientHandler();
				
				OutputStream outputStream = socket.getOutputStream();
				PrintWriter printWriter = new PrintWriter(outputStream,true);
				Scanner scanner = new Scanner(System.in);
				
				InputStream inputStream = socket.getInputStream();
				Scanner scanner2 = new Scanner(inputStream);
				
				String string = null;
				while(true){
					clientHandler.send(scanner,printWriter);
					string = clientHandler.receive(scanner2);
					if(string.equals("Bye!")) break;
				}
				socket.close();
			}
			catch (Exception exception) {
				System.err.println("Server Not Found!");
			}
		}
	}
}

--ClientHandler.java
public class ClientHandler {
	
	private String string = null;
	
	public void send(Scanner scanner, PrintWriter printWriter) throws Exception{
		string = scanner.nextLine();
		printWriter.println(string);
	}
	
	public String receive(Scanner scanner2) throws Exception{
		string = scanner2.nextLine();
		System.out.println(string);
		return string;
	}
}

--Server.java
public class Server {
	
	private static int port = 10000;
	
	public static void main(String[] args) throws Exception{
		try{
			ServerSocket serverSocket = new ServerSocket(port);
			ServerHandler serverHandler = new ServerHandler(serverSocket);
			try{
				while(true) serverHandler.transform();
			}
			catch(Exception e){
				System.err.println("Can Not Create Socket From Port:"+port+".!");
				try{
					serverHandler.close();
				}
				catch(Exception exception){
					System.err.println("ServerSocket has already closed!");
				}
				finally{
					System.exit(0);
				}
			}
		}
		catch(Exception exception){
			System.err.println("Port was used by other applications!");
			System.exit(0);
		}
	}
}

--ServerHandler.java
public class ServerHandler extends Socket implements Runnable{

	private ServerSocket serverSocket;
	private Socket socket;
	
	public ServerHandler(ServerSocket serverSocket){
		this.serverSocket = serverSocket;
	}
	
	public ServerHandler(Socket socket){
		this.socket = socket;
	}
	
	public void transform() throws Exception{
		Socket socket = serverSocket.accept();
		Runnable runnable = new ServerHandler(socket);
		Thread thread = new Thread(runnable);
		thread.start();
	}
	
	public Scanner receive() throws Exception{
		InputStream inputStream = socket.getInputStream();
		Scanner scanner = new Scanner(inputStream);
		return scanner;
	}
	
	public PrintWriter send() throws Exception{
		OutputStream outputStream = socket.getOutputStream();
		PrintWriter printWriter = new PrintWriter(outputStream,true);
		return printWriter;
	}

	public void run(){
		try{
			Scanner scanner = receive();
			PrintWriter printWriter = send();
			String string = null;
			while(scanner.hasNextLine()){
				string = scanner.nextLine();
				System.out.println(string);
				if(string.equalsIgnoreCase("quit") || string.equalsIgnoreCase("exit")){
					printWriter.println("Bye!");
					break;
				}
				else{
					printWriter.println("return: "+string);
				}
			}
			scanner.close();
			printWriter.close();
			socket.close();
		}
		catch (Exception e){
			e.printStackTrace();
		}
	}
}

--XML.java
public class XML {
	
	private Document document;
	
	public static String ip;
	public static String websiteName;
	public static int port;
	public static int time;
	
	public XML(Document document){
		this.document = document;
	}
	
	public Element getRoot(){
		Element root = document.getDocumentElement();
		return root;
	}
	
	public NodeList getChild(Element root){
		NodeList nodeList = root.getChildNodes();
		return nodeList;
	}
	
	public void getItem(NodeList nodeList){
		for(int i=0;i<nodeList.getLength();i++){
			Node node = nodeList.item(i);
			if(node instanceof Element){
				Element element = (Element) node;
				Text text = (Text) element.getFirstChild();
				String string = text.getData().trim();
				if(element.getTagName().equals("ip")) ip = string;
				else if(element.getTagName().equals("websiteName")) websiteName = string;
				else if(element.getTagName().equals("port")) port = Integer.parseInt(string);
				else if(element.getTagName().equals("time")) time = Integer.parseInt(string);
				else System.err.println("Unknown tag.");
			}
		}
	}
}

--Configure.xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<ip>127.0.0.1</ip>
	<websiteName>null</websiteName>
	<port>10000</port>
	<time>50000</time>
</configuration>
```
