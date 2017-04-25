---
title: 用JAVA解析XML
tags: Java,XML
---

一些Java应用如hadoop等都需要XML配置文件，解析XML必然是程序中的一部分。


```javascript?linenums
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document document = builder.parse(new File("Configure.xml"));
Element root = document.getDocumentElement();
NodeList nodeList = root.getChildNodes();
for(int i=0;i<nodeList.getLength();i++){
        Node node = nodeList.item(i);
        if(node instanceof Element){
		Element element = (Element) node;
		Text text = (Text) element.getFirstChild();
		String string = text.getData().trim();
	        if(element.getTagName().equals());
		    else if(element.getTagName().equals());
		    else if(element.getTagName().equals());
		    else if(element.getTagName().equals());
		    else System.err.println("Unknown tag.");
        }
}
```
__这里有一种方案叫静态工厂方法替代构造器方法：DocumentBuilder的产生就是由静态工厂方法产生，从而减少了内存占用（这个Builder是全局通用的，无类差异性）。__
