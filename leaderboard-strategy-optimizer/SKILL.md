# Leaderboard Strategy Optimizer

Cara mengalahkan benchmark Top 1 lewat eksperimen terukur, bukan imitasi buta. Diadaptasi dari playbook eksternal "Leaderboard Strategy Optimizer v1.0.0" yang ditempel pengguna 15 Nov 2026, dievaluasi lalu dikembangkan selektif: kerangka loop-nya dipertahankan, bagian generik dan berbahayanya dibuang atau diikat ke track nyata dan gerbang berdiri.

Skill ini adalah METODE, bukan otorisasi. Skill ini tidak memberi izin aksi wallet, submit, atau bayar apa pun. Semua gerbang berdiri tetap berlaku.

## Evaluasi playbook sumber

Yang bagus dan dipertahankan:

- Benchmark dulu sebelum optimasi, hipotesis terukur, eksperimen terkontrol, validasi di job yang belum pernah dilihat, deploy hanya strategi terverifikasi.
- Larangan fabrikasi performa, larangan mengacaukan hasil simulasi dengan hasil resmi, arsip eksperimen yang bisa direproduksi, analisis kegagalan per tahap.

Yang dibuang atau diubah, dan alasannya:

- Mesin simulasi generik dibuang. Tidak ada simulator untuk award Synmerco (buyer yang memutuskan) atau juri make.fun (pairwise AI yang opaken). Simulasi hanya jujur di Agent Cards (sim match lokal) dan sebagian make.fun (grade lokal). Di track lain, eksperimen adalah trial live kecil yang tercatat, bukan simulasi. Jangan pernah menyebut trial live sebagai simulasi.
- Loop otonom tanpa batas diganti siklus berbatas. Satu siklus sama dengan satu hipotesis, satu eksperimen, satu catatan. Loop kontinu hanya lewat cron dengan scope dan budget eksplisit. Tanpa batas, loop akan menghabiskan trial, biaya, dan kuota submit untuk perbaikan semu.
- Reverse engineering tanpa batas dibatasi ke bukti publik. Pisahkan fakta teramati dari hipotesis. Jangan pernah klaim tahu reasoning privat kompetitor, dan jangan pernah pakai data kompetitor yang tidak sah.
- Deploy tanpa gerbang ditolak. Setiap deployment tunduk ke gerbang berdiri: wallet gate, gate sekali-submit make.fun, dan ya eksplisit untuk aksi berbayar.
- Model "kalahkan Top 1 lewat strategi" salah untuk Synmerco. Rule 18 Operating Brief: award job Synmerco-posted mendahulukan builder yang belum pernah di-award, lalu skor lebih tinggi, dan harga hanya tie-break antar skor sama. Optimizer yang mengabaikan aturan award akan mengoptimasi hal yang salah. Buktinya: tangga harga dan eksperimen 50 persen hampir tidak mengubah peluang menang, hanya payout. Di Synmerco, tuas yang benar adalah pembangunan skor, jendela builder segar, dan cakupan tag.

## Loop satu siklus

Jalankan urutan ini per siklus, dan catat sebelum eksperimen jalan:

- OBSERVE. Ambil snapshot benchmark segar: posisi Top 1, Top 5, Top 10, metrik resmi, dan performa sendiri saat ini. Catat sumber dan waktu cek WIB.
- HYPOTHESIZE. Tulis satu hipotesis yang bisa dibantah: perubahan apa, kenapa diperkirakan menang, metrik apa yang bergerak, dan berapa besar gerakan yang dihitung menang. Daftarkan kriteria menang SEBELUM melihat hasil, dan jangan ubah definisinya sesudah hasil keluar.
- EXPERIMENT. Uji satu variabel per eksperimen. Jangan ubah dua hal sekaligus lalu klaim tahu penyebabnya. Baseline selalu tercatat dan kondisi sebanding: job set sama, aturan sama, jendela waktu sebanding.
- VALIDATE. Uji kandidat di job yang belum pernah dipakai untuk tuning. Pola yang hanya menang di job latih belum menang. Bila eksekusi stokastik, ulangi trial dan laporkan mean plus varians, bukan satu angka terbaik.
- DECIDE. Deploy, buang, atau iterasi, berdasarkan kriteria yang sudah didaftarkan. Deploy hanya bila kandidat mengalahkan baseline di metrik resmi dan lolos validasi unseen.
- MONITOR. Pantau degradasi setelah deploy. Bila performa turun melewati ambang yang sudah ditentukan, rollback ke versi arsip dan laporkan. Jangan kejar kerugian dengan eksperimen dadakan.

Loop reflektif di dalam EXPERIMENT dan VALIDATE (fold-in batch 3, gepa-ai/gepa MIT + stanfordnlp/dspy MIT, 16 Nov 2026):

- **Metrik + baseline sebelum optimasi (DSPY-1).** Tiap siklus menulis metrik dulu — fungsi atas satu contoh/prediksi yang mengembalikan satu angka — lalu mengukur baseline SEBELUM mengubah apa pun. Tiap varian dinilai dengan metrik yang SAMA pada set yang SAMA; hanya varian dengan skor terbaik yang dipromosikan. Tanpa metrik tertulis, tidak ada siklus optimasi. (Sumber: dspy `docs/docs/getting-started/metrics.md`.)
- **Loop reflektif Select → Execute → Reflect → Mutate → Accept (GEPA-1).** (1) Pilih kandidat dari frontier, bukan selalu varian terakhir; (2) eksekusi pada minibatch dengan trace penuh tersimpan; (3) diagnosis TERTULIS atas trace — mengapa gagal, dengan bukti trace; (4) mutasi yang mengutip diagnosis + pelajaran dari semua leluhur varian; (5) terima hanya bila membaik, lalu perbarui frontier. Evaluator WAJIB mengembalikan Actionable Side Information (ASI): skor + umpan balik diagnostik teks, bukan skor telanjang. Tanpa diagnosis tertulis, mutasi tidak boleh dibuat. (Sumber: gepa `README.md`, "How It Works".)
- **Anggaran delegasi (ODR-4).** Disiplin delegasi supervisor — bias ke satu agen, refleksi sebelum/sesudah delegasi, batas keras iterasi + paralelisme — didefinisikan penuh di `precision-task-optimizer`, seksi "Loop gates and chain discipline". Loop ini memakai disiplin itu apa adanya; tidak ada definisi kedua di sini.

## Binding per track

Synmerco:

- Benchmark hidup di bid-log.md, win-strategy.md, dan Operating Brief. Metrik: award rate, payout per job, Synmerco Score, ukuran field, award lag.
- Tiap bid adalah satu trial. Variabel yang sah: cakupan tag, spesialisasi capability, kecepatan bid, dan harga (lemah, lihat Rule 18 di atas).
- Bukan eksperimen: bid buta tanpa log, bid tanpa membaca syarat job, atau mengubah dua variabel sekaligus.
- Deploy berarti mengubah aturan bidding berdiri. Perubahan harga atau scope butuh keputusan pengguna.

make.fun:

- Benchmark adalah leaderboard per bounty plus skor dan rank entri sendiri. Metrik: skor juri, rank, persentil rank.
- Simulasi yang jujur: grade lokal, replay byte-identical, ignored-structure test, robustness overfitting. Skor grade lokal BUKAN prediksi skor juri.
- Deploy berarti submit. Gate sekali-submit berlaku: resubmit hanya bila TERBUKTI mengalahkan entri lama; kompetisi juri adalah judgment call pengguna karena tidak ada juri lokal yang bisa membuktikan perbaikan.
- Monitor lewat entry-watch tiap 30 menit.

Agent Cards:

- Benchmark adalah rating dan rank ladder. Ini satu-satunya track dengan simulator nyata: sim match lokal melawan archetype lawan.
- Program yang tes lebih baik dari program live boleh di-set dan langsung dilaporkan, sesuai aturan berdiri.
- Monitor lewat evaluasi tiap ronde; lawan dipilih dengan filter role advantage, level, rating, rekor, dan skill danger.

Dialectica:

- Benchmark adalah RAR, saldo $TRUED, dan jumlah suspensi. Variabel: strategi arena dan cap harian.
- Guard bersifat report-only. Perubahan strategi atau cap hanya atas perintah pengguna.
- Sinyal penilai yang berulang 2 sampai 3 kali langsung dibekukan jadi checklist gate tertulis, dan semua antrean diaudit sekaligus.

IPX Fan SZN:

- Benchmark adalah Aura dan gap ke batas top 100. Variabel: sudut dan topik konten.
- Tiap konten adalah satu trial dengan skor Alfred sebagai metrik. Evaluasi tiap saran Alfred diterapkan konkret ke draf berikutnya.
- Aturan keras track ini (jeda, tanpa cashtag, tanpa em dash) tidak boleh dilanggar atas nama eksperimen.

## Aturan eksperimen

- Pre-register setiap eksperimen: ID, tanggal WIB, hipotesis, variabel, baseline, metrik, kriteria menang, dan budget. Tanpa pre-register, hasilnya hanya anekdot.
- Satu variabel per eksperimen, baseline tercatat, kondisi sebanding.
- Hasil simulasi tidak pernah dilaporkan sebagai hasil resmi. Sebutkan selalu jalur mana yang menghasilkan angka: sim lokal, trial live, atau hasil resmi.
- Kegagalan adalah data. Klasifikasikan strategis versus operasional, identifikasi tahap gagal, buat hipotesis korektif, dan ajukan eksperimen berikutnya. Jangan ubah komponen yang sukses tanpa tes.
- Jaga diversitas kandidat. Simpan arsip elite berisi strategi terbaik, lineage, konfigurasi parameter, dan metadata reproduksi. Jangan konvergen prematur ke satu pola.
- Stop condition mengikat: budget habis, tiga eksperimen berturut tanpa perbaikan bermakna, atau degradasi melewati ambang. Saat stop, laporkan dan berhenti. Jangan tambah trial untuk menutup kekalahan.
- Bila 2 sampai 3 sinyal penilai yang sama muncul, bekukan jadi checklist gate tertulis, audit semua antrean sekaligus, dan jadikan gate sebelum tiap aksi berikutnya.
- Preconditions loop (fold-in agent-skills `_common/LOOP_PRECONDITIONS.md`, 16 Nov 2026): JANGAN luncurkan loop apa pun sebelum 5 gate ini lolos — (1) oracle penyelesaian yang bisa diverifikasi (exit 0 ⟺ selesai; tujuan subjektif ditolak), (2) hard-stop eksternal (default ≤3 siklus dalam-sesi; batas dari luar, bukan penilaian diri agen), (3) maker ≠ checker (generator tidak menilai karyanya sendiri; checker = subagent sesi terpisah, dan untuk make.fun checker TIDAK boleh membaca file entri sebelum menulis penilaiannya — aturan authorship transkrip), (4) memori persisten di luar percakapan (file/DB/git), (5) kesadaran drift (kualitas bisa memburuk antar iterasi walau tes tetap lolos — batasi loop + baca sampel). Denominator (set pertanyaan, daftar axis, metrik) DIBEKUKAN sebelum siklus 1. Konversi yang diizinkan untuk gate #1: split oracle, frontier exhaustion, coverage set, paritas diferensial. Bila tidak ada konversi yang berlaku, berhenti — jangan luncurkan loop tanpa gate.
- Larangan formula tanpa sumber data (fold-in agent-skills `_common/TOKEN_ECONOMY.md`, 16 Nov 2026): formula tanpa sumber data tidak pernah disajikan dengan grade band atau diikat ke gate. Berlaku untuk skor komposit, estimasi ROI, proyeksi rank, dan "health score" apa pun. Metrik tanpa sumber data ditulis "specified, not implemented", bukan diberi angka.
- **Default generalisasi atas contoh (DSPY-3).** Bila set evaluasi kecil atau miring, utamakan perubahan tingkat pola/instruksi (prinsip umum yang transfer) atas perubahan tingkat contoh (menghafal contoh latih). Varian yang hanya menang di contoh latih tanpa alasan pola yang dinyatakan dicurigai overfit sampai dibuktikan di held-out. Ini default, bukan hukum. (Sumber: dspy `docs/docs/diving-deeper/choosing-an-optimizer.md:35-37`.)
- **Kriteria penerimaan dinyatakan di muka (GEPA-2).** Default `strict_improvement`: terima hanya bila skor minibatch membaik ketat. `improvement_or_equal` hanya bila eksperimen dinyatakan sebagai eksplorasi wilayah solusi baru, dan langkah lateral dicatat sebagai eksplorasi, bukan promosi. (Sumber: gepa `docs/docs/guides/acceptance-criterion.md`.)
- **Seleksi dari frontier Pareto per kunci + eksplorasi (GEPA-3).** Pelihara frontier varian per kunci (contoh validasi / objektif); mutasi berikutnya dipilih dari frontier dengan probabilitas sebanding luasnya kemenangan kunci, bukan selalu dari juara agregat; 10% langkah adalah eksplorasi acak dari pool yang dinyatakan sebagai eksplorasi. Merge dua kandidat Pareto-optimal yang unggul di tugas berbeda diizinkan HANYA bila kedua induk lolos proof bar masing-masing. (Sumber: gepa `docs/docs/guides/candidate-selection.md`.)
- **Pruning dini atas nilai antara (OPT-1).** Untuk evaluasi panjang yang punya nilai antara pada langkah tetap, hentikan dini varian yang nilai antaranya lebih buruk dari median varian-varian sebelumnya pada langkah yang sama. Jangan pruning sebelum langkah pemanasan minimum + sebelum cukup trial selesai untuk median yang bermakna; trial yang di-prune dicatat PRUNED (bukan gagal) dengan nilai antara terakhirnya. (Sumber: optuna `optuna/pruners/_median.py:4-8`.)
- **Successive halving (OPT-2).** Bila varian kandidat banyak, saring semuanya dengan evaluasi murah (minibatch/subset), promosikan hanya fraksi teratas ke evaluasi penuh, ulangi. Anggaran dialokasikan bertingkat, bukan merata. (Sumber: optuna `optuna/pruners/_successive_halving.py:16-19`.)
- **Study note + ledger trial (OPT-3).** Sebelum eksperimen pencarian, tulis study note: objektif + arah tiap objektif, ruang pencarian (tiap dimensi: nama + rentang/pilihan yang diizinkan), anggaran trial maksimum. Tiap trial dicatat: parameter + nilai + status (complete/pruned/failed). "Best trials" = frontier Pareto atas ledger — trial ada di frontier bila tidak ada trial lain yang mendominasinya — bukan satu juara. (Sumber: optuna `optuna/study/study.py:171-174`, `optuna/trial/_trial.py`.)
- **Skema seleksi dinyatakan di muka; default elitis (DEAP-2, parafrasa custom, LGPL — tanpa kutipan sumber).** Tiap eksperimen menyatakan skema seleksinya: ELITIS (juara bertahan ikut bersaing dengan penantang; juara hanya tergusur bila penantang terbukti lebih baik pada metrik yang sama) atau GENERASIONAL (seluruh induk dibuang, generasi berikut hanya dari anak). Default elitis. Skema generasional hanya untuk eksperimen eksplorasi yang dinyatakan demikian, dan juara lama tetap aman di hall of fame (lihat Catatan eksperimen). (Pointer: deap `deap/algorithms.py:248,340`.)
- **Seleksi turnamen untuk evaluasi berisik/mahal (DEAP-3, parafrasa custom, LGPL — tanpa kutipan sumber).** Bila ranking global tidak dapat dipercaya atau terlalu mahal, seleksi memakai turnamen: ambil k varian acak, majukan yang terbaik; ukuran turnamen dinyatakan di muka (turnamen besar = tekanan seleksi tinggi). Hanya untuk seleksi antar-tahap, bukan pengganti proof bar final. (Pointer: deap `deap/tools/selection.py:51-66`.)
- **Probabilitas operator + asal-usul tercatat (DEAP-4, parafrasa custom, LGPL — tanpa kutipan sumber).** Tiap eksperimen menyatakan berapa persen keturunan berasal dari gabung dua induk, dari mutasi satu induk, dan dari seed baru. Tiap varian mencatat asal-usulnya: induk + operator yang dipakai + generasi. Tanpa asal-usul tercatat, varian tidak boleh dipromosikan. (Pointer: deap `deap/algorithms.py:85,248,340`, parameter `cxpb`/`mutpb`.)
- **Split train/val/test + jawaban ditahan (DSPY-2).** Definisi penuh di `skill-effectiveness-bench`, seksi "Protokol siklus": tuning di train, seleksi di val, konfirmasi sekali di test; program yang dinilai tidak pernah melihat jawaban emas. Siklus di skill ini memakai disiplin itu apa adanya.
- **Diagnosis dari trace (GEPA-4).** Definisi penuh di `systematic-debugging`, Phase 3: diagnosis kegagalan wajib membaca trace lengkap sebelum mutasi. Siklus di skill ini memakai disiplin itu apa adanya.

## Dossier keputusan (fold-in batch 3, OpenMOSS/AgentHPOBench Apache 2.0, 16 Nov 2026)

- **Dossier tertulis sebelum tiap mutasi (HPO-1).** Sebelum mengusulkan mutasi apa pun, susun dossier dengan field: log terakhir yang relevan, konfigurasi saat ini, SELURUH riwayat metrik (bukan hanya yang terakhir), anggaran tersisa vs total, ruang pencarian yang dinyatakan, nomor keputusan, dan arah metrik (lebih tinggi/lebih rendah = lebih baik). Mutasi tanpa dossier = tidak sah. (Sumber: AgentHPOBench `README.md`; `agenthpobench/core/agent_base.py:14-27`.)
- **Validasi usulan terhadap ruang, sebelum eksekusi (HPO-2).** Ruang mutasi dinyatakan di muka (parameter yang boleh berubah + rentang/nilai yang sah + asal batasnya). Tiap usulan divalidasi secara mekanis terhadap ruang itu sebelum dieksekusi; parameter tak dikenal atau nilai di luar rentang = DITOLAK dengan pesan yang menyebut parameter + batas yang dilanggar. (Sumber: `agenthpobench/core/agent_base.py:81-110`.)
- **Keputusan tercatat: konfigurasi + penalaran + keyakinan (HPO-3).** Tiap keputusan direkam sebagai konfigurasi baru, penalaran (mengapa, merujuk ke field dossier mana), keyakinan 0–1, dan metadata. Penalaran yang tidak merujuk bukti dossier = keputusan tidak sah. Keyakinan diri TIDAK PERNAH dipakai sebagai bukti penerimaan — hanya untuk kalibrasi retrospektif; bukti hanya dari evaluasi terukur. (Sumber: `agenthpobench/core/agent_base.py:29-36`.)

## Catatan eksperimen

- Simpan di `<workspace>/work/<track>/experiments/` atau di hidden_files goal terkait. Satu file per eksperimen.
- Isi wajib: ID dan tanggal WIB, hipotesis, variabel yang diubah, baseline, metrik dan hasil mentah, keputusan (deploy, buang, iterasi), lineage (eksperimen induk bila ada), dan konfigurasi yang bisa direproduksi.
- Jangan simpan rahasia apa pun di catatan eksperimen: tanpa API key, tanpa private key, tanpa claim code.
- **Hall of fame (DEAP-1, parafrasa custom, LGPL — tanpa kutipan sumber).** Juara terbaik yang pernah terukur disimpan terpisah dari pool varian aktif dan tidak pernah hilang saat pool berganti generasi; daftarnya selalu terurut dengan yang terbaik di depan; bila seri, juara lama dipertahankan; varian yang setara menurut kriteria kesetaraan yang dinyatakan tidak mengisi dua slot. (Pointer: deap `deap/tools/support.py:488-510`.)
- **Anggaran evaluasi + cakupan minibatch (GEPA-5).** Anggaran evaluasi maksimum dinyatakan di muka per eksperimen; screening memakai minibatch, konfirmasi penuh hanya untuk kandidat promosi; komposisi minibatch dicatat karena menentukan failure mode yang terlihat; urutan contoh di-seed agar run dapat direproduksi; cakupan epoch — tiap contoh terlihat sekali sebelum ada yang diulang. (Sumber: gepa `docs/docs/guides/batch-sampling.md`.)
- **Buku catatan run (ML-1, ML-2).** Format run record baku — blok info vs data, empat kategori param/metrik/tag/artefak yang tidak boleh tercampur, tiap metrik terikat digest suite — didefinisikan penuh di `skill-effectiveness-bench`, seksi "Buku catatan run". Catatan eksperimen di skill ini memakai format itu; ledger OPT-3/DEAP-5 dan event log EV-2 adalah isi/artefak di dalam run record tersebut.

## Gerbang yang tidak bisa digeser skill ini

- Wallet gate: preview plus ya eksplisit per aksi on-chain. Hanya ya atas preview spesifik yang mengotorisasi.
- Gate sekali-submit make.fun dan semua aturan kuota submit.
- Tanpa aksi berbayar, deposit, stake, atau pembelian credit tanpa ya eksplisit per aksi.
- Tanpa manipulasi leaderboard, tanpa exploit infrastruktur benchmark, tanpa bypass batasan kompetisi, tanpa data kompetitor yang tidak sah.
- Tanpa klaim tahu reasoning privat kompetitor.
- Rollback selalu tersedia sebelum deploy. Pakai canary deployment bila didukung platform.

## Laporan per siklus

Satu laporan singkat per siklus, teks faktual plus bukti kunci, tanpa narasi step-by-step:

- Benchmark Top 1 saat ini, angka plus sumber plus waktu cek.
- Performa sendiri saat ini di metrik yang sama.
- Kandidat terbaik dan hasil validasinya.
- Jumlah eksperimen, berapa berhasil dan berapa gagal.
- Perbaikan terukur terhadap baseline.
- Biaya eksekusi (trial, gas, token, waktu).
- Hasil generalisasi di job unseen.
- Keputusan deploy dan alasannya.
- Hipotesis berikutnya, bila ada.

## Skill terkait

- **Synmerco Agent Bidding** (skill terpisah, tidak termasuk repo ini) untuk aturan bidding, tangga harga, dan Rule 18.
- **Make.fun Bounty Hunter** (skill terpisah, tidak termasuk repo ini) untuk gate sekali-submit, gate transkrip, dan evaluasi entri.
- [Verification Before Completion](../verification-before-completion/SKILL.md) untuk standar verifikasi sebelum mengklaim hasil.
- [Systematic Debugging](../systematic-debugging/SKILL.md) untuk analisis akar masalah saat eksperimen gagal.
- [Precision Task Optimizer](../precision-task-optimizer/SKILL.md) untuk optimasi eksekusi tugas.
