# Bối cảnh và lỗi bảo mật

Phiên bản production `v1.0.0` trên nhánh `main` trả về toàn bộ bản ghi người
dùng, bao gồm cả `password` và `access_token`. Trong khi đó, nhánh `develop`
đang chứa tính năng `profile summary` chưa sẵn sàng để triển khai.

Bản vá `v1.0.1` dùng danh sách cho phép (allow-list), chỉ trả về ba trường công
khai là `id`, `name` và `email`. Cách này cũng ngăn các trường bí mật được thêm
trong tương lai vô tình xuất hiện trong response.

# Quy trình Gitflow đã thực hiện

1. Tạo `main`, commit phiên bản ổn định và gắn tag `v1.0.0`.
2. Tạo `develop` từ `main`, sau đó commit tính năng đang phát triển.
3. Quay lại `main` và tạo `hotfix/v1.0.1` trực tiếp từ `main`.
4. Sửa lỗi lộ dữ liệu, bổ sung kiểm thử hồi quy và commit trên nhánh hotfix.
5. Merge hotfix vào `main` với `--no-ff`, sau đó tạo annotated tag `v1.0.1`.
6. Merge cùng hotfix vào `develop` với `--no-ff` để bản vá có mặt trong phiên
   bản tương lai.
7. Xóa nhánh hotfix cục bộ sau khi cả hai nhánh dài hạn đã nhận bản vá.

Các lệnh chính:

```bash
git switch main
git switch -c hotfix/v1.0.1
# sửa lỗi, chạy test và commit

git switch main
git merge --no-ff hotfix/v1.0.1 -m "merge: release hotfix v1.0.1 to main"
git tag -a v1.0.1 -m "Release Hotfix 1.0.1"

git switch develop
git merge --no-ff hotfix/v1.0.1 -m "merge: sync hotfix v1.0.1 into develop"
git branch -d hotfix/v1.0.1
```

# Sơ đồ phân nhánh

```text
                         D: tính năng mới ─────── M2 (develop)
                        /                       /
A: stable v1.0.0 ──────┼── H: security fix ───┤
                        \                     \
                         └──────────────────── M1 (main, tag v1.0.1)

H  : commit bản vá trên hotfix/v1.0.1
M1 : merge hotfix vào main để phát hành khẩn cấp
M2 : merge ngược hotfix vào develop để tránh tái xuất hiện lỗi
```

Hai merge commit được giữ lại bằng `--no-ff`, nhờ đó lịch sử thể hiện rõ ranh
giới của nhánh hotfix dù nhánh tạm thời đã được xóa.

# Kiểm thử và xác minh

Chạy kiểm thử bảo mật:

```bash
python -m unittest -v
```

Kiểm tra nhánh, tag và đồ thị commit:

```bash
git branch -a
git tag
git log --graph --oneline --decorate --all
git show-ref --tags
git merge-base --is-ancestor "$(git rev-parse v1.0.1^2)" develop
```

Lệnh cuối lấy parent thứ hai của merge commit được tag `v1.0.1` (chính là
commit hotfix) và trả về mã thoát `0`, chứng minh bản vá là tổ tiên của
`develop`.
