/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 * Click nbfs://nbhost/SystemFileSystem/Templates/Other/File.java to edit this template
 */

/**
 *
 * @author Ephraim
 */
import java.util.ArrayList;
import java.util.List;
import java.io.File;
import java.io.FileOutputStream;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.Semaphore;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.Socket;
import java.net.ServerSocket;


import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;
import javax.xml.transform.dom.DOMSource;
import javax.xml.transform.Transformer;
import javax.xml.transform.TransformerException;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.stream.StreamResult;

import org.w3c.dom.Document;
import org.w3c.dom.Element;
import org.w3c.dom.NodeList;

// This class represents a student in the IT department
class ITStudent {
    private String studentName;
    private String studentID;
    private String programme;
    private List<String> courses;
    private List<Integer> marks;

    public ITStudent() {
        courses = new ArrayList<>();
        marks = new ArrayList<>();
    }

    // Getters and setters for the member variables
     public String getStudentName() {
        return studentName;
    }

    public void setStudentName(String studentName) {
        this.studentName = studentName;
    }

    public String getStudentID() {
        return studentID;
    }

    public void setStudentID(String studentID) {
        this.studentID = studentID;
    }

    public String getProgramme() {
        return programme;
    }

    public void setProgramme(String programme) {
        this.programme = programme;
    }

    public List<String> getCourses() {
        return courses;
    }

    public void setCourses(List<String> courses) {
        this.courses = courses;
    }

    public List<Integer> getMarks() {
        return marks;
    }

    public void setMarks(List<Integer> marks) {
        this.marks = marks;
    }

    // Method to add a course and its associated mark to the student
    public void addCourse(String course, int mark) {
        courses.add(course);
        marks.add(mark);
    }
}

//Producer class with XML-related methods

class Producer implements Runnable {
    private BlockingQueue<Integer> buffer;
    private String sharedDirectory;
    private Random random;
    private Semaphore bufferSemaphore;
    private Semaphore emptySemaphore;
    private Semaphore fullSemaphore;

    public Producer(BlockingQueue<Integer> buffer, String sharedDirectory,
                    Semaphore bufferSemaphore, Semaphore emptySemaphore, Semaphore fullSemaphore) {
        this.buffer = buffer;
        this.sharedDirectory = sharedDirectory;
        this.random = new Random();
        this.bufferSemaphore = bufferSemaphore;
        this.emptySemaphore = emptySemaphore;
        this.fullSemaphore = fullSemaphore;
    }
    
     private ITStudent generateRandomStudent() {
        // Implement your random student data generation logic here
        // Generate random student name
    ITStudent student = new ITStudent();
    String[] names = { "John Doe", "Jane Smith", "Michael Johnson", "Emily Brown", "David Lee" };
    Random random = new Random();
    int randomNameIndex = random.nextInt(names.length);
    student.setStudentName(names[randomNameIndex]);

    // Generate random 8-digit student ID
    int randomID = random.nextInt(100000000);
    student.setStudentID(String.format("%08d", randomID));

    // Generate random programme
    String[] programmes = { "Computer Science", "Information Technology", "Software Engineering" };
    int randomProgrammeIndex = random.nextInt(programmes.length);
    student.setProgramme(programmes[randomProgrammeIndex]);

    // Generate random courses and marks
    String[] courses = { "Mathematics", "Computer Programming", "Database Management", "Data Structures" };
    for (String course : courses) {
        student.addCourse(course, random.nextInt(101)); // Random marks between 0 and 100
    }

    return student;
}

        // For demonstration purposes, let's assume some sample data
    private Document createXmlDocument(ITStudent student) throws ParserConfigurationException {
        DocumentBuilderFactory docFactory = DocumentBuilderFactory.newInstance();
        DocumentBuilder docBuilder = docFactory.newDocumentBuilder();
        Document doc = docBuilder.newDocument();

        Element rootElement = doc.createElement("Student");
        doc.appendChild(rootElement);

        Element name = doc.createElement("Name");
        name.appendChild(doc.createTextNode(student.getStudentName()));
        rootElement.appendChild(name);

        Element id = doc.createElement("ID");
        id.appendChild(doc.createTextNode(student.getStudentID()));
        rootElement.appendChild(id);

        Element programme = doc.createElement("Programme");
        programme.appendChild(doc.createTextNode(student.getProgramme()));
        rootElement.appendChild(programme);

        Element courses = doc.createElement("Courses");
        rootElement.appendChild(courses);

        List<String> courseList = student.getCourses();
        List<Integer> marksList = student.getMarks();
        for (int i = 0; i < courseList.size(); i++) {
            Element course = doc.createElement("Course");
            course.appendChild(doc.createTextNode(courseList.get(i)));
            course.setAttribute("Mark", String.valueOf(marksList.get(i)));
            courses.appendChild(course);
        }

        return doc;
    }
     private void saveToXmlFile(Document doc, int fileNumber) throws Exception {
        String directoryPath = sharedDirectory + File.separator;
        File directory = new File(directoryPath);
        if (!directory.exists()) {
            if (directory.mkdirs()) {
                System.out.println("Created directory: " + directory.getAbsolutePath());
            } else {
                throw new IOException("Failed to create directory: " + directory.getAbsolutePath());
            }
        }

        String xmlFileName = directoryPath + "student" + fileNumber + ".xml";
        File xmlFile = new File(xmlFileName);

        TransformerFactory transformerFactory = TransformerFactory.newInstance();
        Transformer transformer = transformerFactory.newTransformer();
        DOMSource source = new DOMSource(doc);

        try (java.io.OutputStream fos = new FileOutputStream(xmlFile)) {
            StreamResult result = new StreamResult(fos);
            transformer.transform(source, result);
        }
    }

    @Override
    public void run() {
        try {
            for (int i = 1; i <= 10; i++) {
                ITStudent student = generateRandomStudent();
                Document doc = createXmlDocument(student);

                saveToXmlFile(doc, i);

                // Simulate some delay in producing data
                Thread.sleep(random.nextInt(1000));

                // Insert the integer corresponding to the file name into the buffer/queue
                buffer.put(i);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
class Consumer implements Runnable {
    private BlockingQueue<Integer> buffer;
    private Semaphore bufferSemaphore;
    private Semaphore emptySemaphore;
    private Semaphore fullSemaphore;

    public Consumer(BlockingQueue<Integer> buffer,
                    Semaphore bufferSemaphore, Semaphore emptySemaphore, Semaphore fullSemaphore) {
        this.buffer = buffer;
        this.bufferSemaphore = bufferSemaphore;
        this.emptySemaphore = emptySemaphore;
        this.fullSemaphore = fullSemaphore;
    }

    // ... (continue with the rest of the Consumer code)
     // Parse the XML data and perform the required calculations
    
    /*private ITStudent parseXmlFile(File xmlFile) throws Exception {
        DocumentBuilderFactory docFactory = DocumentBuilderFactory.newInstance();
        DocumentBuilder docBuilder = docFactory.newDocumentBuilder();
        Document doc = docBuilder.parse(xmlFile);

        ITStudent student = new ITStudent();

        Element rootElement = doc.getDocumentElement();
        NodeList nameList = rootElement.getElementsByTagName("Name");
        student.setStudentName(nameList.item(0).getTextContent());

        NodeList idList = rootElement.getElementsByTagName("ID");
        student.setStudentID(idList.item(0).getTextContent());

        NodeList programmeList = rootElement.getElementsByTagName("Programme");
        student.setProgramme(programmeList.item(0).getTextContent());

        NodeList coursesList = rootElement.getElementsByTagName("Course");
        for (int i = 0; i < coursesList.getLength(); i++) {
            Element courseElement = (Element) coursesList.item(i);
            String courseName = courseElement.getTextContent();
            int mark = Integer.parseInt(courseElement.getAttribute("Mark"));
            student.addCourse(courseName, mark);
        }

        return student;
    }*/
    
    private ITStudent parseXmlFile(File xmlFile) throws Exception {
    DocumentBuilderFactory docFactory = DocumentBuilderFactory.newInstance();
    DocumentBuilder docBuilder = docFactory.newDocumentBuilder();
    Document doc = docBuilder.parse(xmlFile);

    ITStudent student = new ITStudent();

    Element rootElement = doc.getDocumentElement();

    // Get the student name
    NodeList nameList = rootElement.getElementsByTagName("Name");
    if (nameList.getLength() > 0) {
        student.setStudentName(nameList.item(0).getTextContent());
    }

    // Get the student ID
    NodeList idList = rootElement.getElementsByTagName("ID");
    if (idList.getLength() > 0) {
        student.setStudentID(idList.item(0).getTextContent());
    }

    // Get the programme
    NodeList programmeList = rootElement.getElementsByTagName("Programme");
    if (programmeList.getLength() > 0) {
        student.setProgramme(programmeList.item(0).getTextContent());
    }

    // Get the courses and marks
    NodeList coursesList = rootElement.getElementsByTagName("Course");
    for (int i = 0; i < coursesList.getLength(); i++) {
        Element courseElement = (Element) coursesList.item(i);
        String courseName = courseElement.getTextContent();
        int mark = Integer.parseInt(courseElement.getAttribute("Mark"));
        student.addCourse(courseName, mark);
    }

    return student;
}

    
    @Override
    public void run() {
        try {
            while (true) {
                emptySemaphore.acquire(); // Wait if buffer is empty
                bufferSemaphore.acquire(); // Acquire buffer for reading

                int fileName = buffer.take();
                System.out.println("Consumer: Removing student" + fileName + ".xml from buffer");

                bufferSemaphore.release(); // Release buffer after reading
                fullSemaphore.release(); // Signal that buffer is not full

                // Read the corresponding XML file using the file name
                String xmlFileName = "path/to/shared/directory/student" + fileName + ".xml";
                File xmlFile = new File(xmlFileName);

                ITStudent student = parseXmlFile(xmlFile);

                // Calculate the average mark and pass/fail
                double totalMarks = 0;
                int numCourses = student.getCourses().size();
                for (Integer mark : student.getMarks()) {
                    totalMarks += mark;
                }
                double averageMark = totalMarks / numCourses;
                String passOrFail = (averageMark >= 50.0) ? "Pass" : "Fail";

                // Print student information
                System.out.println("Student Name: " + student.getStudentName());
                System.out.println("Student ID: " + student.getStudentID());
                System.out.println("Programme: " + student.getProgramme());
                System.out.println("Courses and Marks: ");
                List<String> courses = student.getCourses();
                List<Integer> marks = student.getMarks();
                for (int i = 0; i < courses.size(); i++) {
                    System.out.println(courses.get(i) + ": " + marks.get(i));
                }
                System.out.println("Average Mark: " + averageMark);
                System.out.println("Result: " + passOrFail);

                // Clear the content of the XML file (delete the file)
                xmlFile.delete();

                // Simulate some delay in processing data
                Thread.sleep(500);
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
   /* private void processXMLData(String xmlData) {
        // Implement your XML parsing and processing logic here
        // For simplicity, we'll just print the received XML data
        System.out.println("Received XML Data: " + xmlData);}
    }*/

    /*private String readXMLDataFromFile(File xmlFile) {
        // Implement your logic to read XML data from the file
        // ...

        // For demonstration purposes, we'll just return a sample XML data
        return "<Student><Name>John Doe</Name><ID>12345678</ID><Programme>Computer Science</Programme></Student>";
}*/


public class Producer_Consumer_MINI_PROJECT {

    /**
     * @param args the command line arguments
     */
    public static void main(String args[]) {
        // TODO code application logic here
        // Create a shared buffer/queue to hold the integers (file names)
        BlockingQueue<Integer> buffer = new LinkedBlockingQueue<>(10);

        // Shared directory path where the producer saves XML files
        String sharedDirectory = "path/to/shared/directory";

        // Create semaphores for buffer access control
        Semaphore bufferSemaphore = new Semaphore(1);
        Semaphore emptySemaphore = new Semaphore(10);
        Semaphore fullSemaphore = new Semaphore(0);

        // Create producer and consumer threads
        Producer producer = new Producer(buffer, sharedDirectory, bufferSemaphore, emptySemaphore, fullSemaphore);
        Consumer consumer = new Consumer(buffer, bufferSemaphore, emptySemaphore, fullSemaphore);

        Thread producerThread = new Thread(producer);
        Thread consumerThread = new Thread(consumer);

        // Start the threads
        producerThread.start();
        consumerThread.start();
    }
}
