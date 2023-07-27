# Producer-Consumer-problem
 CSC 411
import java.util.LinkedList;
import java.util.Queue;
import java.util.Random;
import java.util.concurrent.Semaphore;
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;
import java.io.File;
import java.io.IOException;
import org.xml.sax.SAXException;
import java.util.ArrayList;

// ITstudent class to store student information
class ITstudent {
    private String studentName;
    private String studentID;
    private String programme;
    private ArrayList<String> courses;
    private ArrayList<Integer> marks;

    public ITstudent(String studentName, String studentID, String programme, ArrayList<String> courses, ArrayList<Integer> marks) {
        this.studentName = studentName;
        this.studentID = studentID;
        this.programme = programme;
        this.courses = courses;
        this.marks = marks;
}

 public String getStudentName() {
        return studentName;
    }

    public String getStudentID() {
        return studentID;
    }

    public String getProgramme() {
        return programme;
    }

    public ArrayList<String> getCourses() {
        return courses;
    }

    public ArrayList<Integer> getMarks() {
        return marks;
    }

    public double calculateAverageMark() {
        int sum = 0;
        for (int mark : marks) {
            sum += mark;
        }
        return (double) sum / marks.size();
    }

    public boolean isPassed() {
        double averageMark = calculateAverageMark();
        return averageMark >= 50;
    }
}

public class ProducerConsumerExample {
    private static final int BUFFER_SIZE = 10;
    private static Queue<Integer> sharedBuffer = new LinkedList<>();
    private static Semaphore bufferMutex = new Semaphore(1); // Mutex for buffer access
    private static Semaphore producerSemaphore = new Semaphore(BUFFER_SIZE); // Empty slots in the buffer
    private static Semaphore consumerSemaphore = new Semaphore(0); // Filled slots in the buffer

    // Producer thread to create and save XML files
    static class Producer extends Thread {
        @Override
        public void run() {
            for (int i = 1; i <= 10; i++) { // We want to create 10 student files (student1.xml, student2.xml, ..., student10.xml)
                ITstudent student = new ITstudent();
                // Save student data to XML file
                saveStudentDataToXML(student, "student" + i + ".xml");

                try {
                    producerSemaphore.acquire(); // Wait for an empty slot in the buffer
                    bufferMutex.acquire(); // Acquire the buffer mutex for mutual exclusion
                    sharedBuffer.add(i);
                    bufferMutex.release(); // Release the buffer mutex
                    consumerSemaphore.release(); // Signal that the buffer has a filled slot
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    // Helper method to save ITstudent data to an XML file
    private static void saveStudentDataToXML(ITstudent student, String fileName) {
        try {
            DocumentBuilderFactory docFactory = DocumentBuilderFactory.newInstance();
            DocumentBuilder docBuilder = docFactory.newDocumentBuilder();

            // Create the root element
            Document doc = docBuilder.newDocument();
            Element rootElement = doc.createElement("Student");
            doc.appendChild(rootElement);

            // Create the child elements for student information
            Element nameElement = doc.createElement("Name");
            nameElement.appendChild(doc.createTextNode(student.getStudentName()));
            rootElement.appendChild(nameElement);

            Element idElement = doc.createElement("ID");
            idElement.appendChild(doc.createTextNode(student.getStudentID()));
            rootElement.appendChild(idElement);

            Element programmeElement = doc.createElement("Programme");
            programmeElement.appendChild(doc.createTextNode(student.getProgramme()));
            rootElement.appendChild(programmeElement);

            Element coursesElement = doc.createElement("Courses");
            rootElement.appendChild(coursesElement);

            // Add courses and marks to the XML
            for (int i = 0; i < student.getCourses().size(); i++) {
                String courseName = student.getCourses().get(i);
                int mark = student.getMarks().get(i);

                Element courseElement = doc.createElement("Course");
                coursesElement.appendChild(courseElement);

                Element courseNameElement = doc.createElement("Name");
                courseNameElement.appendChild(doc.createTextNode(courseName));
                courseElement.appendChild(courseNameElement);

                Element markElement = doc.createElement("Mark");
                markElement.appendChild(doc.createTextNode(String.valueOf(mark)));
                courseElement.appendChild(markElement);
            }

            // Write the content into XML file
            TransformerFactory transformerFactory = TransformerFactory.newInstance();
            Transformer transformer = transformerFactory.newTransformer();
            DOMSource source = new DOMSource(doc);
            StreamResult result = new StreamResult(new File(fileName));
            transformer.transform(source, result);

        } catch (ParserConfigurationException | TransformerException e) {
            e.printStackTrace();
        }
    }

    // Consumer thread to process the XML files
    static class Consumer extends Thread {
        @Override
        public void run() {
            while (true) {
                try {
                    consumerSemaphore.acquire(); // Wait for a filled slot in the buffer
                    bufferMutex.acquire(); // Acquire the buffer mutex for mutual exclusion
                    int fileNumber = sharedBuffer.poll();
                    bufferMutex.release(); // Release the buffer mutex
                    producerSemaphore.release(); // Signal that the buffer has an empty slot

                    String fileName = "student" + fileNumber + ".xml";
                    processStudentDataFromXML(fileName);

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    // Helper method to process student data from an XML file
    private static void processStudentDataFromXML(String fileName) {
        try {
            File file = new File(fileName);
            DocumentBuilderFactory dbFactory = DocumentBuilderFactory.newInstance();
            DocumentBuilder dBuilder = dbFactory.newDocumentBuilder();
            Document doc = dBuilder.parse(file);
            doc.getDocumentElement().normalize();

            String studentName = doc.getElementsByTagName("Name").item(0).getTextContent();
            String studentID = doc.getElementsByTagName("ID").item(0).getTextContent();
            String programme = doc.getElementsByTagName("Programme").item(0).getTextContent();

            NodeList courseNodes = doc.getElementsByTagName("Course");
            ArrayList<String> courses = new ArrayList<>();
            ArrayList<Integer> marks = new ArrayList<>();
            for (int i = 0; i < courseNodes.getLength(); i++) {
                Node courseNode = courseNodes.item(i);
                if (courseNode.getNodeType() == Node.ELEMENT_NODE) {
                    Element courseElement = (Element) courseNode;
                    String courseName = courseElement.getElementsByTagName("Name").item(0).getTextContent();
                    int mark = Integer.parseInt(courseElement.getElementsByTagName("Mark").item(0).getTextContent());

                    courses.add(courseName);
                    marks.add(mark);
                }
            }

            ITstudent student = new ITstudent(studentName, studentID, programme, courses, marks);
            double averageMark = student.calculateAverageMark();
            String passStatus = student.isPassed() ? "Passed" : "Failed";

            System.out.println("Student Name: " + student.getStudentName());
            System.out.println("Student ID: " + student.getStudentID());
            System.out.println("Programme: " + student.getProgramme());
            System.out.println("Courses and Marks:");
            for (int i = 0; i < courses.size(); i++) {
                System.out.println(courses.get(i) + ": " + marks.get(i));
            }
            System.out.println("Average Mark: " + averageMark);
            System.out.println("Pass/Fail: " + passStatus);
            System.out.println();

            // Delete the XML file after processing
            file.delete();

        } catch (ParserConfigurationException | SAXException | IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        Thread producerThread = new Producer();
        Thread consumerThread = new Consumer();
        producerThread.start();
        consumerThread.start();
    }
}
