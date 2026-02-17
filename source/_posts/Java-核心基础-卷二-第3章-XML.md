---
title: 第3章 XML
date: 2026-02-17 15:12:01
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 3.1 XML 概述

- XML（eXtensible Markup Language）是一种用于描述**结构化数据**的标记语言
- XML 与 HTML 的区别：
    - XML 标签由用户自定义，HTML 标签是预定义的
    - XML 严格区分大小写
    - XML 必须有**正确的嵌套**和**关闭标签**
- XML 文档的基本结构：
    - **声明**：`<?xml version="1.0" encoding="UTF-8"?>`
    - **根元素**：每个 XML 文档有且只有一个根元素
    - **子元素**、**属性**、**文本内容**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
    <element attribute="value">文本内容</element>
</root>
```

---

## 3.2 XML 文档的结构

- **元素（Element）**：XML 的基本构建块，可以包含子元素、文本、属性
- **属性（Attribute）**：附加在元素开始标签中的键值对
- **CDATA 段**：`<![CDATA[...]]>` 中的内容不会被解析器解析，用于包含特殊字符
- **处理指令**：`<?target instruction?>` 向应用程序传递信息
- **注释**：`<!-- 注释内容 -->`

### 元素 vs 属性的选择

- 属性适合存储**元数据**（如 id、类型）
- 元素适合存储**实际数据内容**
- 一般建议：优先使用元素，属性仅用于简单的标识性信息

---

## 3.3 解析 XML 文档

Java 提供两种主要的 XML 解析方式：

| 特性 | **DOM 解析** | **SAX 解析** |
| --- | --- | --- |
| 解析方式 | 将整个文档加载为树形结构 | 基于事件驱动，逐行读取 |
| 内存占用 | 高（整棵树在内存中） | 低（不保留文档结构） |
| 适用场景 | 需要随机访问、修改文档 | 大文件、只需顺序读取 |
| 可否修改 | ✅ 可以 | ❌ 不可以 |

---

## 3.4 使用 DOM 解析器

### 基本步骤

1. 创建 `DocumentBuilderFactory`
2. 通过工厂创建 `DocumentBuilder`
3. 调用 `parse()` 方法获取 `Document` 对象
4. 遍历 DOM 树获取数据

```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
DocumentBuilder builder = factory.newDocumentBuilder();
Document doc = builder.parse(new File("data.xml"));

Element root = doc.getDocumentElement();
NodeList children = root.getChildNodes();

for (int i = 0; i < children.getLength(); i++) {
    Node child = children.item(i);
    if (child instanceof Element) {
        Element e = (Element) child;
        String text = e.getTextContent();
        String attr = e.getAttribute("id");
    }
}
```

### 常用 DOM API

- `Document.getDocumentElement()` → 获取根元素
- `Element.getElementsByTagName(String)` → 按标签名获取子元素列表
- `Element.getAttribute(String)` → 获取属性值
- `Node.getChildNodes()` → 获取所有子节点（包含文本节点）
- `Node.getTextContent()` → 获取文本内容

<aside>
⚠️

`getChildNodes()` 返回的节点列表中包含**空白文本节点**，遍历时需要用 `instanceof Element` 过滤。

</aside>

---

## 3.5 使用 SAX 解析器

- SAX（Simple API for XML）是**事件驱动**的解析方式
- 继承 `DefaultHandler`，重写回调方法：
    - `startDocument()` / `endDocument()`
    - `startElement()` / `endElement()`
    - `characters()` — 处理元素中的文本内容

```java
SAXParserFactory factory = SAXParserFactory.newInstance();
SAXParser parser = factory.newSAXParser();

DefaultHandler handler = new DefaultHandler() {
    @Override
    public void startElement(String uri, String localName,
            String qName, Attributes attrs) {
        System.out.println("开始元素: " + qName);
    }

    @Override
    public void characters(char[] ch, int start, int length) {
        System.out.println("文本: " + new String(ch, start, length));
    }

    @Override
    public void endElement(String uri, String localName, String qName) {
        System.out.println("结束元素: " + qName);
    }
};

parser.parse(new File("data.xml"), handler);
```

<aside>
💡

`characters()` 方法可能被**多次调用**来传递同一个元素的文本内容，需要使用 `StringBuilder` 来拼接。

</aside>

---

## 3.6 使用 StAX 解析器

- StAX（Streaming API for XML）是一种**拉模式**的流式解析
- 与 SAX 的区别：SAX 是推模式（解析器驱动），StAX 是拉模式（应用程序驱动）
- 使用 `XMLInputFactory` 和 `XMLStreamReader`

```java
XMLInputFactory factory = XMLInputFactory.newInstance();
XMLStreamReader reader = factory.createXMLStreamReader(
    new FileInputStream("data.xml"));

while (reader.hasNext()) {
    int event = reader.next();
    switch (event) {
        case XMLStreamConstants.START_ELEMENT:
            System.out.println("元素: " + reader.getLocalName());
            break;
        case XMLStreamConstants.CHARACTERS:
            System.out.println("文本: " + reader.getText().trim());
            break;
    }
}
reader.close();
```

---

## 3.7 生成 XML 文档

### 使用 DOM 生成

```java
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
Document doc = factory.newDocumentBuilder().newDocument();

Element root = doc.createElement("students");
doc.appendChild(root);

Element student = doc.createElement("student");
student.setAttribute("id", "1");
student.setTextContent("张三");
root.appendChild(student);

// 输出到文件
Transformer transformer = TransformerFactory.newInstance().newTransformer();
transformer.setOutputProperty(OutputKeys.INDENT, "yes");
transformer.transform(new DOMSource(doc), new StreamResult(new File("output.xml")));
```

### 使用 StAX 生成

```java
XMLOutputFactory factory = XMLOutputFactory.newInstance();
XMLStreamWriter writer = factory.createXMLStreamWriter(
    new FileOutputStream("output.xml"), "UTF-8");

writer.writeStartDocument("UTF-8", "1.0");
writer.writeStartElement("students");
writer.writeStartElement("student");
writer.writeAttribute("id", "1");
writer.writeCharacters("张三");
writer.writeEndElement();
writer.writeEndElement();
writer.writeEndDocument();
writer.close();
```

---

## 3.8 XPath

- XPath 是一种在 XML 文档中**定位节点**的查询语言
- 使用 `XPathFactory` 和 `XPath` 对象执行查询

### 常用 XPath 表达式

| 表达式 | 说明 |
| --- | --- |
| `/root/element` | 从根路径选取 |
| `//element` | 选取所有匹配元素（任意深度） |
| `/root/element[@attr='val']` | 按属性值筛选 |
| `/root/element[1]` | 选取第一个匹配元素（从 1 开始） |
| `text()` | 获取文本内容 |

### Java 中使用 XPath

```java
XPathFactory xpFactory = XPathFactory.newInstance();
XPath xpath = xpFactory.newXPath();

// 查询单个节点
Node node = (Node) xpath.evaluate(
    "/students/student[@id='1']", doc, XPathConstants.NODE);

// 查询节点列表
NodeList nodes = (NodeList) xpath.evaluate(
    "//student", doc, XPathConstants.NODESET);

// 查询文本值
String name = xpath.evaluate("/students/student[1]/text()", doc);
```

---

## 3.9 XML 命名空间

- 当多个 XML 文档合并时，可能出现**元素名冲突**
- 命名空间通过 URI 唯一标识，使用 `xmlns` 声明

```xml
<root xmlns:h="http://www.w3.org/TR/html4/"
      xmlns:f="http://www.example.com/furniture">
    <h:table>
        <h:tr><h:td>HTML 表格</h:td></h:tr>
    </h:table>
    <f:table>
        <f:name>家具桌子</f:name>
    </f:table>
</root>
```

- Java 解析时需要开启命名空间感知：

```java
factory.setNamespaceAware(true);
```

---

## 3.10 DTD 与 XML Schema 验证

### DTD（Document Type Definition）

- 定义 XML 文档的合法结构
- 语法较老，功能有限

```xml
<!DOCTYPE students [
    <!ELEMENT students (student*)>
    <!ELEMENT student (#PCDATA)>
    <!ATTLIST student id CDATA #REQUIRED>
]>
```

### XML Schema（XSD）

- 比 DTD 更强大，支持**数据类型**、**命名空间**
- 使用 XML 语法编写

```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="students">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="student" maxOccurs="unbounded">
                    <xs:complexType>
                        <xs:simpleContent>
                            <xs:extension base="xs:string">
                                <xs:attribute name="id" type="xs:string" use="required"/>
                            </xs:extension>
                        </xs:simpleContent>
                    </xs:complexType>
                </xs:element>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
```

### Java 中启用验证

```java
factory.setValidating(true);                    // DTD 验证
// 或
factory.setSchema(schemaFactory.newSchema(xsd)); // Schema 验证
```

---

## 3.11 XSLT 转换

- XSLT（XSL Transformations）用于将 XML 文档**转换**为其他格式（HTML、文本等）
- 使用 `TransformerFactory` 和 `Transformer`

```java
TransformerFactory tFactory = TransformerFactory.newInstance();
Transformer transformer = tFactory.newTransformer(
    new StreamSource("style.xsl"));

transformer.transform(
    new DOMSource(doc),
    new StreamResult(new File("output.html")));
```

---

## 📝 本章小结

- XML 是描述结构化数据的通用标记语言
- Java 提供 **DOM**（树形/随机访问）、**SAX**（事件推模式）、**StAX**（流式拉模式）三种解析方式
- **XPath** 可以方便地定位和查询 XML 节点
- **命名空间** 解决元素名冲突问题
- **DTD / XML Schema** 用于验证 XML 文档结构
- **XSLT** 用于 XML 到其他格式的转换
- 选择建议：小文件用 DOM，大文件用 SAX/StAX，需要查询用 XPath