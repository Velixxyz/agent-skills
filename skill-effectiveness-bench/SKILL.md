---
name: skill-effectiveness-bench
description: Mengukur apakah sebuah skill benar-benar membantu sebelum dipasang atau dipertahankan: baterai tugas lokal tetap, baseline tanpa-skill lebih dulu di sesi yang sama, median 3 run, tes tersembunyi, kontrol dummy, tanpa penilaian model untuk headline. Dipakai saat menilai skill baru, memutuskan trim/hapus skill lama, atau membuktikan klaim "skill ini membantu". Triggers: "apakah skill ini membantu", "bench skill", "ukur efektivitas skill", "buktikan skill ini", "lift check skill".
risk: low - evaluasi lokal read-only, tanpa aksi wallet, tanpa submit live; pemasangan tetap keputusan parent.
source: currenjin/skillbench (MIT) metode benchmark + kontrol dummy dari skill-authoring-run-first + bukti internal (catatan agen, 16 Nov 2026).
---

# Skill Effectiveness Bench

Satu-satunya cara terukur menjawab "apakah skill ini membantu?" sebelum
skill itu membebani konteks setiap giliran. Tanpa bench ini, lift-check
satu run tanpa median dan tanpa tes tersembunyi adalah anekdot — dan
anekdot tidak boleh memasang skill.

Diadaptasi dari metode skillbench (currenjin/skillbench, MIT). Angka
hasil skillbench DITOLAK (pilot n=1, baseline lintas-minggu, run
didominasi error harness — TIDAK TERBUKTI); yang diadopsi hanya
metodenya, dalam bentuk custom di bawah.

## Apa yang diubah dari upstream

- DIPOTONG: kadensi mingguan, leaderboard publik, publikasi suite
  pensiun. Evaluasi kami on-demand, sebelum pasang/trim skill.
- DIPOTONG: runner Claude Code CLI (`claude -p`, pinning `skills.yaml`).
  Diganti sesi subagent segar + usage row sebagai sumber token/biaya.
- DITAMBAH: baterai 4 tugas lokal tetap dari pekerjaan nyata (bukan
  hanya bugfix Python) — lihat Baterai di bawah.
- DITAMBAH: lengan kontrol dummy (teks netral sepanjang yang sama, dari
  `skill-authoring-run-first`): dengan-skill / tanpa-skill / dummy.
  Inilah yang membedakan lift nyata dari efek panjang konteks.
- DIPERTAHANKAN: baseline lebih dulu di sesi yang sama, median 3 run +
  flag varians, solve = tes tersembunyi lolos (bukan penilaian model),
  log mentah tersimpan, biaya per solve dilaporkan.

## Baterai (tugas tetap, dibekukan sebelum siklus)

Empat tugas, masing-masing ditulis sebagai paket tugas (format
`skill-authoring-run-first`, seksi Format paket tugas eval): `task.toml`
+ `instruction.md` + `tests/` (suite grading tersembunyi — TIDAK dibaca
runner) + `solution/` (referensi, hanya untuk sanity gate). Tugas tidak
boleh diubah di tengah siklus; denominator dibekukan sebelum run 1.

1. **make.fun** — grade lokal entri kecil vs baseline yang diketahui
   (differential + determinisme 2 run). Tanpa submit live, tanpa baca
   file entri lama (aturan authorship transkrip berlaku).
2. **Riset web3** — verifikasi klaim proyek dari 2+ sumber independen,
   dengan ground truth yang sudah ditetapkan parent.
3. **Bidding Synmerco** — draf bid + pesan untuk satu job fiktif vs
   rubrik (harga, pesan, kepatuhan scope). Tanpa bid live.
4. **Audit diff** — temuan vs ground truth pada satu diff kecil yang
   sudah dinilai parent (temuan benar / false positive / terlewat).

Setiap tugas WAJIB lolos sanity gate dua-arah SEBELUM dipakai: suite
tersembunyi GAGAL pada fixture rusak, LOLOS pada fixture yang sudah
diperbaiki, suite publik lolos di keduanya. Tugas yang tidak lolos gate
tidak dipakai — "cek yang tidak pernah disaksikan gagal" tidak bisa
dipercaya.

**Registry eval berversi (EV-1, fold-in batch 3, openai/evals MIT, 16 Nov 2026).**
Tiap suite tugas dibekukan sebagai entri registry dengan format tetap
— `id: <nama>.<split>.v<n>`, `description:` satu kalimat (kemampuan
apa yang diuji), `metrics:` daftar metrik, `samples:` file sampel +
hash-nya. Versi naik bila SATU PUN sampel/metrik berubah; suite lama
tidak pernah diedit in-place. Laporan merujuk suite persis ("suite v2,
hash …"). (Sumber: `evals/registry/evals/2d_movement.yaml`,
`evals/eval.py`.)

**Skema baris grading standar (DV-1, fold-in batch 3,
confident-ai/deepeval Apache 2.0, 16 Nov 2026).** Setiap baris grading
memuat `input` (apa yang diminta), `actual_output` (apa yang
dihasilkan lengan), `expected_output` (jawaban/keluaran emas),
`context` (fakta yang seharusnya dipakai), `tools_called` vs
`expected_tools` (jejak tool aktual vs yang diharapkan), dan
`token_cost` (biaya run). Baris yang tidak memuat keenam field ini
dinyatakan tidak lengkap. (Sumber:
`deepeval/test_case/llm_test_case.py`.)

**Stage beku (DVC-2, fold-in batch 3, iterative/dvc Apache 2.0, 16 Nov
2026).** Suite + grading yang sudah dibekukan (EV-1) ditandai FROZEN
dalam deklarasi DAG (lihat DVC-1 di Protokol siklus): repro/pipeline
TIDAK PERNAH menjalankan ulang stage beku secara implisit, bahkan bila
dep-nya berubah — perubahan atas stage beku hanya lewat keputusan
eksplisit yang menaikkan versi suite. (Sumber:
`dvc/stage/params.py:8`.)

**Lengan eval routing (K-9, eksperimental).** Terpisah dari keempat
tugas di atas, tiap skill membawa suite eval routing kecil: 3 prompt
realistis + 1-2 near-miss yang menyebut skill lain yang lebih tepat.
Suite hasil generate berstatus `pending review` dan TIDAK masuk gate
apa pun sampai ditinjau manusia. Yang diukur: routing yang benar +
tidak terpicu oleh near-miss. Status eksperimental sampai satu siklus
bench membuktikannya di harness kami; tidak ada angka hit-rate yang
diklaim.

## Protokol siklus

1. Bekukan baterai + denominator. Catat model + versi yang dipakai
   (satu model per siklus; ganti model = siklus baru, bukan perbandingan
   lintas-siklus).
2. Lengan baseline (tanpa skill) jalan LEBIH DULU, di jendela sesi yang
   sama dengan lengan lain. Perbandingan lintas-sesi/minggu dilarang.
3. Tiga lengan × 3 run per tugas, masing-masing di subagent sesi segar:
   dengan-skill, tanpa-skill, dummy. Runner TIDAK membaca suite
   tersembunyi, TIDAK membaca `solution/`.
4. Solve = tes tersembunyi lolos. Laporkan median 3 run + flag varians
   bila run tidak sepakat. Satu run tidak pernah jadi headline.
5. Biaya per solve = median token/biaya dari usage row atas run yang
   solved. Log mentah tiap run tersimpan di direktori kerja siklus.
6. Laporan per tugas: solved (median), varians, biaya per solve, dan
   field `not_verified` (apa yang TIDAK terverifikasi dalam siklus ini).

**Grader dua-tugas + beban pembuktian (A-F2).** Grader — subagent sesi
terpisah, maker≠checker — punya dua tugas: menilai output DAN mengkritik
eval-nya sendiri. Nilai lolos atas asersi yang lemah lebih buruk daripada
tidak berguna: ia menciptakan kepercayaan palsu. Grader wajib melaporkan
asersi yang terpenuhi secara trivial dan outcome penting yang tidak
diperiksa asersi mana pun. Saat ragu, beban pembuktian untuk lolos ada
pada ekspektasi: ragu = gagal, bukan lolos. Asersi yang baik bersifat
membedakan: lolos saat skill benar-benar berhasil, gagal saat tidak.
Laporan grading memakai format per-asersi: teks asersi, lolos/gagal,
bukti.

**Komparasi buta + pola per-asersi (A-F3).** Perbandingan kualitatif
antar-lengan dilakukan BUTA: juri menilai kualitas output saja, tanpa
menebak lengan mana yang menghasilkan output yang dinilainya. Juri
melaporkan PASS/FAIL + alasan per asersi, tanpa angka gabungan — skor
rubrik TIDAK PERNAH dirata-rata menjadi skor komposit (larangan F-6).
Analisis pola per asersi masuk laporan sebagai kolom: selalu lolos di
kedua lengan (tidak membedakan nilai skill), selalu gagal di kedua
lengan, hanya lolos dengan skill, hanya lolos tanpa skill (skill
mungkin merugikan), varians tinggi (ekspektasi flaky atau perilaku
non-deterministik).

**Stempel-waktu rencana + log deviasi (K-4).** Sebelum melihat outcome
run mana pun, stempel-waktu pertanyaan, kandidat, prediksi, outcome,
eksklusi, transformasi, analisis, multiplisitas, rencana missing-data,
dan aturan berhenti. Ide atau analisis yang muncul sesudah melihat
hasil WAJIB berlabel eksploratori; analisis yang direncanakan tetap
dilaporkan walau diganti. Tiap deviasi dicatat: tanggal, alasan, siapa
memutuskan, dampak yang diperkirakan. Jangan pernah menulis ulang pola
yang teramati sebagai prediksi a priori.

**Menafsir flag varians: taksonomi flaky (Q-F2).** Bila run tidak
sepakat, klasifikasikan dulu penyebabnya sebelum menyalahkan skill:
(1) race/timing, (2) shared state, (3) ketergantungan waktu, (4)
dependensi eksternal. Definisi kerja: tingkat lolos di bawah 100%
dalam 10+ run → tandai flaky (konvensi, bukan fakta terukur); 0% =
rusak, bukan flaky.

**Split tiga-arah + jawaban ditahan (DSPY-2, fold-in batch 3,
stanfordnlp/dspy MIT, 16 Nov 2026).** Untuk evaluasi apa pun yang
melibatkan tuning: tuning di train, seleksi varian di val, konfirmasi
SEKALI di test; test tidak pernah dipakai untuk tuning maupun seleksi.
Program/agen yang dievaluasi tidak pernah melihat jawaban emas — hanya
metrik yang memegangnya. Suite tuning dan suite konfirmasi adalah set
berbeda yang dibekukan terpisah. (Sumber:
`docs/docs/getting-started/metrics.md`.)

**Baseline jujur termurah dulu (DSPY-4, fold-in batch 3, dspy MIT, 16
Nov 2026).** Sebelum optimasi berat apa pun, jalankan baseline paling
bodoh yang masuk akal (nol-tuning) dan catat skornya; bila baseline
sudah mencapai target, optimasi berat adalah usaha terbuang — berhenti.
(Sumber: `docs/docs/diving-deeper/choosing-an-optimizer.md:70`.)

**Event log sebagai receipt (EV-2, fold-in batch 3, openai/evals MIT,
16 Nov 2026).** Tiap run menulis event log (JSONL): satu event per
sampel (benar/salah + expected + picked + sample_id) + event
sampling/error + event laporan akhir. SEMUA angka headline dihitung
ulang dari event log oleh fungsi turunan, bukan ditulis tangan di
laporan. Laporan yang angkanya tidak bisa direproduksi dari event log
= tidak sah. (Sumber: `evals/record.py`, `evals/metrics.py:12-17`.)

**Pemisahan empat peran (EV-3, fold-in batch 3, evals MIT, 16 Nov
2026).** Dalam tiap siklus, empat peran dipisah secara artefak: (1)
SISTEM YANG DIUJI (lengan + inputnya), (2) PROSEDUR (suite + urutan +
seed), (3) PEREKAM (event log mentah), (4) METRIK (fungsi turunan atas
event log). Satu agen/file tidak boleh merangkap prosedur + metrik +
pelaporan tanpa jejak yang bisa diaudit ulang oleh pihak lain.
(Sumber: `evals/eval.py:46-54`.)

**Seed tetap + ketidakpastian bootstrap (EV-4, fold-in batch 3, evals
MIT, 16 Nov 2026).** Tiap run menyatakan seed-nya; RNG per sampel
diturunkan dari seed agar run dapat direproduksi. Untuk metrik tipe
akurasi atas sampel kecil, laporkan simpangan baku bootstrap di
samping angka titik — angka titik tanpa ukuran ketidakpastian tidak
boleh dipakai untuk vonis "lebih bagus". (Sumber: `evals/eval.py`,
`evals/metrics.py:21-23`.)

**Metrik = skor + ambang + alasan; error = gagal (DV-2, fold-in batch
3, deepeval Apache 2.0, 16 Nov 2026).** Tiap asersi grading adalah
metrik dengan empat field wajib: `score` (angka), `threshold` (ambang
lolos, dinyatakan SEBELUM run), `reason` (alasan/bukti teks), `success`
(boolean turunan). Aturan keras: tanpa threshold yang dinyatakan di
muka, tidak ada vonis (success = None, bukan "lolos"); error saat
grading = GAGAL, bukan dilewati; `reason` wajib diisi untuk tiap
asersi — boolean tanpa alasan tidak sah sebagai bukti. (Sumber:
`deepeval/metrics/base_metric.py:172-181`.)

**Grading sebagai test yang bisa dieksekusi (DV-3, fold-in batch 3,
deepeval Apache 2.0, 16 Nov 2026).** Script grading tiap suite WAJIB
dapat dijalankan sebagai perintah test dengan exit code 0 = semua
asersi lolos, non-zero = ada yang gagal, tanpa intervensi manusia.
Grading yang hanya bisa dijalankan oleh agen yang membaca output =
tidak sah sebagai harness. (Sumber: deepeval `README.md`, pola
`assert_test`.)

**Pipeline eval sebagai DAG stage (DVC-1, fold-in batch 3, dvc Apache
2.0, 16 Nov 2026).** Rantai multi-langkah (bangun → jalankan → nilai →
bandingkan) dinyatakan sebagai DAG stage — tiap stage punya `cmd`,
`deps` (file/input + hash), `params`, `outs`, `metrics`. Output suatu
stage HANYA sah bila hash semua deps-nya masih cocok dengan yang
tercatat; bila satu dep berubah, stage itu + semua stage hilirnya
WAJIB dijalankan ulang sebelum hasilnya dipakai — hasil lama dari dep
yang sudah berubah tidak boleh dikutip. Diterapkan sebagai deklarasi
tertulis + pemeriksaan hash, BUKAN dengan menjalankan DVC. (Sumber:
`dvc/stage/params.py:3-13`, `dvc/stage/__init__.py:354-368`.)

**Perbandingan atas check ID bersama; regresi tidak dinet (ATL-3,
fold-in batch 3, lilishyxf/AgentTraceLab MIT, 16 Nov 2026).**
Perbandingan kandidat vs baseline HANYA atas asersi/check yang IDENTIK
di kedua run (ID bersama). Regresi (dulu lolos, kini gagal) dilaporkan
sebagai daftar terpisah dan TIDAK PERNAH dinet ke dalam satu delta
skor — delta skor boleh dilaporkan, tetapi daftar regresi wajib
menyertainya. Check yang hanya ada di satu sisi dinyatakan tidak
sebanding, bukan dihitung sebagai nol. (Sumber:
`src/agenttracelab/comparison.py:6-22`.)

**Randomisasi urutan pairwise (ODR-6, fold-in batch 3,
langchain-ai/open_deep_research MIT, 16 Nov 2026).** Pada penilaian
pairwise buta, acak urutan tampil tiap pasangan untuk mengontrol bias
posisi; catat seed/urutan di log run. (Sumber:
`tests/pairwise_evaluation.py:127`.)

## Aturan keputusan

- **Pasang/pertahankan** hanya bila lengan dengan-skill mengalahkan
  baseline DAN dummy pada median, tanpa flag varians yang belum
  dijelaskan.
- **Trim/hapus** bila tidak ada lift terhadap dummy — panjang konteks
  saja bukan lift.
- **Tolak** bila lebih buruk dari baseline pada tugas apa pun yang
  material, apa pun alasannya.
- Hasil n=1, perbandingan lintas-sesi, atau penilaian model sebagai
  headline TIDAK PERNAH jadi dasar keputusan.
- Bench tidak pernah mengesampingkan gerbang: skill yang "membantu"
  dengan melonggarkan gerbang wallet, gate sekali-submit, atau authorship
  transkrip tetap ditolak, apa pun skor bench-nya.
- Status `executed` terikat revisi: bila skill atau baterai berubah
  setelah siklus, hasil siklus itu basi, bukan lolos.

## Audit kesiapan (fold-in batch 3, 16 Nov 2026)

**Skor ≠ kepercayaan pada evaluasi (ATL-1, AgentTraceLab MIT).**
SEBELUM vonis bench apa pun, jalankan AUDIT KESIAPAN yang memisahkan
dua pertanyaan: (a) berapa skornya, (b) seberapa bisa dipercaya
evaluasinya. Rekomendasi audit: `ready` (bukti lengkap, vonis boleh
dipakai), `hold` (bukti kurang — vonis ditahan), atau
`diagnostic_only` (hanya untuk diagnosis, bukan vonis). Bukti yang
hilang untuk klaim kritis = `unknown` dan MEMBLOKIR kesiapan — tidak
pernah dihitung diam-diam sebagai lolos. (Sumber:
`docs/evaluation-readiness.md`, `src/agenttracelab/readiness.py`.)

**Pass-rate sebagai bound berbasis bukti (ATL-2, AgentTraceLab MIT).**
Laporkan pass-rate sebagai INTERVAL [bawah, atas] — bawah = hanya
hasil yang terverifikasi independen, atas = termasuk potensi lolos
yang belum terverifikasi — ditambah bound bawah Wilson 95% untuk angka
titik. Angka tunggal hanya sah bila SETIAP hasil terverifikasi
independen. Trial berulang harus FRESH (ID trace baru, bukan ulangan
atas jejak lama). (Sumber: `docs/evaluation-readiness.md` prinsip 2
dan 5, `src/agenttracelab/readiness.py:34-35`.)

**Audit integritas artefak (HPO-4, OpenMOSS/AgentHPOBench Apache 2.0).**
Setelah tiap siklus, jalankan audit atas artefak hasil: (a) jejak
harus PERSIS baseline + N intervensi yang dinyatakan (tidak lebih,
tidak kurang, tidak ada langkah yang hilang); (b) jumlah keputusan
tercatat harus cocok dengan jumlah intervensi; (c) kegagalan artefak
(error, timeout, file hilang) yang tidak muncul di artefak hasil =
TEMUAN AUDIT, bukan hal yang boleh disembunyikan; (d) provenance
lengkap (konfigurasi + metrik + log + keputusan) harus ada dalam satu
artefak. (Sumber:
`agenthpobench/evaluation/result_integrity.py:85-88,246-249`.)

## Buku catatan run (fold-in batch 3, mlflow/mlflow Apache 2.0, 16 Nov 2026)

**Skema run record (ML-1).** Tiap siklus bench/optimasi menulis satu
run record per run, dua blok: INFO (`run_id`, `experiment` = strategi/
skill yang diuji, `status`, `start_time`, `end_time`, `artifact_uri` =
direktori bukti) dan DATA (`params`, `metrics`, `tags`). Run record
adalah wadah bersama untuk ledger OPT-3/DEAP-5 dan event log EV-2.
(Sumber: `mlflow/entities/run.py:14,28-29`,
`mlflow/entities/run_data.py:36-38`, `mlflow/entities/run_info.py`.)

**Empat kategori tidak boleh tercampur (ML-2).** PARAM (apa yang di-set
sebelum run: varian, ambang, anggaran), METRIK (apa yang diukur sesudah
run: skor per step + timestamp + digest suite yang dipakai), TAG (label
klasifikasi: lengan/suite/seed), ARTEFAK (file bukti: event log, output
mentah). Tiap metrik wajib menyebut digest suite/dataset yang
menghasilkannya — angka dari suite berbeda tidak boleh dibandingkan
seolah setara. (Sumber: `mlflow/tracking/fluent.py` (`log_param`/
`log_metric`/`set_tag`/`log_artifact`), `mlflow/entities/metric.py`.)

**Lifecycle run eksplisit (ML-3).** Tiap run ditutup dengan STATUS
eksplisit (FINISHED / FAILED / KILLED) + `end_time`. Run yang gagal
(error, timeout, dibatalkan) TETAP tercatat dengan status FAILED dan
alasan — tidak pernah dihapus, tidak pernah ditulis ulang seolah tidak
terjadi. (Sumber: `mlflow/tracking/fluent.py:718`.)

## Checklist kesehatan eval (sebelum siklus pertama)

Fold-in batch 2 (anthropics/skills `eval-audit.md`, 16 Nov 2026),
diparafrasa custom. Jalankan SEBELUM siklus bench pertama dan sebelum
keputusan pasang/trim apa pun — harness yang tidak sehat mengukur
dirinya sendiri, bukan skillnya:

- **Oracle + null sebelum full pass.** Jalankan (a) oracle — jawaban
  referensi, atau varian yang seharusnya lolos nyaris sempurna — dan
  (b) baseline null — output kosong, jawaban konstan, atau kelas
  mayoritas — melewati seluruh pipeline. Oracle yang tidak lolos
  berarti harness atau grader rusak; null yang tidak gagal berarti
  grader terlalu longgar. Dua run ini menangkap sebagian besar bug
  wiring sebelum satu full pass dibayar.
- **Kegagalan infra dipisah dari kegagalan model.** Timeout, error API
  atau rate-limit, output tak ter-parse, respons terpotong, tool yang
  gagal, atau grader yang gagal sendiri dicatat ke sidecar error
  dengan kelas kegagalannya — TIDAK PERNAH dinilai 0 atau lolos lalu
  dicampur dengan jawaban nyata. (429 = hard stop, tidak pernah
  di-retry; kelasnya dicatat, siklusnya berhenti.)
- **Headline dihitung ulang dari baris mentah.** Angka yang dilaporkan
  dihitung ulang dari nilai per-kasus, jangan percaya field agregat.
  Kekeliruan mean-vs-sum dan per-rep-vs-per-case menghasilkan
  terobosan palsu.
- **Noise floor diukur dulu.** Ketahui varians baseline (lengan dummy,
  run berulang tanpa perubahan) sebelum mengklaim lift; lift yang
  lebih kecil dari noise bukan lift.
- **Cakupan dua arah.** Setiap cek sudah disaksikan gagal DAN lolos
  (sanity gate dua-arah di Baterai); cek yang hanya pernah lolos
  tidak bisa dipercaya.
- **Provenansi gold dicatat.** Ground truth tiap tugas membawa sumber
  + tanggalnya; ground truth yang salah mengukur hal yang salah
  dengan presisi.
- **Disiplin juri.** Headline tidak pernah berasal dari penilaian
  model; grader berjalan di sesi terpisah dari maker dan tidak
  menilai karyanya sendiri.

## Quality gates

- [ ] Checklist kesehatan eval lolos (oracle + null, pemisahan infra,
      headline dari baris mentah, noise floor, provenansi gold),
      sebelum run 1.

- [ ] Baterai dibekukan + sanity gate dua-arah lolos untuk tiap tugas,
      sebelum run 1.
- [ ] Baseline jalan dulu, di jendela sesi yang sama.
- [ ] 3 lengan × 3 run, sesi segar, suite tersembunyi tidak dibaca runner.
- [ ] Laporan menyebut median + varians + biaya per solve + `not_verified`.
- [ ] Tidak ada aksi wallet, submit live, atau bid live di dalam siklus.
- [ ] Rencana + prediksi distempel-waktu sebelum outcome dilihat;
      deviasi tercatat (tanggal/alasan/pemutus/dampak) (K-4).
- [ ] Analisis eksploratori berlabel eksploratori; analisis yang
      direncanakan tetap dilaporkan (K-4).
- [ ] Split train/val/test + jawaban ditahan dinyatakan sebelum tuning
      (DSPY-2); event log ditulis dan headline dihitung ulang darinya
      (EV-2).
- [ ] Audit kesiapan jalan sebelum vonis; pass-rate dilaporkan sebagai
      bound berbasis bukti + bound bawah Wilson (ATL-1, ATL-2).
- [ ] Run record ditulis per run (info/data, empat kategori terpisah,
      status eksplisit termasuk run gagal) (ML-1 s.d. ML-3).

## Contoh pemakaian

"Bench skill X sebelum dipasang" → bangun/bekukan baterai → jalankan
siklus 3 lengan × 3 run → laporkan median per tugas + keputusan
pasang/trim/tolak dengan bukti. "Apakah skill Y masih layak
dipertahankan?" → siklus yang sama terhadap skill terpasang; tanpa lift
vs dummy, Y kandidat trim (penghapusan tetap butuh ya eksplisit pengguna
per skill, sesuai `skill-authoring-run-first` mode prune).

## Limitations

- Bench mengukur lift pada 4 tugas lokal, bukan kualitas absolut atau
  performa di tugas yang belum pernah dilihat.
- Bench tidak mengukur biaya konteks jangka panjang; skill yang lolos
  bench tetapi jarang dipanggil tetap kandidat prune.
- Suite tersembunyi hanya sebagus ground truth parent; ground truth
  yang salah mengukur hal yang salah dengan presisi.

## When-to-Use / Do-not-use

Pakai: menilai skill baru sebelum dipasang; memutuskan trim/hapus skill
lama; membuktikan atau membantah klaim "skill ini membantu"; membandingkan
dua versi skill yang sama.

Jangan pakai: menggantikan run-first (`skill-authoring-run-first` tetap
menentukan APA isi skill — bench hanya mengukur apakah isinya membantu);
menilai skill dari satu run; membandingkan hasil lintas model/sesi;
memberi skor komposit atas keempat tugas (laporan per tugas, tidak
pernah dijumlah — lihat larangan skor komposit di
`precision-task-optimizer`).
