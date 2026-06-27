# Bài Tập Ôn Tập — Lab 3: Tiến Trình và Tiểu Trình

---

# Bài Tập 1 — Quan Hệ Cha-Con Giữa Các Tiến Trình

---

## Phần 1a — Vẽ cây quan hệ cha-con

Dựa vào bảng dữ liệu trong đề bài:

| UID | PID | PPID | COMMAND      |
|-----|-----|------|--------------|
| 88  | 86  | 1    | WindowServer |
| 501 | 281 | 86   | iTunes       |
| 501 | 282 | 86   | Terminal     |
| 0   | 287 | 282  | login        |
| 501 | 461 | 293  | firefox-bin  |
| 501 | 531 | 86   | Safari       |
| 501 | 726 | 86   | Mail         |
| 501 | 751 | 293  | Aquamacs     |
| 501 | 293 | 287  | -bash        |

Cách đọc: mỗi tiến trình có PPID là PID của tiến trình cha. Ví dụ tiến trình 282 (Terminal) có PPID = 86 → cha là WindowServer.

### Cây quan hệ parent-child

```
1 (init)
└── 86 WindowServer
    ├── 281 iTunes
    ├── 282 Terminal
    │   └── 287 login
    │       └── 293 -bash
    │           ├── 461 firefox-bin
    │           └── 751 Aquamacs
    ├── 531 Safari
    └── 726 Mail
```

---

## Phần 1b — Dùng lệnh ps tìm tiến trình cha

Lệnh `ps -f` liệt kê các tiến trình kèm thông tin đầy đủ, trong đó có cột **PPID** là PID của tiến trình cha.

### Xem tất cả tiến trình hiện tại

```bash
ps -f
```

### Tìm cha của một tiến trình cụ thể theo PID

```bash
ps -f -p <PID cần tìm>
```

### Ví dụ

```bash
ps -f -p 282
```

Kết quả:

```
UID   PID  PPID  CMD
501   282    86  Terminal
```

Nhìn vào cột **PPID = 86** → tiến trình cha của 282 là **WindowServer (86)**.

---

## Phần 1c — Cài đặt và dùng pstree

### Cài pstree

```bash
sudo apt-get install psmisc
```

Nhập mật khẩu, nhấn `Y` rồi Enter khi được hỏi.

### Xem toàn bộ cây tiến trình

```bash
pstree -p
```

### Tìm cha của một tiến trình theo PID

```bash
pstree -s -p <PID cần tìm>
```

### Ví dụ tìm cha của firefox-bin (PID 461)

```bash
pstree -s -p 461
```

Kết quả:

```
init(1)───WindowServer(86)───-bash(293)───firefox-bin(461)
```

Đọc từ trái sang phải — tiến trình cha trực tiếp của **461** là **-bash (293)**.

### Giải thích các option của pstree

| Option | Ý nghĩa |
|--------|---------|
| `-p` | Hiển thị kèm PID |
| `-s` | Hiển thị ngược lên tiến trình cha (show parents) |

---

## Tổng kết — Lệnh cần nhớ

| Lệnh | Tác dụng |
|------|---------|
| `ps -f` | Liệt kê tiến trình hiện tại kèm thông tin đầy đủ |
| `ps -f -p <PID>` | Xem thông tin tiến trình theo PID, nhìn cột PPID để biết cha |
| `pstree -p` | Xem toàn bộ cây tiến trình kèm PID |
| `pstree -s -p <PID>` | Tìm chuỗi cha của tiến trình theo PID |

---

## Bài tập 2 — Chương trình in ra gì?

### Mã nguồn

```c
#include <stdio.h>
int main() {
    pid_t pid;
    int num_coconuts = 17;
    pid = fork();
    if (pid == 0) {
        num_coconuts = 42;
        exit(0);
    } else {
        wait(NULL); /* wait until the child terminates */
    }
    printf("I see %d coconuts!\n", num_coconuts);
    exit(0);
}
```

### Kết quả in ra

```
I see 17 coconuts!
```

### Giải thích

Khi `fork()` được gọi, hệ điều hành tạo ra một bản sao của tiến trình cha. Mỗi tiến trình có **vùng nhớ riêng biệt**, do đó biến `num_coconuts` ở tiến trình con và tiến trình cha là **hai biến độc lập**.

- Tiến trình con (`pid == 0`): đổi `num_coconuts = 42` rồi gọi `exit(0)` — kết thúc ngay, không in gì.
- Tiến trình cha (`pid > 0`): chờ con kết thúc bằng `wait(NULL)`, sau đó in giá trị `num_coconuts` của chính nó — vẫn là **17** vì chưa bị thay đổi.

---

## Bài tập 3 — Thuộc tính của pthread

### Các hàm pthread_attr_* thường dùng

| Hàm | Chức năng |
|-----|-----------|
| `pthread_attr_init(&attr)` | Khởi tạo đối tượng thuộc tính |
| `pthread_attr_destroy(&attr)` | Giải phóng đối tượng thuộc tính |
| `pthread_attr_setdetachstate(&attr, state)` | Đặt trạng thái joinable/detached |
| `pthread_attr_setstacksize(&attr, size)` | Đặt kích thước stack |
| `pthread_attr_setschedpolicy(&attr, policy)` | Đặt chính sách lập lịch |
| `pthread_attr_setschedparam(&attr, &param)` | Đặt tham số lập lịch (độ ưu tiên) |

### Chương trình minh họa

```c
/*######################################
# University of Information Technology #
# IT007 Operating System               #
# <Your name>, <your Student ID>       #
# File: bai3_pthread_attr.c            #
######################################*/

#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

void *thread_func(void *arg) {
    int id = *(int *)arg;
    printf("Thread %d dang chay voi stack size tuy chinh\n", id);
    pthread_exit(NULL);
}

int main() {
    pthread_t tid[2];
    pthread_attr_t attr;
    int ids[2] = {1, 2};

    // Khoi tao thuoc tinh
    pthread_attr_init(&attr);

    // Dat trang thai joinable (co the goi pthread_join)
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_JOINABLE);

    // Dat kich thuoc stack la 1MB
    pthread_attr_setstacksize(&attr, 1024 * 1024);

    // Tao 2 tieu trinh voi thuoc tinh tuy chinh
    for (int i = 0; i < 2; i++) {
        int rc = pthread_create(&tid[i], &attr, thread_func, &ids[i]);
        if (rc != 0) {
            printf("ERROR: Khong the tao thread %d\n", i);
            exit(-1);
        }
    }

    // Giai phong doi tuong thuoc tinh sau khi tao xong
    pthread_attr_destroy(&attr);

    // Cho cac tieu trinh ket thuc
    for (int i = 0; i < 2; i++) {
        pthread_join(tid[i], NULL);
    }

    printf("Tat ca tieu trinh da ket thuc.\n");
    return 0;
}
```

### Biên dịch và chạy

```bash
gcc -pthread bai3_pthread_attr.c -o bai3_pthread_attr
./bai3_pthread_attr
```

### Kết quả mong đợi

```
Thread 1 dang chay voi stack size tuy chinh
Thread 2 dang chay voi stack size tuy chinh
Tat ca tieu trinh da ket thuc.
```

---

## Bài tập 4 — Mở vim và bắt Ctrl+C

### Yêu cầu

- In ra dòng chào với mã số sinh viên
- Mở file `abcd.txt` bằng vim
- Khi người dùng nhấn **Ctrl+C**: tắt vim và in thông báo tạm biệt

### Chương trình

```c
/*######################################
# University of Information Technology #
# IT007 Operating System               #
# <Your name>, <your Student ID>       #
# File: bai4_signal_vim.c              #
######################################*/

#include <stdio.h>
#include <stdlib.h>
#include <signal.h>

void on_sigint() {
    printf("\nYou are pressed CTRL+C! Goodbye!\n");
    system("pkill vim");   // tat vim editor
    exit(0);
}

int main() {
    // Buoc a: in dong chao
    printf("Welcome to IT007, I am <your_Student_ID>!\n");

    // Dang ky bat tin hieu SIGINT (Ctrl+C)
    signal(SIGINT, on_sigint);

    // Buoc b: mo abcd.txt bang vim
    system("vim abcd.txt");

    return 0;
}
```

### Biên dịch và chạy

```bash
gcc bai4_signal_vim.c -o bai4_signal_vim
./bai4_signal_vim
```

### Luồng hoạt động

```
Chạy chương trình
      ↓
In: "Welcome to IT007, I am <your_Student_ID>!"
      ↓
Mở vim abcd.txt
      ↓
Người dùng nhấn Ctrl+C
      ↓
Bắt SIGINT → gọi on_sigint()
      ↓
In: "You are pressed CTRL+C! Goodbye!"
      ↓
Tắt vim → thoát chương trình
```

### Lưu ý

- Thay `<your_Student_ID>` bằng mã số sinh viên thực tế của bạn.
- Lệnh `pkill vim` sẽ tắt tất cả tiến trình vim đang chạy trên hệ thống. Nếu chỉ muốn tắt vim được mở bởi chương trình này, cần lưu PID của tiến trình vim và dùng `kill <PID>`.

---
