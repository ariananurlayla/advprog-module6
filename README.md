## Milestone 1: Single threaded web server
### Commit 1 Reflection Notes

TCP (Transmission Control Protocol) adalah salah satu protokol utama yang digunakan dalam model komunikasi jaringan yang dikenal sebagai TCP/IP (Transmission Control Protocol/Internet Protocol).
TCP bertanggung jawab untuk mengatur pengiriman data antara dua perangkat dalam jaringan komputer, biasanya antara server dan klien.

`let buf_reader = BufReader::new(&mut stream);` Method handle_connection bertugas menangani koneksi TCP dengan membaca dan memproses request HTTP yang diterima dari klien.
Membuat objek BufReader yang melakukan wrapping pada reference mutable TcpStream.
BufReader membantu dalam membaca data dari sebuah Reader, seperti TcpStream.
Fungsi utama dari BufReader adalah untuk melakukan buffering (penyimpanan sementara data dalam memori) saat membaca data.
BufReader digunakan karena memiliki keunggulan dalam efisiensi jika dibandingkan read instance lain.

Membaca setiap baris oleh instance BufReader dan menyimpannya dalam Vec<> bernama http_request:
```
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();
```
- ```.lines()``` mengembalikan iterator yang memberikan setiap baris yang dibaca dari buf_reader
- ```.map(|result| result.unwrap())``` memetakan iterator sebelumnya menjadi iterator baru yang berisi hasil dari ```result.unwrap()``` berupa String (memetakan ekstraksi String value dari result yang didapat dari hasil pengambilan baris di atasnya).
- ```.take_while(|line| !line.is_empty())``` membaca baris-baris dari BufReader sampai menemui baris kosong. Baris-baris request HTTP biasanya diakhiri oleh sebuah baris kosong yang menandakan akhir dari request sehingga  hal ini membantu pembacaan request HTTP.
- ```.collect();``` mengumpulkan semua HTTP requestnya dan dibuat menjadi ```Vec<_>``` (mengumpulkan iterator menjadi vector).

`println!("Request: {:#?}", http_request);` Mencetak nilai dari request HTTP (dengan format debug) ke konsol.