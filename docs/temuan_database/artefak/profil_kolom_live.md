# Profil Kolom Live — database `penandaan` (2026-08-13)

Per kolom: jumlah NULL (SQL NULL, bukan sentinel string), persentase, dan cacah nilai distinct.
Diikuti katalog nilai untuk kolom berkardinalitas rendah.

---


### coverage_balai  —  668 rows
  - id_balai                             bigint       null=        0 (  0.0%)  distinct=88
  - nama_balai                           text         null=        0 (  0.0%)  distinct=88
  - id_kabupaten                         integer      null=        0 (  0.0%)  distinct=514
  - kabupaten_kota                       text         null=        0 (  0.0%)  distinct=514
  - sync                                 timestamp without time zone null=        0 (  0.0%)  distinct=1

### mv_penandaan  —  292,758 rows
  - id                                   bigint       null=        0 (  0.0%)  distinct=292758
  - komoditi                             text         null=        0 (  0.0%)  distinct=8
  - nomor_surat                          text         null=        0 (  0.0%)  distinct=22495
  - nomorsampel                          text         null=   31,984 ( 10.9%)  distinct=260702
  - tgl_start                            date         null=        0 (  0.0%)  distinct=1283
  - tgl_end                              date         null=        0 (  0.0%)  distinct=1288
  - nama_produk                          text         null=        0 (  0.0%)  distinct=84807
  - ed_nie                               date         null=   29,278 ( 10.0%)  distinct=3505
  - pendaftar                            text         null=        0 (  0.0%)  distinct=4327
  - produsen                             text         null=        0 (  0.0%)  distinct=20492
  - nama_balai                           text         null=        1 (  0.0%)  distinct=83
  - kesimpulan_penilaian_balai           text         null=        0 (  0.0%)  distinct=6
  - kesimpulan_penilaian_pusat           text         null=        0 (  0.0%)  distinct=5
  - catatan                              text         null=        0 (  0.0%)  distinct=27101
  - sync                                 timestamp without time zone null=        0 (  0.0%)  distinct=1
    VALUES[komoditi] (8): KOSMETIKA=78,550 | PRODUK PANGAN=75,629 | OBAT=53,175 | OBAT TRADISIONAL (OT)=39,189 | ROKOK=32,081 | SUPLEMEN KESEHATAN=10,757 | OBAT KUASI=2,688 | KEMASAN PANGAN=689
    VALUES[kesimpulan_penilaian_balai] (6): MK=177,440 | =85,257 | TMK=14,140 | TMK MINOR=7,424 | TMK MAYOR=6,607 | TMK Minor=1,890
    VALUES[kesimpulan_penilaian_pusat] (5): MK=151,777 | TMK=76,547 | VP=59,831 | TMK MAYOR=2,367 | TMK MINOR=2,236

### mv_penandaan_agg  —  94,737 rows
  - periode_type                         text         null=        0 (  0.0%)  distinct=2
  - tanggal_periode                      date         null=        0 (  0.0%)  distinct=1290
  - komoditi                             text         null=        0 (  0.0%)  distinct=8
  - nama_balai                           text         null=        2 (  0.0%)  distinct=83
  - kesimpulan_penilaian_balai           text         null=        0 (  0.0%)  distinct=6
  - kesimpulan_penilaian_pusat           text         null=        0 (  0.0%)  distinct=5
  - jumlah_penandaan                     bigint       null=        0 (  0.0%)  distinct=138
  - jumlah_surat_unik                    bigint       null=        0 (  0.0%)  distinct=29
  - jumlah_sampel_unik                   bigint       null=        0 (  0.0%)  distinct=139
  - jumlah_produk_unik                   bigint       null=        0 (  0.0%)  distinct=127
  - avg_durasi_hari                      double precision null=        0 (  0.0%)  distinct=888
  - min_durasi_hari                      integer      null=        0 (  0.0%)  distinct=45
  - max_durasi_hari                      integer      null=        0 (  0.0%)  distinct=53
  - last_updated                         timestamp without time zone null=        0 (  0.0%)  distinct=1
    VALUES[periode_type] (2): day=63,987 | month=30,750
    VALUES[komoditi] (8): KOSMETIKA=23,761 | PRODUK PANGAN=23,014 | OBAT TRADISIONAL (OT)=14,201 | OBAT=11,819 | ROKOK=10,749 | SUPLEMEN KESEHATAN=7,387 | OBAT KUASI=2,996 | KEMASAN PANGAN=810
    VALUES[kesimpulan_penilaian_balai] (6): MK=49,183 | =22,570 | TMK=11,356 | TMK MINOR=5,497 | TMK MAYOR=4,894 | TMK Minor=1,237
    VALUES[kesimpulan_penilaian_pusat] (5): MK=43,921 | TMK=27,913 | VP=19,678 | TMK MINOR=1,643 | TMK MAYOR=1,582
    VALUES[jumlah_surat_unik] (29): 1=65,521 | 2=17,415 | 3=6,057 | 4=2,571 | 5=1,194 | 6=720 | 7=425 | 8=253 | 9=180 | 10=94 | 11=81 | 12=61 | 13=34 | 14=28 | 17=21 | 15=20 | 16=17 | 20=11 | 18=8 | 19=7 | 21=6 | 24=3 | 26=3 | 34=2 | 22=1 | 33=1 | 29=1 | 25=1 | 23=1
    VALUES[min_durasi_hari] (45): 0=89,883 | 30=976 | 29=668 | 1=657 | 2=276 | 3=256 | 4=231 | 7=192 | 27=188 | 6=151 | 28=136 | 5=136 | 8=102 | 26=69 | 13=61 | 10=61 | 31=59 | 22=52 | 9=52 | 12=51 | 25=49 | 20=44 | 14=43 | 11=42 | 24=38 | 15=35 | 16=35 | 19=29 | 18=28 | 23=26 | 21=24 | 17=21 | 34=15 | 32=12 | 37=9 | 35=7 | 33=6 | 36=5 | 61=3 | 44=2 | 91=2 | 43=2 | 365=1 | 52=1 | 40=1
    VALUES[max_durasi_hari] (53): 0=87,707 | 30=1,342 | 1=1,007 | 29=800 | 2=431 | 3=400 | 4=344 | 7=312 | 28=230 | 27=221 | 6=207 | 5=201 | 8=141 | 31=130 | 13=87 | 9=84 | 25=82 | 11=76 | 26=74 | 20=69 | 15=68 | 24=67 | 12=64 | 22=63 | 10=62 | 14=55 | 19=53 | 16=48 | 18=48 | 21=37 | 17=37 | 23=34 | 34=26 | 32=22 | 33=20 | 35=18 | 37=16 | 36=8 | 61=8 | 60=4 | 91=4 | 334=4 | 43=4 | 365=4 | 40=2 | 92=2 | 181=2 | 44=2 | 38=2 | 63=2 | 64=2 | 47=2 | 52=2

### mv_penandaan_log  —  3,534,975 rows
  - id_penandaan                         bigint       null=        0 (  0.0%)  distinct=500820
  - trx_steps                            text         null=        7 (  0.0%)  distinct=18
  - status_code                          bigint       null=        7 (  0.0%)  distinct=19
  - status_label                         text         null=   40,020 (  1.1%)  distinct=11
  - fullname                             text         null=   13,928 (  0.4%)  distinct=1803
  - nama_balai                           text         null=   13,956 (  0.4%)  distinct=91
  - catatan                              text         null=1,599,440 ( 45.2%)  distinct=38758
  - tanggal_proses                       timestamp without time zone null=  298,413 (  8.4%)  distinct=802914
  - sync                                 timestamp without time zone null=        0 (  0.0%)  distinct=1
    VALUES[trx_steps] (18): draft=576,586 | pusat=536,586 | spv_1=516,563 | kepala_balai=489,702 | spv_1_pusat=359,921 | direktur=354,181 | selesai=350,804 | spv_2_pusat=251,602 | spv_2=56,843 | ditolak_spv_1=38,479 | ditolak_pusat=1,461 | ditolak_spv_1_pusat=1,416 | ditolak_kepala_balai=637 | ditolak_spv_2=78 | ditolak_spv_2_pusat=63 | ditolak_direktur=43 | <NULL>=7 | receive_tps=2 | spv1=1
    VALUES[status_code] (19): 0=576,135 | 4=536,084 | 1=518,409 | 3=490,508 | 5=359,974 | 7=354,065 | 999=350,735 | 6=251,411 | 2=56,841 | 991=36,316 | 994=1,461 | 995=1,415 | 14=758 | 993=636 | 992=78 | 996=63 | 997=43 | 12=35 | <NULL>=7 | 990=1
    VALUES[status_label] (11): Operator - Draft Sampling=576,135 | MT - Pembuatan SPK=536,084 | Supervisor - Verifikasi=518,409 | TPS - Penerimaan SPU=490,508 | Deputi MT - Pembuatan SPK=359,974 | Penguji - Entri Hasil Pengujian=354,065 | Sampel Rujukan Selesai=350,735 | Penyelia - Pembuatan SPP=251,411 | Supervisor 2 - Verifikasi=56,841 | <NULL>=40,020 | Operator - Perbaikan Sampel=758 | Kepala Balai=35

### mv_penandaan_timeline  —  500,820 rows
  - id_penandaan                         bigint       null=        0 (  0.0%)  distinct=500820
  - tgl_start                            date         null=       17 (  0.0%)  distinct=2330
  - tgl_end                              date         null=       17 (  0.0%)  distinct=2339
  - tanggal_kirim_kabalai                date         null=   21,484 (  4.3%)  distinct=2052
  - tanggal_kirim_direktur               date         null=  147,429 ( 29.4%)  distinct=969
  - tanggal_kirim_pusat                  date         null=   23,201 (  4.6%)  distinct=1907
  - status                               bigint       null=        0 (  0.0%)  distinct=19
  - mulai_kabalai                        integer      null=   21,484 (  4.3%)  distinct=359
  - kabalai_direktur                     integer      null=  147,510 ( 29.5%)  distinct=436
  - direktur_pusat                       integer      null=  147,536 ( 29.5%)  distinct=7
  - sync                                 timestamp without time zone null=        0 (  0.0%)  distinct=1
    VALUES[status] (19): 999=350,276 | 4=119,715 | 0=17,239 | 5=3,430 | 7=3,061 | 1=2,886 | 3=1,522 | 994=870 | 2=842 | 991=519 | 995=180 | 993=147 | 6=100 | 8=11 | 12=10 | 14=6 | 992=4 | 996=1 | 997=1
    VALUES[direktur_pusat] (7): 0=347,511 | <NULL>=147,536 | 1=5,764 | 19=4 | 5=2 | 142=1 | 177=1 | 209=1

### target_balai  —  532 rows
  - id                                   bigint       null=        0 (  0.0%)  distinct=532
  - nama_balai                           text         null=        0 (  0.0%)  distinct=76
  - komoditi                             text         null=        0 (  0.0%)  distinct=7
  - tahun                                bigint       null=        0 (  0.0%)  distinct=1
  - target_penandaan                     bigint       null=        0 (  0.0%)  distinct=253
  - target_pengawasan                    bigint       null=        0 (  0.0%)  distinct=53
  - target_pengujian                     bigint       null=        0 (  0.0%)  distinct=254
  - target_pengujian_pangan              bigint       null=       17 (  3.2%)  distinct=68
  - target_pengujian_pangan_fortifikasi  bigint       null=       17 (  3.2%)  distinct=19
  - target_sarana_distribusi             bigint       null=        0 (  0.0%)  distinct=177
  - target_sarana_produksi               bigint       null=        0 (  0.0%)  distinct=74
  - sync                                 timestamp without time zone null=        0 (  0.0%)  distinct=1
    VALUES[nama_balai] (76): BALAI BESAR POM DI PALEMBANG=7 | BALAI BESAR POM DI BANJARBARU=7 | BALAI BESAR POM DI GORONTALO=7 | BALAI POM DI TASIKMALAYA=7 | BALAI BESAR POM DI PEKANBARU=7 | BALAI BESAR POM DI SURABAYA=7 | Loka POM di Kabupaten Sijunjung=7 | Loka POM di Kabupaten Belitung=7 | BALAI BESAR POM DI PONTIANAK=7 | Loka POM di Kota Lubuklinggau=7 | BALAI BESAR POM DI KENDARI=7 | BALAI POM DI DUMAI =7 | BALAI POM DI AMBON=7 | BALAI BESAR POM DI PALU=7 | BALAI POM DI BALIKPAPAN=7 | BALAI BESAR POM DI BANDAR LAMPUNG=7 | BALAI BESAR POM DI SEMARANG=7 | Loka POM di Kabupaten Kepulauan Sangihe=7 | Loka POM di Kab. Sumba Timur=7 | BALAI BESAR POM DI SERANG=7 | BALAI BESAR POM DI MANADO=7 | Loka POM di Kab. Sambas=7 | BALAI BESAR POM DI BANDUNG=7 | Loka POM di Kab. Belu=7 | BALAI POM DI TARAKAN=7 | BALAI POM DI TANJUNGBALAI=7 | Loka POM di Kabupaten Buleleng=7 | Loka POM di Kabupaten Aceh Selatan=7 | BALAI POM DI PAYAKUMBUH=7 | BALAI POM DI JAMBI=7 | BALAI POM DI BENGKULU=7 | BALAI POM DI JEMBER=7 | BALAI BESAR POM DI BANDA ACEH=7 | BALAI POM DI BATAM=7 | BALAI POM DI TANGERANG=7 | BALAI BESAR POM DI MEDAN=7 | BALAI POM DI PALOPO=7 | BALAI POM DI TULANGBAWANG=7 | BALAI POM DI ENDE=7 | BALAI POM DI TABALONG=7 | Loka POM di Kabupaten Merauke=7 | Loka POM di Kabupaten Manggarai Barat=7 | BALAI POM DI SANGGAU=7 | BALAI POM DI MANOKWARI=7 | BALAI POM DI INDRAGIRI HULU=7 | BALAI POM DI BOGOR=7 | BALAI POM DI BAU-BAU=7 | BALAI BESAR POM DI PALANGKARAYA=7 | BALAI POM DI PANGKALPINANG=7 | BALAI POM DI SOFIFI=7 | BALAI BESAR POM DI YOGYAKARTA=7 | BALAI BESAR POM DI JAKARTA=7 | BALAI BESAR POM DI KUPANG=7 | Loka POM di Kabupaten Tanah Bumbu=7 | BALAI BESAR POM DI MATARAM=7 | BALAI BESAR POM DI SAMARINDA=7 | Loka POM di Kabupaten Banggai=7 | BALAI POM DI SURAKARTA=7 | BALAI POM DI TOBA=7 | BALAI BESAR POM DI DENPASAR=7 | Loka POM di Kabupaten Aceh Tengah=7 | BALAI BESAR POM DI PADANG=7 | Loka POM di Kabupaten Rejang Lebong=7 | BALAI POM DI KEDIRI=7 | Loka POM di Kabupaten Mimika=7 | BALAI BESAR POM DI MAKASSAR=7 | BALAI POM DI BANYUMAS=7 | BALAI BESAR POM DI JAYAPURA=7 | Loka POM di Kabupaten Kotawaringin Barat=7 | Loka POM di Kabupaten Bungo=7 | Loka POM di Kabupaten Pulau Morotai=7 | BALAI POM DI BIMA=7 | Loka POM di Kabupaten Tanimbar=7 | BALAI POM DI MAMUJU=7 | Loka POM di Kabupaten Sorong=7 | Loka POM di Kota Tanjung Pinang=7
    VALUES[komoditi] (7): Obat Kuasi=76 | Produk Pangan=76 | Obat Tradisional (OT)=76 | Obat=76 | Kosmetika=76 | Suplemen Kesehatan=76 | Rokok=76
    VALUES[tahun] (1): 2024=532
    VALUES[target_pengawasan] (53): 0=76 | 10=55 | 110=42 | 15=42 | 120=38 | 35=26 | 75=22 | 235=21 | 40=19 | 300=17 | 5=16 | 360=14 | 432=10 | 576=10 | 85=9 | 30=8 | 288=8 | 25=8 | 100=7 | 70=7 | 270=6 | 60=6 | 80=6 | 20=6 | 65=5 | 305=5 | 130=5 | 250=3 | 50=3 | 160=3 | 175=2 | 90=2 | 210=2 | 170=2 | 620=2 | 320=2 | 215=1 | 150=1 | 95=1 | 79=1 | 200=1 | 115=1 | 381=1 | 105=1 | 420=1 | 356=1 | 440=1 | 180=1 | 133=1 | 600=1 | 125=1 | 530=1 | 260=1
    VALUES[target_pengujian_pangan] (68): 0=439 | <NULL>=17 | 65=3 | 160=3 | 64=2 | 80=2 | 50=2 | 60=2 | 110=2 | 760=1 | 215=1 | 575=1 | 875=1 | 540=1 | 95=1 | 643=1 | 555=1 | 627=1 | 481=1 | 76=1 | 100=1 | 387=1 | 942=1 | 919=1 | 132=1 | 66=1 | 894=1 | 199=1 | 114=1 | 163=1 | 82=1 | 450=1 | 69=1 | 105=1 | 141=1 | 150=1 | 670=1 | 122=1 | 553=1 | 177=1 | 212=1 | 566=1 | 241=1 | 435=1 | 538=1 | 171=1 | 620=1 | 254=1 | 116=1 | 607=1 | 41=1 | 448=1 | 185=1 | 120=1 | 90=1 | 957=1 | 560=1 | 71=1 | 210=1 | 70=1 | 198=1 | 75=1 | 573=1 | 155=1 | 347=1 | 173=1 | 909=1 | 397=1 | 196=1
    VALUES[target_pengujian_pangan_fortifikasi] (19): 0=462 | <NULL>=17 | 15=10 | 75=9 | 20=6 | 60=4 | 125=4 | 70=4 | 80=3 | 50=2 | 85=2 | 31=1 | 30=1 | 65=1 | 105=1 | 110=1 | 10=1 | 100=1 | 39=1 | 40=1
    VALUES[target_sarana_produksi] (74): 0=327 | 1=38 | 2=12 | 3=11 | 4=10 | 5=7 | 12=6 | 25=6 | 6=5 | 13=5 | 21=5 | 7=4 | 11=4 | 10=4 | 33=3 | 38=3 | 40=3 | 18=3 | 31=3 | 8=3 | 28=2 | 60=2 | 23=2 | 62=2 | 9=2 | 15=2 | 30=2 | 14=2 | 65=2 | 44=2 | 36=2 | 24=2 | 17=2 | 48=2 | 22=2 | 50=2 | 16=1 | 71=1 | 26=1 | 72=1 | 70=1 | 75=1 | 96=1 | 207=1 | 144=1 | 109=1 | 19=1 | 20=1 | 34=1 | 32=1 | 261=1 | 66=1 | 175=1 | 51=1 | 153=1 | 27=1 | 190=1 | 235=1 | 43=1 | 42=1 | 309=1 | 69=1 | 54=1 | 139=1 | 55=1 | 52=1 | 85=1 | 73=1 | 164=1 | 87=1 | 124=1 | 56=1 | 35=1 | 162=1
