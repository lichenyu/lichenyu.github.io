title: Java - XML解析示例
date: 2019/03/22
categories:
- Program Development
tags:
- Java
---


```java
import java.io.ByteArrayInputStream;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;

public class XmlParserDemo
{
    /*
<?xml version="1.0" encoding="UTF-8"?>
<routetable>
    <route>
        <da>138517</da>
        <ne>SMSC1</ne>
    </route>
    <route>
        <da>139139</da>
        <ne> SMSC2</ne>
    </route>
</routetable>
     */
    private static final String xmlString = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>"
                                                + "<routetable>"
                                                + "<route><da>138517</da>"
                                                + "<ne>SMSC1</ne></route>" 
                                                + "<route><da>139139</da>"
                                                + "<ne>SMSC2</ne></route>"
                                                + "</routetable>";

    public static void main(String[] args)
    {
        try
        {
         // 生成一个Dom解析器
            DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
            DocumentBuilder db = dbf.newDocumentBuilder();

            // 解析XML
            //Document document = db.parse(new File("test.xml"));
            Document document = db.parse(new ByteArrayInputStream(xmlString.getBytes()));
            document.getDocumentElement().normalize();
            
            NodeList table = document.getElementsByTagName("routetable");
            System.out.println("routetable list len = " + table.getLength());
            for(int i = 0; i < table.getLength(); ++i)
            {
                System.out.println("routetable " + i + " name = " + table.item(i).getNodeName());
                NodeList list = table.item(i).getChildNodes();
                System.out.println("route list len = " + list.getLength());
            }
            
            System.out.println();
            
            NodeList routeList = document.getElementsByTagName("route");
            for(int i = 0; i < routeList.getLength(); ++i)
            {
                Node route = routeList.item(i);
                if(route.getNodeType() == Node.ELEMENT_NODE)
                {
                    Element element = (Element) route;
                    System.out.println("id = " + (element.getAttribute("id").isEmpty() ? "no ID" : element.getAttribute("id")));
                    System.out.println("da -> " + element.getElementsByTagName("da").item(0).getTextContent());
                    System.out.println("ne -> " + element.getElementsByTagName("ne").item(0).getTextContent());
                }
            }
        }
        catch(Exception e)
        {
            e.printStackTrace();
        }
    }
}
```
