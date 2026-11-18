# TA-06: Pemodelan dan Simulasi Sistem Dinamik Termal Liquid Cold-Plate pada Rak Server GPU AI

**Identitas Mahasiswa:**
- **Nama:** Sigit Miraj Permana
- **NIM:** 301240037
- **Kelas:** 5B - Teknik Informatika
- **Fakultas:** Fakultas Teknologi Informasi, Universitas Bale Bandung (FTI UNIBBA)

---

## Deskripsi Proyek
Proyek ini memodelkan dan mensimulasikan **sistem dinamik termal berbasis *Liquid Cold-Plate* (Direct-to-Chip Cooling)** pada rak server komputasi kecerdasan buatan (AI) berdensitas tinggi menggunakan pendekatan **Sistem Persamaan Diferensial Biasa (ODE)** non-linier yang diselesaikan secara numerik menggunakan `scipy.integrate.solve_ivp` (metode Runge-Kutta Orde 4/5 - RK45).

Model ini merepresentasikan dinamika perpindahan panas antara tiga nodus termal utama pada rak server GPU AI (misalnya konfigurasi 8x NVIDIA H100 SXM5):
1. **Suhu Die Silikon GPU ($T_{gpu}$):** Dipengaruhi oleh beban daya komputasi AI ($P_{gpu}(t)$) yang fluktuatif (fase *idle*, *forward-pass*, *backpropagation*, hingga *thermal throttling*), konduksi termal melalui *Thermal Interface Material* (TIM) ke *cold-plate*, serta disipasi konveksi pasif ke udara sekitar.
2. **Suhu Blok Tembaga Liquid Cold-Plate ($T_{cp}$):** Menyerap panas dari paket GPU dan melepaskannya ke cairan pendingin (*coolant* berbasis campuran air-glikol / *deionized water*) melalui konveksi paksa.
3. **Suhu Cairan Pendingin Keluar ($T_{cool,out}$):** Dipengaruhi oleh laju aliran massa cairan ($\dot{m}(t)$) yang diatur oleh kontroler pompa variabel (PWM / PID) serta efisiensi penukar panas (*Coolant Distribution Unit* / CDU) untuk menjaga suhu suplai ($T_{cool,in}$).

---

## Struktur Direktori
```text
TA06_Simulasi_Termal_Liquid_ColdPlate_301240037/
├── TA06_Thermal_Liquid_ColdPlate_GPU_301240037.ipynb   # Notebook utama pemodelan ODE, simulasi skenario, & visualisasi
└── README.md                                           # Dokumentasi proyek, formulasi matematis, & panduan instalasi
```

---

## Formulasi Matematis (Sistem ODE)

Dinamika perubahan suhu pada setiap komponen dimodelkan melalui neraca energi termal berikut:

$$C_{gpu} \frac{dT_{gpu}}{dt} = P_{gpu}(t) - \frac{T_{gpu} - T_{cp}}{R_{tim}} - h_{amb} A_{amb} (T_{gpu} - T_{amb})$$

$$C_{cp} \frac{dT_{cp}}{dt} = \frac{T_{gpu} - T_{cp}}{R_{tim}} - h_{conv}(\dot{m}) A_{cp} (T_{cp} - T_{cool,avg})$$

$$C_{cool} \frac{dT_{cool,out}}{dt} = h_{conv}(\dot{m}) A_{cp} (T_{cp} - T_{cool,avg}) - \dot{m}(t) c_{p,cool} (T_{cool,out} - T_{cool,in})$$

dengan $T_{cool,avg} = \frac{T_{cool,in} + T_{cool,out}}{2}$ dan koefisien konveksi paksa bergantung pada laju aliran melalui korelasi bilangan Nusselt ($h_{conv}(\dot{m}) = h_0 \left(\frac{\dot{m}}{\dot{m}_0}\right)^{0.8}$).

---

## Skenario Simulasi di dalam Notebook
1. **Skenario 1 (Baseline - Constant Flow Rate):** Beban pelatihan LLM dinamis (lonjakan daya hingga 5.6 kW per node) dengan laju aliran pompa konstan.
2. **Skenario 2 (Adaptive Pump Control / Proportional-Integral):** Pengaturan laju aliran pompa variabel berbasis suhu *junction* GPU untuk menjaga $T_{gpu} < 75^\circ\text{C}$ sekaligus menghemat daya pompa ($P_{pump} \propto \dot{m}^3$).
3. **Skenario 3 (Anomali & Degradasi Sistem):** Simulasi kegagalan parsial pompa (penurunan debit 60%) dan mekanisme perlindungan *hardware thermal throttling* otomatis saat $T_{gpu} \ge 83^\circ\text{C}$.
4. **Analisis Sensitivitas Parameter:** Pengaruh resistansi termal TIM ($R_{tim}$) dan suhu suplai CDU ($T_{cool,in}$) terhadap margin termal puncak.

---

## Cara Menjalankan Proyek
1. Pastikan Python 3.9+ telah terpasang di lingkungan kerja Anda (VS Code / Jupyter Notebook).
2. Buat dan aktifkan *virtual environment* (`venv`):
   ```powershell
   python -m venv venv
   .\venv\Scripts\activate
   ```
3. Instal pustaka yang diperlukan:
   ```powershell
   pip install numpy scipy matplotlib pandas seaborn jupyter
   ```
4. Buka dan jalankan seluruh sel pada file `TA06_Thermal_Liquid_ColdPlate_GPU_301240037.ipynb`:
   ```powershell
   jupyter notebook TA06_Thermal_Liquid_ColdPlate_GPU_301240037.ipynb
   ```
