#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <string>
#include <algorithm>
#include <limits>
#include <cctype>
using namespace std;

// ==============================
// Book Class
// ==============================
struct Book {
    int bookID;
    string title;
    string author;
    bool isIssued;

    Book() : bookID(0), title(""), author(""), isIssued(false) {}
    Book(int id, const string &t, const string &a, bool issued = false)
        : bookID(id), title(t), author(a), isIssued(issued) {}

    string serialize() const {
        ostringstream oss;
        oss << bookID << '|' << title << '|' << author << '|' << (isIssued ? 1 : 0);
        return oss.str();
    }

    static Book deserialize(const string &line) {
        Book b;
        istringstream iss(line);
        string token;
        getline(iss, token, '|'); b.bookID = stoi(token);
        getline(iss, b.title, '|');
        getline(iss, b.author, '|');
        getline(iss, token, '|'); b.isIssued = (token == "1");
        return b;
    }
};

// ==============================
// Student Class
// ==============================
struct Student {
    string studentID;
    string name;
    vector<int> issuedBooks;

    Student() : studentID(""), name("") {}
    Student(const string &id, const string &n) : studentID(id), name(n) {}

    string serialize() const {
        ostringstream oss;
        oss << studentID << '|' << name << '|';
        for (size_t i = 0; i < issuedBooks.size(); ++i) {
            if (i) oss << ',';
            oss << issuedBooks[i];
        }
        return oss.str();
    }

    static Student deserialize(const string &line) {
        Student s;
        istringstream iss(line);
        string token;
        getline(iss, s.studentID, '|');
        getline(iss, s.name, '|');
        getline(iss, token, '|');
        if (!token.empty()) {
            istringstream bs(token);
            string btok;
            while (getline(bs, btok, ',')) s.issuedBooks.push_back(stoi(btok));
        }
        return s;
    }
};

// ==============================
// Transaction Class
// ==============================
struct Transaction {
    int bookID;
    string studentID;
    string issueDate;
    string dueDate;
    string returnDate;
    bool returned;

    Transaction() : bookID(0), studentID(""), issueDate(""), dueDate(""), returnDate(""), returned(false) {}
    Transaction(int b, const string &s, const string &iss, const string &due)
        : bookID(b), studentID(s), issueDate(iss), dueDate(due), returnDate(""), returned(false) {}

    string serialize() const {
        ostringstream oss;
        oss << bookID << '|' << studentID << '|' << issueDate << '|' << dueDate << '|' << returnDate << '|' << (returned ? 1 : 0);
        return oss.str();
    }

    static Transaction deserialize(const string &line) {
        Transaction t;
        istringstream iss(line);
        string token;
        getline(iss, token, '|'); t.bookID = stoi(token);
        getline(iss, t.studentID, '|');
        getline(iss, t.issueDate, '|');
        getline(iss, t.dueDate, '|');
        getline(iss, t.returnDate, '|');
        getline(iss, token, '|'); t.returned = (token == "1");
        return t;
    }
};

// ==============================
// Utils Namespace
// ==============================
namespace Utils {
    string trim(const string &s) {
        size_t a = s.find_first_not_of(" \t\r\n");
        if (a == string::npos) return "";
        size_t b = s.find_last_not_of(" \t\r\n");
        return s.substr(a, b - a + 1);
    }

    bool validDateFormat(const string &d) {
        if (d.size() != 10) return false;
        if (d[4] != '-' || d[7] != '-') return false;
        for (size_t i = 0; i < d.size(); ++i) {
            if (i == 4 || i == 7) continue;
            if (!isdigit(static_cast<unsigned char>(d[i]))) return false;
        }
        return true;
    }

    int daysBetween(const string &d1, const string &d2) {
        if (!validDateFormat(d1) || !validDateFormat(d2)) return 0;
        int y1, m1, day1, y2, m2, day2;
        char dash;
        istringstream s1(d1), s2(d2);
        s1 >> y1 >> dash >> m1 >> dash >> day1;
        s2 >> y2 >> dash >> m2 >> dash >> day2;
        int days1 = y1 * 365 + m1 * 30 + day1;
        int days2 = y2 * 365 + m2 * 30 + day2;
        return days2 - days1;
    }
}

// ==============================
// Library Class
// ==============================
class Library {
private:
    vector<Book> books;
    vector<Student> students;
    vector<Transaction> transactions;

    const string booksFile = "books.txt";
    const string studentsFile = "students.txt";
    const string transactionsFile = "transactions.txt";

    int findBookIndex(int bookID) {
        for (size_t i = 0; i < books.size(); ++i)
            if (books[i].bookID == bookID) return (int)i;
        return -1;
    }

    int findStudentIndex(const string &studentID) {
        for (size_t i = 0; i < students.size(); ++i)
            if (students[i].studentID == studentID) return (int)i;
        return -1;
    }

public:
    Library() { loadAll(); }

    void loadAll() {
        books.clear(); students.clear(); transactions.clear();
        string line;

        ifstream finb(booksFile);
        while (getline(finb, line)) {
            line = Utils::trim(line);
            if (!line.empty()) books.push_back(Book::deserialize(line));
        }
        finb.close();

        ifstream fins(studentsFile);
        while (getline(fins, line)) {
            line = Utils::trim(line);
            if (!line.empty()) students.push_back(Student::deserialize(line));
        }
        fins.close();

        ifstream fint(transactionsFile);
        while (getline(fint, line)) {
            line = Utils::trim(line);
            if (!line.empty()) transactions.push_back(Transaction::deserialize(line));
        }
        fint.close();
    }

    void saveAll() {
        ofstream foutb(booksFile);
        for (const auto &b : books) foutb << b.serialize() << '\n';
        foutb.close();

        ofstream fouts(studentsFile);
        for (const auto &s : students) fouts << s.serialize() << '\n';
        fouts.close();

        ofstream foutt(transactionsFile);
        for (const auto &t : transactions) foutt << t.serialize() << '\n';
        foutt.close();
    }

    bool addBook(const Book &b) {
        if (findBookIndex(b.bookID) != -1) return false;
        books.push_back(b); saveAll(); return true;
    }

    bool deleteBook(int bookID) {
        int idx = findBookIndex(bookID);
        if (idx == -1 || books[idx].isIssued) return false;
        books.erase(books.begin() + idx);
        saveAll();
        return true;
    }

    vector<Book> listBooks() const { return books; }
    Book* getBookByID(int bookID) {
        int idx = findBookIndex(bookID);
        return idx == -1 ? nullptr : &books[idx];
    }

    bool addStudent(const Student &s) {
        if (findStudentIndex(s.studentID) != -1) return false;
        students.push_back(s); saveAll(); return true;
    }

    Student* getStudentByID(const string &studentID) {
        int idx = findStudentIndex(studentID);
        return idx == -1 ? nullptr : &students[idx];
    }

    bool issueBook(int bookID, const string &studentID, const string &issueDate, const string &dueDate) {
        int bidx = findBookIndex(bookID);
        int sidx = findStudentIndex(studentID);
        if (bidx == -1 || sidx == -1 || books[bidx].isIssued) return false;

        books[bidx].isIssued = true;
        students[sidx].issuedBooks.push_back(bookID);
        transactions.push_back(Transaction(bookID, studentID, issueDate, dueDate));
        saveAll();
        return true;
    }

    bool returnBook(int bookID, const string &studentID, const string &returnDate, int &fine) {
        fine = 0;
        int bidx = findBookIndex(bookID);
        int sidx = findStudentIndex(studentID);
        if (bidx == -1 || sidx == -1) return false;

        int tidx = -1;
        for (size_t i = 0; i < transactions.size(); ++i) {
            if (transactions[i].bookID == bookID && transactions[i].studentID == studentID && !transactions[i].returned) {
                tidx = (int)i; break;
            }
        }
        if (tidx == -1) return false;

        books[bidx].isIssued = false;
        auto &vec = students[sidx].issuedBooks;
        auto it = find(vec.begin(), vec.end(), bookID);
        if (it != vec.end()) vec.erase(it);

        transactions[tidx].returnDate = returnDate;
        transactions[tidx].returned = true;

        int diff = Utils::daysBetween(transactions[tidx].dueDate, returnDate);
        if (diff > 0) fine = diff * 5;

        saveAll();
        return true;
    }
};

// ==============================
// MAIN PROGRAM
// ==============================
void pause() {
    cout << "\nPress Enter to continue...";
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
}

void adminMenu(Library &lib);
void studentMenu(Library &lib);

int main() {
    Library lib;
    while (true) {
        cout << "\n==== SMART LIBRARY MENU ====\n";
        cout << "1. Admin Login\n2. Student Login\n3. Register Student\n4. Exit\nEnter choice: ";
        int ch; cin >> ch; cin.ignore(numeric_limits<streamsize>::max(), '\n');
        if (ch == 1) {
            string pwd; cout << "Enter admin password: "; getline(cin, pwd);
            if (pwd == "admin") adminMenu(lib);
            else cout << "Wrong password!\n";
        } else if (ch == 2) studentMenu(lib);
        else if (ch == 3) {
            string sid, name;
            cout << "Enter Student ID: "; getline(cin, sid);
            cout << "Enter Name: "; getline(cin, name);
            if (lib.addStudent(Student(sid, name))) cout << "Student registered.\n";
            else cout << "Student already exists.\n";
        } else if (ch == 4) break;
        else cout << "Invalid option!\n";
    }
    return 0;
}

void adminMenu(Library &lib) {
    while (true) {
        cout << "\n--- ADMIN MENU ---\n1. Add Book\n2. Delete Book\n3. View Books\n4. Back\nEnter choice: ";
        int ch; cin >> ch; cin.ignore(numeric_limits<streamsize>::max(), '\n');
        if (ch == 1) {
            int id; string title, author;
            cout << "Book ID: "; cin >> id; cin.ignore(numeric_limits<streamsize>::max(), '\n');
            cout << "Title: "; getline(cin, title);
            cout << "Author: "; getline(cin, author);
            if (lib.addBook(Book(id, title, author))) cout << "Book added.\n"; else cout << "Book ID exists!\n";
        } else if (ch == 2) {
            int id; cout << "Book ID to delete: "; cin >> id; cin.ignore();
            if (lib.deleteBook(id)) cout << "Deleted.\n"; else cout << "Error deleting book.\n";
        } else if (ch == 3) {
            for (auto &b : lib.listBooks())
                cout << b.bookID << " | " << b.title << " | " << b.author << " | Issued: " << (b.isIssued ? "Yes" : "No") << "\n";
        } else if (ch == 4) break;
        pause();
    }
}

void studentMenu(Library &lib) {
    string sid;
    cout << "Enter Student ID: "; getline(cin, sid);
    Student *s = lib.getStudentByID(sid);
    if (!s) { cout << "Student not found!\n"; return; }

    while (true) {
        cout << "\n--- STUDENT MENU (" << s->name << ") ---\n";
        cout << "1. View All Books\n2. Issue Book\n3. Return Book\n4. My Books\n5. Back\nEnter choice: ";
        int ch; cin >> ch; cin.ignore(numeric_limits<streamsize>::max(), '\n');
        if (ch == 1) {
            for (auto &b : lib.listBooks())
                cout << b.bookID << " | " << b.title << " | " << b.author << " | Issued: " << (b.isIssued ? "Yes" : "No") << "\n";
        } else if (ch == 2) {
            int id; string issueDate, dueDate;
            cout << "Book ID: "; cin >> id; cin.ignore();
            cout << "Issue Date (YYYY-MM-DD): "; getline(cin, issueDate);
            cout << "Due Date (YYYY-MM-DD): "; getline(cin, dueDate);
            if (lib.issueBook(id, sid, issueDate, dueDate)) cout << "Issued successfully.\n";
            else cout << "Issue failed.\n";
        } else if (ch == 3) {
            int id, fine; string returnDate;
            cout << "Book ID: "; cin >> id; cin.ignore();
            cout << "Return Date (YYYY-MM-DD): "; getline(cin, returnDate);
            if (lib.returnBook(id, sid, returnDate, fine)) cout << "Returned. Fine: " << fine << "\n";
            else cout << "Return failed.\n";
        } else if (ch == 4) {
            for (int bid : s->issuedBooks) {
                Book *b = lib.getBookByID(bid);
                if (b) cout << b->bookID << " | " << b->title << " | " << b->author << "\n";
            }
        } else if (ch == 5) break;
        pause();
    }
}
