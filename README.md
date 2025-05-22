#include <iostream>
#include <fstream>
#include <string>
using namespace std;

struct Tugas {
    string nama;
    string deadline; 
    string matkul;
    bool selesai;
    Tugas* next; 
};

Tugas* head = nullptr; 

int ubahTanggalKeAngka(const string& tgl) {
    int d, m, y;
    sscanf(tgl.c_str(), "%d-%d-%d", &d, &m, &y); 
    return y * 10000 + m * 100 + d; 
}

Tugas* buatTugas(string nama, string deadline, string matkul) { 
    Tugas* t = new Tugas{nama, deadline, matkul, false, nullptr}; 
    return t; 
}

void tambahTugas(string nama, string deadline, string matkul) { 
    Tugas* baru = buatTugas(nama, deadline, matkul); 
    baru->next = head; 
    head = baru; 
    cout << "Tugas ditambahkan\n";
}
