
from collections import deque

# =============================
# 1. TRIE UNTUK PENCARIAN MENU
# =============================
class TrieNode:
    def __init__(self):
        self.children = {}
        self.end_of_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for char in word.lower():
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.end_of_word = True

    def search_prefix(self, prefix):
        node = self.root
        for char in prefix.lower():
            if char not in node.children:
                return []
            node = node.children[char]
        
        result = []
        self._collect_words(node, prefix.lower(), result)
        return result

    def _collect_words(self, node, word, result):
        if node.end_of_word:
            result.append(word)
        for char, child in node.children.items():
            self._collect_words(child, word + char, result)

# ============================
# 2. DATA MENU & INISIALISASI
# ============================
menu = {
    "nasi goreng": 15000,
    "nasi ayam": 18000,
    "nasi katsu": 20000,
    "mie goreng": 14000,
    "es teh": 5000,
    "jus alpukat": 10000
}

trie = Trie()
for item in menu:
    trie.insert(item)

antrian_pesanan = deque()  
riwayat_transaksi = []    

# ======================================
# 3. FUNGSI PELANGGAN (CUSTOMER-FACING)
# ======================================
def tampilkan_menu():
    print("\n===== DAFTAR MENU =====")
    for nama, harga in menu.items():
        print(f"- {nama.title()} : Rp{harga:,}")    

def cari_menu():
    prefix = input("\nMasukkan kata kunci menu: ")
    hasil = trie.search_prefix(prefix)   
    if hasil:
        print("\nMenu ditemukan:")
        for item in hasil:
            print("-", item.title())
    else:
        print("Menu tidak ditemukan.")

def tambah_pesanan():
    nama = input("\nMasukkan Nama Anda : ")
    stack_makanan = [] 

    while True:
        tampilkan_menu()
        print("\n[Ketik 'undo' untuk membatalkan pilihan terakhir]")
        item = input("Pilih menu : ").lower()

        if item == "undo":
            if stack_makanan:
                batal = stack_makanan.pop()
                print(f"[-] '{batal[0].title()}' dihapus dari pesanan.")
            else:
                print("[!] Keranjang masih kosong.")
            continue

        if item not in menu:
            print("[!] Menu tidak tersedia.")
            continue

        jumlah = int(input("Jumlah : "))
        stack_makanan.append((item, jumlah))
        print(f"[+] {item.title()} ditambahkan ke pesanan Anda.")

        lagi = input("Tambah menu lain? (y/n): ")
        if lagi.lower() != "y":
            break

    if stack_makanan:
        antrian_pesanan.append({
            "pelanggan": nama,
            "pesanan": stack_makanan
        })
        print("\n Pesanan Anda berhasil dikirim ke Dapur/Kasir!")
        print("Nomor Antrian Anda :", len(antrian_pesanan))
    else:
        print("\n Pemesanan dibatalkan.")

# ========================
# 4. FUNGSI ADMIN / KASIR
# ========================
def proses_pesanan():
    if not antrian_pesanan:
        print("\n Belum ada pesanan masuk.")
        return

    data = antrian_pesanan.popleft() 
    total = 0

    print("\n===== CETAK STRUK TRANSAKSI =====")
    print("Pelanggan :", data["pelanggan"])
    for item, jumlah in data["pesanan"]:
        subtotal = menu[item] * jumlah
        total += subtotal
        print(f"{item.title()} x{jumlah} = Rp{subtotal:,}")

    print("-" * 30)
    print("TOTAL BAYAR : Rp", format(total, ","))

    riwayat_transaksi.append({
        "pelanggan": data["pelanggan"],
        "pesanan": data["pesanan"],
        "total": total
    })
    print("Transaksi selesai. Dana masuk ke riwayat.")

def undo_transaksi():
    if not riwayat_transaksi:
        print("\n[!] Tidak ada transaksi yang bisa dibatalkan.")
        return

    batal = riwayat_transaksi.pop() 
    print("\n TRANSAKSI DIBATALKAN")
    print("Dana dikembalikan untuk Pelanggan :", batal["pelanggan"])

def lihat_riwayat():
    if not riwayat_transaksi:
        print("\n[!] Belum ada pemasukan hari ini.")
        return

    print("\n===== BUKU KAS KANTIN (Terbaru ke Terlama) =====")
    total_pendapatan = 0
    for i, trx in enumerate(reversed(riwayat_transaksi), start=1):
        print(f"{i}. {trx['pelanggan']} - Rp{trx['total']:,}")
        total_pendapatan += trx['total']
    
    print("-" * 30)
    print(f"TOTAL PENDAPATAN SEMENTARA: Rp{total_pendapatan:,}")

# ==================================
# 5. MENU NAVIGASI (PENGATUR PERAN)
# ==================================
def menu_pelanggan():
    while True:
        print("\n=== LOKET PELANGGAN ===")
        print("1. Lihat Daftar Menu")
        print("2. Cari Menu Spesifik")
        print("3. Mulai Pesan Makanan")
        print("0. Kembali ke Layar Awal")
        pilihan = input("Pilihan Anda: ")

        if pilihan == "1":
            tampilkan_menu()
        elif pilihan == "2":
            cari_menu()
        elif pilihan == "3":
            tambah_pesanan()
        elif pilihan == "0":
            break
        else:
            print("Pilihan tidak valid.")

def menu_admin():
    password = input("\nMasukkan Password Admin: ")
    if password != "admin123":
        print("Password Salah! Akses ditolak.")
        return

    while True:
        print("\n=== DASHBOARD KASIR ===")
        print(f"Sisa Antrian: {len(antrian_pesanan)} pesanan")
        print("1. Proses Antrian Selanjutnya")
        print("2. Batalkan Transaksi Terakhir (Undo)")
        print("3. Lihat Buku Kas / Riwayat")
        print("0. Logout (Kembali)")
        pilihan = input("Pilihan Anda: ")

        if pilihan == "1":
            proses_pesanan()
        elif pilihan == "2":
            undo_transaksi()
        elif pilihan == "3":
            lihat_riwayat()
        elif pilihan == "0":
            print("Berhasil Logout.")
            break
        else:
            print("Pilihan tidak valid.")

# =====================
# PROGRAM UTAMA (MAIN)
# =====================
if __name__ == "__main__":
    while True:
        print("\n" + "="*30)
        print("SELAMAT DATANG DI KANTIN BWI")
        print("="*30)
        print("Silakan pilih peran Anda:")
        print("1. Pelanggan (Pesan Makanan)")
        print("2. Petugas / Kasir (Login)")
        print("0. Matikan Mesin")
        
        pilihan = input("Pilihan: ")

        if pilihan == "1":
            menu_pelanggan()
        elif pilihan == "2":
            menu_admin()
        elif pilihan == "0":
            print("Mesin dimatikan. Sampai jumpa!")
            break
        else:
            print("Pilihan tidak valid.")
