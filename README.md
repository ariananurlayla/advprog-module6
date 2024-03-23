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

## Milestone 2: Returning HTML
### Commit 2 Reflection Notes
![image](https://github.com/ariananurlayla/advprog-module6/assets/117559846/81cef064-8179-452e-b6ee-e4d21ce126ff)
Dalam method handle_connection, status_line diinisialisasi sebagai string yang mengindikasikan bahwa respons berhasil dilakukan. Kemudian, konten file hello.html dibaca dan diubah menjadi string menggunakan fs::read_to_string("hello.html"). Panjang dari string contents dihitung dan disimpan dalam variabel length dengan menggunakan fungsi len(). Setelah itu, semua variabel tersebut diformat menjadi sebuah string yang disimpan dalam variabel response.
- `let status_line = "HTTP/1.1 200 OK` Menyiapkan status line untuk response HTTP yang akan dikirim ke klien. Kode status 200 (OK) ini menyatakan bahwa respons sukses.
- `let contents = std::fs::read_to_string("hello.html").unwrap();` Membaca  file hello.html ke dalam string contents.
- `let length = contents.len();` Menghitung panjang (jumlah byte) dari isi file hello.html
- `let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");` Membuat response HTTP yang akan dikirim ke klien. Response terdiri dari status line, header Content-Length yang menyatakan panjang konten, dan isi konten dari file hello.html
- `stream.write_all(response.as_bytes()).unwrap();` Mengirim response ke klien dalam bentuk byte array setelah diubah dari string menggunakan `as_bytes()`. Jika terjadi kesalahan dalam proses pengiriman, program akan keluar dengan menggunakan unwrap()

## Milestone 3: Validating request and selectively responding
### Commit 3 Reflection Notes
![image](https://github.com/ariananurlayla/advprog-module6/assets/117559846/6cdce661-e8c0-49d4-9ed7-68350bfc168b)

Dilakukan pengecekan untuk validasi baris pertama dari HTTP request untuk membedakan respons.
Baris kode berikut bertujuan untuk mendapatkan baris permintaan (request line) dari request HTTP yang masuk agar nantinya dapat diidentifikasi alamat rutenya:
`let request_line = buf_reader.lines().next().unwrap().unwrap();`
Apabila routenya ditemukan, maka akan akan diarahkan pada halaman hello.html. Selain itu, apabila route requestnya tidak ditemukan maka akan dihandle oleh 404.html.
Ditemukan perulangan kode branching if else sehingga perlu dilakukan refactoring dengan mengeluarkan duplikasi dari block. Kemudian, bagian yang membedakannya yang akan masuk ke dalam branching if else block.

## Milestone 4: Simulation slow response
### Commit 4 Reflection Notes
`let (status_line, filename) = match &request_line[..] {`

`match` digunakan untuk membandingkan nilai dari request_line dengan pola tertentu.
Penggunaan `&request_line[..]` mengubah request_line menjadi slice dari string yang membantu membandingkan bagian dari string dengan pola yang diinginkan.

`"GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),` Jika request_line adalah "GET / HTTP/1.1", maka status_line akan diatur sebagai "HTTP/1.1 200 OK" yang menandakan bahwa respons berhasil, dan filename diatur sebagai "hello.html" yang merupakan nama file yang akan dikirimkan sebagai respons.

``` 
  GET /sleep HTTP/1.1" => {
      thread::sleep(Duration::from_secs(10));
      ("HTTP/1.1 200 OK", "hello.html")
  }
```

Jika request_line adalah "GET /sleep HTTP/1.1", maka akan ada delay selama 10 detik menggunakan `thread::sleep(Duration::from_secs(10))`. Kemudian, status_line diatur sebagai "HTTP/1.1 200 OK" dan filename diatur sebagai "hello.html". Ini menunjukkan bahwa server akan membalas setelah delay selama 10 detik.

`_ => ("HTTP/1.1 404 NOT FOUND", "404.html"),` Jika request_line tidak cocok dengan pola yang telah ditentukan sebelumnya, maka status_line akan diatur sebagai "HTTP/1.1 404 NOT FOUND" yang menunjukkan bahwa halaman tidak ditemukan, dan filename diatur sebagai "404.html" yang merupakan halaman kesalahan standar yang akan dikirimkan sebagai respons.