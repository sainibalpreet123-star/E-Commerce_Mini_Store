#include <iostream>
#include <fstream>
#include <vector>
#include <iomanip>
#include <string>
#include <stdexcept>
#include <sstream>
using namespace std;

class Product {
public:
    int id;
    string name;
    double price;
    int quantity;

    Product() {}
    Product(int i, string n, double p, int q) {
        id = i; name = n; price = p; quantity = q;
    }

    void display() const {
        cout << left << setw(10) << id << setw(20) << name 
             << setw(10) << price << setw(10) << quantity << endl;
    }
};

class Admin {
public:
    void addProduct() {
        ofstream fout("products.txt", ios::app);
        if (!fout) {
            cout << "Error opening file.\n";
            return;
        }

        Product p;
        cout << "Enter Product ID: "; cin >> p.id;
        cin.ignore();
        cout << "Enter Product Name: "; getline(cin, p.name);
        cout << "Enter Price: "; cin >> p.price;
        cout << "Enter Quantity: "; cin >> p.quantity;

        fout << p.id << "," << p.name << "," << p.price << "," << p.quantity << "\n";
        fout.close();
        cout << "Product Added Successfully!\n";
    }

    void viewProducts() {
        ifstream fin("products.txt");
        if (!fin) {
            cout << "No Products Found.\n";
            return;
        }
        string line;
        cout << left << setw(10) << "ID" << setw(20) << "Name" 
             << setw(10) << "Price" << setw(10) << "Qty" << endl;
        cout << "--------------------------------------------------------\n";
        while (getline(fin, line)) {
            stringstream ss(line);
            string id, name, price, qty;
            getline(ss, id, ',');
            getline(ss, name, ',');
            getline(ss, price, ',');
            getline(ss, qty, ',');
            cout << left << setw(10) << id << setw(20) << name 
                 << setw(10) << price << setw(10) << qty << endl;
        }
        fin.close();
    }
};

class Customer {
    vector<Product> cart;

public:
    void showProducts() {
        ifstream fin("products.txt");
        if (!fin) {
            cout << "No Products Available.\n";
            return;
        }
        cout << left << setw(10) << "ID" << setw(20) << "Name" 
             << setw(10) << "Price" << setw(10) << "Qty" << endl;
        cout << "--------------------------------------------------------\n";
        string line;
        while (getline(fin, line)) {
            stringstream ss(line);
            Product p;
            string id, name, price, qty;
            getline(ss, id, ',');
            getline(ss, name, ',');
            getline(ss, price, ',');
            getline(ss, qty, ',');
            p.id = stoi(id);
            p.name = name;
            p.price = stod(price);
            p.quantity = stoi(qty);
            p.display();
        }
        fin.close();
    }

    void addToCart() {
        int pid, qty;
        cout << "Enter Product ID to Add: ";
        cin >> pid;
        cout << "Enter Quantity: ";
        cin >> qty;

        ifstream fin("products.txt");
        string line;
        bool found = false;
        while (getline(fin, line)) {
            stringstream ss(line);
            Product p;
            string id, name, price, quantity;
            getline(ss, id, ',');
            getline(ss, name, ',');
            getline(ss, price, ',');
            getline(ss, quantity, ',');
            if (stoi(id) == pid) {
                found = true;
                if (qty <= stoi(quantity)) {
                    cart.push_back(Product(stoi(id), name, stod(price), qty));
                    cout << "Added to Cart!\n";
                } else {
                    cout << "Not enough stock!\n";
                }
                break;
            }
        }
        if (!found) cout << "Product Not Found.\n";
        fin.close();
    }

    void generateBill() {
        double total = 0;
        cout << "\n---------- BILL ----------\n";
        cout << left << setw(10) << "ID" << setw(20) << "Name"
             << setw(10) << "Qty" << setw(10) << "Price" << setw(10) << "Total\n";
        cout << "------------------------------------------------------------\n";
        for (auto &p : cart) {
            double t = p.price * p.quantity;
            cout << left << setw(10) << p.id << setw(20) << p.name 
                 << setw(10) << p.quantity << setw(10) << p.price 
                 << setw(10) << t << endl;
            total += t;
        }
        cout << "------------------------------------------------------------\n";
        cout << "Grand Total: Rs. " << total << endl;

        ofstream fout("bill.txt", ios::app);
        fout << "Total: " << total << endl;
        fout.close();
        cout << "Bill Saved to bill.txt\n";
    }
};

class LoginSystem {
public:
    void registerUser() {
        string user, pass;
        cout << "Enter Username: ";
        cin >> user;
        cout << "Enter Password: ";
        cin >> pass;

        ofstream fout("users.txt", ios::app);
        fout << user << "," << pass << "\n";
        fout.close();
        cout << "Registration Successful!\n";
    }

    bool login() {
        string user, pass, u, p, line;
        cout << "Username: "; cin >> user;
        cout << "Password: "; cin >> pass;

        ifstream fin("users.txt");
        while (getline(fin, line)) {
            stringstream ss(line);
            getline(ss, u, ',');
            getline(ss, p, ',');
            if (u == user && p == pass) {
                cout << "Login Successful!\n";
                return true;
            }
        }
        cout << "Invalid Credentials!\n";
        return false;
    }
};

int main() {
    LoginSystem loginSys;
    Admin admin;
    Customer cust;
    int choice;

    while (true) {
        cout << "\n====== E-Commerce Mini Store ======\n";
        cout << "1. Register\n2. Login\n3. Admin Panel\n4. Exit\n";
        cout << "Enter Choice: ";
        cin >> choice;

        try {
            if (choice == 1) loginSys.registerUser();
            else if (choice == 2) {
                if (loginSys.login()) {
                    int c;
                    while (true) {
                        cout << "\n1. View Products\n2. Add to Cart\n3. Generate Bill\n4. Logout\n";
                        cout << "Enter Choice: ";
                        cin >> c;
                        if (c == 1) cust.showProducts();
                        else if (c == 2) cust.addToCart();
                        else if (c == 3) cust.generateBill();
                        else if (c == 4) break;
                        else throw invalid_argument("Invalid Menu Option!");
                    }
                }
            }
            else if (choice == 3) {
                int code;
                cout << "Enter Admin Passcode: ";
                cin >> code;
                if (code == 1234) {
                    int a;
                    while (true) {
                        cout << "\n1. Add Product\n2. View Products\n3. Exit Admin\n";
                        cout << "Enter Choice: ";
                        cin >> a;
                        if (a == 1) admin.addProduct();
                        else if (a == 2) admin.viewProducts();
                        else if (a == 3) break;
                        else throw invalid_argument("Invalid Option!");
                    }
                } else cout << "Wrong Passcode!\n";
            }
            else if (choice == 4) {
                cout << "Thank you for visiting!\n";
                break;
            } else {
                throw invalid_argument("Invalid Input!");
            }
        } catch (exception &e) {
            cout << "Error: " << e.what() << endl;
        }
    }
    return 0;
}
 
