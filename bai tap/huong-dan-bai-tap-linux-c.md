# Hướng Dẫn Thực Hành: Lập Trình C và Quản Lý Tiến Trình trên Linux

> **Mục tiêu:** Làm quen với môi trường lập trình C trên Linux, sử dụng các công cụ gcc, make, gdb và thực hành quản lý tiến trình/tiểu trình.

---

## Bài 01 — Cài Đặt Công Cụ Cần Thiết

### Mục tiêu
Chuẩn bị đầy đủ môi trường phát triển trước khi bắt đầu lập trình.

### Các bước thực hiện

```bash
apt-get install vim build-essential gdb
```

| Công cụ | Vai trò |
|---------|---------|
| `vim` | Soạn thảo mã nguồn trực tiếp trên terminal |
| `gcc` | Trình biên dịch C (GNU Compiler Collection) |
| `make` | Tự động hóa quá trình biên dịch qua Makefile |
| `gdb` | Gỡ lỗi chương trình (GNU Debugger) |

### Kiểm tra cài đặt thành công

```bash
gcc --version
make --version
gdb --version
```

---

## Bài 02 — Viết Chương Trình Hello World

### Mục tiêu
Tạo file C đầu tiên, làm quen với vim và cú pháp cơ bản.

### Các bước thực hiện

**Bước 1:** Mở vim để tạo file mới

```bash
vim hello.c
```

**Bước 2:** Nhấn `i` để vào chế độ INSERT, nhập mã nguồn sau:

```c
#include <stdio.h>

int main() {
    printf("Hello, I am Nguyen Van A\n");
    return 0;
}
```

> Thay `Nguyen Van A` bằng tên thật của bạn.

**Bước 3:** Lưu và thoát vim

- Nhấn `Esc` để thoát chế độ INSERT
- Gõ `:wq` rồi Enter để lưu và thoát

### Các lệnh vim cơ bản cần nhớ

| Lệnh | Chức năng |
|------|-----------|
| `i` | Vào chế độ chèn (INSERT) |
| `Esc` | Trở về chế độ thường (NORMAL) |
| `:w` | Lưu file |
| `:q` | Thoát vim |
| `:wq` | Lưu và thoát |
| `:q!` | Thoát không lưu |

---

## Bài 03 — Biên Dịch và Chạy Chương Trình

### Mục tiêu
Sử dụng gcc để biên dịch mã nguồn thành file thực thi.

### Các bước thực hiện

**Bước 1:** Biên dịch bằng gcc

```bash
gcc hello.c -o hello
```

- `hello.c` — file mã nguồn đầu vào
- `-o hello` — chỉ định tên file thực thi đầu ra

**Bước 2:** Chạy chương trình

```bash
./hello
```

**Kết quả mong đợi:**

```
Hello, I am Nguyen Van A
```

### Các flag gcc thường dùng

| Flag | Ý nghĩa |
|------|---------|
| `-o <tên>` | Đặt tên file thực thi |
| `-Wall` | Bật tất cả cảnh báo |
| `-g` | Thêm thông tin debug (dùng với gdb) |
| `-pthread` | Liên kết thư viện POSIX threads |

---

## Bài 04 — Tổ Chức Project với Makefile

### Mục tiêu
Dùng `make` để tự động hóa việc biên dịch, chạy và dọn dẹp project.

### Các bước thực hiện

**Bước 1:** Tạo Makefile

```bash
vim Makefile
```

**Bước 2:** Nhập nội dung sau (chú ý dùng **TAB** thụt đầu dòng, không dùng space):

```makefile
CC = gcc
CFLAGS = -Wall

all: hello run

hello: hello.c
	$(CC) $(CFLAGS) hello.c -o hello

run: hello
	./hello

clean:
	rm -f hello
```

**Bước 3:** Chạy make

```bash
make all      # Biên dịch và chạy chương trình
make clean    # Xóa file thực thi
make hello    # Chỉ biên dịch
make run      # Chỉ chạy (nếu đã biên dịch)
```

> **Lưu ý quan trọng:** Dòng lệnh trong Makefile **bắt buộc** phải dùng ký tự TAB, không phải space. Nếu dùng space, `make` sẽ báo lỗi.

---

## Bài 05 — Thực Hành Gỡ Lỗi với GDB

### Mục tiêu
Sử dụng gdb để tìm và sửa lỗi trong chương trình tính giai thừa (factorial).

### Chuẩn bị: Tạo file factorial.c có lỗi

```c
#include <stdio.h>

int factorial(int n) {
    int result;        // Lỗi: chưa khởi tạo biến result!
    int j = 1;
    for (j = 1; j <= n; j++) {
        result *= j;   // result chứa giá trị rác → kết quả sai
    }
    return result;
}

int main() {
    printf("5! = %d\n", factorial(5));
    return 0;
}
```

### Các bước gỡ lỗi

**Bước 1:** Biên dịch với flag `-g` để thêm thông tin debug

```bash
gcc -g factorial.c -o factorial
```

**Bước 2:** Khởi động gdb

```bash
gdb ./factorial
```

**Bước 3:** Thực hiện các lệnh gdb

```
(gdb) break factorial      # Đặt breakpoint tại hàm factorial
(gdb) run                  # Chạy chương trình
(gdb) print result         # Xem giá trị biến result (sẽ thấy giá trị rác)
(gdb) print j              # Xem biến j
(gdb) next                 # Chạy từng dòng
(gdb) continue             # Tiếp tục chạy đến breakpoint tiếp theo
(gdb) quit                 # Thoát gdb
```

### Sửa lỗi

Thay dòng `int result;` thành `int result = 1;` trong `factorial.c`, rồi biên dịch và chạy lại.

### Tổng hợp lệnh gdb hay dùng

| Lệnh | Ý nghĩa |
|------|---------|
| `break <tên_hàm>` | Đặt breakpoint tại hàm |
| `break <số_dòng>` | Đặt breakpoint tại dòng |
| `run` | Chạy chương trình |
| `next` | Chạy dòng tiếp theo (không vào hàm con) |
| `step` | Chạy dòng tiếp theo (vào trong hàm con) |
| `print <biến>` | In giá trị biến |
| `info locals` | Liệt kê tất cả biến cục bộ |
| `continue` | Tiếp tục chạy đến breakpoint tiếp |
| `quit` | Thoát gdb |

---

## Bài 06 — Quản Lý Tiến Trình trong Linux

### Mục tiêu
Nắm vững các lệnh theo dõi và điều khiển tiến trình trên Linux.

### 6.1 Xem tiến trình đang chạy

```bash
top               # Xem tiến trình theo thời gian thực (nhấn q để thoát)
ps -f             # Liệt kê tiến trình của phiên hiện tại (dạng đầy đủ)
ps aux            # Liệt kê tất cả tiến trình trên hệ thống
```

Các cột quan trọng trong `ps -f`:

| Cột | Ý nghĩa |
|-----|---------|
| `PID` | Process ID (định danh tiến trình) |
| `PPID` | Parent PID (tiến trình cha) |
| `CMD` | Lệnh đã khởi chạy tiến trình |
| `%CPU` | Phần trăm CPU đang dùng |

### 6.2 Chạy tiến trình ở background và foreground

```bash
./hello &         # Chạy tiến trình ở background (thêm dấu &)
jobs              # Liệt kê các job đang chạy ở background
fg %1             # Đưa job số 1 lên foreground
bg %1             # Tiếp tục chạy job số 1 ở background
```

### 6.3 Tạm dừng và chấm dứt tiến trình

```bash
Ctrl + Z          # Tạm dừng tiến trình đang chạy (suspend)
Ctrl + C          # Kết thúc tiến trình đang chạy (interrupt)
kill <PID>        # Gửi tín hiệu TERM để dừng tiến trình
kill -9 <PID>     # Buộc dừng tiến trình ngay lập tức
```

---

## Bài 07 — Thực Hành với Tiểu Trình (pthread)

### Mục tiêu
Viết chương trình tạo nhiều tiểu trình chạy song song bằng thư viện POSIX threads.

### Chuẩn bị: Tạo file mythread.c

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

#define NUM_THREADS 3

void *thread_function(void *arg) {
    int thread_id = *(int *)arg;
    printf("Tiểu trình %d đang chạy...\n", thread_id);
    sleep(1);
    printf("Tiểu trình %d kết thúc.\n", thread_id);
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];
    int thread_ids[NUM_THREADS];

    // Tạo các tiểu trình
    for (int i = 0; i < NUM_THREADS; i++) {
        thread_ids[i] = i + 1;
        pthread_create(&threads[i], NULL, thread_function, &thread_ids[i]);
        printf("Đã tạo tiểu trình %d\n", i + 1);
    }

    // Chờ tất cả tiểu trình kết thúc
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    printf("Tất cả tiểu trình đã kết thúc.\n");
    return 0;
}
```

### Các bước thực hiện

**Bước 1:** Biên dịch với flag `-pthread`

```bash
gcc -pthread mythread.c -o mythread
```

**Bước 2:** Chạy chương trình

```bash
./mythread
```

**Bước 3:** Quan sát kết quả — các tiểu trình chạy **xen kẽ không theo thứ tự cố định**, thể hiện tính song song.

### Các hàm pthread cơ bản

| Hàm | Chức năng |
|-----|-----------|
| `pthread_create(thread, attr, func, arg)` | Tạo tiểu trình mới |
| `pthread_join(thread, retval)` | Chờ tiểu trình kết thúc |
| `pthread_exit(retval)` | Kết thúc tiểu trình hiện tại |
| `pthread_self()` | Lấy ID tiểu trình hiện tại |

---

## Tổng Kết

| Bài | Kỹ năng đạt được |
|-----|-----------------|
| 01 | Cài đặt môi trường lập trình C trên Linux |
| 02 | Soạn thảo mã nguồn với vim |
| 03 | Biên dịch và chạy chương trình C với gcc |
| 04 | Tổ chức project và tự động hóa với Makefile |
| 05 | Gỡ lỗi chương trình bằng gdb |
| 06 | Quản lý tiến trình với top, ps, jobs, fg, bg |
| 07 | Lập trình đa tiểu trình với pthread |

---

*Tài liệu hướng dẫn thực hành — Lập trình hệ thống trên Linux*
