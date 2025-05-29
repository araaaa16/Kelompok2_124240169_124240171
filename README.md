#include <iostream>     // Input/output
#include <fstream>      // File handling
#include <string>       // Tipe data string
using namespace std;

struct Tugas {
    string nama;
    string deadline;    // Format: DD-MM-YYYY
    string matkul;
    bool selesai;
    Tugas* next;        // POINTER - menunjuk ke node berikutnya (linked list)
};

// POINTER - Penunjuk ke node pertama dalam linked list
Tugas* head = nullptr;

// FUNCTION: ngubah tanggal dari format DD-MM-YYYY menjadi angka YYYYMMDD
int ubahTanggalKeAngka(const string& tgl) {
    int d, m, y;
    sscanf(tgl.c_str(), "%d-%d-%d", &d, &m, &y);
    return y * 10000 + m * 100 + d;
}

// POINTER + LINKED LIST - buat node tugas baru
Tugas* buatTugas(string nama, string deadline, string matkul) {
    Tugas* t = new Tugas{nama, deadline, matkul, false, nullptr};
    return t;
}

// LINKED LIST - menambahkan tugas ke awal linked list
void tambahTugas(string nama, string deadline, string matkul) {
    Tugas* baru = buatTugas(nama, deadline, matkul);
    baru->next = head;
    head = baru;
    cout << "Tugas ditambahkan.\n";
}

// LINKED LIST - menampilkan semua tugas dengan tampilan tabel
void tampilkanTugas() {
    if (!head) {
        cout << "Tidak ada tugas.";
        return;
    }

   cout << "\n----------------------------------------------------------------------------------------\n";
    cout << "| No | Nama Tugas                    | Deadline      | Mata Kuliah        | Status     |\n";
    cout << "---------------------------------------------------------------------------------------\n";

    Tugas* t = head;
    int i = 1;
    while (t) {
        printf("| %2d | %-29s | %-13s | %-18s | %-10s |\n", i++,
               t->nama.c_str(), t->deadline.c_str(), t->matkul.c_str(),
               t->selesai ? "Selesai" : "Belum");
        t = t->next;
    }
    cout << "----------------------------------------------------------------------------------------\n";
}

// SEARCHING - mencari tugas berdasarkan kata kunci
void cariTugas(string kunci) {
    Tugas* t = head;
    bool ditemukan = false;
    while (t) {
        if (t->nama.find(kunci) != string::npos) {
            cout << "Ditemukan: " << t->nama << " | " << t->deadline
                 << " | " << t->matkul << " | "
                 << (t->selesai ? "Selesai" : "Belum") << endl;
            ditemukan = true;
        }
        t = t->next;
    }
    if (!ditemukan)
        cout << "Tugas tidak ditemukan.\n";
}

// EDITING - edit informasi tugas
void editTugas(string namaLama) {
    Tugas* t = head;
    while (t) {
        if (t->nama == namaLama) {
            cout << "Edit tugas\n";
            cout << "Nama baru: "; getline(cin, t->nama);
            cout << "Deadline baru (DD-MM-YYYY): "; getline(cin, t->deadline);
            cout << "Mata kuliah baru: "; getline(cin, t->matkul);
            cout << "Tugas diperbarui.\n";
            return;
        }
        t = t->next;
    }
    cout << " Tugas tidak ditemukan.\n";
}

// tandai tugas sebagai selesai
void tandaiSelesai(string nama) {
    Tugas* t = head;
    while (t) {
        if (t->nama == nama) {
            t->selesai = true;
            cout << "Tugas ditandai selesai.\n";
            return;
        }
        t = t->next;
    }
    cout << "Tugas tidak ditemukan.\n";
}

// menghapus tugas
void hapusTugas(string nama) {
    Tugas* t = head;
    Tugas* sebelum = nullptr;
    while (t) {
        if (t->nama == nama) {
            if (!sebelum) head = t->next;
            else sebelum->next = t->next;
            delete t;  // POINTER - dealokasi memori
            cout << "Tugas dihapus.\n";
            return;
        }
        sebelum = t;
        t = t->next;
    }
    cout << "Tugas tidak ditemukan.\n";
}

// FILE HANDLING - Simpan semua tugas ke file
void simpanKeFile(string namaFile) {
    ofstream out(namaFile);
    Tugas* t = head;
    while (t) {
        out << t->nama << "|" << t->deadline << "|" << t->matkul << "|" << t->selesai << "\n";
        t = t->next;
    }
    out.close();
    cout << "Data disimpan ke file.\n";
}

// FILE HANDLING - muat data dari file
void muatDariFile(string namaFile) {
    ifstream in(namaFile);
    if (!in) return;

    string baris;
    while (getline(in, baris)) {
        string nama = "", deadline = "", matkul = "", status = "";
        int i = 0, tahap = 0;
        while (i < baris.size()) {
            if (baris[i] == '|') { tahap++; i++; continue; }
            if (tahap == 0) nama += baris[i];
            else if (tahap == 1) deadline += baris[i];
            else if (tahap == 2) matkul += baris[i];
            else if (tahap == 3) status += baris[i];
            i++;
        }
        Tugas* t = buatTugas(nama, deadline, matkul);
        t->selesai = (status == "1");
        t->next = head;
        head = t;
    }
    in.close();
}

// SORTING - Bubble sort berdasarkan deadline
void urutkanDeadline() {
    if (!head || !head->next) return;

    bool tukar;
    do {
        tukar = false;
        Tugas* sekarang = head;
        Tugas* sebelum = nullptr;
        while (sekarang && sekarang->next) {
            Tugas* setelah = sekarang->next;
            if (ubahTanggalKeAngka(sekarang->deadline) > ubahTanggalKeAngka(setelah->deadline)) {
                tukar = true;
                if (sebelum) sebelum->next = setelah;
                else head = setelah;
                sekarang->next = setelah->next;
                setelah->next = sekarang;
                sebelum = setelah;
            } else {
                sebelum = sekarang;
                sekarang = sekarang->next;
            }
        }
    } while (tukar);
    cout << "Tugas diurutkan berdasarkan deadline.\n";
}

// MENU UTAMA
void menu() {
    int pilih;
    string nama, deadline, matkul;

    muatDariFile("todolist.txt");

    do {
        cout << "\n ============================== \n";
        cout << "|       TO-DO LIST TUGAS       |\n";
        cout << "|------------------------------|\n";
        cout << "| 1. Tambah Tugas              |\n";
        cout << "| 2. Tampilkan Semua Tugas     |\n";
        cout << "| 3. Cari Tugas                |\n";
        cout << "| 4. Edit Tugas                |\n";
        cout << "| 5. Tandai Selesai            |\n";
        cout << "| 6. Hapus Tugas               |\n";
        cout << "| 7. Simpan & Keluar           |\n";
        cout << " ============================== \n"; 

        cout << "Pilihan Anda: ";
        cin >> pilih;
        cin.ignore();
        system("cls");  
