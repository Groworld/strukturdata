"""
Sistem Informasi Antrian Pemesanan dan Penjualan
Kantin BWI - Menggunakan Struktur Data Queue, Stack, dan Trie
=============================================================
Jurusan Informatika - Proyek Struktur Data
"""

from datetime import datetime
from collections import deque


# ============================================================
# 1. TRIE — Pencarian dan Autocomplete Menu
# ============================================================

class TrieNode:
    def __init__(self):
        self.children = {}      # karakter -> TrieNode
        self.is_end = False     # penanda akhir kata/menu
        self.menu_data = None   # simpan data menu (harga, stok)


class Trie:
    """
    Trie digunakan untuk:
    - Menyimpan daftar menu kantin
    - Pencarian menu dengan prefix (autocomplete)
    - Kompleksitas pencarian O(m) — m = panjang kata
    """

    def __init__(self):
        self.root = TrieNode()

    def insert(self, nama_menu: str, harga: int, stok: int):
        """Memasukkan nama menu ke Trie."""
        node = self.root
        for huruf in nama_menu.lower():
            if huruf not in node.children:
                node.children[huruf] = TrieNode()
            node = node.children[huruf]
        node.is_end = True
        node.menu_data = {"nama": nama_menu, "harga": harga, "stok": stok}

    def search(self, nama_menu: str) -> dict | None:
        """Cari menu exact. Return data menu atau None jika tidak ada."""
        node = self.root
        for huruf in nama_menu.lower():
            if huruf not in node.children:
                return None
            node = node.children[huruf]
        return node.menu_data if node.is_end else None

    def autocomplete(self, prefix: str) -> list[dict]:
        """
        Kembalikan semua menu yang diawali prefix tertentu.
        Contoh: prefix 'nasi' -> ['Nasi Goreng', 'Nasi Uduk', 'Nasi Kuning']
        """
        node = self.root
        for huruf in prefix.lower():
            if huruf not in node.children:
                return []
            node = node.children[huruf]

        hasil = []
        self._kumpulkan_menu(node, hasil)
        return hasil

    def _kumpulkan_menu(self, node: TrieNode, hasil: list):
        """Rekursif: kumpulkan semua menu dari node saat ini."""
        if node.is_end:
            hasil.append(node.menu_data)
        for child in node.children.values():
            self._kumpulkan_menu(child, hasil)

    def update_stok(self, nama_menu: str, jumlah: int) -> bool:
        """Kurangi stok setelah menu dipesan."""
        node = self.root
        for huruf in nama_menu.lower():
            if huruf not in node.children:
                return False
            node = node.children[huruf]
        if node.is_end and node.menu_data["stok"] >= jumlah:
            node.menu_data["stok"] -= jumlah
            return True
        return False


# ============================================================
# 2. STACK — Keranjang Belanja dan Riwayat Transaksi
# ============================================================

class Stack:
    """
    Stack digunakan untuk:
    - Keranjang belanja (push item, pop untuk undo)
    - Riwayat transaksi (LIFO — transaksi terakhir di atas)
    - Mencetak struk dari riwayat pembelian
    """

    def __init__(self, label="Stack"):
        self._data = []
        self.label = label

    def push(self, item):
        """Tambah item ke atas stack."""
        self._data.append(item)
        print(f"  [Stack/{self.label}] PUSH: {item}")

    def pop(self):
        """Ambil dan hapus item teratas. Return None jika kosong."""
        if self.is_empty():
            print(f"  [Stack/{self.label}] Stack kosong, tidak ada yang di-pop.")
            return None
        item = self._data.pop()
        print(f"  [Stack/{self.label}] POP: {item}")
        return item

    def peek(self):
        """Lihat item teratas tanpa menghapus."""
        return self._data[-1] if not self.is_empty() else None

    def is_empty(self) -> bool:
        return len(self._data) == 0

    def size(self) -> int:
        return len(self._data)

    def lihat_semua(self) -> list:
        """Kembalikan semua item (urutan dari bawah ke atas)."""
        return list(self._data)

    def cetak_struk(self, nama_pelanggan: str, nomor_antrian: int):
        """Cetak struk belanja dari isi stack."""
        print("\n" + "="*40)
        print(f"  STRUK PEMBELIAN - KANTIN BWI")
        print(f"  Pelanggan : {nama_pelanggan}")
        print(f"  No. Antrian: {nomor_antrian}")
        print(f"  Waktu     : {datetime.now().strftime('%d/%m/%Y %H:%M')}")
        print("-"*40)
        total = 0
        for idx, item in enumerate(self._data, 1):
            subtotal = item["harga"] * item["qty"]
            print(f"  {idx}. {item['nama']:<20} x{item['qty']}  Rp{subtotal:>8,}")
            total += subtotal
        print("-"*40)
        print(f"  {'TOTAL':<30} Rp{total:>8,}")
        print("="*40)
        return total


# ============================================================
# 3. QUEUE — Antrian Pemesanan Pelanggan
# ============================================================

class Queue:
    """
    Queue digunakan untuk:
    - Antrian pelanggan (FIFO — siapa duluan dilayani duluan)
    - Manajemen urutan pemesanan di kasir
    - Estimasi waktu tunggu
    """

    def __init__(self):
        self._data = deque()   # deque lebih efisien dari list untuk queue
        self._nomor_urut = 0

    def enqueue(self, pesanan: dict) -> int:
        """
        Tambah pesanan ke antrian.
        Return nomor antrian yang diberikan ke pelanggan.
        """
        self._nomor_urut += 1
        pesanan["nomor_antrian"] = self._nomor_urut
        pesanan["waktu_masuk"] = datetime.now().strftime("%H:%M:%S")
        self._data.append(pesanan)
        print(f"  [Queue] ENQUEUE: {pesanan['nama_pelanggan']} — No. Antrian #{self._nomor_urut}")
        return self._nomor_urut

    def dequeue(self) -> dict | None:
        """
        Panggil pelanggan berikutnya dari antrian.
        Return pesanan atau None jika antrian kosong.
        """
        if self.is_empty():
            print("  [Queue] Antrian kosong.")
            return None
        pesanan = self._data.popleft()
        print(f"  [Queue] DEQUEUE: Memanggil {pesanan['nama_pelanggan']} — No. #{pesanan['nomor_antrian']}")
        return pesanan

    def peek(self) -> dict | None:
        """Lihat pelanggan berikutnya tanpa menghapus dari antrian."""
        return self._data[0] if not self.is_empty() else None

    def is_empty(self) -> bool:
        return len(self._data) == 0

    def size(self) -> int:
        return len(self._data)

    def tampilkan_antrian(self):
        """Tampilkan status antrian saat ini."""
        print(f"\n  Status Antrian ({len(self._data)} orang menunggu):")
        for i, p in enumerate(self._data):
            print(f"    #{p['nomor_antrian']} {p['nama_pelanggan']} — masuk {p['waktu_masuk']}")


# ============================================================
# 4. SISTEM KANTIN — Integrasi Queue + Stack + Trie
# ============================================================

class SistemKantinBWI:
    """
    Kelas utama yang mengintegrasikan ketiga struktur data:
    - Trie  : pencarian & autocomplete menu
    - Stack : keranjang belanja per pelanggan
    - Queue : antrian pemesanan di kasir
    """

    def __init__(self):
        self.trie_menu = Trie()
        self.antrian = Queue()
        self.riwayat_transaksi = Stack(label="RiwayatTransaksi")
        self._inisialisasi_menu()

    def _inisialisasi_menu(self):
        """Isi Trie dengan daftar menu kantin."""
        daftar_menu = [
            ("Nasi Goreng",      12000, 20),
            ("Nasi Uduk",        10000, 15),
            ("Nasi Kuning",      11000, 10),
            ("Ayam Bakar",       18000, 12),
            ("Ayam Goreng",      15000, 20),
            ("Ayam Penyet",      17000,  8),
            ("Mie Goreng",       10000, 15),
            ("Mie Rebus",         9000, 10),
            ("Es Teh",            3000, 50),
            ("Es Jeruk",          4000, 40),
            ("Es Campur",         6000, 20),
            ("Bakso",            12000, 15),
            ("Soto Ayam",        13000, 10),
        ]
        for nama, harga, stok in daftar_menu:
            self.trie_menu.insert(nama, harga, stok)
        print(f"✅ Menu kantin berhasil dimuat ({len(daftar_menu)} item)\n")

    def cari_menu(self, keyword: str) -> list[dict]:
        """Cari menu dengan autocomplete Trie."""
        hasil = self.trie_menu.autocomplete(keyword)
        if hasil:
            print(f"  Saran menu untuk '{keyword}':")
            for m in hasil:
                print(f"    - {m['nama']:<20} Rp{m['harga']:,}  (stok: {m['stok']})")
        else:
            print(f"  Menu dengan awalan '{keyword}' tidak ditemukan.")
        return hasil

    def buat_pesanan(self, nama_pelanggan: str) -> "PesananPelanggan":
        """Buat sesi pesanan baru untuk satu pelanggan."""
        return PesananPelanggan(nama_pelanggan, self.trie_menu)

    def kirim_ke_kasir(self, pesanan: "PesananPelanggan") -> int:
        """Masukkan pesanan selesai ke antrian kasir."""
        data = {
            "nama_pelanggan": pesanan.nama,
            "keranjang": pesanan.keranjang,
        }
        nomor = self.antrian.enqueue(data)
        return nomor

    def layani_pelanggan(self):
        """Kasir memanggil dan memproses pelanggan berikutnya."""
        pesanan = self.antrian.dequeue()
        if not pesanan:
            return

        print(f"\n🍽️  Melayani: {pesanan['nama_pelanggan']}")
        total = pesanan["keranjang"].cetak_struk(
            pesanan["nama_pelanggan"],
            pesanan["nomor_antrian"]
        )

        # Simpan ke riwayat transaksi (Stack)
        self.riwayat_transaksi.push({
            "waktu": datetime.now().strftime("%H:%M"),
            "pelanggan": pesanan["nama_pelanggan"],
            "total": total,
            "nomor": pesanan["nomor_antrian"],
        })
        return total

    def laporan_hari_ini(self):
        """Tampilkan laporan dari Stack riwayat transaksi."""
        semua = self.riwayat_transaksi.lihat_semua()
        if not semua:
            print("Belum ada transaksi hari ini.")
            return
        total_omset = sum(t["total"] for t in semua)
        print("\n" + "="*40)
        print("  LAPORAN PENJUALAN — KANTIN BWI")
        print(f"  Tanggal: {datetime.now().strftime('%d/%m/%Y')}")
        print("-"*40)
        for t in semua:
            print(f"  #{t['nomor']:02} {t['pelanggan']:<18} Rp{t['total']:>8,}")
        print("-"*40)
        print(f"  Total Transaksi : {len(semua)}")
        print(f"  Total Omset     : Rp{total_omset:,}")
        print("="*40)


class PesananPelanggan:
    """Satu sesi belanja pelanggan — keranjang menggunakan Stack."""

    def __init__(self, nama: str, trie_menu: Trie):
        self.nama = nama
        self.keranjang = Stack(label=f"Keranjang-{nama}")
        self.trie_menu = trie_menu

    def tambah_item(self, nama_menu: str, qty: int = 1) -> bool:
        """Tambah item ke keranjang (Stack.push)."""
        menu = self.trie_menu.search(nama_menu)
        if not menu:
            print(f"  ❌ Menu '{nama_menu}' tidak ditemukan.")
            return False
        if menu["stok"] < qty:
            print(f"  ❌ Stok '{nama_menu}' tidak cukup (sisa: {menu['stok']}).")
            return False
        self.keranjang.push({"nama": menu["nama"], "harga": menu["harga"], "qty": qty})
        self.trie_menu.update_stok(nama_menu, qty)
        return True

    def batal_item_terakhir(self):
        """Undo pesanan terakhir (Stack.pop)."""
        item = self.keranjang.pop()
        if item:
            self.trie_menu.update_stok(item["nama"], -item["qty"])  # kembalikan stok
            print(f"  ↩️  Item '{item['nama']}' dibatalkan dari keranjang.")

    def lihat_keranjang(self):
        """Tampilkan isi keranjang saat ini."""
        isi = self.keranjang.lihat_semua()
        if not isi:
            print(f"  Keranjang {self.nama} kosong.")
            return
        print(f"\n  Keranjang {self.nama}:")
        for i, item in enumerate(isi, 1):
            print(f"    {i}. {item['nama']} x{item['qty']} = Rp{item['harga']*item['qty']:,}")


# ============================================================
# 5. DEMO — Simulasi Penggunaan Sistem
# ============================================================

if __name__ == "__main__":
    print("="*50)
    print("  SIMULASI SISTEM KANTIN BWI")
    print("="*50 + "\n")

    kantin = SistemKantinBWI()

    # --- Demo Trie: Autocomplete ---
    print("🔍 Demo Pencarian Menu (Trie Autocomplete)")
    print("-"*40)
    kantin.cari_menu("nasi")
    print()
    kantin.cari_menu("ay")
    print()

    # --- Pelanggan 1: Andi ---
    print("👤 Pelanggan 1: Andi")
    print("-"*40)
    andi = kantin.buat_pesanan("Andi")
    andi.tambah_item("Nasi Goreng", 1)
    andi.tambah_item("Ayam Bakar", 1)
    andi.tambah_item("Es Teh", 2)
    andi.tambah_item("Mie Goreng", 1)   # tambah dulu
    andi.batal_item_terakhir()           # lalu dibatalkan (undo)
    andi.lihat_keranjang()
    no_andi = kantin.kirim_ke_kasir(andi)
    print(f"  ✅ Andi masuk antrian #{no_andi}\n")

    # --- Pelanggan 2: Budi ---
    print("👤 Pelanggan 2: Budi")
    print("-"*40)
    budi = kantin.buat_pesanan("Budi")
    budi.tambah_item("Soto Ayam", 1)
    budi.tambah_item("Es Jeruk", 1)
    budi.lihat_keranjang()
    no_budi = kantin.kirim_ke_kasir(budi)
    print(f"  ✅ Budi masuk antrian #{no_budi}\n")

    # --- Pelanggan 3: Citra ---
    print("👤 Pelanggan 3: Citra")
    print("-"*40)
    citra = kantin.buat_pesanan("Citra")
    citra.tambah_item("Ayam Goreng", 2)
    citra.tambah_item("Es Campur", 1)
    citra.lihat_keranjang()
    no_citra = kantin.kirim_ke_kasir(citra)
    print(f"  ✅ Citra masuk antrian #{no_citra}\n")

    # --- Tampilkan status antrian ---
    print("📋 Status Antrian Kasir")
    print("-"*40)
    kantin.antrian.tampilkan_antrian()

    # --- Kasir melayani satu per satu ---
    print("\n💰 Kasir Mulai Melayani")
    print("-"*40)
    kantin.layani_pelanggan()
    kantin.layani_pelanggan()
    kantin.layani_pelanggan()

    # --- Laporan akhir hari ---
    kantin.laporan_hari_ini()
