"""DompetKu - Sistem Pengelolaan Keuangan Pribadi Mahasiswa (versi Streamlit).
Jalankan: streamlit run streamlit_app.py
"""
from abc import ABC, abstractmethod
from datetime import date, timedelta


# =====================================================================
#  MODEL OOP (sama dengan versi web sebelumnya)
# =====================================================================
class Kategori:
    def __init__(self, nama, jenis):
        self.__nama = nama
        self.__jenis = jenis  # "pemasukan" / "pengeluaran"

    @property
    def nama(self):
        return self.__nama

    @property
    def jenis(self):
        return self.__jenis


class Dompet:
    def __init__(self, id, nama, jenis, saldo=0):
        self.id = id
        self.__nama = nama
        self.__jenis = jenis
        self.__saldo = saldo

    @property
    def nama(self):
        return self.__nama

    @property
    def jenis(self):
        return self.__jenis

    @property
    def saldo(self):
        return self.__saldo

    def tambah_saldo(self, n):
        self.__saldo += n

    def kurangi_saldo(self, n):
        if n > self.__saldo:
            raise ValueError(f"Saldo {self.__nama} tidak cukup")
        self.__saldo -= n


class Transaksi(ABC):
    def __init__(self, id, tanggal, nominal, deskripsi, kategori, dompet):
        if nominal <= 0:
            raise ValueError("Nominal harus lebih dari 0")
        self.id, self.tanggal, self.nominal = id, tanggal, nominal
        self.deskripsi, self.kategori, self.dompet = deskripsi, kategori, dompet

    @abstractmethod
    def terapkan(self):
        """Memperbarui saldo dompet (polimorfisme)."""

    def to_dict(self):
        return {"id": self.id, "tanggal": str(self.tanggal), "nominal": self.nominal,
                "deskripsi": self.deskripsi, "kategori": self.kategori.nama,
                "dompet": self.dompet.nama, "jenis": self.kategori.jenis}


class Pemasukan(Transaksi):
    def terapkan(self):
        self.dompet.tambah_saldo(self.nominal)


class Pengeluaran(Transaksi):
    def terapkan(self):
        self.dompet.kurangi_saldo(self.nominal)


class Anggaran:
    def __init__(self, kategori, batas):
        self.kategori, self.batas = kategori, batas

    def hitung_terpakai(self, transaksi):
        return sum(t.nominal for t in transaksi
                   if isinstance(t, Pengeluaran) and t.kategori is self.kategori)

    def persentase(self, terpakai):
        return round(terpakai / self.batas * 100, 1)

    def melebihi(self, terpakai):
        return terpakai > self.batas


class TargetKeuangan(ABC):
    """Kelas dasar semua target (abstraksi)."""
    def __init__(self, nama, kategori, target, tenggat, terkumpul=0):
        self.nama, self.kategori, self.target = nama, kategori, target
        self.tenggat, self.terkumpul = tenggat, terkumpul

    def setor(self, nominal):
        if nominal <= 0:
            raise ValueError("Nominal setoran harus lebih dari 0")
        self.terkumpul += nominal

    def progres(self):
        return round(min(self.terkumpul / self.target * 100, 100), 1)

    def sisa(self):
        return max(self.target - self.terkumpul, 0)

    @abstractmethod
    def jenis(self):
        """Nama jenis target (polimorfisme)."""


class TargetTabungan(TargetKeuangan):
    def jenis(self):
        return "Tabungan"


class TargetInvestasi(TargetKeuangan):
    def __init__(self, nama, kategori, target, tenggat, estimasi_return, terkumpul=0):
        super().__init__(nama, kategori, target, tenggat, terkumpul)
        self.estimasi_return = estimasi_return  # persen per tahun

    def jenis(self):
        return "Investasi"

    def proyeksi_nilai(self):
        """Perkiraan nilai dana saat tenggat dengan imbal hasil majemuk."""
        try:
            tahun = max((date.fromisoformat(str(self.tenggat)) - date.today()).days, 0) / 365
        except ValueError:
            tahun = 0
        return round(self.terkumpul * (1 + self.estimasi_return / 100) ** tahun)


class Pengguna:
    def __init__(self, nama, email, jurusan):
        self.nama, self.email, self.jurusan = nama, email, jurusan
        self.daftar_dompet = []

    def tambah_dompet(self, dompet):
        self.daftar_dompet.append(dompet)

    def total_saldo(self):
        return sum(d.saldo for d in self.daftar_dompet)


class PengelolaKeuangan:
    def __init__(self, pengguna):
        self.pengguna = pengguna
        self.kategori = {}
        self.transaksi = []
        self.anggaran = []
        self.target = []

    def _dompet(self, id):
        return next(d for d in self.pengguna.daftar_dompet if d.id == id)

    def catat_transaksi(self, nama_kategori, dompet_id, nominal, deskripsi, tanggal=None):
        kat = self.kategori[nama_kategori]
        kelas = Pemasukan if kat.jenis == "pemasukan" else Pengeluaran
        t = kelas(len(self.transaksi) + 1, tanggal or date.today(), nominal,
                  deskripsi, kat, self._dompet(dompet_id))
        t.terapkan()  # polimorfisme: tiap subclass punya aturan sendiri
        self.transaksi.append(t)
        return t

    def setor_tabungan(self, idx, dompet_id, nominal):
        t = self.target[int(idx)]
        if nominal <= 0:
            raise ValueError("Nominal setoran harus lebih dari 0")
        self._dompet(dompet_id).kurangi_saldo(nominal)
        t.setor(nominal)

    def tambah_target(self, jenis, nama, kategori, nominal, tenggat=None, estimasi_return=0):
        nominal = float(nominal or 0)
        if not str(nama).strip() or nominal <= 0:
            raise ValueError("Nama dan nominal target wajib diisi")
        tenggat = tenggat or "-"
        if jenis == "Investasi":
            t = TargetInvestasi(nama.strip(), kategori, nominal, tenggat, float(estimasi_return or 0))
        else:
            t = TargetTabungan(nama.strip(), kategori, nominal, tenggat)
        self.target.append(t)

    def hapus_target(self, idx):
        del self.target[int(idx)]

    def peringatan(self):
        hasil = []
        for a in self.anggaran:
            p = a.persentase(a.hitung_terpakai(self.transaksi))
            if p > 100:
                hasil.append(f"Anggaran {a.kategori.nama} terlampaui ({p}%)!")
            elif p >= 80:
                hasil.append(f"Anggaran {a.kategori.nama} hampir habis ({p}%)")
        return hasil

    def tambah_kategori(self, nama, jenis="pengeluaran", batas=None):
        nama = nama.strip()
        if not nama:
            raise ValueError("Nama kategori wajib diisi")
        if jenis not in ("pemasukan", "pengeluaran"):
            raise ValueError("Jenis kategori tidak valid")
        if nama not in self.kategori:
            self.kategori[nama] = Kategori(nama, jenis)
        kat, batas = self.kategori[nama], float(batas or 0)
        if batas > 0 and kat.jenis == "pengeluaran":
            for a in self.anggaran:
                if a.kategori is kat:
                    a.batas = batas  # kategori sudah punya anggaran: perbarui
                    return
            self.anggaran.append(Anggaran(kat, batas))

    def hapus_anggaran(self, idx):
        del self.anggaran[int(idx)]

    def daftar_objek(self):
        p = self.pengguna
        o = [{"objek": p.nama, "class": "Pengguna",
              "atribut": {"nama": p.nama, "email": p.email, "jurusan": p.jurusan},
              "method": ["tambahDompet()", "totalSaldo()"]}]
        for d in p.daftar_dompet:
            o.append({"objek": d.nama, "class": "Dompet",
                      "atribut": {"nama": d.nama, "jenis": d.jenis, "saldo": d.saldo},
                      "method": ["tambahSaldo()", "kurangiSaldo()"]})
        for a in self.anggaran:
            o.append({"objek": "Anggaran " + a.kategori.nama, "class": "Anggaran",
                      "atribut": {"kategori": a.kategori.nama, "batas": a.batas},
                      "method": ["hitungTerpakai()", "persentase()", "melebihi()"]})
        for t in self.target:
            atr = {"nama": t.nama, "kategori": t.kategori, "target": t.target,
                   "terkumpul": t.terkumpul, "tenggat": t.tenggat}
            mtd = ["setor()", "progres()", "sisa()", "jenis()"]
            if isinstance(t, TargetInvestasi):
                atr["estimasiReturn"] = t.estimasi_return
                mtd.append("proyeksiNilai()")
            o.append({"objek": t.nama, "class": type(t).__name__, "atribut": atr, "method": mtd})
        for t in self.transaksi[-5:]:
            o.append({"objek": f"Transaksi #{t.id}", "class": type(t).__name__,
                      "atribut": {"nominal": t.nominal, "deskripsi": t.deskripsi,
                                  "kategori": t.kategori.nama, "dompet": t.dompet.nama},
                      "method": ["terapkan()"]})
        return o

    def ringkasan(self):
        masuk = sum(t.nominal for t in self.transaksi if isinstance(t, Pemasukan))
        keluar = sum(t.nominal for t in self.transaksi if isinstance(t, Pengeluaran))
        per_kat = {}
        for t in self.transaksi:
            if isinstance(t, Pengeluaran):
                per_kat[t.kategori.nama] = per_kat.get(t.kategori.nama, 0) + t.nominal
        return {
            "terdaftar": True, "objek": self.daftar_objek(),
            "pengguna": self.pengguna.nama,
            "total_saldo": self.pengguna.total_saldo(),
            "pemasukan": masuk, "pengeluaran": keluar,
            "dompet": [{"id": d.id, "nama": d.nama, "jenis": d.jenis, "saldo": d.saldo}
                       for d in self.pengguna.daftar_dompet],
            "per_kategori": per_kat,
            "anggaran": [{"kategori": a.kategori.nama, "batas": a.batas,
                          "terpakai": (u := a.hitung_terpakai(self.transaksi)),
                          "persen": a.persentase(u)} for a in self.anggaran],
            "target": [{"nama": t.nama, "jenis": t.jenis(), "kategori": t.kategori,
                        "target": t.target, "terkumpul": t.terkumpul, "sisa": t.sisa(),
                        "progres": t.progres(), "tenggat": str(t.tenggat),
                        "return": getattr(t, "estimasi_return", None),
                        "proyeksi": t.proyeksi_nilai() if isinstance(t, TargetInvestasi) else None}
                       for t in self.target],
            "peringatan": self.peringatan(),
            "kategori": [{"nama": k.nama, "jenis": k.jenis} for k in self.kategori.values()],
            "transaksi": [t.to_dict() for t in reversed(self.transaksi)][:15],
        }


def buat_sistem(d):
    pengguna = Pengguna(d["nama"].strip(), d.get("email", "").strip(), d.get("jurusan", "").strip())
    app = PengelolaKeuangan(pengguna)
    for i, (nm, jn) in enumerate([("Tunai", "tunai"), ("Rekening Bank", "bank"), ("E-Wallet", "ewallet")], 1):
        pengguna.tambah_dompet(Dompet(i, nm, jn, float(d.get(jn) or 0)))
    for nama, jenis in [("Uang Saku", "pemasukan"), ("Freelance", "pemasukan"),
                        ("Lainnya", "pengeluaran")]:
        app.kategori[nama] = Kategori(nama, jenis)
    for item in d.get("anggaran", []):
        if not str(item.get("kategori", "")).strip():
            continue  # lewati baris kosong
        app.tambah_kategori(item["kategori"], "pengeluaran", item.get("batas"))
    tg = d.get("target")
    if tg and str(tg.get("nama", "")).strip():
        app.tambah_target(tg.get("jenis", "Tabungan"), tg["nama"], tg.get("kategori", "Lainnya"),
                          tg.get("nominal"), tg.get("tenggat"), tg.get("return"))
    return app


# =====================================================================
#  TAMPILAN STREAMLIT
# =====================================================================
import json
import re

import altair as alt
import pandas as pd
import streamlit as st

st.set_page_config(page_title="DompetKu", page_icon="💰", layout="wide")
st.markdown("""<style>
.stButton>button[kind="primary"]{background:#00c176;border:0;color:#04130c;font-weight:800}
[data-testid="stMetricValue"]{font-weight:800}
</style>""", unsafe_allow_html=True)

S = st.session_state
STAT = r'''classDiagram
  class Pengguna {
    -nama: str
    -email: str
    -jurusan: str
    +tambahDompet(dompet)
    +totalSaldo() float
  }
  class Dompet {
    -nama: str
    -jenis: str
    -saldo: float
    +tambahSaldo(n)
    +kurangiSaldo(n)
  }
  class Transaksi {
    <<abstract>>
    -id: int
    -tanggal: date
    -nominal: float
    -deskripsi: str
    +terapkan()*
  }
  class Pemasukan {
    +terapkan()
  }
  class Pengeluaran {
    +terapkan()
  }
  class Kategori {
    -nama: str
    -jenis: str
  }
  class Anggaran {
    -batas: float
    +hitungTerpakai()
    +persentase()
    +melebihi()
  }
  class TargetKeuangan {
    <<abstract>>
    -nama: str
    -kategori: str
    -target: float
    -terkumpul: float
    -tenggat: date
    +setor(nominal)
    +progres()
    +sisa()
    +jenis()*
  }
  class TargetTabungan {
    +jenis()
  }
  class TargetInvestasi {
    -estimasiReturn: float
    +jenis()
    +proyeksiNilai()
  }
  class PengelolaKeuangan {
    +catatTransaksi()
    +setorTabungan()
    +tambahTarget()
    +tambahKategori()
    +ringkasan()
    +peringatan()
    +daftarObjek()
  }
  Transaksi <|-- Pemasukan
  Transaksi <|-- Pengeluaran
  Pengguna "1" --> "*" Dompet
  Dompet "1" --> "*" Transaksi
  Kategori "1" --> "*" Transaksi
  Kategori "1" --> "0..1" Anggaran
  Pengguna "1" --> "*" TargetKeuangan
  TargetKeuangan <|-- TargetTabungan
  TargetKeuangan <|-- TargetInvestasi
  PengelolaKeuangan o-- Dompet
  PengelolaKeuangan o-- Transaksi
  PengelolaKeuangan o-- Anggaran
  PengelolaKeuangan o-- TargetKeuangan'''
KAT = {"Tabungan": ["Dana Darurat", "Liburan", "Pendidikan", "Gadget / Laptop", "Kendaraan",
                    "Rumah / Properti", "Pernikahan", "Lainnya"],
       "Investasi": ["Reksa Dana", "Saham", "Emas", "Obligasi / SBN", "Deposito", "Crypto"]}


# ---------- helper angka (titik ribuan) ----------
def fmt(n):
    return f"{int(round(n)):,}".replace(",", ".")


def rp(n):
    return "Rp" + fmt(n)


def angka(s):
    digit = "".join(c for c in str(s) if c.isdigit())
    return int(digit) if digit else 0


def _titik(key):
    n = angka(S[key])
    S[key] = fmt(n) if n else ""


def uang(label, key, awal=0):
    """Input nominal: titik ribuan muncul otomatis setelah Enter / pindah kolom."""
    if key not in S:
        S[key] = fmt(awal) if awal else ""
    st.text_input(label, key=key, on_change=_titik, args=(key,), placeholder="0")


def jalankan(fn, sukses, *args):
    try:
        fn(*args)
        S.pesan = ("ok", sukses)
    except (ValueError, KeyError, IndexError, StopIteration) as e:
        S.pesan = ("error", str(e) or "Data tidak valid")


def tampil_pesan():
    if "pesan" in S:
        jenis, teks = S.pop("pesan")
        (st.error if jenis == "error" else st.success)(teks)


# ---------- form target (dipakai di pendaftaran & dashboard) ----------
def form_target(p):
    jenis = st.selectbox("Jenis target", list(KAT), key=f"{p}_tg_jenis")
    st.selectbox("Kategori", KAT[jenis], key=f"{p}_tg_kat_{jenis}")
    st.text_input("Nama target", key=f"{p}_tg_nama", placeholder="cth: Laptop baru, Beli emas")
    uang("Nominal target (Rp)", f"{p}_tg_nominal")
    st.date_input("Tenggat", value=date.today() + timedelta(days=365), key=f"{p}_tg_tenggat")
    if jenis == "Investasi":
        st.number_input("Estimasi imbal hasil (% per tahun)", 0.0, 100.0, 8.0, 0.5, key=f"{p}_tg_ret")


def baca_target(p):
    jenis = S[f"{p}_tg_jenis"]
    return {"jenis": jenis, "kategori": S[f"{p}_tg_kat_{jenis}"], "nama": S[f"{p}_tg_nama"],
            "nominal": angka(S[f"{p}_tg_nominal"]), "tenggat": str(S[f"{p}_tg_tenggat"]),
            "return": S.get(f"{p}_tg_ret", 0) if jenis == "Investasi" else 0}


# ---------- diagram ----------
def diagram_dinamis(objek):
    bersih = lambda s: re.sub(r"[^\w .@-]", "", str(s))[:28]
    kel = {}
    for o in objek:
        kel.setdefault(o["class"], o)
    m = "classDiagram\n"
    for c, o in kel.items():
        m += f"class {c} {{\n"
        m += "".join(f"-{k}\n" for k in o["atribut"]) + "".join(f"+{f}\n" for f in o["method"]) + "}\n"
    for i, o in enumerate(objek):
        m += f'class o{i}["{bersih(o["objek"])} : {o["class"]}"] {{\n<<objek>>\n'
        for k, v in o["atribut"].items():
            m += f"{k} = {bersih(v) or '-'}\n"
        m += f"}}\no{i} ..> {o['class']} : instance\n"
    return m


def gambar(teks, tinggi):
    html = f"""<div style="background:#fff;padding:10px;border-radius:10px;overflow:auto"><div id="g"></div></div>
<script src="https://cdn.jsdelivr.net/npm/mermaid@10.9.1/dist/mermaid.min.js"></script>
<script>
mermaid.initialize({{startOnLoad:false, theme:'default'}});
mermaid.render('gsvg', {json.dumps(teks)})
  .then(r => {{ document.getElementById('g').innerHTML = r.svg; }})
  .catch(e => {{ document.getElementById('g').textContent = 'Diagram gagal dibuat: ' + e.message; }});
</script>"""
    if hasattr(st, "iframe"):  # Streamlit terbaru
        st.iframe(html, height=tinggi)
    else:  # Streamlit lama
        import streamlit.components.v1 as components
        components.html(html, height=tinggi, scrolling=True)


# ---------- aksi (callback tombol) ----------
def aksi_daftar():
    def kerja():
        if not S.r_nama.strip():
            raise ValueError("Nama wajib diisi")
        ang = []
        for _, r in S["_ang"].iterrows():
            nama = r["Kategori"]
            if pd.notna(nama) and str(nama).strip():
                batas = r["Batas per bulan (Rp)"]
                ang.append({"kategori": str(nama), "batas": float(batas) if pd.notna(batas) else 0})
        S.sistem = buat_sistem({
            "nama": S.r_nama, "email": S.r_email, "jurusan": S.r_jurusan,
            "tunai": angka(S.r_tunai), "bank": angka(S.r_bank), "ewallet": angka(S.r_ewallet),
            "anggaran": ang, "target": baca_target("r")})
    jalankan(kerja, "Berhasil masuk")


def keluar():
    for k in list(S.keys()):
        del S[k]


def aksi_catat():
    def kerja():
        S.sistem.catat_transaksi(S.t_kat, S.t_dompet, angka(S.t_nominal), S.t_desk.strip() or "-")
        S.t_nominal, S.t_desk = "", ""
    jalankan(kerja, "Transaksi tersimpan")


def aksi_kategori():
    def kerja():
        S.sistem.tambah_kategori(S.k_nama, S.k_jenis, angka(S.k_batas))
        S.k_nama, S.k_batas = "", ""
    jalankan(kerja, "Kategori disimpan")


def aksi_target():
    t = baca_target("g")

    def kerja():
        S.sistem.tambah_target(t["jenis"], t["nama"], t["kategori"], t["nominal"], t["tenggat"], t["return"])
        S.g_tg_nama, S.g_tg_nominal = "", ""
    jalankan(kerja, "Target ditambahkan")


def aksi_setor():
    def kerja():
        S.sistem.setor_tabungan(S.s_target, S.s_dompet, angka(S.s_nominal))
        S.s_nominal = ""
    jalankan(kerja, "Setoran berhasil")


# ---------- halaman pendaftaran ----------
def halaman_daftar():
    st.title("💰 DompetKu")
    st.caption("Isi data berikut untuk membuat dashboard keuanganmu. Data ini menjadi objek di dalam sistem.")
    tampil_pesan()
    st.subheader("Data diri")
    c = st.columns(3)
    c[0].text_input("Nama lengkap", key="r_nama")
    c[1].text_input("Email", key="r_email")
    c[2].text_input("Jurusan", key="r_jurusan")
    st.subheader("Saldo awal dompet")
    c = st.columns(3)
    with c[0]:
        uang("Tunai (Rp)", "r_tunai")
    with c[1]:
        uang("Rekening Bank (Rp)", "r_bank")
    with c[2]:
        uang("E-Wallet (Rp)", "r_ewallet")
    st.subheader("Anggaran bulanan (bebas pilih kategori)")
    st.caption("Klik baris kosong paling bawah untuk menambah kategori, pilih baris lalu tekan Delete untuk menghapus.")
    awal = pd.DataFrame({"Kategori": ["Makan", "Transport", "Hiburan", "Kuliah"],
                         "Batas per bulan (Rp)": [600000, 200000, 150000, 300000]})
    S["_ang"] = st.data_editor(awal, num_rows="dynamic", hide_index=True, key="r_ang", column_config={
        "Batas per bulan (Rp)": st.column_config.NumberColumn(min_value=0, step=10000, format="localized")})
    st.subheader("Target keuangan (opsional)")
    form_target("r")
    st.button("Masuk ke Dashboard", type="primary", on_click=aksi_daftar)


# ---------- halaman dashboard ----------
def tabel_transaksi(sis):
    baris = [t.to_dict() for t in reversed(sis.transaksi)]
    return pd.DataFrame([{"Tanggal": t["tanggal"], "Deskripsi": t["deskripsi"], "Kategori": t["kategori"],
                          "Dompet": t["dompet"],
                          "Nominal": ("+" if t["jenis"] == "pemasukan" else "-") + rp(t["nominal"])}
                         for t in baris])


def daftar_target(d):
    for t in d["target"]:
        st.progress(min(t["progres"], 100) / 100,
                    text=f"{t['nama']} ({t['jenis']} · {t['kategori']}): {rp(t['terkumpul'])} dari {rp(t['target'])} ({t['progres']}%)")
        if t["proyeksi"] is not None:
            st.caption(f"Estimasi {t['return']}%/tahun, proyeksi nilai saat tenggat {rp(t['proyeksi'])}")
    if not d["target"]:
        st.info("Belum ada target. Tambahkan di tab Anggaran & Target.")


def halaman_dashboard():
    sis = S.sistem
    d = sis.ringkasan()
    h1, h2 = st.columns([8, 1])
    h1.markdown(f"## Halo, {d['pengguna']} 👋")
    h2.button("Ganti", on_click=keluar)
    tampil_pesan()
    t1, t2, t3, t4 = st.tabs(["📊 Dashboard", "🧾 Transaksi", "🎯 Anggaran & Target", "🧩 Diagram Class"])

    with t1:
        for p in d["peringatan"]:
            (st.error if "terlampaui" in p else st.warning)(p)
        net = d["pemasukan"] - d["pengeluaran"]
        m = st.columns(3)
        m[0].metric("Total Saldo", rp(d["total_saldo"]), delta=("+" if net >= 0 else "-") + rp(abs(net)))
        m[1].metric("Pemasukan", rp(d["pemasukan"]))
        m[2].metric("Pengeluaran", rp(d["pengeluaran"]))
        a, b = st.columns(2)
        with a:
            st.subheader("Pengeluaran per Kategori")
            if d["per_kategori"]:
                df = pd.DataFrame({"Kategori": list(d["per_kategori"]), "Jumlah": list(d["per_kategori"].values())})
                st.altair_chart(alt.Chart(df).mark_arc(innerRadius=70).encode(
                    theta="Jumlah:Q", color="Kategori:N", tooltip=["Kategori", "Jumlah"]))
            else:
                st.info("Belum ada pengeluaran.")
            st.subheader("Dompet")
            for x in d["dompet"]:
                st.write(f"**{x['nama']}**: {rp(x['saldo'])}")
        with b:
            st.subheader("Anggaran Bulanan")
            for x in d["anggaran"]:
                ikon = "🔴" if x["persen"] > 100 else "🟡" if x["persen"] >= 80 else "🟢"
                st.progress(min(x["persen"], 100) / 100,
                            text=f"{ikon} {x['kategori']}: {rp(x['terpakai'])} dari {rp(x['batas'])} ({x['persen']}%)")
            if not d["anggaran"]:
                st.info("Belum ada anggaran.")
            st.subheader("Target Keuangan")
            daftar_target(d)
        st.subheader("Transaksi Terbaru")
        tb = tabel_transaksi(sis)
        if len(tb):
            st.dataframe(tb.head(10), hide_index=True)
        else:
            st.info("Belum ada transaksi.")

    with t2:
        st.subheader("Catat Transaksi")
        nama_dompet = {x["id"]: x["nama"] for x in d["dompet"]}
        c = st.columns(2)
        c[0].selectbox("Kategori", [k["nama"] for k in d["kategori"]], key="t_kat",
                       format_func=lambda n: f"{n} ({sis.kategori[n].jenis})")
        c[1].selectbox("Dompet", list(nama_dompet), key="t_dompet", format_func=nama_dompet.get)
        c = st.columns(2)
        with c[0]:
            uang("Nominal (Rp)", "t_nominal")
        c[1].text_input("Deskripsi", key="t_desk")
        st.button("Simpan Transaksi", type="primary", on_click=aksi_catat)
        st.subheader("Semua Transaksi")
        tb = tabel_transaksi(sis)
        if len(tb):
            st.dataframe(tb, hide_index=True)
        else:
            st.info("Belum ada transaksi.")

    with t3:
        kiri, kanan = st.columns(2)
        with kiri:
            st.subheader("Kategori & Anggaran")
            st.text_input("Nama kategori", key="k_nama", placeholder="cth: Kos, Ngopi, Gaji")
            st.selectbox("Jenis", ["pengeluaran", "pemasukan"], key="k_jenis")
            uang("Batas bulanan (Rp, opsional, hanya pengeluaran)", "k_batas")
            st.button("Simpan Kategori", type="primary", on_click=aksi_kategori)
            st.markdown("**Anggaran aktif**")
            for i, x in enumerate(d["anggaran"]):
                c = st.columns([6, 1])
                c[0].write(f"{x['kategori']}: {rp(x['terpakai'])} dari {rp(x['batas'])} ({x['persen']}%)")
                c[1].button("✕", key=f"ha{i}", on_click=jalankan, args=(sis.hapus_anggaran, "Anggaran dihapus", i))
        with kanan:
            st.subheader("Target Keuangan")
            for i, t in enumerate(d["target"]):
                c = st.columns([6, 1])
                c[0].write(f"**{t['nama']}** ({t['jenis']} · {t['kategori']}): {t['progres']}%")
                c[1].button("✕", key=f"ht{i}", on_click=jalankan, args=(sis.hapus_target, "Target dihapus", i))
            daftar_target(d)
            if d["target"]:
                st.markdown("**Setor ke target**")
                st.selectbox("Target", list(range(len(d["target"]))), key="s_target",
                             format_func=lambda i: d["target"][i]["nama"])
                st.selectbox("Dari dompet", list(nama_dompet), key="s_dompet", format_func=nama_dompet.get)
                uang("Nominal setoran (Rp)", "s_nominal")
                st.button("Setor", type="primary", on_click=aksi_setor)
            st.markdown("**Tambah target baru**")
            form_target("g")
            st.button("Simpan Target", type="primary", on_click=aksi_target)

    with t4:
        st.subheader("Class Diagram (rancangan umum)")
        gambar(STAT, 760)
        st.subheader("Diagram Class + Objek (sesuai data yang kamu input)")
        st.caption("Kotak class berisi atribut (-) dan method (+). Kotak objek berisi nilai atribut dari inputmu, "
                   "dan panah putus-putus menunjukkan objek dibuat dari class mana.")
        gambar(diagram_dinamis(d["objek"]), 900)
        st.subheader("Tabel Objek, Class, Atribut, dan Method")
        st.dataframe(pd.DataFrame([{"Objek": o["objek"], "Class": o["class"],
                                    "Atribut = Nilai": "; ".join(f"{k} = {v}" for k, v in o["atribut"].items()),
                                    "Method": ", ".join(o["method"])} for o in d["objek"]]), hide_index=True)


if "sistem" in S:
    halaman_dashboard()
else:
    halaman_daftar()
