# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Quang Huy |
| MSSV | 2A202602461 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/qhuy180105-boop/K4-L3-DAY21-NguyenQuangHuy-2A202602461-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Lần 3 đạt điểm f1_score cao nhất (0.7149), đạt ngưỡng chất lượng f1_score >= 0.65 của pipeline. Lần 1 có accuracy cao hơn (0.8780) nhưng f1_score lại thấp hơn (0.7109 so với 0.7149), cho thấy accuracy có thể gây hiểu nhầm trên dữ liệu mất cân bằng. Có sự đánh đổi giữa n_estimators và learning_rate: khi giảm learning_rate=0.05, n_estimators=50 và max_depth=2 ở Lần 2, mô hình bị underfitting (f1=0.6051). Tăng n_estimators lên 200 và max_depth lên 5 giúp mô hình Gradient Boosting biểu diễn tốt hơn các đặc trưng phức tạp.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult Census Income có phân bố lớp mất cân bằng nghiêm trọng với chỉ 24.8% số mẫu thuộc lớp thu nhập cao (>50K). Một mô hình vô dụng luôn đoán "thu nhập thấp" cho mọi mẫu vẫn đạt accuracy 75.2% mặc dù không phát hiện được bất kỳ người thu nhập cao nào. Do đó, accuracy không phản ánh năng lực thực tế. F1-score của lớp dương (target = 1) đánh giá chính xác sự cân bằng giữa Precision và Recall riêng cho lớp thiểu số. Khi tính f1_score, ta không dùng `average="weighted"` hay `average="macro"` vì các cách tính này bị lớp đa số (75.2%) kéo điểm số lên cao, che lấp hiệu năng thực sự trên lớp thiểu số.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Lỗi dvc push/pull authentication | Sa-key.json chưa cấu hình trong remote credentialpath của DVC. | Chạy `dvc remote modify labstore credentialpath sa-key.json` và lưu JSON vào secret STORAGE_CREDENTIALS. |
| SSH connection denied khi deploy | SSH key chưa thêm vào authorized_keys hoặc format key bị sai. | Tạo key Ed25519, thêm public key vào authorized_keys trên VM và lưu private key vào SERVER_SSH_KEY. |
| Service FastAPI trên VM bị hỏng khi start | Server khởi động trước khi model.joblib được upload lên Cloud Storage. | Đảm bảo job Train upload thành công model.joblib lên Cloud Storage trước khi job Release restart service. |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.8820 |

**Nhận xét:** Khi bổ sung 22.361 mẫu ở Bước 3 (tổng 44.722 mẫu), f1_score tăng từ 0.7149 lên 0.7354 và accuracy tăng từ 0.8740 lên 0.8820. Do dữ liệu mới trích xuất từ cùng nguồn nên có cùng phân phối, việc gấp đôi tập huấn luyện giúp mô hình học ranh giới phân loại chính xác hơn. Kết quả này kiểm chứng thành công pipeline MLOps tự động chạy trọn vẹn từ commit dữ liệu đến triển khai.
