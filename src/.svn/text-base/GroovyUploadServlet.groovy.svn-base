/**
 * Created by IntelliJ IDEA.
 * User: mjmc
 * Date: Jan 5, 2010
 * Time: 12:22:45 PM
 */

import groovy.xml.XmlUtil
import java.text.ParseException
import javax.servlet.ServletException
import javax.servlet.http.HttpServlet
import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse
import javax.xml.XMLConstants
import javax.xml.parsers.DocumentBuilder
import javax.xml.parsers.DocumentBuilderFactory
import javax.xml.parsers.ParserConfigurationException
import javax.xml.transform.stream.StreamSource
import javax.xml.validation.Schema
import javax.xml.validation.SchemaFactory
import javax.xml.validation.Validator
import org.apache.commons.io.IOUtils
import org.w3c.dom.Document
import org.w3c.dom.Element
import org.w3c.dom.Node
import org.xml.sax.InputSource
import org.xml.sax.SAXException

class GroovyUploadServlet extends HttpServlet {
    private static org.apache.log4j.Logger LOG = org.apache.log4j.Logger.getLogger("GroovyUploadServlet");

    String VALIDATION_FAILURE_MESSAGE;

    @Override
    public void doPost(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException {
        // test with:
        //   curl -k --verbose --request POST --header "Content-Type: text/xml" --data @/StaplesInvoice.xml http://localhost:8080/upload/

        FileOutputStream out, outDone;
        String filename, s, filenameWithDotXML, duns, einvoiceid, payloadid, timestamp_from_einvoice, DOCTYPE;
        PrintWriter writer = res.getWriter();

        try {

            s = IOUtils.toString(req.getInputStream());
            def cxml = new XmlSlurper().parseText(s)
            duns = cxml.Header.From
            einvoiceid = cxml.Request.InvoiceDetailRequest.InvoiceDetailRequestHeader.@invoiceID
            payloadid = cxml.@payloadID
            timestamp_from_einvoice = cxml.@timestamp
            LOG.info("DUNS: ${duns}    eInvoiceID: ${einvoiceid}     payloadID: ${payloadid}    timestamp_from_einvoice: ${timestamp_from_einvoice}")
            LOG.info("recieved input stream = \n" + s);

            // Special DOCTYPE for Fisher; Kevin had me add this
            if (duns.equalsIgnoreCase("004321519")) {

                DOCTYPE = "<!DOCTYPE cXML \"http://xml.cXML.org/schemas/cXML/1.2.014/cXML.dtd\">" + "\r\n"
            } else {
                DOCTYPE = "<!DOCTYPE cXML SYSTEM \"http://xml.cXML.org/schemas/cXML/1.2.014/cXML.dtd\">" + "\r\n"
            }

            if (validate(addNamespaceDefinition(s), getServletContext().getResource("/WEB-INF/electronicInvoice.xsd"))) {

                filename = "/vendorfiles/einvoice/" + duns + "_" + einvoiceid + "_" + System.nanoTime();
                filenameWithDotXML = filename + ".xml";
                out = new FileOutputStream(filenameWithDotXML);
                outDone = new FileOutputStream(filename + ".done");

                out.write(s.getBytes());
                outDone.write(0);
                Mailer.sendSuccessEmail(s)



                StringBuffer cxml_response = new StringBuffer();
                cxml_response.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>" + "\r\n");
                cxml_response.append(DOCTYPE);
                cxml_response.append("<cXML timestamp=\"${timestamp_from_einvoice}\" payloadID=\"${payloadid}\">" + "\r\n");
                cxml_response.append("<Response>" + "\r\n");
                cxml_response.append("<Status code=\"200\" text=\"OK\">" + "eInvoice Received!!!" + "</Status>" + "\r\n");
                cxml_response.append("</Response>" + "\r\n");
                cxml_response.append("</cXML>" + "\r\n");

                writer.print(cxml_response);


            } else {

                res.setStatus(406);
                res.sendError(406, VALIDATION_FAILURE_MESSAGE);
                StringBuffer cxml_response_error = new StringBuffer();
                cxml_response_error.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>" + "\r\n");
                cxml_response_error.append(DOCTYPE);
                cxml_response_error.append("<cXML>" + "\r\n");
                cxml_response_error.append("<Response>" + "\r\n");
                cxml_response_error.append("<Status code=\"406\" text=\"Not Acceptable\">" + "XML parsing error: ${VALIDATION_FAILURE_MESSAGE}" + "</Status>" + "\r\n");
                cxml_response_error.append("</Response>" + "\r\n");
                cxml_response_error.append("</cXML>" + "\r\n");

                writer.print(cxml_response_error);
                Mailer.sendFailureEmail(s, VALIDATION_FAILURE_MESSAGE)
            }


        }

        catch (SAXException e) {
            e.printStackTrace();
        }
    }





    public boolean validate(String s, URL schemaXsdUrl) throws ParseException, SAXException, ParserConfigurationException, IOException {

        SchemaFactory schemaFactory = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
        Schema schema = schemaFactory.newSchema(schemaXsdUrl);
        Validator validator = schema.newValidator();

        try {
            validator.validate(new StreamSource(new StringReader(s)));
            return true;
        } catch (SAXException e) {
            VALIDATION_FAILURE_MESSAGE = e.getLocalizedMessage();
            LOG.error("Reason: " + VALIDATION_FAILURE_MESSAGE);
            return false;
        }

    }


    String addNamespaceDefinition(String s) throws IOException {

        DocumentBuilderFactory builderFactory = DocumentBuilderFactory.newInstance();
        builderFactory.setValidating(false); // It's not needed to validate here
        builderFactory.setIgnoringElementContentWhitespace(true);
        DocumentBuilder builder;
        try {
            builder = builderFactory.newDocumentBuilder();  // Create the parser
        } catch (ParserConfigurationException e) {
            LOG.error("Error getting document builder - " + e.getMessage());
            throw new RuntimeException(e);
        }

        Document xmlDoc;

        try {
            xmlDoc = builder.parse(new InputSource(new StringReader(s)));
        } catch (Exception e) {

            LOG.error("Error parsing the file - " + e.getMessage());

            return null;
        }

        Node node = xmlDoc.getDocumentElement();
        Element element = (Element) node;

        String xmlnsValue = element.getAttribute("xmlns");
        String xmlnsXsiValue = element.getAttribute("xmlns:xsi");

        if (xmlnsValue.equals("http://www.kuali.org/kfs/purap/electronicInvoice") &&
                xmlnsXsiValue.equals("http://www.w3.org/2001/XMLSchema-instance")) {
            LOG.info("xmlns and xmlns:xsi attributes already exists in the invoice xml");
        } else {
            LOG.info("setting attributes");
            element.setAttribute("xmlns", "http://www.kuali.org/kfs/purap/electronicInvoice");
            element.setAttribute("xmlns:xsi", "http://www.w3.org/2001/XMLSchema-instance");
        }
        return XmlUtil.serialize(xmlDoc.documentElement)

    }





    @Override
    public void doGet(HttpServletRequest req, HttpServletResponse res) throws ServletException, java.io.IOException {

        // just display a static page to let us know the Upload Servlet is running
        res.setContentType("text/html");
        PrintWriter out = res.getWriter();
        out.println("<html>");
        out.println("<head>");
        String title = "eInvoice Upload";
        out.println("<title>" + title + "</title>");
        out.println("</head>");
        out.println("<body bgcolor=\"green\">");
        out.println("<h1>" + title + "</h1>");

    }


}