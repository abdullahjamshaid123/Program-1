#include <iostream>
#include <vector>
#include <fstream>
#include <algorithm>
#include <stdexcept>
#include <map>
using namespace std;

// Encryption function (simple Caesar cipher for demonstration)
string encrypt(const string& data, int key) {
    string encrypted = data;
    for (char& c : encrypted) {
        if (isalpha(c)) {
            char base = isupper(c) ? 'A' : 'a';
            c = (c - base + key) % 26 + base;
        }
    }
    return encrypted;
}

// Decryption function
string decrypt(const string& data, int key) {
    return encrypt(data, 26 - key); // Reverse the Caesar cipher
}

// Class to store individual identity and family information
class Citizen {
private:
    string id;
    string name;
    int age;
    string gender;
    vector<string> familyMembers; // List of family members

public:
    Citizen(string id, string name, int age, string gender) : id(id), name(name), age(age), gender(gender) {}

    void addFamilyMember(const string& member) {
        familyMembers.push_back(member);
    }

    void display() const {
        cout << "ID: " << id << ", Name: " << name << ", Age: " << age << ", Gender: " << gender << endl;
        cout << "Family Members: ";
        for (const auto& member : familyMembers) {
            cout << member << " ";
        }
        cout << endl;
    }

    string getID() const { return id; }
    string getName() const { return name; }
    int getAge() const { return age; }
    string getGender() const { return gender; }
    vector<string> getFamilyMembers() const { return familyMembers; }

    // Serialize data for secure storage
    string serialize() const {
        string data = id + "," + name + "," + to_string(age) + "," + gender;
        for (const auto& member : familyMembers) {
            data += "," + member;
        }
        return data;
    }

    // Deserialize data from secure storage
    static Citizen deserialize(const string& data) {
        size_t pos1 = data.find(",");
        size_t pos2 = data.find(",", pos1 + 1);
        size_t pos3 = data.find(",", pos2 + 1);

        string id = data.substr(0, pos1);
        string name = data.substr(pos1 + 1, pos2 - pos1 - 1);
        int age = stoi(data.substr(pos2 + 1, pos3 - pos2 - 1));
        string gender = data.substr(pos3 + 1, data.find(",", pos3 + 1) - pos3 - 1);

        Citizen citizen(id, name, age, gender);

        size_t pos = data.find(",", pos3 + 1);
        while (pos != string::npos) {
            size_t nextPos = data.find(",", pos + 1);
            string member = data.substr(pos + 1, nextPos - pos - 1);
            citizen.addFamilyMember(member);
            pos = nextPos;
        }

        return citizen;
    }
};

// Secure Intelligence System
class IntelligenceSystem {
private:
    map<string, Citizen> database; // Stores citizens by ID
    int encryptionKey; // Secret key for encryption

public:
    IntelligenceSystem(int key) : encryptionKey(key) {}

    void addCitizen(const Citizen& citizen) {
        database[citizen.getID()] = citizen;
    }

    void displayCitizen(const string& id) const {
        auto it = database.find(id);
        if (it != database.end()) {
            it->second.display();
        } else {
            cout << "Citizen not found." << endl;
        }
    }

    void saveToFile(const string& filename) const {
        ofstream file(filename);
        if (file.is_open()) {
            for (const auto& entry : database) {
                string encryptedData = encrypt(entry.second.serialize(), encryptionKey);
                file << encryptedData << endl;
            }
            cout << "Data saved to " << filename << " securely." << endl;
        } else {
            cout << "Unable to save data to file." << endl;
        }
    }

    void loadFromFile(const string& filename) {
        ifstream file(filename);
        if (file.is_open()) {
            database.clear(); // Clear existing data
            string line;
            while (getline(file, line)) {
                string decryptedData = decrypt(line, encryptionKey);
                Citizen citizen = Citizen::deserialize(decryptedData);
                database[citizen.getID()] = citizen;
            }
            cout << "Data loaded from " << filename << " securely." << endl;
        } else {
            cout << "Unable to load data from file." << endl;
        }
    }
};

// Main function
int main() {
    IntelligenceSystem system(5); // Encryption key = 5
    int choice;
    string filename = "secure_data.txt";

    do {
        cout << "\nGovernment Intelligence System" << endl;
        cout << "1. Add Citizen" << endl;
        cout << "2. Display Citizen Information" << endl;
        cout << "3. Save Data to File" << endl;
        cout << "4. Load Data from File" << endl;
        cout << "0. Exit" << endl;
        cout << "Enter your choice: ";
        cin >> choice;

        try {
            switch (choice) {
                case 1: {
                    string id, name, gender;
                    int age;
                    cout << "Enter citizen ID: ";
                    cin >> id;
                    cout << "Enter name: ";
                    cin.ignore();
                    getline(cin, name);
                    cout << "Enter age: ";
                    cin >> age;
                    cout << "Enter gender: ";
                    cin >> gender;

                    Citizen citizen(id, name, age, gender);

                    char addMore;
                    do {
                        string member;
                        cout << "Enter family member name: ";
                        cin.ignore();
                        getline(cin, member);
                        citizen.addFamilyMember(member);

                        cout << "Add another family member? (y/n): ";
                        cin >> addMore;
                    } while (addMore == 'y' || addMore == 'Y');

                    system.addCitizen(citizen);
                    break;
                }
                case 2: {
                    string id;
                    cout << "Enter citizen ID: ";
                    cin >> id;
                    system.displayCitizen(id);
                    break;
                }
                case 3:
                    system.saveToFile(filename);
                    break;
                case 4:
                    system.loadFromFile(filename);
                    break;
                case 0:
                    cout << "Exiting the program. Goodbye!" << endl;
                    break;
                default:
                    cout << "Invalid choice. Please try again." << endl;
            }
        } catch (const exception& e) {
            cout << "Error: " << e.what() << endl;
        }
    } while (choice != 0);

    return 0;
}
