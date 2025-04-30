// Book.java public abstract class Book { protected String title; protected String author; protected String ISBN; protected boolean isAvailable;

public Book(String title, String author, String ISBN) {
    this.title = title;
    this.author = author;
    this.ISBN = ISBN;
    this.isAvailable = true;
}

public abstract String getType();

public String getTitle() { return title; }
public String getAuthor() { return author; }
public String getISBN() { return ISBN; }
public boolean isAvailable() { return isAvailable; }
public void setAvailable(boolean available) { this.isAvailable = available; }

@Override
public String toString() {
    return getType() + " - " + title + " by " + author + " (" + ISBN + ") - " + (isAvailable ? "متوفر" : "معار");
}

}

// PaperBook.java public class PaperBook extends Book { public PaperBook(String title, String author, String ISBN) { super(title, author, ISBN); }

@Override
public String getType() {
    return "كتاب ورقي";
}

}

// EBook.java public class EBook extends Book { public EBook(String title, String author, String ISBN) { super(title, author, ISBN); }

@Override
public String getType() {
    return "كتاب إلكتروني";
}

}

// Borrower.java import java.util.*;

public class Borrower { private String name; private String universityID; private List<Book> borrowedBooks;

public Borrower(String name, String universityID) {
    this.name = name;
    this.universityID = universityID;
    this.borrowedBooks = new ArrayList<>();
}

public String getName() { return name; }
public String getUniversityID() { return universityID; }
public List<Book> getBorrowedBooks() { return borrowedBooks; }

public void borrowBook(Book book) {
    borrowedBooks.add(book);
    book.setAvailable(false);
}

public void returnBook(Book book) {
    borrowedBooks.remove(book);
    book.setAvailable(true);
}

@Override
public String toString() {
    return name + " (" + universityID + ")";
}

}

// BorrowingProcess.java import java.time.LocalDate;

public class BorrowingProcess { private Book book; private Borrower borrower; private LocalDate borrowDate; private LocalDate returnDate;

public BorrowingProcess(Book book, Borrower borrower) {
    this.book = book;
    this.borrower = borrower;
    this.borrowDate = LocalDate.now();
}

public void setReturnDate() {
    this.returnDate = LocalDate.now();
}

@Override
public String toString() {
    return borrower + " استعار الكتاب " + book.getTitle() + " في " + borrowDate + (returnDate != null ? ", وأرجعه في " + returnDate : "");
}

}

// LibraryManager.java import java.util.*;

public class LibraryManager { private static List<Book> books = new ArrayList<>(); private static List<Borrower> borrowers = new ArrayList<>(); private static List<BorrowingProcess> processes = new ArrayList<>(); private static Scanner scanner = new Scanner(System.in);

public static void main(String[] args) {
    while (true) {
        System.out.println("\n*** نظام إدارة المكتبة ***");
        System.out.println("1. إضافة كتاب");
        System.out.println("2. إضافة مستعير");
        System.out.println("3. إعارة كتاب");
        System.out.println("4. استرجاع كتاب");
        System.out.println("5. البحث عن كتاب أو مستعير");
        System.out.println("6. عرض الكتب المستعارة من مستعير");
        System.out.println("7. إنهاء");
        System.out.print("اختيارك: ");
        int choice = scanner.nextInt();
        scanner.nextLine();
        switch (choice) {
            case 1 -> addBook();
            case 2 -> addBorrower();
            case 3 -> lendBook();
            case 4 -> returnBook();
            case 5 -> search();
            case 6 -> showBorrowedBooks();
            case 7 -> System.exit(0);
            default -> System.out.println("خيار غير صالح");
        }
    }
}

private static void addBook() {
    System.out.print("العنوان: ");
    String title = scanner.nextLine();
    System.out.print("المؤلف: ");
    String author = scanner.nextLine();
    System.out.print("رقم ISBN: ");
    String isbn = scanner.nextLine();
    System.out.print("نوع الكتاب (1: ورقي, 2: إلكتروني): ");
    int type = scanner.nextInt();
    scanner.nextLine();
    Book book = (type == 1) ? new PaperBook(title, author, isbn) : new EBook(title, author, isbn);
    books.add(book);
    System.out.println("تمت إضافة الكتاب.");
}

private static void addBorrower() {
    System.out.print("اسم المستعير: ");
    String name = scanner.nextLine();
    System.out.print("الرقم الجامعي: ");
    String id = scanner.nextLine();
    borrowers.add(new Borrower(name, id));
    System.out.println("تمت إضافة المستعير.");
}

private static void lendBook() {
    System.out.print("رقم ISBN للكتاب: ");
    String isbn = scanner.nextLine();
    Book book = findBook(isbn);
    if (book == null || !book.isAvailable()) {
        System.out.println("الكتاب غير متاح.");
        return;
    }
    System.out.print("الرقم الجامعي للمستعير: ");
    String id = scanner.nextLine();
    Borrower borrower = findBorrower(id);
    if (borrower == null) {
        System.out.println("المستعير غير موجود.");
        return;
    }
    borrower.borrowBook(book);
    processes.add(new BorrowingProcess(book, borrower));
    System.out.println("تمت الإعارة.");
}

private static void returnBook() {
    System.out.print("رقم ISBN للكتاب: ");
    String isbn = scanner.nextLine();
    Book book = findBook(isbn);
    if (book == null || book.isAvailable()) {
        System.out.println("الكتاب غير معار.");
        return;
    }
    System.out.print("الرقم الجامعي للمستعير: ");
    String id = scanner.nextLine();
    Borrower borrower = findBorrower(id);
    if (borrower == null) {
        System.out.println("المستعير غير موجود.");
        return;
    }
    borrower.returnBook(book);
    for (BorrowingProcess bp : processes) {
        if (bp.toString().contains(isbn) && bp.toString().contains(id)) {
            bp.setReturnDate();
            break;
        }
    }
    System.out.println("تم الاسترجاع.");
}

private static void search() {
    System.out.print("كلمة البحث: ");
    String keyword = scanner.nextLine().toLowerCase();
    System.out.println("\nنتائج البحث في الكتب:");
    books.stream().filter(b -> b.getTitle().toLowerCase().contains(keyword) || b.getAuthor().toLowerCase().contains(keyword))
         .forEach(System.out::println);
    System.out.println("\nنتائج البحث في المستعيرين:");
    borrowers.stream().filter(b -> b.getName().toLowerCase().contains(keyword))
             .forEach(System.out::println);
}

private static void showBorrowedBooks() {
    System.out.print("الرقم الجامعي للمستعير: ");
    String id = scanner.nextLine();
    Borrower borrower = findBorrower(id);
    if (borrower == null) {
        System.out.println("المستعير غير موجود.");
        return;
    }
    System.out.println("الكتب المستعارة:");
    borrower.getBorrowedBooks().forEach(System.out::println);
}

private static Book findBook(String isbn) {
    return books.stream().filter(b -> b.getISBN().equals(isbn)).findFirst().orElse(null);
}

private static Borrower findBorrower(String id) {
    return borrowers.stream().filter(b -> b.getUniversityID().equals(id)).findFirst().orElse(null);
}

}

