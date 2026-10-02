# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: Nhóm 01 (`ket-qua-nhom-01`); mã phòng LC chưa được cung cấp.
- Thành viên: xem `TEAMMATES.md` (họ tên/MSSV, vai trò từng lượt). Phân vai trong bảng là đề xuất luân phiên, nhóm cần xác nhận đúng với thực tế.
- Trạng thái: `provided-results` (tổng hợp output có sẵn; không có nhật ký xác định từng người chạy).
- Người thực sự chạy; ngày/giờ; hệ máy/architecture: người chạy chưa được ghi; output tạo ngày 2026-10-02 23:21:38–23:22:52 (UTC+7, đổi từ `smoke.json`); runtime Linux/Docker `amd64`, 4 CPU/4 GB container; host cụ thể chưa ghi.
- Image tag và image ID; phiên bản repo: `day13-pointpillars:lc-20261001-amd64`; `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`; revision `0831856d921609312d42c7582c366e5a311bb7b1` (working tree dirty theo manifest).
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp: `input/demo.pcd` / `demo`, KITTI demo 000008 chuyển đổi, 17.238 điểm, SHA256 `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`; chỉ xác nhận được gói Student chạy trên mẫu này; nơi/fingerprint LC chưa được cung cấp. x/y giữ nguyên, z đã cộng 1,73 m; reflectance bị bỏ, rgb=0 placeholder. Không có dữ liệu Robotaxi/VinFast.
- Checkpoint: PointPillars KITTI có sẵn trong image; ghi checkpoint ID/hash nếu LC cấp: `/opt/PointPillars/pretrained/epoch_160.pth`; SHA256 `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
- Phạm vi: front-window; score threshold: 0.3.
- Giả định kênh thứ tư/intensity và nguồn z_ground: reflectance nguồn đã bị loại khỏi PCD; adapter dùng kênh hằng theo thiết kế gói. `z_ground` được ước lượng từ scan, không phải ground truth; cả ba JSON ghi 0,075 m.

## Ba lượt inference thật

A/B/C là ba lượt trên cùng PCD. Runner chạy đủ ba lượt từ một lệnh. Lấy Số hộp từ n_boxes, mean_z từ mean_z trong run-A/B/C/summary.csv; không tự tính lại hoặc đoán. mean_z không phải điểm chất lượng. Mở side-*.png, đối chiếu boxes-*.json để ghi quan sát. Số hộp không phải đáp án cần khớp nhóm khác.

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | ---: | ---: | ---: | ---: | --- | --- |
| A | 0 | 0.16 | 1 | 0.330 | `run-A/boxes-demo-delta-0-voxel-0.16.json`; `run-A/side-demo-delta-0-voxel-0.16.png`; `run-A/summary.csv` | Một hộp `vehicles`, tâm x=13.15, y=-0.45, z=0.330 m, score 0.322. |
| B | 1.73 | 0.16 | 13 | 1.034 | `run-B/boxes-demo-delta-1.73-voxel-0.16.json`; `run-B/side-demo-delta-1.73-voxel-0.16.png`; `run-B/summary.csv` | 10 `vehicles`, 1 `two-wheels`, 2 `pedestrian`; hộp đầu score 0.933. Side có hộp từ khoảng x=2 đến 56 m; các hộp xa có điểm hỗ trợ thưa. |
| C | 1.73 | 0.32 | 6 | 1.091 | `run-C/boxes-demo-delta-1.73-voxel-0.32.json`; `run-C/side-demo-delta-1.73-voxel-0.32.png`; `run-C/summary.csv` | Cả 6 hộp là `pedestrian`; số hộp và class khác B; một frame không đủ để kết luận chất lượng. |

- A/B — chỉ đổi delta: A có 1 hộp; B có 13 hộp. `run-A/side-demo-delta-0-voxel-0.16.png` và `run-B/side-demo-delta-1.73-voxel-0.16.png` cho thấy output và phân bố hộp khác rõ; mean_z lần lượt 0.330 và 1.034 m, chênh 0.704 m, không phải 1.73 m. Đây là chạy lại model trên input khác, không chỉ dịch hộp cũ; điều em còn chưa chắc là mức độ đúng của các hộp do không có ground truth. PCD cũng đã chuyển z +1.73 m và pipeline ước lượng/hiệu chỉnh ground.
- B/C — chỉ đổi pillar: B có 13 hộp; C có 6 hộp. Ảnh/file `run-B/...voxel-0.16.png` và `run-C/...voxel-0.32.png` thể hiện tập hộp và class thay đổi; B có 10 vehicles, 1 two-wheels, 2 pedestrian, còn C có 6 pedestrian. mean_z là 1.034 và 1.091 m. Không đủ bằng chứng để kết luận cấu hình nào tốt hơn vì chỉ có một frame và không có nhãn chuẩn.
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào? Side chỉ chiếu x-z, trục x giới hạn đến 70 m và z từ -4 đến 4 m; không cho thấy y hoặc hình học 3D. Điểm xa/thưa và vật thể chồng lấp làm miss khó nhận biết; góc chiếu có thể làm yaw/chồng lấp trông khác, nên cần đối chiếu nhiều view trước khi quyết định.
- JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp? Cả A/B/C đều là dự đoán KITTI demo, không phải frame Robotaxi và chưa có ground truth; không JSON nào đủ cơ sở để import vào CVAT. Các `qc-cases/case-*.json` là ca helper training-only. Cần xác minh đúng nguồn/frame và transform, xem nhiều view, kiểm class/tâm/kích thước/yaw và QC thủ công. Không import prediction KITTI hay ca QC vào job Robotaxi.

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | ---: | --- | --- | --- | --- |
| case-correct | 0/13 | 0 m | Không | Chưa đủ để kết luận đúng; QC prediction nguồn từng hộp | `qc-cases/case-correct.json`, `side-correct.png`; bản sao không sửa của prediction B. |
| case-batch-z | 13/13 | -1.805 m mỗi hộp | Không | Dừng batch, kiểm transform/đảo phép z toàn pipeline | `qc-cases/case-batch-z.json`, `side-batch-z.png`; z của cả 13 hộp giảm 1.805 m. |
| case-one-box-z | 1/13 | -1.805 m ở hộp đầu | Không | Kiểm hộp đó và các view liên quan | `qc-cases/case-one-box-z.json`, `side-one-box-z.png`; hộp đầu z đổi 0.9215 thành -0.8835 m, các trường khác giữ nguyên. |

Ghi rõ helper tạo biến đổi có chủ đích từ prediction, không phải kết quả inference riêng hoặc nhãn đúng. `practice/pipeline-qc-cases.py` tạo ca từ prediction B có SHA256 `c2a8db247353f0ef00299ff50acff997b7b4a87bdcfcefe792815acb5652cc80`. Độ lệch là `-(delta + z_ground) = -(1.73 + 0.075) = -1.805 m`.

## Nhận xét cá nhân

Mỗi thành viên tự viết một mục: vai trò đã làm; một quan sát A/B/C có dẫn file hoặc hộp/vùng; diễn giải phép z thuận/ngược; một quyết định lỗi batch và hành động; điều chưa chắc. Chỉ đọc kết quả chuẩn bị trước thì ghi rõ chưa tự chạy.

### Đỗ Nguyễn Việt Linh

- Vai trò đề xuất trong bảng TEAMMATES: vận hành runner lượt A; cần thành viên xác nhận.
- Chỉ phân tích output chuẩn bị trước, chưa xác minh là người trực tiếp chạy. Quan sát: A có 1 hộp, B có 13 hộp (`run-A/summary.csv`, `run-B/summary.csv`); output không phải cùng hộp chỉ được dịch z.
- Phép thuận: `z_model = z_source - z_ground - delta`; phép ngược: `z_source = z_model + z_ground + delta`. Bỏ phép ngược làm z thấp hơn 1.805 m.
- Nếu cả batch lệch z cùng lượng thì dừng và kiểm transform; nếu một hộp lệch thì kiểm hộp và view liên quan.
- Chưa chắc: class/yaw/hình học đúng đến đâu vì không có ground truth và chỉ một hình chiếu Side.

### Nguyễn Hoài Thanh

- Vai trò đề xuất trong bảng TEAMMATES: đọc JSON/cấu hình lượt A; cần thành viên xác nhận.
- Chỉ phân tích output chuẩn bị trước, chưa xác minh là người trực tiếp chạy. Quan sát: B có 13 hộp với mean_z 1.034 m, C có 6 hộp với mean_z 1.091 m (`run-B/summary.csv`, `run-C/summary.csv`); class cũng thay đổi.
- Phép thuận: `z_model = z_source - z_ground - delta`; phép ngược: `z_source = z_model + z_ground + delta`. Bỏ phép ngược làm z thấp hơn `delta + z_ground`.
- Lỗi batch đồng loạt: dừng pipeline để kiểm transform; lỗi cục bộ: kiểm hộp bị ảnh hưởng và view liên quan.
- Chưa chắc: cấu hình pillar 0.16 hay 0.32 m tốt hơn nếu không có nhãn chuẩn.

### Trần Nhật Tân

- Vai trò đề xuất trong bảng TEAMMATES: ghi log/kết quả lượt A; cần thành viên xác nhận.
- Chỉ phân tích output chuẩn bị trước, chưa xác minh là người trực tiếp chạy. Quan sát: ca `case-batch-z` làm z giảm đều 1.805 m trên 13/13 hộp, trong khi class/x/y/yaw không đổi (`qc-cases/case-batch-z.json`).
- Phép thuận trừ `z_ground` và `delta` khỏi z nguồn trước inference; phép ngược cộng lại cả hai. Nếu thiếu bước cộng ngược sẽ thấy lỗi z có hệ thống.
- Với lỗi z cả batch, dừng batch và kiểm phép chuyển đổi trước khi sửa hộp; với ca một hộp, kiểm hộp đó và view lân cận.
- Chưa chắc: dự đoán có đúng đối tượng thật không do ca QC chỉ là biến đổi mô phỏng và không phải ground truth.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca: artifact chỉ xác nhận gói Student KITTI; xác nhận quyền/ca của LC chưa được cung cấp.
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung: report ghi `provided-results`; có smoke và A/B/C hoàn tất, người chạy chưa được xác định. LC cần xác nhận thực hành của từng thành viên.
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT: output A/B/C và ba ca QC có trong `../ket-qua-nhom-01/`; LC chưa xác nhận việc lưu bản gốc/import. Ca QC đánh dấu training-only, không import CVAT.
- Nhận xét từng thành viên và quyết định dừng pipeline: LC chưa ghi nhận. Bằng chứng ca mô phỏng cho thấy batch-z lệch 13/13 nên dừng kiểm pipeline; one-box-z lệch 1/13 nên kiểm cá thể.
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do: LC chưa cung cấp quyết định; chờ LC xác nhận quyền/ca và nhóm xác nhận vai trò, việc thực hành. Không dùng prediction KITTI hoặc ca lỗi cho job Robotaxi.
