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

import java.util.*;
import java.util.stream.*;
import java.util.Comparator;

class Employee {
    String name;
    int age;
    double salary;

    Employee(String name, int age, double salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }

    public String toString() {
        return name + " | Age: " + age + " | Salary: " + salary;
    }
}

class Student {
    String name;
    double marks;

    Student(String name, double marks) {
        this.name = name;
        this.marks = marks;
    }

    public String toString() {
        return name + " | Marks: " + marks;
    }
}

class Product {
    String name;
    double price;
    String category;

    Product(String name, double price, String category) {
        this.name = name;
        this.price = price;
        this.category = category;
    }

    public String toString() {
        return name + " | " + category + " | Price: " + price;
    }
}

public class LambdaStreamDemo {
    public static void main(String[] args) {

        // ---------------- Part A: Sorting Employees ----------------
        System.out.println("=== Part A: Sorting Employee Objects ===");
        List<Employee> employees = Arrays.asList(
            new Employee("Ravi", 30, 55000),
            new Employee("Anita", 25, 65000),
            new Employee("Karan", 28, 48000),
            new Employee("Meena", 35, 72000)
        );

        System.out.println("\nSorted by Name (Alphabetical):");
        employees.stream()
                 .sorted((e1, e2) -> e1.name.compareTo(e2.name))
                 .forEach(System.out::println);

        System.out.println("\nSorted by Age (Ascending):");
        employees.stream()
                 .sorted(Comparator.comparingInt(e -> e.age))
                 .forEach(System.out::println);

        System.out.println("\nSorted by Salary (Descending):");
        employees.stream()
                 .sorted((e1, e2) -> Double.compare(e2.salary, e1.salary))
                 .forEach(System.out::println);

        // ---------------- Part B: Filtering and Sorting Students ----------------
        System.out.println("\n=== Part B: Filtering and Sorting Students ===");
        List<Student> students = Arrays.asList(
            new Student("Amit", 82.5),
            new Student("Neha", 68.4),
            new Student("Rohit", 91.0),
            new Student("Sita", 74.2),
            new Student("Vikas", 88.7)
        );

        System.out.println("\nStudents Scoring Above 75%, Sorted by Marks:");
        students.stream()
                .filter(s -> s.marks > 75)
                .sorted(Comparator.comparingDouble(s -> s.marks))
                .map(s -> s.name)
                .forEach(System.out::println);

        // ---------------- Part C: Stream Operations on Product Dataset ----------------
        System.out.println("\n=== Part C: Stream Operations on Product Dataset ===");
        List<Product> products = Arrays.asList(
            new Product("Laptop", 75000, "Electronics"),
            new Product("Phone", 50000, "Electronics"),
            new Product("Shoes", 4000, "Fashion"),
            new Product("Shirt", 2000, "Fashion"),
            new Product("TV", 65000, "Electronics"),
            new Product("Watch", 8000, "Fashion")
        );

        System.out.println("\nGrouping Products by Category:");
        Map<String, List<Product>> grouped = products.stream()
            .collect(Collectors.groupingBy(p -> p.category));
        grouped.forEach((cat, list) -> {
            System.out.println(cat + ": " + list);
        });

        System.out.println("\nMost Expensive Product in Each Category:");
        Map<String, Optional<Product>> maxByCategory = products.stream()
            .collect(Collectors.groupingBy(
                p -> p.category,
                Collectors.maxBy(Comparator.comparingDouble(p -> p.price))
            ));
        maxByCategory.forEach((cat, prod) ->
            System.out.println(cat + ": " + prod.get())
        );

        double avgPrice = products.stream()
            .collect(Collectors.averagingDouble(p -> p.price));
        System.out.println("\nAverage Price of All Products: " + avgPrice);
    }
}
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

