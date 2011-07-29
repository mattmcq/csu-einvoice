/**
 * Created by IntelliJ IDEA.
 * User: mjmc
 * Date: Feb 23, 2010
 * Time: 11:37:54 AM
 */

import javax.mail.Message
import javax.mail.MessagingException
import javax.mail.Session
import javax.mail.internet.InternetAddress
import javax.mail.internet.MimeMessage
import javax.xml.parsers.ParserConfigurationException
import org.xml.sax.SAXException

class Mailer {
    private static org.apache.log4j.Logger LOG = org.apache.log4j.Logger.getLogger("Mailer");

    static void sendSuccessEmail(String full_xml) throws IOException, SAXException, ParserConfigurationException, MessagingException {
        def cxml = new XmlSlurper().parseText(full_xml.toString())
        String duns = cxml.Header.From
        String einvoiceid = cxml.Request.InvoiceDetailRequest.InvoiceDetailRequestHeader.@invoiceID

        LOG.info("duns is " + duns)
        String subject = "eInvoice: " + einvoiceid + "    DUNS: " + duns;

        // only send success emails if we are on the test system Tuttle
        if (InetAddress.localHost.hostName == "Tuttle.is.colostate.edu") {
            LOG.info("Sending SUCCESS email for " + subject);
            sendMail("bfs_kuali_implementation@colostate.edu", "einvoice@colostate.edu", subject, "eInvoice full text: \n\n" + full_xml);
        }
    }

    static void sendFailureEmail(String full_xml, String error) throws IOException, SAXException, ParserConfigurationException, MessagingException {
        def cxml = new XmlSlurper().parseText(full_xml.toString())
        String duns = cxml.Header.From
        String einvoiceid = cxml.Request.InvoiceDetailRequest.InvoiceDetailRequestHeader.@invoiceID
        String subject = "ERROR - eInvoice: " + einvoiceid + "    DUNS: " + duns;
        LOG.info("Sending FAILURE email for " + subject);
        sendMail("bfs_kuali_implementation@colostate.edu", "einvoice@colostate.edu", subject, "eInvoice error: \n${error} \n\n\n\neInvoice full text: \n" + full_xml);
    }

    static private sendMail(to, from, subject, message) {
        def props = new Properties();
        props.put("mail.smtp.host", "smtp.colostate.edu");
        Session session = Session.getInstance(props);

        def mimeMessage = new MimeMessage(session)
        mimeMessage.setRecipients Message.RecipientType.TO, to
        mimeMessage.setSubject subject
        mimeMessage.setFrom new InternetAddress(from)
        mimeMessage.setContent message.toString(), "text/plain"

        def transport = session.getTransport("smtp")
        transport.connect()
        transport.sendMessage mimeMessage, mimeMessage.allRecipients
    }


}