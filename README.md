# dxos-transparency-log

Repo neo công khai của [DX-OS Trust Core](https://github.com/PSA-OVNCR/dxos) (đội PSA_OVNCR, OLP PMNM 2026).

DX-OS ghi mọi sự kiện quan trọng vào một sổ cái dạng cây Merkle (RFC 6962). Định kỳ, hệ thống ký một
**Signed Tree Head (STH)** bằng khóa ed25519 giữ trong OpenBao và đẩy STH đó vào repo này thành tệp
`sth/<tree_size 12 chữ số>.json`. Vì nhánh `main` được bảo vệ, bên vận hành DX-OS **không** viết lại được
lịch sử đã neo, kể cả khi có quyền root trên máy chủ.

## Quy tắc của repo
- Nhánh `main`: cấm force-push, cấm xóa, bắt buộc lịch sử tuyến tính; không ai được bỏ qua.
- Máy chủ DX-OS đẩy STH bằng một **deploy key** chỉ có quyền ghi vào repo này. Quyền quản trị repo nằm ở tài khoản
  thành viên có 2FA, không đặt trên máy chủ.
- Không ai sửa tay các tệp trong `sth/`.

## Tự kiểm tra
```bash
git clone https://github.com/PSA-OVNCR/dxos-transparency-log
python -m pip install "dxos-proof @ git+https://github.com/PSA-OVNCR/dxos@v0.1.0#subdirectory=libs/dxos-proof" \
                      "dxos-witness @ git+https://github.com/PSA-OVNCR/dxos@v0.1.0#subdirectory=witness"
dxos-witness sync --server https://<địa-chỉ-DX-OS> --anchor-dir dxos-transparency-log --state-dir ~/.dxos-witness
```
Mã thoát `0`: bình thường. Mã `2`: bất thường; hãy giữ lại thư mục `--state-dir` làm bằng chứng và báo cho đội qua
[Issues của repo dxos](https://github.com/PSA-OVNCR/dxos/issues).

| Kết quả | Ý nghĩa |
|---|---|
| `FIRST_RUN` / `OK` | STH mới nhất có chữ ký đúng, khớp repo neo, nhất quán với lần kiểm trước |
| `BAD_SIGNATURE` | Chữ ký sai, hoặc khóa công khai của cùng phiên bản bị thay |
| `ANCHOR_MISMATCH` | STH máy chủ trả khác tệp trong repo neo |
| `REWRITE_DETECTED` | Cùng kích thước mà root khác, cây nhỏ đi, hoặc phần lịch sử đã neo bị viết lại |
| `INCONSISTENT` | Consistency proof từ lần trước tới nay không hợp lệ |

Muốn làm witness độc lập chạy tự động trên GitHub Actions: dùng repo mẫu
[dxos-witness-kit](https://github.com/PSA-OVNCR/dxos-witness-kit).

## Witness tự động trong repo này
`.github/workflows/witness.yml` chạy mỗi giờ khi đã đặt biến `DXOS_SERVER`. Kiểm thất bại thì workflow báo đỏ công khai.
Witness này nằm trong tổ chức của đội nên không được tính là độc lập.

## Giấy phép
Apache-2.0 (xem `LICENSE`).
