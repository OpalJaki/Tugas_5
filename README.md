Nama: Muhammad Naufal Zaki  
Kelas: SKU2B

## Kode:
```nasm
section .data
    prompt_item db "Enter item (or type 'done' to finish): ", 0
    prompt_item_len equ $ - prompt_item

    item_label db " - ", 0
    item_label_len equ $ - item_label

    list_title db 10, "Your shopping list:", 10, 0
    list_title_len equ $ - list_title

    newline db 10
    done_str db "done", 0

section .bss
    input resb 32
    items resb 32 * 5
    item_count resb 1

section .text
    global _start

_start:
    xor byte [item_count], 0

input_loop:
    mov eax, 4
    mov ebx, 1
    mov ecx, prompt_item
    mov edx, prompt_item_len
    int 0x80

    mov eax, 3
    mov ebx, 0
    mov ecx, input
    mov edx, 32
    int 0x80

    mov ecx, input
replace_newline:
    cmp byte [ecx], 10
    je set_zero
    cmp byte [ecx], 0
    je end_replace
    inc ecx
    jmp replace_newline

set_zero:
    mov byte [ecx], 0
end_replace:

    mov esi, input
    mov edi, done_str
    mov ecx, 4
    repe cmpsb
    je show_list

    movzx esi, byte [item_count]
    cmp esi, 5
    jge show_list

    mov edi, items
    mov eax, 32
    mul esi
    add edi, eax

    mov esi, input
    mov ecx, 32
    rep movsb

    inc byte [item_count]
    jmp input_loop

show_list:
    mov eax, 4
    mov ebx, 1
    mov ecx, list_title
    mov edx, list_title_len
    int 0x80

    xor ecx, ecx

print_loop:
    mov cl, [item_count]
    test cl, cl
    jz exit

    dec byte [item_count]

    movzx esi, byte [item_count]
    mov edi, items
    mov eax, 32
    mul esi
    add edi, eax

    mov eax, 4
    mov ebx, 1
    mov ecx, item_label
    mov edx, item_label_len
    int 0x80

    mov eax, 4
    mov ebx, 1
    mov ecx, edi
    mov edx, 32
print_item_loop:
    cmp byte [ecx + edx - 1], 0
    jne end_trim
    dec edx
    jmp print_item_loop
end_trim:
    int 0x80

    mov eax, 4
    mov ebx, 1
    mov ecx, newline
    mov edx, 1
    int 0x80

    jmp print_loop

exit:
    mov eax, 1
    xor ebx, ebx
    int 0x80
```
## Output:

![Screenshot 2025-04-14 145439](https://github.com/user-attachments/assets/faa96e44-4425-44df-9386-4da549e9463a)

## Penjelasan:

Program ini ditulis dalam bahasa Assembly (untuk arsitektur x86 Linux 32-bit). Fungsi utama dari program ini adalah membuat daftar belanja interaktif, di mana pengguna dapat memasukkan item satu per satu. Program akan terus meminta input hingga pengguna mengetikkan "done", dan setelah itu, daftar belanja akan ditampilkan.

```nasm
section .data
    prompt_item db "Enter item (or type 'done' to finish): ", 0
    prompt_item_len equ $ - prompt_item

    item_label db " - ", 0
    item_label_len equ $ - item_label

    list_title db 10, "Your shopping list:", 10, 0
    list_title_len equ $ - list_title

    newline db 10
    done_str db "done", 0
```

Bagian ini berisi data statis (konstanta), yang digunakan untuk menampilkan pesan atau string yang tetap selama program berjalan.
prompt_item: Pesan yang ditampilkan kepada pengguna untuk meminta input item. Variabel-variabel dalam kode ini memiliki peran khusus untuk mengatur tampilan dan alur program. item_label digunakan untuk menampilkan tanda " - " di depan setiap item yang dimasukkan, memberikan format yang jelas pada daftar belanja. list_title adalah judul yang akan ditampilkan di atas daftar belanja setelah semua item dimasukkan, memberikan konteks kepada pengguna. newline berisi karakter newline (\n) yang digunakan untuk memisahkan setiap item yang ditampilkan di layar, memastikan tampilan yang rapi dan terstruktur. Terakhir, done_str adalah string yang digunakan untuk memeriksa apakah input yang dimasukkan oleh pengguna adalah "done", yang menandakan bahwa pengguna telah selesai memasukkan item dan daftar belanja dapat ditampilkan.

```nasm
section .bss
    input resb 32
    items resb 32 * 5
    item_count resb 1

section .text
    global _start

_start:
    xor byte [item_count], 0

input_loop:
    mov eax, 4
    mov ebx, 1
    mov ecx, prompt_item
    mov edx, prompt_item_len
    int 0x80

    mov eax, 3
    mov ebx, 0
    mov ecx, input
    mov edx, 32
    int 0x80

    mov ecx, input
```
Di bagian .bss, variabel-variabel yang belum diinisialisasi dideklarasikan dan ruang memori untuk mereka dialokasikan. Bagian kode ini mengalokasikan beberapa ruang penyimpanan untuk menangani input dan daftar belanja. input resb 32 mengalokasikan 32 byte untuk buffer input yang akan digunakan untuk menyimpan data yang dimasukkan oleh pengguna. Buffer ini dapat menampung string dengan panjang maksimum 31 karakter, termasuk karakter terminator null (\0). items resb 32 * 5 mengalokasikan total 160 byte (32 byte untuk setiap item × 5 item maksimal) untuk menyimpan daftar belanja yang dimasukkan oleh pengguna. Setiap item dalam daftar ini dapat memiliki panjang hingga 32 byte. Sedangkan item_count resb 1 mendeklarasikan variabel item_count, yang digunakan untuk menyimpan jumlah item yang telah dimasukkan oleh pengguna. Variabel ini hanya memerlukan 1 byte untuk menghitung jumlah item yang telah dimasukkan.

Di bagian .text berisi kode yang akan dieksekusi oleh CPU, dimulai dengan titik masuk _start yang merupakan awal dari eksekusi prograM dan di bagian _start, program dimulai dengan menginisialisasi item_count ke 0 menggunakan instruksi xor byte [item_count], 0. Hal ini memastikan bahwa tidak ada item yang dimasukkan pada awalnya.

Setelah inisialisasi, program memasuki input loop, yang meminta pengguna untuk memasukkan item. Proses ini dilakukan dengan beberapa langkah berikut:
Program menampilkan prompt untuk meminta input pengguna menggunakan sistem call write. Register eax diatur ke 4 (nomor sistem call untuk write), ebx diatur ke 1 (stdout), dan ecx diatur ke alamat string prompt_item, yang berisi pesan "Enter item (or type 'done' to finish):".

Program membaca input dari pengguna menggunakan sistem call read dengan register eax diatur ke 3 (nomor sistem call untuk read). Program membaca hingga 32 byte dari input standar (stdin) dan menyimpannya di buffer input.

Setelah input dibaca, program memeriksa apakah pengguna mengetikkan kata "done". Jika ya, program melanjutkan untuk menampilkan daftar belanja yang telah dimasukkan. Jika tidak, input akan disalin ke dalam array items dan item_count akan bertambah. Program akan terus meminta input hingga pengguna mengetikkan "done" atau mencapai jumlah maksimum item yang dapat dimasukkan.

```nasm
replace_newline:
    cmp byte [ecx], 10
    je set_zero
    cmp byte [ecx], 0
    je end_replace
    inc ecx
    jmp replace_newline

set_zero:
    mov byte [ecx], 0
end_replace:

    mov esi, input
    mov edi, done_str
    mov ecx, 4
    repe cmpsb
    je show_list

    movzx esi, byte [item_count]
    cmp esi, 5
    jge show_list

    mov edi, items
    mov eax, 32
    mul esi
    add edi, eax

    mov esi, input
    mov ecx, 32
    rep movsb

    inc byte [item_count]
    jmp input_loop
```
Bagian kode ini berfungsi untuk menghapus karakter newline (\n) dari input yang dimasukkan pengguna dan memeriksa apakah input yang diberikan adalah kata "done". Proses pertama dimulai dengan perbandingan karakter dalam buffer input. Program memeriksa setiap karakter dalam buffer input yang disimpan di alamat yang ditunjuk oleh register ecx. Jika karakter yang ditemukan adalah newline (karakter ASCII 10), program melompat ke label set_zero untuk mengganti karakter newline tersebut dengan karakter null terminator (\0), yang menandakan akhir dari string. Jika karakter tersebut adalah null terminator (\0), maka program melompat ke label end_replace, menandakan bahwa akhir dari string telah tercapai.

Setelah menghapus newline, program kemudian memeriksa apakah input yang diberikan adalah kata "done". Hal ini dilakukan dengan membandingkan input yang dimasukkan dengan string done_str (yang berisi "done"). Jika input tersebut cocok dengan "done", program melompat ke label show_list, yang bertugas untuk menampilkan daftar item belanja yang sudah dimasukkan.

Jika input bukan "done", program akan melanjutkan untuk menyimpan item tersebut. Program pertama-tama memeriksa apakah jumlah item yang sudah dimasukkan melebihi batas maksimal (5 item). Jika belum, program menyimpan input ke dalam array items dengan menggunakan perhitungan offset, kemudian menambah item_count untuk mencatat jumlah item yang telah dimasukkan. Program kemudian kembali ke loop untuk meminta input berikutnya.

```nasm
show_list:
    mov eax, 4
    mov ebx, 1
    mov ecx, list_title
    mov edx, list_title_len
    int 0x80

    xor ecx, ecx

print_loop:
    mov cl, [item_count]
    test cl, cl
    jz exit

    dec byte [item_count]

    movzx esi, byte [item_count]
    mov edi, items
    mov eax, 32
    mul esi
    add edi, eax

    mov eax, 4
    mov ebx, 1
    mov ecx, item_label
    mov edx, item_label_len
    int 0x80

    mov eax, 4
    mov ebx, 1
    mov ecx, edi
    mov edx, 32
print_item_loop:
    cmp byte [ecx + edx - 1], 0
    jne end_trim
    dec edx
    jmp print_item_loop
end_trim:
    int 0x80

    mov eax, 4
    mov ebx, 1
    mov ecx, newline
    mov edx, 1
    int 0x80

    jmp print_loop

exit:
    mov eax, 1
    xor ebx, ebx
    int 0x80
```
Bagian kode ini bertanggung jawab untuk menampilkan daftar belanja yang telah dimasukkan oleh pengguna setelah mereka mengetikkan "done". Pertama, program menampilkan judul "Your shopping list:" dengan menggunakan sistem call write. Kemudian, program menginisialisasi register ecx ke 0 untuk menghitung jumlah item yang telah dimasukkan berdasarkan nilai item_count.

Selanjutnya, program memasuki sebuah loop yang akan menampilkan setiap item dalam daftar. Program memeriksa nilai item_count, dan jika tidak ada item yang dimasukkan, program akan melompat ke label exit untuk keluar. Jika ada item, program mengurangi item_count dan kemudian menampilkan label item (" - ") menggunakan sistem call write. Setelah itu, program mencetak nama item dengan menghitung alamat item yang disimpan dalam array items dan menggunakan sistem call write lagi untuk mencetaknya.

Program juga memeriksa apakah karakter terakhir dalam item adalah null terminator (\0) dan menghilangkannya dengan mengurangi panjang item sebelum mencetaknya. Setelah menampilkan nama item, program mencetak karakter newline (\n) untuk memberi jarak antar item. Loop ini berlanjut hingga semua item dicetak.

Setelah semua item selesai ditampilkan, program keluar dengan memanggil sistem call exit, yang mengakhiri eksekusi program.



