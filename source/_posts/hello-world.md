## Forensics - Forensics for newbie

Statement: [Here](https://drive.google.com/drive/folders/1jKuvDrzLHvQ_wgcKewByXHO6Aas1OkTy)

Description:
```bash
Challenge Name: Forensics for newbie
Category: Forensics

Description: Yesterday, my supervisor has sent to my colleague a secret file. After that, I saw his screen was running something with a black background. I think he was did something with that secret file. Can you help me to view that file?
```

Bài toán cấp file `chall.dmp`
```bash
chall.dmp: MS Windows 64bit crash dump, version 15.19045, 4 processors, DumpType (0x1), 1048333 pages
```

File đó là memory dump, sử dụng tool Volatility để phân tích.
Từ phần Description, ta biết đồng nghiệp đã được cấp một secret file, sau đó thao tác trên `black background`, có vẻ như là terminal. Để xem đồng nghiệp đã làm gì, sử dụng Plugin `windows.console` để capture cuộc hội thoại giữa shell và đồng nghiệp.

```bash
vol -f chall.dmp windows.console
```

Một phần của output:
```bash
C:\Users\ctf-player>cd Desktop

C:\Users\ctf-player\Desktop>7z a -pKetSoSadisveryfunny -mhe secret.7z secret.png

7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03

Scanning the drive:
1 file, 19596 bytes (20 KiB)

Creating archive: secret.7z

Add new data to archive: 1 file, 19596 bytes (20 KiB)


Files read from disk: 1
Archive size: 17074 bytes (17 KiB)
Everything is Ok

C:\Users\ctf-player\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is E053-B4C5

 Directory of C:\Users\ctf-player\Desktop

11/22/2025  12:34 AM    <DIR>          .
11/22/2025  12:34 AM    <DIR>          ..
11/22/2025  12:26 AM           530,640 DumpIt.exe
11/22/2025  12:34 AM            17,074 secret.7z
11/21/2025  10:38 PM            19,596 secret.png
               3 File(s)        567,310 bytes
               2 Dir(s)  39,006,892,032 bytes free

C:\Users\ctf-player\Desktop>del secret.png
```
Cần biết trong `secret.png` có gì (có thể là flag).
Dựa vào `C:\Users\ctf-player\Desktop>7z a -pKetSoSadisveryfunny -mhe secret.7z secret.png`, `secret.png` đã bị nén cùng với mật khẩu để giải mã là `KetSoSadisveryfunny`.

Đến đây cần truy cập `secret.7z` như thế nào?
Sử dụng command với plugin `windows.filescan`, trả về virtual address của các file trên RAM.

```bash
$ vol -f chall.dmp windows.filescan | grep secret.7z

0xdd84dba88cd0.0\Users\ctf-player\Desktop\secret.7z
```

Khi đã biết được địa chỉ ảo của `secret.7z`, tiến hành truy xuất nó bằng plugin `windows.dumpfile`: 
```bash
$ vol -f chall.dmp windows.dumpfile --virtaddr 0xdd84dba88cd0
Volatility 3 Framework 2.28.2
Progress:  100.00               PDB scanning finished
Cache   FileObject      FileName        Result

DataSectionObject       0xdd84dba88cd0  secret.7z       file.0xdd84dba88cd0.0xdd84de7e1e30.DataSectionObject.secret.7z.dat
```
Tiến hành unzip `file.0xdd84dba88cd0.0xdd84de7e1e30.DataSectionObject.secret.7z.dat` (xóa `.dat `) với password là `KetSoSadisveryfunny`

<img width="384" height="462" alt="image" src="https://github.com/user-attachments/assets/8be3cf1f-73a4-49cc-afd5-ecd7abd42c37" />

Flag: 

<img width="970" height="536" alt="secret" src="https://github.com/user-attachments/assets/a6e26d3a-93e6-4864-852f-3f7845e07557" />
