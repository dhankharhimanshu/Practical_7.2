Part B: CRUD Operations on Product Table
1. Database Setup
CREATE DATABASE shopdb;

USE shopdb;

CREATE TABLE Product (
  ProductID INT PRIMARY KEY,
  ProductName VARCHAR(50),
  Price DOUBLE,
  Quantity INT
);

2. Java Program (ProductCRUD.java)
import java.sql.*;
import java.util.Scanner;

public class ProductCRUD {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/shopdb";
        String user = "root";
        String pass = "password";
        Scanner sc = new Scanner(System.in);

        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            Connection con = DriverManager.getConnection(url, user, pass);
            con.setAutoCommit(false);

            while (true) {
                System.out.println("\n=== PRODUCT MENU ===");
                System.out.println("1. Add Product");
                System.out.println("2. View All Products");
                System.out.println("3. Update Product");
                System.out.println("4. Delete Product");
                System.out.println("5. Exit");
                System.out.print("Enter choice: ");
                int choice = sc.nextInt();

                switch (choice) {
                    case 1:
                        System.out.print("Enter ID, Name, Price, Quantity: ");
                        int id = sc.nextInt();
                        String name = sc.next();
                        double price = sc.nextDouble();
                        int qty = sc.nextInt();
                        PreparedStatement ps1 = con.prepareStatement("INSERT INTO Product VALUES (?, ?, ?, ?)");
                        ps1.setInt(1, id);
                        ps1.setString(2, name);
                        ps1.setDouble(3, price);
                        ps1.setInt(4, qty);
                        ps1.executeUpdate();
                        con.commit();
                        System.out.println("✅ Product added successfully!");
                        break;

                    case 2:
                        Statement st = con.createStatement();
                        ResultSet rs = st.executeQuery("SELECT * FROM Product");
                        System.out.println("ID\tName\tPrice\tQty");
                        while (rs.next()) {
                            System.out.println(rs.getInt(1) + "\t" + rs.getString(2) + "\t" +
                                               rs.getDouble(3) + "\t" + rs.getInt(4));
                        }
                        break;

                    case 3:
                        System.out.print("Enter ProductID and new Price: ");
                        int pid = sc.nextInt();
                        double newPrice = sc.nextDouble();
                        PreparedStatement ps2 = con.prepareStatement("UPDATE Product SET Price=? WHERE ProductID=?");
                        ps2.setDouble(1, newPrice);
                        ps2.setInt(2, pid);
                        ps2.executeUpdate();
                        con.commit();
                        System.out.println("✅ Product updated!");
                        break;

                    case 4:
                        System.out.print("Enter ProductID to delete: ");
                        int delId = sc.nextInt();
                        PreparedStatement ps3 = con.prepareStatement("DELETE FROM Product WHERE ProductID=?");
                        ps3.setInt(1, delId);
                        ps3.executeUpdate();
                        con.commit();
                        System.out.println("🗑️ Product deleted!");
                        break;

                    case 5:
                        con.close();
                        System.out.println("Exiting...");
                        return;

                    default:
                        System.out.println("Invalid choice!");
                }
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            sc.close();
        }
    }
}
Part C: Student Management System Using JDBC and MVC
1. Model — Student.java
package model;

public class Student {
    private int studentID;
    private String name;
    private String department;
    private double marks;

    public Student(int studentID, String name, String department, double marks) {
        this.studentID = studentID;
        this.name = name;
        this.department = department;
        this.marks = marks;
    }

    public int getStudentID() { return studentID; }
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getMarks() { return marks; }
}

⚙️ 2. Controller — StudentDAO.java
package controller;
import java.sql.*;
import model.Student;

public class StudentDAO {
    Connection con;

    public StudentDAO() throws Exception {
        Class.forName("com.mysql.cj.jdbc.Driver");
        con = DriverManager.getConnection("jdbc:mysql://localhost:3306/schooldb", "root", "password");
    }

    public void addStudent(Student s) throws SQLException {
        PreparedStatement ps = con.prepareStatement("INSERT INTO Student VALUES (?, ?, ?, ?)");
        ps.setInt(1, s.getStudentID());
        ps.setString(2, s.getName());
        ps.setString(3, s.getDepartment());
        ps.setDouble(4, s.getMarks());
        ps.executeUpdate();
        System.out.println("✅ Student added successfully!");
    }

    public void viewAll() throws SQLException {
        Statement st = con.createStatement();
        ResultSet rs = st.executeQuery("SELECT * FROM Student");
        System.out.println("ID\tName\tDept\tMarks");
        while (rs.next()) {
            System.out.println(rs.getInt(1) + "\t" + rs.getString(2) + "\t" +
                               rs.getString(3) + "\t" + rs.getDouble(4));
        }
    }

    public void updateMarks(int id, double marks) throws SQLException {
        PreparedStatement ps = con.prepareStatement("UPDATE Student SET Marks=? WHERE StudentID=?");
        ps.setDouble(1, marks);
        ps.setInt(2, id);
        ps.executeUpdate();
        System.out.println("✅ Marks updated!");
    }

    public void deleteStudent(int id) throws SQLException {
        PreparedStatement ps = con.prepareStatement("DELETE FROM Student WHERE StudentID=?");
        ps.setInt(1, id);
        ps.executeUpdate();
        System.out.println("🗑️ Student deleted!");
    }
}

3. View — StudentApp.java
package view;
import java.util.*;
import model.Student;
import controller.StudentDAO;

public class StudentApp {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        try {
            StudentDAO dao = new StudentDAO();
            while (true) {
                System.out.println("\n=== STUDENT MENU ===");
                System.out.println("1. Add Student");
                System.out.println("2. View All Students");
                System.out.println("3. Update Marks");
                System.out.println("4. Delete Student");
                System.out.println("5. Exit");
                System.out.print("Enter choice: ");
                int ch = sc.nextInt();

                switch (ch) {
                    case 1:
                        System.out.print("Enter ID, Name, Dept, Marks: ");
                        int id = sc.nextInt();
                        String name = sc.next();
                        String dept = sc.next();
                        double marks = sc.nextDouble();
                        dao.addStudent(new Student(id, name, dept, marks));
                        break;
                    case 2:
                        dao.viewAll();
                        break;
                    case 3:
                        System.out.print("Enter ID and new Marks: ");
                        int sid = sc.nextInt();
                        double newMarks = sc.nextDouble();
                        dao.updateMarks(sid, newMarks);
                        break;
                    case 4:
                        System.out.print("Enter ID to delete: ");
                        int delId = sc.nextInt();
                        dao.deleteStudent(delId);
                        break;
                    case 5:
                        System.out.println("Exiting...");
                        return;
                    default:
                        System.out.println("Invalid choice!");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            sc.close();
        }
    }
}
