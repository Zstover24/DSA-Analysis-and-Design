# DSA-Analysis-and-Design
**Working code for sorting and printing list of courses.**
#include <fstream>
#include <string>
#include <iostream>
#include <algorithm>
#include <climits>
#include <vector>
#include "CSVparser.hpp"


using namespace std;

const unsigned int DEFAULT_SIZE = 179;

struct Course {
    string courseNumber;
    string title;
    vector <string> prerequisites;
};

class HashTable {

private:
    // Define structures to hold courses
    struct Node {
        Course course;
        unsigned int key;
        Node* next;

        // default constructor
        Node() {
            key = UINT_MAX;
            next = nullptr;
        }

        // initialize with a course
        Node(Course aCourse) : Node() {
            course = aCourse;
        }

        // initialize with a course and a key
        Node(Course aCourse, unsigned int aKey) : Node(aCourse) {
            key = aKey;
        }
    };

    vector<Node> nodes;

    unsigned int tableSize = DEFAULT_SIZE;

    unsigned int hash(int key);

public:
    HashTable();
    HashTable(unsigned int size);
    virtual ~HashTable();
    void Insert(Course course);
    void PrintAll();
    Course Search(string courseNumber);
};

// default constructor
HashTable::HashTable() {
    // Initalize node structure by resizing tableSize
    nodes.resize(tableSize);
}

// Constructor for specifying size of the table
HashTable::HashTable(unsigned int size) {
    // invoke local tableSize to size with this->
    tableSize = size;

    // resize nodes size
    nodes.resize(tableSize);
}

// Destructor
HashTable::~HashTable() {
    // erase nodes beginning
    nodes.erase(nodes.begin());
}

// Calculate the hash value of a given key.
unsigned int HashTable::hash(int key) {
    // return key tableSize
    return key % tableSize;
}

// Insert a course
void HashTable::Insert(Course course) {
    // create the key for the given course
    unsigned key = hash(atoi(course.courseNumber.c_str()));

    Node* newNode = new Node(course, key);
    // retrieve node using key
    Node* oldNode = &nodes[key];

    // if node is not used
    if (oldNode->key == UINT_MAX) {
        *oldNode = *newNode;
    }

    // else find the next open node
    else {
        while (oldNode->next != nullptr) {
            oldNode = oldNode->next;
        }

        // add new newNode to end
        oldNode->next = new Node(course, key);
    }

}

// Print all courses
void HashTable::PrintAll() {
    // for node begin to end iterate
    for (auto it = nodes.begin(); it != nodes.end(); ++it) {
        Node* node = &(*it);

        //   if key not equal to UINT_MAX
        if (node->key != UINT_MAX) {

            // Output course
            cout << " Course Number: " << node->course.courseNumber << " Title: " << node->course.title << endl;

            // node is equal to next node
            Node* nextNode = node->next;
            while (nextNode != nullptr) {

                cout << " Course Number: " << nextNode->course.courseNumber << " Title: " << nextNode->course.title << endl;
                // node is equal to next node
                nextNode = nextNode->next;
            }


        }

    }
}

// Search for the specified courseNumber
Course HashTable::Search(string courseNumber) {

    Course course;


    // create the key for the given course
    unsigned key = hash(atoi(courseNumber.c_str()));

    Node* node = &(nodes.at(key));



    while (node != nullptr) {

        // if entry found for the key
        if (node->key == key && node->course.courseNumber == courseNumber) {

            return node->course;  // Found the course

        }



        // Check prerequisites

        for (const auto& prereq : node->course.prerequisites) {

            if (prereq == courseNumber) {

                return node->course;  // Found as a prerequisite

            }

        }



        node = node->next;

    }



    // Course not found

    return course;

}

// Display the course information
void displayCourses(Course course) {
    cout << course.courseNumber << ", " << course.title << ", ";
    for (auto prereq : course.prerequisites) {
        cout << prereq << ", " << endl;
    }

    return;
}

// Load a CSV file containing courses into a container
void loadCourses(string csvPath, HashTable* hashTable) {
    cout << "Loading CSV file " << csvPath << endl;

    // initialize the CSV Parser using the given path
    csv::Parser file = csv::Parser(csvPath);

    

    try {
        // loop to read rows of a CSV file
        for (unsigned int i = 0; i < file.rowCount(); i++) {

            Course course;

            // Create a data structure and add to the collection of courses
            course.courseNumber = file[i][0];
            course.title = file[i][1];
            //course.prerequisites = file[i][j];
            for (unsigned int j = 2; j < file[i].size(); j++) {
                course.prerequisites.push_back(file[i][j]);
            }

            // push this course to the end
            hashTable->Insert(course);
        }
    }
    catch (csv::Error& e) {
        std::cerr << e.what() << std::endl;
    }
}

int main(int argc, char* argv[]) {

    // process command line arguments
    string csvPath, courseKey;
    switch (argc) {
    case 2:
        csvPath = argv[1];
        courseKey = "";
        break;
    case 3:
        csvPath = argv[1];
        courseKey = argv[2];
        break;
    default:
        csvPath = "Project_Two_Test.txt";
        courseKey = "CSCI300";
        break;
    }
    // Define a hash table to hold all the courses
    HashTable* courseTable;

    Course course;
    courseTable = new HashTable();
    string cNumber;
    int choice = 0;
    while (choice != 9) {
        cout << "Welcome to the course planner!" << endl;
        cout << "  1. Load Data Structure." << endl;
        cout << "  2. Print Course List." << endl;
        cout << "  3. Print Course." << endl;
        cout << "  9. Exit." << endl;
        cout << "What would you like to do?";
        cin >> choice;

        cout << "Current choice: " << choice << endl; 
        if (choice == 3) {
            cout << "What course would you like to know about?" << endl;
            cin >> courseKey;
        }

        switch (choice) {
        case 1:

            loadCourses(csvPath, courseTable);

            break;

        case 2:
            courseTable->PrintAll();

            break;

        case 3:

            course = courseTable->Search(courseKey);

            if (!course.courseNumber.empty()) {
                displayCourses(course);
            }

            else {
                cout << "Course Number " << courseKey << " not found." << endl;
            }

            break;

        case 9:
            cout << "Exiting the course planner." << endl;
            break;

        default:
            cout << "Invalid choice. Please choose a valid option." << endl;

            break;
        }
    }

    cout << "Thank you for using the course planner" << endl;

    return 0;
}


The problem i was solving was giving academic advisors a program to load courses into that would then sort and store them to be searched and printed at a later date. I approached this by using a hashtable i know now that a BST would have been better for sorting and searching the list and a vector would have been easier to implement but hopefully i can use this as a learning experience moving forward and apply this in another project or in a real worl problem. At first i had quite a few issues with my code that took about a week to get all of the kinks out. Persistence and sometimes taking a step back and taking a break so i can reflect on what may actually be causing the problems was extremely helpful in overcoming all of my challenges. 
