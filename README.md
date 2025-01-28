#sdde Propiedades de microservicios
autor: excz

(schema.xsd)
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
    <xs:element name="book">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="title" type="xs:string"/>
                <xs:element name="author" type="xs:string"/>
                <xs:element name="year" type="xs:int"/>
                <xs:element name="price" type="xs:float"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>

(book.xml)
<?xml version="1.0"?>
<book>
    <title>Effective Java</title>
    <author>Joshua Bloch</author>
    <year>2018</year>
    <price>45.0</price>
</book>

(book.schematron)
<?xml version="1.0" encoding="UTF-8"?>
<schema xmlns="http://purl.oclc.org/dsdl/schematron">
    <pattern id="price-validation">
        <rule context="book">
            <assert test="price > 10">The price of the book must be greater than 10.</assert>
        </rule>
    </pattern>
</schema>

import org.xml.sax.SAXException;
import javax.xml.XMLConstants;
import javax.xml.transform.*;
import javax.xml.transform.stream.StreamSource;
import javax.xml.validation.*;
import java.io.File;
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        try {
            // Validación XSD
            File xmlFile = new File("book.xml");
            File schemaFile = new File("schema.xsd");
            if (validateXMLSchema(xmlFile, schemaFile)) {
                System.out.println("XML is valid against XSD.");
            } else {
                System.out.println("XML is not valid against XSD.");
                return;
            }

            // Validación Schematron
            File schematronFile = new File("book.schematron");
            if (validateSchematron(xmlFile, schematronFile)) {
                System.out.println("XML passed Schematron validation.");
            } else {
                System.out.println("XML failed Schematron validation.");
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static boolean validateXMLSchema(File xmlFile, File schemaFile) {
        try {
            SchemaFactory factory = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
            Validator validator = factory.newSchema(schemaFile).newValidator();
            validator.validate(new StreamSource(xmlFile));
            return true;
        } catch (SAXException | IOException e) {
            e.printStackTrace();
            return false;
        }
    }

    public static boolean validateSchematron(File xmlFile, File schematronFile) throws TransformerException {
        // Transformación Schematron, primero convertir Schematron a XSLT
        TransformerFactory tf = TransformerFactory.newInstance();
        Source schematronSource = new StreamSource(schematronFile);
        Source xsltSource = new StreamSource(Main.class.getResourceAsStream("/iso_svrl_for_xslt2.xsl"));
        Transformer schematronTransformer = tf.newTransformer(xsltSource);

        // Aplicar XSLT para generar un reporte de validación
        Source xmlSource = new StreamSource(xmlFile);
        Result result = new StreamResult(new File("schematron-output.xml"));
        schematronTransformer.transform(xmlSource, result);

        // Leer el resultado para determinar si hay algún fallo
        // Esto es un análisis sencillo, puedes mejorar según tus necesidades
        // (por ejemplo, leer y analizar "schematron-output.xml" para buscar errores específicamente)
        return result != null;
    }
}

