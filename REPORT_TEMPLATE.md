# CSC4005 Lab 4 Report – CRNN for UrbanSound8K

---

# 1. Thông tin sinh viên

- Họ tên: Phan Việt Hùng
- Mã sinh viên: 1671040015
- Lớp: KHMT 1601
- Link GitHub : https://github.com/FIT-DNU-CS-16-01/csc4005-lab4-VietHung04.git
- Link W&B project: https://wandb.ai/phanhung2004dl-dainam-vietnam/csc4005-lab4-urbansound8k-crnn

---

# 2. Mục tiêu thí nghiệm

Trong Lab 4, mục tiêu chính là xây dựng mô hình CRNN (Convolutional Recurrent Neural Network) để phân loại âm thanh môi trường trên bộ dữ liệu UrbanSound8K.  

Mô hình sử dụng đặc trưng log-mel spectrogram nhằm biểu diễn tín hiệu âm thanh dưới dạng ảnh phổ theo thời gian và tần số, giúp mô hình học được các đặc trưng quan trọng của audio.  

Khác với mô hình 1D-CNN ở Lab 3 chỉ tập trung học đặc trưng cục bộ, CRNN kết hợp cả CNN và RNN nên có khả năng học thêm mối quan hệ theo thời gian giữa các frame âm thanh.  

Mục tiêu cuối cùng của bài lab là:
- đánh giá khả năng phân loại của CRNN,
- so sánh với mô hình 1D-CNN ở Lab 3,
- phân tích learning curves và confusion matrix để hiểu hành vi của mô hình.

---

# 3. Cấu hình dữ liệu

| Thành phần | Giá trị |
|---|---|
| Dataset | UrbanSound8K |
| Số lớp | 10 |
| Train folds | 1–8 |
| Validation fold | 9 |
| Test fold | 10 |
| Feature | log-mel spectrogram |
| Sampling rate | 16 kHz |
| Duration | 4 giây |

---

# 4. Cấu hình mô hình

## 4.1 Baseline Model – CRNN

| Thành phần | Giá trị |
|---|---|
| Model | CRNN |
| CNN blocks | 3 |
| RNN type | GRU |
| Hidden size | 64 |
| Dropout | 0.3 |
| Optimizer | Adam |
| Learning rate | 0.001 |
| Batch size | 32 |
| Epochs | 25 |

## 4.2 Extension Model – BiLSTM CRNN

| Thành phần | Giá trị |
|---|---|
| Model | CRNN + BiLSTM |
| CNN blocks | 3 |
| RNN type | BiLSTM |
| Hidden size | 128 |
| Dropout | 0.4 |
| Optimizer | Adam |
| Learning rate | 0.0007 |
| Batch size | 32 |
| Epochs | 25 (early stopping tại epoch 16) |

---

# 5. Kết quả huấn luyện

| Run | best_val_acc | test_acc | Ghi chú |
|---|---:|---:|---|
| logmel_crnn_gru_baseline | 0.7414 | 0.7503 | Kết quả tốt và ổn định |
| extension_bilstm_crnn | 0.6287 | 0.6404 | Overfitting nhẹ, dừng sớm |

## Nhận xét

Mô hình baseline sử dụng GRU cho kết quả tốt hơn mô hình mở rộng BiLSTM.  
Baseline đạt:
- Validation Accuracy cao nhất: **74.14%**
- Test Accuracy: **75.03%**

Trong khi đó mô hình BiLSTM:
- Validation Accuracy: **62.87%**
- Test Accuracy: **64.04%**

Điều này cho thấy kiến trúc BiLSTM phức tạp hơn chưa chắc mang lại hiệu quả tốt hơn trên UrbanSound8K khi số epoch và dữ liệu còn hạn chế.

---

# 6. Learning curves

## Nhận xét Baseline CRNN

- Train loss và validation loss giảm tương đối ổn định.
- Validation accuracy tăng đều theo epoch.
- Không xuất hiện hiện tượng overfitting quá mạnh.
- Learning rate scheduler hoạt động tốt khi giảm learning rate ở các epoch cuối.
- Mô hình hội tụ khá ổn định sau khoảng epoch 20.

## Nhận xét BiLSTM CRNN

- Train accuracy tiếp tục tăng nhưng validation accuracy dao động.
- Validation loss không cải thiện ổn định ở các epoch cuối.
- Xuất hiện dấu hiệu overfitting nhẹ.
- Early stopping được kích hoạt tại epoch 16 nhằm tránh mô hình học quá mức trên tập train.

---

# 7. Confusion matrix

## Nhận xét

Một số lớp được phân loại khá tốt:
- siren
- dog_bark
- drilling

Các lớp dễ bị nhầm:
- children_playing và street_music
- air_conditioner và engine_idling

Nguyên nhân có thể do:
- đặc trưng phổ âm thanh tương đối giống nhau,
- nhiều tiếng nhiễu nền,
- âm thanh có cường độ thấp,
- một số clip chứa nhiều nguồn âm thanh cùng lúc.

Ngoài ra, UrbanSound8K là dataset thực tế nên mức độ nhiễu khá cao, làm tăng độ khó của bài toán.

---

# 8. So sánh với Lab 3 – 1D-CNN

| Tiêu chí | Lab 3: 1D-CNN | Lab 4: CRNN |
|---|---|---|
| Feature chính | MFCC / log-mel | log-mel |
| Khả năng học pattern cục bộ | Có | Có |
| Khả năng học quan hệ thời gian | Hạn chế | Tốt hơn |
| Test accuracy | ~75% | 75.03% |
| Nhận xét | Mô hình đơn giản, train nhanh | Học temporal dependency tốt hơn |

## Nhận xét

CRNN có ưu điểm lớn hơn 1D-CNN ở khả năng học quan hệ theo thời gian nhờ RNN layer.  
CNN giúp trích xuất đặc trưng cục bộ trên spectrogram, trong khi GRU/LSTM học được sự thay đổi của âm thanh theo thời gian.  

Tuy nhiên:
- mô hình CRNN phức tạp hơn,
- thời gian train lâu hơn,
- cần nhiều tài nguyên tính toán hơn.

---

# 9. Kết luận

Trong Lab 4, em đã xây dựng thành công mô hình CRNN cho bài toán phân loại âm thanh môi trường trên UrbanSound8K.  

Mô hình sử dụng log-mel spectrogram giúp biểu diễn tín hiệu âm thanh hiệu quả hơn cho deep learning.  

Baseline CRNN với GRU đạt kết quả tốt nhất:
- Validation Accuracy: 74.14%
- Test Accuracy: 75.03%

So với Lab 3, CRNN có khả năng học temporal dependency tốt hơn nhờ sự kết hợp giữa CNN và RNN. Tuy nhiên, mô hình cũng phức tạp hơn và thời gian huấn luyện dài hơn đáng kể.  

Mô hình BiLSTM mở rộng chưa đạt kết quả tốt như mong đợi do có thể bị overfitting và chưa được tối ưu hyperparameter đầy đủ.  

Nếu tiếp tục cải thiện, em có thể:
- tăng cường audio augmentation,
- thử attention mechanism,
- sử dụng pretrained audio models,
- tuning learning rate và batch size,
- tăng số lượng dữ liệu huấn luyện.

---

