---
title: Java.NIO小例子
tags: Java ,NIO,SocketChannel,ServerSocketChannel
---

```javascript?linenums
--Client.java
public class Client {
	
	private static String ip = null;
	private static int port = 10000;
	
	public static void main(String[] args) throws Exception{
		DocumentBuilderFactory documentBuilderFactory = DocumentBuilderFactory.newInstance();
		DocumentBuilder documentBuilder = documentBuilderFactory.newDocumentBuilder();
		File file = new File("/Work Files/Java/SocketChannelNetworkWithXML/src/socketChannelNetworkWithXML/Configure.xml");
		Document document = documentBuilder.parse(file);
		XML xml = new XML(document);
		xml.explain();
		ip = XML.ip;
		port = XML.port;
		
		getConnect();
	}
	
	public static void getConnect() throws Exception{
		InetSocketAddress inetSocketAddress = new InetSocketAddress(ip, port);
		SocketChannel socketChannel = SocketChannel.open(inetSocketAddress);
		ClientHandler clientHandler = new ClientHandler();
		
		OutputStream outputStream = Channels.newOutputStream(socketChannel);
		PrintWriter printWriter = new PrintWriter(outputStream, true);
		Scanner scanner = new Scanner(System.in);
		
		InputStream inputStream = Channels.newInputStream(socketChannel);
		Scanner scanner2 = new Scanner(inputStream);
		
		while(true){
			clientHandler.send(outputStream,printWriter,scanner);
			String string = clientHandler.receive(inputStream,scanner2);
			if(string.equals("Bye!")) break;
		}
		printWriter.close();
		scanner.close();
		scanner2.close();
		socketChannel.close();
	}
}

--ClientHandler.java
public class ClientHandler {
	
	private String string = null;
	
	public void send(OutputStream outputStream, PrintWriter printWriter, Scanner scanner){
		string = scanner.nextLine();
		printWriter.println(string);
	}
	
	public String receive(InputStream inputStream, Scanner scanner2){
		string = scanner2.nextLine();
		System.out.println(string);
		return string;
	}
}

--Server.java
public class Server {
	
	private static int port = 10000;
	
	public static void main(String[] args) throws Exception{
		ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
		InetSocketAddress inetSocketAddress = new InetSocketAddress(port);
		serverSocketChannel.socket().bind(inetSocketAddress);
		SocketChannel socketChannel = null;
		while(true){
			socketChannel = serverSocketChannel.accept();
			ServerHandler serverHandler = new ServerHandler(socketChannel);
			serverHandler.getInThread();
		}
	}

}

--ServerHandler.java
public class ServerHandler implements Runnable{
	
	private SocketChannel socketChannel = null;
	private ExecutorService executorService = Executors.newCachedThreadPool();
	
	public ServerHandler(SocketChannel socketChannel){
		this.socketChannel = socketChannel;
	}
	
	public void getInThread(){
		Runnable runnable = new ServerHandler(socketChannel);
		executorService.execute(runnable);
	}

	public void run(){
		Socket socket = socketChannel.socket();
		BufferedReader bufferedReader = null;
		PrintWriter printWriter = null;
		try{
			bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
			printWriter = new PrintWriter(socket.getOutputStream(),true);
			String string = null;
			while((string = bufferedReader.readLine()) != null){
				System.out.println(string);
				if(string.equalsIgnoreCase("quit") || string.equalsIgnoreCase("exit")){
					printWriter.println("Bye!");
					break;
				}
				else{
					printWriter.println("return :"+string);
				}
			}
		}
		catch(Exception exception){
			try {
				bufferedReader.close();
			}
			catch (Exception e) {
				e.printStackTrace();
			}
			printWriter.close();
		}
		finally{
			if(socketChannel != null){
				try{
					socketChannel.close();
				}
				catch(Exception exception2){
					System.err.println("Connection has stop yet!");
				}
			}
		}
	}
}

--XML.java
public class XML {
	
	private Document document;
	public static String ip = "127.0.0.1";
	public static int port = 10000;
	
	public XML(Document document){
		this.document = document;
	}
	
	public void explain(){
		Element root = document.getDocumentElement();
		NodeList nodeList = root.getChildNodes();
		for(int i=0;i<nodeList.getLength();i++){
			Node node = nodeList.item(i);
			if(node instanceof Element){
				Element element = (Element) node;
				Text text = (Text) element.getFirstChild();
				String string = text.getData().trim();
				if(element.getTagName().equals("ip")) ip = string;
				else if(element.getTagName().equals("port")) port = Integer.parseInt(string);
				else ;
			}
		}
	}
}

--Configure.java
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<ip>127.0.0.1</ip>
	<port>10000</port>
</configuration>
```
