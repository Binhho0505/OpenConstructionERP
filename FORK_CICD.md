# Quy tắc CI/CD — fork Binhho0505/OpenConstructionERP

Áp dụng cho người phát triển trên fork này. Dự án gốc: `datadrivenconstruction/OpenConstructionERP`.

## 1. Nhánh (branch)

| Nhánh | Vai trò | Quy tắc |
|---|---|---|
| `main` | Bản chạy được, luôn xanh CI. Máy local chạy Docker từ nhánh này | **Không commit thẳng.** Chỉ nhận qua Pull Request đã qua CI |
| `feature/<mo-ta>` | Tính năng mới | Tách từ `main`, mỗi nhánh một việc |
| `fix/<mo-ta>` | Sửa lỗi | Như trên |
| `sync/upstream-YYYY-MM-DD` | Ghép bản mới của dự án gốc | Xử lý xung đột ở đây, rồi PR vào `main` |
| `ci/<mo-ta>` | Sửa pipeline, Docker, script | Như trên |

Tên nhánh viết thường, không dấu, nối bằng `-`.

## 2. Commit

- Theo chuẩn **Conventional Commits**: `feat:`, `fix:`, `chore:`, `docs:`, `ci:`, `refactor:`, `test:`.
- Một commit = một thay đổi có nghĩa. Mô tả ngắn, nói *vì sao*.
- **Không bao giờ** commit `.env`, mật khẩu, token, file dữ liệu khách hàng thật.

## 3. Pull Request → `main`

1. Đẩy nhánh: `git push -u origin feature/...`
2. Tạo PR vào `main` **của fork này** (không phải repo gốc): `gh pr create --base main --repo Binhho0505/OpenConstructionERP`
3. Workflow **Fork CI** (`.github/workflows/fork-ci.yml`) phải xanh cả 3 job: Backend, Frontend, Docker.
4. Tự review lại diff, rồi **Squash merge**.
5. Xoá nhánh sau khi merge.

Khuyến nghị bật Branch protection cho `main` (Settings → Branches): bắt buộc PR + bắt buộc check `Fork CI`.

## 4. CI — kiểm tra gì

| Job | Lệnh |
|---|---|
| Backend | `ruff check`, `ruff format --check`, `pytest` |
| Frontend | `npm run lint`, `npm run build` (đã gồm typecheck) |
| Docker | build `deploy/docker/Dockerfile.unified` (không đẩy image) |

Các workflow có sẵn của dự án gốc (release, publish PyPI, ký số…) cần secret của họ → **tắt trên fork**, chỉ giữ `fork-ci.yml`, `codeql.yml`, `secret-scan.yml`.

## 5. CD — đưa code mới về máy local

Máy local luôn chạy theo `origin/main`. Sau khi PR được merge:

```powershell
C:\Users\Admin\source\repos\_ops\deploy-local.ps1 -Project oce
```

Script sẽ: kiểm tra CI của commit mới nhất trên `main` đã xanh → `git pull --ff-only` → `docker compose up -d --build` → chờ `http://localhost:8090/api/health` trả về OK. CI đỏ thì **dừng, không triển khai**.

## 6. Đồng bộ với dự án gốc (mỗi 1–2 tuần)

```powershell
git fetch upstream
git switch -c sync/upstream-$(Get-Date -f yyyy-MM-dd) origin/main
git merge upstream/main
# xử lý xung đột, chạy test, push, tạo PR vào main
```

## 7. Môi trường local

| Thành phần | Địa chỉ |
|---|---|
| Ứng dụng | http://localhost:8090 |
| Compose project | `oce` (`docker compose -p oce ...`) |
| Bí mật | file `.env` cạnh `docker-compose.quickstart.yml` (đã gitignore) |

## 8. Giấy phép

AGPL-3.0: fork phải công khai; nếu cung cấp dịch vụ qua mạng thì phải công bố mã nguồn đã sửa.
